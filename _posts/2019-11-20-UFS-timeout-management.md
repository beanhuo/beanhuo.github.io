---
layout: post
title: UFS timeout management
date: 2019-11-20 
tags: Linux
---


## UFS timeout management  

UFS only has the SW timeout timer in the block layer, it doesn't have a HW timer on the
host side.

the SW timer setup flow:

```
scsi_queue_rq() //scsi_lib.c
	blk_mq_start_request(req)
	scsi_dispatch_cmd()
	
```

SCSI-based storage, you can change its queue timeout through sysfs node "timeout"



the queue timeout is set in the sd_probe();



In the block layer, blk-mq will initialize the queue timeout when allocating a queue:

```
blk_mq_init_allocated_queue()
	  INIT_WORK(&q->timeout_work, blk_mq_timeout_work);
	  blk_queue_rq_timeout(q, set->timeout ? set->timeout : 30 * HZ);
```
  
For the UFS, it will go this way, so the timeout value in the block layer for the UFS is
30s probably.


in the case of timeout, the scsi_timeout() will be called:

```

scsi_timeout()-->

enum blk_eh_timer_return scsi_times_out(struct request *req)                     
{                                                                                
        struct scsi_cmnd *scmd = blk_mq_rq_to_pdu(req);                          
        enum blk_eh_timer_return rtn = BLK_EH_DONE;                              
        struct Scsi_Host *host = scmd->device->host;                             
                                                                                 
        trace_scsi_dispatch_cmd_timeout(scmd);                                   
        scsi_log_completion(scmd, TIMEOUT_ERROR);                                
                                                                                 
        if (host->eh_deadline != -1 && !host->last_reset)                        
                host->last_reset = jiffies;                                      
                                                                                 
        if (host->hostt->eh_timed_out)   //UFS didn't initialize this hooker                                        
                rtn = host->hostt->eh_timed_out(scmd);                           
                                                                                 
        if (rtn == BLK_EH_DONE) {                                                
                /*                                                               
                 * Set the command to complete first in order to prevent a real  
                 * completion from releasing the command while error handling    
                 * is using it. If the command was already completed, then the   
                 * lower level driver beat the timeout handler, and it is safe   
                 * to return without escalating error recovery.                  
                 *                                                               
                 * If timeout handling lost the race to a real completion, the   
                 * block layer may ignore that due to a fake timeout injection,  
                 * so return RESET_TIMER to allow error handling another shot    
                 * at this command.                                              
                 */                                                              
                if (test_and_set_bit(SCMD_STATE_COMPLETE, &scmd->state))         
                        return BLK_EH_RESET_TIMER;                               
                if (scsi_abort_command(scmd) != SUCCESS) {  // send abort CMD                     
                        set_host_byte(scmd, DID_TIME_OUT);                       
                        scsi_eh_scmd_add(scmd);                                  
                }                                                                
        }                                                                        
                                                                                 
        return rtn;                                                              
}
```
```
static int                                                                       
scsi_abort_command(struct scsi_cmnd *scmd)                                       
{                                                                                
        struct scsi_device *sdev = scmd->device;                                 
        struct Scsi_Host *shost = sdev->host;                                    
        unsigned long flags;                                                     
                                                                                 
        if (scmd->eh_eflags & SCSI_EH_ABORT_SCHEDULED) {                         
                /*                                                               
                 * Retry after abort failed, escalate to next level.             
                 */                                                              
                SCSI_LOG_ERROR_RECOVERY(3,                                       
                        scmd_printk(KERN_INFO, scmd,                             
                                    "previous abort failed\n"));                 
                BUG_ON(delayed_work_pending(&scmd->abort_work));                 
                return FAILED;                                                   
        }                                                                        
                                                                                 
        spin_lock_irqsave(shost->host_lock, flags);                              
        if (shost->eh_deadline != -1 && !shost->last_reset)                      
                shost->last_reset = jiffies;                                     
        spin_unlock_irqrestore(shost->host_lock, flags);                         
                                                                                 
        scmd->eh_eflags |= SCSI_EH_ABORT_SCHEDULED;                              
        SCSI_LOG_ERROR_RECOVERY(3,                                               
                scmd_printk(KERN_INFO, scmd, "abort scheduled\n"));              
        queue_delayed_work(shost->tmf_work_q, &scmd->abort_work, HZ / 100);      
        return SUCCESS;                                                          
} 
```

scmd->abort_work->scmd_eh_abort_handler()

```

scmd_eh_abort_handler(struct work_struct *work)                                  
{                                                                                
        struct scsi_cmnd *scmd =                                                 
                container_of(work, struct scsi_cmnd, abort_work.work);           
        struct scsi_device *sdev = scmd->device;                                 
        enum scsi_disposition rtn;                                               
                                                                                 
        if (scsi_host_eh_past_deadline(sdev->host)) {                            
                SCSI_LOG_ERROR_RECOVERY(3,                                       
                        scmd_printk(KERN_INFO, scmd,                             
                                    "eh timeout, not aborting\n"));              
        } else {                                                                 
                SCSI_LOG_ERROR_RECOVERY(3,                                       
                        scmd_printk(KERN_INFO, scmd,                             
                                    "aborting command\n"));                      
                rtn = scsi_try_to_abort_cmd(sdev->host->hostt, scmd); //issue abort CMD           
                if (rtn == SUCCESS) {                                            
                        set_host_byte(scmd, DID_TIME_OUT);                       
                        if (scsi_host_eh_past_deadline(sdev->host)) {            
                                SCSI_LOG_ERROR_RECOVERY(3,                       
                                        scmd_printk(KERN_INFO, scmd,             
                                                    "eh timeout, not retrying "  
                                                    "aborted command\n"));       
                        } else if (!scsi_noretry_cmd(scmd) &&                    
                                   scsi_cmd_retry_allowed(scmd) &&               
                                scsi_eh_should_retry_cmd(scmd)) {                
                                SCSI_LOG_ERROR_RECOVERY(3,                       
                                        scmd_printk(KERN_WARNING, scmd,          
                                                    "retry aborted command\n")); 
                                scsi_queue_insert(scmd, SCSI_MLQUEUE_EH_RETRY);  //retry 
                                return;                                          
                        } else {                                                 
                                SCSI_LOG_ERROR_RECOVERY(3,                       
                                        scmd_printk(KERN_WARNING, scmd,          
                                                    "finish aborted command\n"));
                                scsi_finish_command(scmd);                       
                                return;                                          
                        }                                                        
                } else {                                                         
                        SCSI_LOG_ERROR_RECOVERY(3,                               
                                scmd_printk(KERN_INFO, scmd,                     
                                            "cmd abort %s\n",                    
                                            (rtn == FAST_IO_FAIL) ?              
                                            "not send" : "failed"));             
                }                                                                
        }                                                                        
                                                                                 
        scsi_eh_scmd_add(scmd);                                                  
} 
```

the maximum retries is set in static int sd_probe(struct device *dev)
by default is 5.

if retries failed, Pass command off to upper layer for finishing of I/O.



In case of abort CMD failure, there will call scsi_error_handler(),


```
int scsi_error_handler(void *data)                                               
{                                                                                
        struct Scsi_Host *shost = data;                                          
                                                                                 
        /*                                                                       
         * We use TASK_INTERRUPTIBLE so that the thread is not                   
         * counted against the load average as a running process.                
         * We never actually get interrupted because kthread_run                 
         * disables signal delivery for the created thread.                      
         */                                                                      
        while (true) {                                                           
                /*                                                               
                 * The sequence in kthread_stop() sets the stop flag first       
                 * then wakes the process.  To avoid missed wakeups, the task    
                 * should always be in a non running state before the stop       
                 * flag is checked                                               
                 */                                                              
                set_current_state(TASK_INTERRUPTIBLE);                           
                if (kthread_should_stop())                                       
                        break;                                                   
                                                                                 
                if ((shost->host_failed == 0 && shost->host_eh_scheduled == 0) ||
                    shost->host_failed != scsi_host_busy(shost)) {               
                        SCSI_LOG_ERROR_RECOVERY(1,                               
                                shost_printk(KERN_INFO, shost,                   
                                             "scsi_eh_%d: sleeping\n",           
                                             shost->host_no));                   
                        schedule();                                              
                        continue;                                                
                }                                                                
                                                                                 
                __set_current_state(TASK_RUNNING);                               
                SCSI_LOG_ERROR_RECOVERY(1,                                       
                        shost_printk(KERN_INFO, shost,                           
                                     "scsi_eh_%d: waking up %d/%d/%d\n",         
                                     shost->host_no, shost->host_eh_scheduled,   
                                     shost->host_failed,                         
                                     scsi_host_busy(shost)));                    
                                                                                 
                /*                                                               
                 * We have a host that is failing for some reason.  Figure out   
                 * what we need to do to get it up and online again (if we can). 
                 * If we fail, we end up taking the thing offline.               
                 */                                                              
                if (!shost->eh_noresume && scsi_autopm_get_host(shost) != 0) {   
                        SCSI_LOG_ERROR_RECOVERY(1,                               
                                shost_printk(KERN_ERR, shost,                    
                                             "scsi_eh_%d: unable to autoresume\n",
                                             shost->host_no));                   
                        continue;                                                
                }                                                                
                                                                                 
                if (shost->transportt->eh_strategy_handler)                      
                        shost->transportt->eh_strategy_handler(shost);   // will call UFS error handler        
                else                                                             
                        scsi_unjam_host(shost);                                  
                                                                                 
                /* All scmds have been handled */                                
                shost->host_failed = 0;                                          
                                                                                 
                /*                                                               
                 * Note - if the above fails completely, the action is to take   
                 * individual devices offline and flush the queue of any         
                 * outstanding requests that may have been pending.  When we     
                 * restart, we restart any I/O to any other devices on the bus   
                 * which are still online.                                       
                 */                                                              
                scsi_restart_operations(shost);                                  
                if (!shost->eh_noresume)                                         
                        scsi_autopm_put_host(shost);                             
        }                                                                        
        __set_current_state(TASK_RUNNING);                                       
                                                                                 
        SCSI_LOG_ERROR_RECOVERY(1,                                               
                shost_printk(KERN_INFO, shost,                                   
                             "Error handler scsi_eh_%d exiting\n",               
                             shost->host_no));                                   
        shost->ehandler = NULL;                                                  
        return 0;                                                                
}                                                                                
   
```

To understand UFS (Universal Flash Storage) timeout management in the context of the Linux SCSI subsystem, let's break down the key aspects of the provided code and its flow:

### Timeout Setup in the Block Layer

1. **Initialization of the Queue Timeout**:
   - The queue timeout for SCSI-based storage devices is set during the `sd_probe()` function.
   - The actual setup of the timeout occurs in `blk_mq_init_allocated_queue()` where `blk_queue_rq_timeout(q, set->timeout ? set->timeout : 30 * HZ);` is called, setting the default timeout to 30 seconds (`30 * HZ`).

2. **Command Dispatch and Timeout Handling**:
   - When a command is queued, `scsi_queue_rq()` is invoked, which calls `blk_mq_start_request(req)` and `scsi_dispatch_cmd()`.
   - If a command times out, `scsi_timeout()` is called, which in turn calls `scsi_times_out(req)`.

### Handling Timeouts

1. **Primary Timeout Handling in `scsi_times_out`**:
   - Within `scsi_times_out`, various checks and logs are performed.
   - If the host's `eh_timed_out` function is not set (`UFS didn't initialize this hooker`), the function proceeds to attempt to abort the command using `scsi_abort_command(scmd)`.

2. **Command Abort Handling in `scsi_abort_command`**:
   - The function schedules an abort work via `queue_delayed_work(shost->tmf_work_q, &scmd->abort_work, HZ / 100);` and returns `SUCCESS`.

3. **Abort Work Execution in `scmd_eh_abort_handler`**:
   - This handler tries to abort the command via `scsi_try_to_abort_cmd()`.
   - Depending on the result, it may either retry the command, finish it, or add it to the error handling list using `scsi_eh_scmd_add(scmd)`.

### Error Handling Flow

1. **SCSI Error Handler**:
   - If the command abort fails, the `scsi_error_handler()` is invoked.
   - The error handler manages error recovery for the SCSI host. It checks if there are any failed commands and invokes the appropriate error handling strategy.

2. **UFS Specific Error Handling**:
   - For UFS devices, if a specific error handler (`eh_strategy_handler`) is defined in the transport layer (`shost->transportt`), it will be invoked to handle the error.

### Practical Implications for UFS

- **Timeout Value**: The default timeout value is 30 seconds.
- **Error Recovery**: If a UFS command times out, the system will try to abort the command and, upon failure, will invoke the error handler.
- **Retries**: Commands can be retried up to a maximum of 5 times as set in `sd_probe()`.

### Conclusion

The timeout management for UFS in the Linux kernel leverages the existing SCSI subsystem's mechanisms for handling timeouts and errors. The flow involves initializing a default timeout, attempting to abort timed-out commands, and invoking error recovery strategies as needed. The specific details of UFS error handling depend on the transport layer implementation for UFS, which can define custom error handling strategies.

