---
layout: post
title: eMMC timeout management in Linux&Uboot
date: 2019-11-20 
tags: Linux
---
	
	# eMMC timeout management
	
For the eMMC, we have thwo levels of timeout managemnt: one is in block layer,
the second one is in the eMMC core, it shows as below:

      ---     VFS     ---
      ---     FS      ---
      --- Block layer ---  > set up SW timeout timer for each request before dispatch the request to the next layers
      	       |
      	       |
      	       V
      ---    MMC Core ---  > 1) if it is data tranfer, setup HW/SW timeout for each request, for data line timeout
               |             2) setup a SW timeout for waiting for the completion interruption
               |
               V
       --- eMMC Device ---


 ## >1< -- block layer
 
    For the data tranfer request, which is  REQ_OP_READ and REQ_OP_WRITE, there will
    set the SW timeout value with queue's q->rq_timeout 
    

    Queue timeout value, which is set in mmc_mq_init() during eMMC
    initialization stage. By default, the queue timeout value is 1 minute.
    This timeout value is used by block layer. Block layer will setup a timer
    for each request in the block layer, and set its timeout value to 1m. This
    timer is to prevent a command from being retried forever.
	
    eMMC initialize queue timeout value:
    
    '
    mmc_init_queue()
       mmc_mq_init()
          blk_queue_rq_timeout(mq->queue, 60 * HZ)
    '
          
    
    For the non-data transfer request, eg, discard, erase, flush. By default, there
    will set the request->timeout with 600*HZ, which is 10m. this is hardcode set    
    

		mmc_mq_queue_rq() {
		switch(issue_type) {
		 default:
			req->timeout = 600 * HZ;
		}
			-blk_mq_start_request()
				--blk_add_timer(struct request *req)
			-mmc_blk_mq_issue_rq() /* dispatch the request to the eMMC core */
		}
		 
		Each request has its own timer, when the request is completed, the timer
		will be cancelled.
	
	
## >2< -- eMMC core

	In the eMMC core, there is also a timer for each request coming
	from the upper layer.
	Here has two conditions, with CQE and without CQE:

	** 1) with CQE **
	CQE will use HW timer in the emmc core for the data transfer, and its callback flow:
	
		mmc_blk_mq_issue_rq()
		  -mmc_blk_cqe_issue_rw_rq()
		    -mmc_blk_cqe_start_req()
		      -mmc_cqe_start_req()
		        ->cqhci_request().
		          .cqe_enable = cqhci_enable.
		              cq_host->ops->enable(mmc);                                       
                              /*enter vendor specific driver ops*/                
			      ..sdhci_cqe_enable
			/* Set maximum timeout */  
			sdhci_set_timeout(host, NULL);
			/*set register SDHCI_TIMEOUT_CONTROL   0x2E to be 0xe */
			
	  Note: no matter how eMMC upper layer caculate the timeout value in 
	  mmc_blk_data_prep()->mmc_set_data_timeout(), the CQE driver will
	  always set maximum timeout value for CQE request.
	  
	  
	HW timeout TMCLK is set in the eMMC host capacity regisger 0x40:
	sdhci_setup_host()
	  sdhci_read_caps()---...>host->caps = sdhci_readl(host, SDHCI_CAPABILITIES);

	** 2) without CQE **
	
	If CQE is disabled, there will be two timers for the request coming from upper
	layer. one is HW timer for data line timeout of data transfer, another one is
	the SW timer for waiting for the completion interruption:
	
	*HW timer setup flow:*
	'
		mmc_blk_mq_issue_rq()
		  -mmc_blk_mq_issue_rw_rq()
		    	-mmc_start_request()
		    	....>sdhci_send_command(host, cmd)
		    	   If (it is data W/R)
		    	     ->sdhci_set_timeout(host, cmd);
	'
	
	Note: CQE is off, before send the command to the eMMC, eMMC driver will
	set DHCI_TIMEOUT_CONTROL to be value set in mmc_set_data_timeout()
	
	
	---------
	For some SDHCI controller, if they have SDHCI_QUIRK2_DISABLE_HW_TIMEOUT,
	and calculated timeout value is too big and beyond the HW timer capacity,
	eMMc driver will disable this HW timeout timer for data transfer.
	
	
	*SW timeout timer setup*
	
	No matter it is data transfer request or non-data transfer request(such as flush,
	erase, discard), this SW timer will be explicityly setup before issuing this
	request to the eMMC device;
	
	  mmc_blk_mq_issue_rq() or mmc_switch()---->sdhci_send_command()
	  
	'timeout = jiffies;                                                       
        if (host->data_timeout)                                                  
                timeout += nsecs_to_jiffies(host->data_timeout);                 
        else if (!cmd->data && cmd->busy_timeout > 9000)                         
                timeout += DIV_ROUND_UP(cmd->busy_timeout, 1000) * HZ + HZ;      
        else                                                                     
                timeout += 10 * HZ;                                              
        sdhci_mod_timer(host, cmd->mrq, timeout);
	  '
	
	
	for the non-data transfer command, eMMC will choose SW timeout. For the
	timeout value depends on the exact command. for example, sanitize timeout
	which is defined by SW:
	
	'
	 #define MMC_SANITIZE_TIMEOUT_MS         (240 * 1000) /* 240s */
	 #define MMC_BKOPS_TIMEOUT_MS            (120 * 1000) /* 120s */
	 ' 

	
	


	For the SWITCH CMD6 timeout, there will be retried 10 times in case of failure:

'
	sdhci_request()
		sdhci_send_command_retry() {

		int timeout = 10; /* Approx. 10 ms */
		while (!sdhci_send_command(host, cmd)) {
			 ...
			 usleep_range(1000, 1250);
			 ...
			if failed ,there will retry until success or timoeut!
		}
	}
'


##How to disable Hw timeout for data-transfer


1. Firstly you should add this quirk SDHCI_QUIRK2_DISABLE_HW_TIMEOUT in the eMMC host driver,
TI added this quirk since v4.18

'
static const struct sdhci_pltfm_data sdhci_omap_pdata = {                            
        .quirks = SDHCI_QUIRK_BROKEN_CARD_DETECTION |                                
                  SDHCI_QUIRK_DATA_TIMEOUT_USES_SDCLK |                              
                  SDHCI_QUIRK_CAP_CLOCK_BASE_BROKEN |                                
                  SDHCI_QUIRK_NO_HISPD_BIT |                                         
                  SDHCI_QUIRK_BROKEN_ADMA_ZEROLEN_DESC,                              
        .quirks2 = SDHCI_QUIRK2_ACMD23_BROKEN |                                      
                   SDHCI_QUIRK2_PRESET_VALUE_BROKEN |                                
                   SDHCI_QUIRK2_RSP_136_HAS_CRC |                                    
                   SDHCI_QUIRK2_DISABLE_HW_TIMEOUT,                                  
        .ops = &sdhci_omap_ops,                                                  
};
'

/*
* Disable HW timeout if the requested timeout is more than the maximum          
* obtainable timeout.                                                           
*/                                                                           
#define SDHCI_QUIRK2_DISABLE_HW_TIMEOUT

2. then the timeout value requried by eMMC should be bigger that host HW timer.

'

void __sdhci_set_timeout(struct sdhci_host *host, struct mmc_command *cmd)           
{                                                                                
        bool too_big = false;                                                    
        u8 count = sdhci_calc_timeout(host, cmd, &too_big);                      
                                                                                 
        if (too_big &&                                                           
            host->quirks2 & SDHCI_QUIRK2_DISABLE_HW_TIMEOUT) {                   
                sdhci_calc_sw_timeout(host, cmd); /// caclulate SW timeout value                             
                sdhci_set_data_timeout_irq(host, false); // Disable HW timer                     
        } else if (!(host->ier & SDHCI_INT_DATA_TIMEOUT)) {                      
                sdhci_set_data_timeout_irq(host, true);                          
        }                                                                        
                                                                                 
        sdhci_writeb(host, count, SDHCI_TIMEOUT_CONTROL);         
}
' 


