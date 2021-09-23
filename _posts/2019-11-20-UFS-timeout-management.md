---
layout: post
title: UFS timeout management in Linux&Uboot
date: 2019-11-20 
tags: Linux
---

### UFS timeout management  

UFS only has the SW timeout timer in the block layer, it doesn't have a HW timer on the
host side.

the SW timer setup flow:

```
scsi_queue_rq()
	blk_mq_start_request(req)
	scsi_dispatch_cmd()
	
```

scsi based storage, you can change its queue timeout through sysfs node "timeout"



the queue timeout is set in the sd_probe();



In the block layer, blk-mq will initialize the queue timeout when allocating a queue:

```
blk_mq_init_allocated_queue()
	  INIT_WORK(&q->timeout_work, blk_mq_timeout_work);
	  blk_queue_rq_timeout(q, set->timeout ? set->timeout : 30 * HZ);
```
  
For the UFS, it will go this way, so the timeout value in the block layer for the UFS is
30s probably.
