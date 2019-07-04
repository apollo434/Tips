
**NOTE**

How to create the patch when some commit need to delete and the modification saved at the same time

```
375  git diff HEAD^^^..HEAD --stat
376  git diff HEAD^^^..HEAD -p
377  git diff HEAD^^^..HEAD > patch.diff
378  vim patch.diff
379  git branch
380  ls
381  pwd
382  git status
383  git add arch/x86/kernel/cpu/common.c arch/x86/kernel/cpu/transmeta.c arch/x86/kvm/vmx.c arch/x86/lguest/boot.c
384  git commit -m "working"
385  mv patch.diff ../
386  git log
387  git checkout 5f80d0e7906f7dfc4a9dd946b486435c00383d99 -b patching
388  git log
389  git status
390  git apply ../patch.diff
391  git status
392  history

```

How to creat remote repertory

```
$ git checkout -b local-branch commit-id

push to remote:

$ git push local-branch:remote-branch

To delete the remote branch if needed:

$ git push :remote-branch
```

how to creat personal local git branch

```
1. Update the personal local git branch

#git fetch

2. Create the local branch

#git checkout -b XXX-WRL8.0 origin/WRL8.0

3. Make changes

4. Push local branch to remote

#git push origin XXX-WRL8.0:XXX-WRL8.0
```
How to use Simics

```
1. Install Simics structure directory: ~/lx2160/
2. Install the Simics directory: ~/lx2160/simics-5/

```

How to make itb image for kernel/rootfs image?
```
1. kernel config
$ bitbake linux-windriver -c menuconfig
+CONFIG_SERIAL_AMBA_PL011=y
+CONFIG_SERIAL_AMBA_PL011_CONSOLE=y

2. make fsl-lx2160a-rdb.dtb
$ bitbake -c devshell virtual/kernel
$ make freescale/fsl-lx2160a-rdb.dtb

(3. option: if you needed update rootfs(initramfs), create cpio.gz)
# tar -jxvf wrlinux-image-glibc-cgl-nxp-lx2160.tar.bz2 -C dist/
# cd dist
# find . | cpio -ovH newc | gzip > ../wrlinux-image-glibc-cgl-nxp-lx2160.cpio.gz

4. make itb image
$ cp -r utools <project>/build/tmp/deploy/nxp-lx2160
$ export PATH=<project>/build/tmp/deploy/nxp-lx2160/utools:$PATH
$ mkimage -f wrl-lx2160ardb.its wrl-lx2160ardb.itb
---

New simics script for easy settings bootparams
---
$ cp -r lx2160.wrl9 <your simics project>
Note: <your simics project>/daybreak-csm
```

How to change/set u-boot/kernel images?
```
Right click "lx2-wrl9.simics" on Simics -> Run as -> Run Configrations... ->
 Script: targets/lx2160.wrl9/lx2-wrl9.simics -> Click "Parameters..."
 Click "Runtime" and set these name and "Browse.." and set your images
  uboot_image:
  security_monitor_image:
  linux_kernel_image:
```

How to change kernel bootparams?
```
$ cat lx2-wrl9.simics
# Auto-generated file. Any changes will be overwritten!
decl { substitute "/opt/wrs/simics/5.0/WindRiver.internal/simics-5/simics-5.0.139/../simics-daybreak-5.0.pre4/targets/daybreak-csm/daybreak-lx2-nxp-uboot.simics" }

$kernel_cmdline = "console=ttyAMA0,115200 root=/dev/ram0 rw rootfstype=ramfs rdinit=/sbin/init earlycon=pl011,mmio32,0x21c0000 default_hugepagesz=2m hugepagesz=2m hugepages=256 selinux=0"
$linux_kernel_addr_dram = 0xa0000000

run-command-file "/opt/wrs/simics/5.0/WindRiver.internal/simics-5/simics-5.0.139/../simics-daybreak-5.0.pre4/targets/daybreak-csm/daybreak-lx2-nxp-uboot.simics"

You can edit "kernel_cmdline" line.
 Or
Right click "lx2-wrl9.simics" on Simics -> Run as -> Run Configrations... ->
 Script: targets/lx2160.wrl9/lx2-wrl9.simics -> Click "Parameters..."
 Select "kernel_cmdline" and input paramers
---

file list:
20180426.ccm
├── fsl-lx2160a-rdb.dtb								lx2160r-rdb dtb file
├── Image-nxp-lx2160.bin							kernel image
├── simics
│   └── lx2160.wrl9
│       └── lx2-wrl9.simics							my simics script
├── utools
│   ├── dtc											wrlinux//scripts
│   ├── dumpimage									u-boot/output/tools
│   └── mkimage										u-boot/output/tools
├── wrlinux-image-glibc-cgl-nxp-lx2160.tar.bz2		glibc cgl rootfs
└── wrl-lx2160ardb.its								its for make itb image

```

***Perf command***
```
perf record -a -g -o /tmp/perf.data
perf report -i /tmp/perf.data > perf.txt
vi perf.txt
set nowrap
Shift + 4 ====> end of raw
Shift + 0 ====> begin of raw
```

***ASLR***
```
# cat /proc/sys/kernel/randomize_va_space

If the output is 1 or 2, the ASLR is enabled.
```

***Calculate timer***
```
How to add command via /proc/sys/kernel
kernel/sysctl
{
  .procname = "panic_on_oops",
  .data     = &panic_on_oops,
  .maxlen   = sizeof(int),
  .mode     = 0644,
  .proc_handler = proc_dointvec,
}

getnstimeofday64()
do_gettimeofday()
sched_clock()

```

***Record info in memory and print it before crash***
```
From 974bb85e0002ecdd158e2e8d0f564abfb659afff Mon Sep 17 00:00:00 2001
From: Jiwei Sun <jiwei.sun@windriver.com>
Date: Fri, 19 Apr 2019 13:12:46 +0800
Subject: [PATCH v2] writeback: record some useful information for ext2 usb
 storage

Debug patch V2

Signed-off-by: Jiwei Sun <jiwei.sun@windriver.com>
v2 changes:
  - disable preempt before invoke smp_processor_id()
  - show the call trace after hung task is checked
---
 block/blk-core.c               |   2 +
 block/blk-softirq.c            |  14 +-
 drivers/scsi/scsi_lib.c        |   6 +
 drivers/usb/storage/scsiglue.c |   2 +
 drivers/usb/storage/usb.c      |  11 +
 fs/mpage.c                     |  11 +-
 init/main.c                    |   2 +
 kernel/hung_task.c             | 560 ++++++++++++++++++++++++++++++++++++++++-
 mm/filemap.c                   |  20 +-
 mm/page-writeback.c            |   5 +-
 10 files changed, 626 insertions(+), 7 deletions(-)

diff --git a/block/blk-core.c b/block/blk-core.c
index 03b5f8d..af08d45 100644
--- a/block/blk-core.c
+++ b/block/blk-core.c
@@ -2441,12 +2441,14 @@ EXPORT_SYMBOL(blk_fetch_request);
  *     %false - this request doesn't have any more data
  *     %true  - this request has more data
  **/
+extern void blk_sftirq_set_running_flags(unsigned int flags);
 bool blk_update_request(struct request *req, int error, unsigned int nr_bytes)
 {
        int total_bytes;

        trace_block_rq_complete(req->q, req, nr_bytes);

+       blk_sftirq_set_running_flags(0x4000);
        if (!req->bio)
                return false;

diff --git a/block/blk-softirq.c b/block/blk-softirq.c
index 53b1737..e7f9313 100644
--- a/block/blk-softirq.c
+++ b/block/blk-softirq.c
@@ -18,10 +18,14 @@ static DEFINE_PER_CPU(struct list_head, blk_cpu_done);
  * Softirq action handler - move entries to local list and loop over them
  * while passing them to the queue registered handler.
  */
+extern void blk_sftirq_enter(void);
+extern void blk_sftirq_exit(void);
+extern void set_blk_sftirq_raise_time(unsigned int flags);
 static void blk_done_softirq(struct softirq_action *h)
 {
        struct list_head *cpu_list, local_list;

+       blk_sftirq_enter();
        local_irq_disable();
        cpu_list = this_cpu_ptr(&blk_cpu_done);
        list_replace_init(cpu_list, &local_list);
@@ -34,6 +38,7 @@ static void blk_done_softirq(struct softirq_action *h)
                list_del_init(&rq->ipi_list);
                rq->q->softirq_done_fn(rq);
        }
+       blk_sftirq_exit();
 }

 #ifdef CONFIG_SMP
@@ -47,8 +52,10 @@ static void trigger_softirq(void *data)
        list = this_cpu_ptr(&blk_cpu_done);
        list_add_tail(&rq->ipi_list, list);

-       if (list->next == &rq->ipi_list)
+       if (list->next == &rq->ipi_list) {
+               set_blk_sftirq_raise_time(0x1000);
                raise_softirq_irqoff(BLOCK_SOFTIRQ);
+       }

        local_irq_restore(flags);
 }
@@ -91,6 +98,7 @@ static int blk_cpu_notify(struct notifier_block *self, unsigned long action,
                local_irq_disable();
                list_splice_init(&per_cpu(blk_cpu_done, cpu),
                                 this_cpu_ptr(&blk_cpu_done));
+               set_blk_sftirq_raise_time(0x2000);
                raise_softirq_irqoff(BLOCK_SOFTIRQ);
                local_irq_enable();
        }
@@ -144,8 +152,10 @@ do_local:
                 * entries there, someone already raised the irq but it
                 * hasn't run yet.
                 */
-               if (list->next == &req->ipi_list)
+               if (list->next == &req->ipi_list) {
+                       set_blk_sftirq_raise_time(0x4000);
                        raise_softirq_irqoff(BLOCK_SOFTIRQ);
+               }
        } else if (raise_blk_irq(ccpu, req))
                goto do_local;

diff --git a/drivers/scsi/scsi_lib.c b/drivers/scsi/scsi_lib.c
index 448ebda..a9581f0 100644
--- a/drivers/scsi/scsi_lib.c
+++ b/drivers/scsi/scsi_lib.c
@@ -814,6 +814,7 @@ static int __scsi_error_from_host_byte(struct scsi_cmnd *cmd, int result)
  *             c) We can call scsi_end_request() with -EIO to fail
  *                the remainder of the request.
  */
+extern void blk_sftirq_set_running_flags(unsigned int flags);
 void scsi_io_completion(struct scsi_cmnd *cmd, unsigned int good_bytes)
 {
        int result = cmd->result;
@@ -827,6 +828,7 @@ void scsi_io_completion(struct scsi_cmnd *cmd, unsigned int good_bytes)
              ACTION_DELAYED_RETRY} action;
        unsigned long wait_for = (cmd->allowed + 1) * req->timeout;

+       blk_sftirq_set_running_flags(0x2000);
        if (result) {
                sense_valid = scsi_command_normalize_sense(cmd, &sshdr);
                if (sense_valid)
@@ -1618,6 +1620,7 @@ static void scsi_softirq_done(struct request *rq)
        unsigned long wait_for = (cmd->allowed + 1) * rq->timeout;
        int disposition;

+       blk_sftirq_set_running_flags(0x1000);
        INIT_LIST_HEAD(&cmd->eh_entry);

        atomic_inc(&cmd->device->iodone_cnt);
@@ -1640,12 +1643,15 @@ static void scsi_softirq_done(struct request *rq)
                        scsi_finish_command(cmd);
                        break;
                case NEEDS_RETRY:
+                       blk_sftirq_set_running_flags(0x10);
                        scsi_queue_insert(cmd, SCSI_MLQUEUE_EH_RETRY);
                        break;
                case ADD_TO_MLQUEUE:
+                       blk_sftirq_set_running_flags(0x20);
                        scsi_queue_insert(cmd, SCSI_MLQUEUE_DEVICE_BUSY);
                        break;
                default:
+                       blk_sftirq_set_running_flags(0x40);
                        if (!scsi_eh_scmd_add(cmd, 0))
                                scsi_finish_command(cmd);
        }
diff --git a/drivers/usb/storage/scsiglue.c b/drivers/usb/storage/scsiglue.c
index 0e400f3..1ff5f4a 100644
--- a/drivers/usb/storage/scsiglue.c
+++ b/drivers/usb/storage/scsiglue.c
@@ -321,6 +321,7 @@ static int target_alloc(struct scsi_target *starget)

 /* queue a command */
 /* This is always called with scsi_lock(host) held */
+extern void record_wtbk_task_real_wake_time(void);
 static int queuecommand_lck(struct scsi_cmnd *srb,
                        void (*done)(struct scsi_cmnd *))
 {
@@ -344,6 +345,7 @@ static int queuecommand_lck(struct scsi_cmnd *srb,
        /* enqueue the command and wake up the control thread */
        srb->scsi_done = done;
        us->srb = srb;
+       record_wtbk_task_real_wake_time();
        complete(&us->cmnd_ready);

        return 0;
diff --git a/drivers/usb/storage/usb.c b/drivers/usb/storage/usb.c
index 6c10c88..33f95da 100644
--- a/drivers/usb/storage/usb.c
+++ b/drivers/usb/storage/usb.c
@@ -300,15 +300,22 @@ void fill_inquiry_response(struct us_data *us, unsigned char *data,
 }
 EXPORT_SYMBOL_GPL(fill_inquiry_response);

+extern void init_usb_strg_task(void);
+extern void usb_strg_task_enter_sleep(void);
+extern void usb_strg_task_raise_blk_sftirq(void);
+extern void usb_strg_task_exit_sleep(void);
 static int usb_stor_control_thread(void * __us)
 {
        struct us_data *us = (struct us_data *)__us;
        struct Scsi_Host *host = us_to_host(us);
+       init_usb_strg_task();
        for (;;) {
                usb_stor_dbg(us, "*** thread sleeping\n");
+               usb_strg_task_enter_sleep();
                if (wait_for_completion_interruptible(&us->cmnd_ready))
                        break;
+               usb_strg_task_exit_sleep();

                usb_stor_dbg(us, "*** thread awakened\n");

@@ -387,6 +394,7 @@ static int usb_stor_control_thread(void * __us)
                if (us->srb->result != DID_ABORT << 16) {
                        usb_stor_dbg(us, "scsi cmd done, result=0x%x\n",
                                     us->srb->result);
+                       usb_strg_task_raise_blk_sftirq();
                        us->srb->scsi_done(us->srb);
                } else {
 SkipForAbort:
@@ -765,6 +773,7 @@ static int get_pipes(struct us_data *us)
        return 0;
 }

+extern struct task_struct *usb_storage_task;
 /* Initialize all the dynamic resources we need */
 static int usb_stor_acquire_resources(struct us_data *us)
 {
@@ -792,6 +801,7 @@ static int usb_stor_acquire_resources(struct us_data *us)
                                "Unable to start control thread\n");
                return PTR_ERR(th);
        }
+       usb_storage_task = th;
        us->ctl_thread = th;

        return 0;
@@ -806,6 +816,7 @@ static void usb_stor_release_resources(struct us_data *us)
         */
        usb_stor_dbg(us, "-- sending exit command to thread\n");
        complete(&us->cmnd_ready);
+       usb_storage_task = NULL;
        if (us->ctl_thread)
                kthread_stop(us->ctl_thread);

diff --git a/fs/mpage.c b/fs/mpage.c
index 3e79220..9008687 100644
--- a/fs/mpage.c
+++ b/fs/mpage.c
@@ -42,6 +42,8 @@
  * status of that page is hard.  See end_buffer_async_read() for the details.
  * There is no point in duplicating all that complexity.
  */
+extern void blk_sftirq_wakeup_wb(unsigned long page);
+extern void set_wtbk_wake_task_flag(unsigned int flag);
 static void mpage_end_io(struct bio *bio, int err)
 {
        struct bio_vec *bv;
@@ -49,6 +51,7 @@ static void mpage_end_io(struct bio *bio, int err)

        bio_for_each_segment_all(bv, bio, i) {
                struct page *page = bv->bv_page;
+               blk_sftirq_wakeup_wb((unsigned long)page);
                page_endio(page, bio_data_dir(bio), err);
        }

@@ -681,8 +684,10 @@ mpage_writepages(struct address_space *mapping,

        blk_start_plug(&plug);

-       if (!get_block)
+       if (!get_block) {
+               set_wtbk_wake_task_flag(0x200);
                ret = generic_writepages(mapping, wbc);
+       }
        else {
                struct mpage_data mpd = {
                        .bio = NULL,
@@ -692,8 +697,10 @@ mpage_writepages(struct address_space *mapping,
                };

                ret = write_cache_pages(mapping, wbc, __mpage_writepage, &mpd);
-               if (mpd.bio)
+               if (mpd.bio) {
+                       set_wtbk_wake_task_flag(0x400);
                        mpage_bio_submit(WRITE, mpd.bio);
+               }
        }
        blk_finish_plug(&plug);
        return ret;
diff --git a/init/main.c b/init/main.c
index 2a89545..0074d43 100644
--- a/init/main.c
+++ b/init/main.c
@@ -489,6 +489,7 @@ static void __init mm_init(void)
        ioremap_huge_init();
 }

+extern void init_ext2_hung_task_debug(void);
 asmlinkage __visible void __init start_kernel(void)
 {
        char *command_line;
@@ -548,6 +549,7 @@ asmlinkage __visible void __init start_kernel(void)
         */
        setup_log_buf(0);
        pidhash_init();
+       init_ext2_hung_task_debug();
        vfs_caches_init_early();
        sort_main_extable();
        trap_init();
diff --git a/kernel/hung_task.c b/kernel/hung_task.c
index e0f90c2..4e13fe1 100644
--- a/kernel/hung_task.c
+++ b/kernel/hung_task.c
@@ -72,6 +72,562 @@ static struct notifier_block panic_block = {
        .notifier_call = hung_task_panic,
 };

+#define PAGE_NODE_CNT  (20)
+#define USB_STRG_TASK_CNT      (10)
+#define BLK_SFTIRQ_RAISE_CNT   (10)
+struct page_node {
+       unsigned long page;
+       unsigned long long wku_wtbk_time;
+};
+
+struct blk_sftirq {
+       unsigned int flags;
+       unsigned int running_flag;
+
+       spinlock_t lock;
+       unsigned int page_cnt;
+       struct page_node page_node[PAGE_NODE_CNT];
+
+       unsigned long long sleeping_time;
+       unsigned long long wakeup_time;
+};
+
+struct blk_sftirq_raise {
+       struct task_struct *task;
+       unsigned int raise_softirq_cpu;
+       unsigned long long raise_softirq_time;
+};
+
+struct usb_strg_task_back {
+       unsigned int running_flag;
+       unsigned long long done_time;
+       unsigned long long wakeup_time;
+       unsigned long long sleeping_time;
+};
+
+struct usb_strg_task {
+       struct task_struct *task;
+       spinlock_t lock;
+       unsigned int cnt;
+       struct usb_strg_task_back usb_strg_task_back[USB_STRG_TASK_CNT];
+};
+
+struct wtbk_wake_task {
+       struct task_struct *task;
+       unsigned int running_flag;
+       struct list_head node;
+       unsigned long long wku_usb_strg_time;
+       unsigned long long wku_usb_strg_real_time;
+};
+
+struct wtbk_wait_task {
+       struct task_struct *task;
+       unsigned long wait_page;
+       struct list_head node;
+       unsigned long long sleeping_time;
+};
+
+struct ext2_hung_task_debug {
+       spinlock_t wake_lock;
+       struct list_head wtbk_wake_list;
+       struct wtbk_wake_task *wtbk_wake_task_cache;
+
+       spinlock_t wait_lock;
+       struct list_head wtbk_wait_list;
+       struct wtbk_wait_task *wtbk_wait_task_cache;
+
+       struct usb_strg_task usb_strg_task;
+
+       struct blk_sftirq blk_sftirq[NR_CPUS];
+
+       spinlock_t blk_sftirq_lock;
+       unsigned int cnt;
+       struct blk_sftirq_raise blk_sftirq_raise[BLK_SFTIRQ_RAISE_CNT];
+};
+
+struct ext2_hung_task_debug ext2_hung_task_debug;
+
+void blk_sftirq_set_running_flags(unsigned int flags)
+{
+       struct blk_sftirq *blk_sftirq;
+       unsigned int cpu;
+
+       preempt_disable();
+       cpu = smp_processor_id();
+       preempt_enable();
+
+       blk_sftirq = &ext2_hung_task_debug.blk_sftirq[cpu];
+       if (blk_sftirq->flags == 0xa5) {
+               blk_sftirq->running_flag |= flags;
+       }
+}
+
+void blk_sftirq_wakeup_wb(unsigned long page)
+{
+       struct blk_sftirq *blk_sftirq;
+       unsigned int cpu;
+
+       preempt_disable();
+       cpu = smp_processor_id();
+       preempt_enable();
+
+       blk_sftirq = &ext2_hung_task_debug.blk_sftirq[cpu];
+       if (blk_sftirq->flags == 0xa5) {
+               blk_sftirq->running_flag |= 0x2;
+               spin_lock(&blk_sftirq->lock);
+               blk_sftirq->page_cnt++;
+               if (blk_sftirq->page_cnt >= PAGE_NODE_CNT)
+                       blk_sftirq->page_cnt = 0;
+               blk_sftirq->page_node[blk_sftirq->page_cnt].page = page;
+               blk_sftirq->page_node[blk_sftirq->page_cnt].wku_wtbk_time = running_clock();
+               spin_unlock(&blk_sftirq->lock);
+       }
+}
+
+void blk_sftirq_exit(void)
+{
+       struct blk_sftirq *blk_sftirq;
+       unsigned int cpu;
+
+       preempt_disable();
+       cpu = smp_processor_id();
+       preempt_enable();
+
+       blk_sftirq = &ext2_hung_task_debug.blk_sftirq[cpu];
+       blk_sftirq->flags = 0x0;
+       blk_sftirq->running_flag |= 0x4;
+       blk_sftirq->sleeping_time = running_clock();
+}
+
+void blk_sftirq_enter(void)
+{
+       struct blk_sftirq *blk_sftirq;
+       unsigned int cpu;
+
+       preempt_disable();
+       cpu = smp_processor_id();
+       preempt_enable();
+
+       blk_sftirq = &ext2_hung_task_debug.blk_sftirq[cpu];
+       blk_sftirq->flags = 0xa5;
+       blk_sftirq->running_flag = 0x01;
+       blk_sftirq->wakeup_time = running_clock();
+}
+
+void usb_strg_task_enter_sleep(void)
+{
+       struct usb_strg_task *usb_strg_task = &ext2_hung_task_debug.usb_strg_task;
+       struct usb_strg_task_back *usb_strg_task_back;
+
+       spin_lock(&usb_strg_task->lock);
+       usb_strg_task_back = &(usb_strg_task->usb_strg_task_back[usb_strg_task->cnt]);
+       usb_strg_task_back->running_flag |= 0x4;
+       usb_strg_task_back->sleeping_time = running_clock();
+       spin_unlock(&usb_strg_task->lock);
+}
+
+void usb_strg_task_raise_blk_sftirq(void)
+{
+       struct usb_strg_task *usb_strg_task = &ext2_hung_task_debug.usb_strg_task;
+       struct usb_strg_task_back *usb_strg_task_back;
+
+       spin_lock(&usb_strg_task->lock);
+       usb_strg_task_back = &(usb_strg_task->usb_strg_task_back[usb_strg_task->cnt]);
+       usb_strg_task_back->running_flag |= 0x2;
+       usb_strg_task_back->done_time = running_clock();
+       spin_unlock(&usb_strg_task->lock);
+}
+
+void usb_strg_task_exit_sleep(void)
+{
+       struct usb_strg_task *usb_strg_task = &ext2_hung_task_debug.usb_strg_task;
+       struct usb_strg_task_back *usb_strg_task_back;
+
+       spin_lock(&usb_strg_task->lock);
+       usb_strg_task->cnt++;
+       if (usb_strg_task->cnt >= USB_STRG_TASK_CNT)
+               usb_strg_task->cnt = 0;
+       usb_strg_task_back = &(usb_strg_task->usb_strg_task_back[usb_strg_task->cnt]);
+       usb_strg_task_back->running_flag = 0x1;
+       usb_strg_task_back->wakeup_time = running_clock();
+       spin_unlock(&usb_strg_task->lock);
+}
+
+void init_usb_strg_task(void)
+{
+       struct usb_strg_task *tmp = &ext2_hung_task_debug.usb_strg_task;
+
+       tmp->task = current;
+       tmp->cnt = 0;
+}
+
+static void set_usb_strg_task_flag(unsigned int flag)
+{
+       struct usb_strg_task *usb_strg_task = &ext2_hung_task_debug.usb_strg_task;
+       struct usb_strg_task_back *usb_strg_task_back;
+
+       if (usb_strg_task->task == current) {
+               spin_lock(&usb_strg_task->lock);
+               usb_strg_task_back = &(usb_strg_task->usb_strg_task_back[usb_strg_task->cnt]);
+               usb_strg_task_back->running_flag |= flag;
+               spin_unlock(&usb_strg_task->lock);
+       }
+}
+
+void set_blk_sftirq_raise_time(unsigned int flags)
+{
+       struct blk_sftirq_raise *tmp;
+       unsigned int cpu;
+
+       preempt_disable();
+       cpu = smp_processor_id();
+       preempt_enable();
+
+       set_usb_strg_task_flag(flags);
+       spin_lock(&ext2_hung_task_debug.blk_sftirq_lock);
+       ext2_hung_task_debug.cnt++;
+       if (ext2_hung_task_debug.cnt >= BLK_SFTIRQ_RAISE_CNT)
+               ext2_hung_task_debug.cnt = 0;
+       tmp = &ext2_hung_task_debug.blk_sftirq_raise[ext2_hung_task_debug.cnt];
+
+       tmp->task = current;
+       tmp->raise_softirq_time = running_clock();
+       tmp->raise_softirq_cpu = cpu;
+       spin_unlock(&ext2_hung_task_debug.blk_sftirq_lock);
+}
+
+#include <linux/slab.h>
+static unsigned int wrdb_flag = 0;
+int wait_num = 0;
+void release_wtbk_task_wait_info(unsigned long task)
+{
+       struct wtbk_wait_task *wtbk_wait_task;
+
+       while(wrdb_flag == 1)
+               msleep(100);
+       wtbk_wait_task = (struct wtbk_wait_task *)task;
+
+       if (ext2_hung_task_debug.wtbk_wait_task_cache == wtbk_wait_task)
+               ext2_hung_task_debug.wtbk_wait_task_cache = NULL;
+
+       spin_lock(&ext2_hung_task_debug.wait_lock);
+       wait_num--;
+       list_del(&wtbk_wait_task->node);
+       spin_unlock(&ext2_hung_task_debug.wait_lock);
+
+       kfree(wtbk_wait_task);
+}
+
+void record_wtbk_task_wait_page(unsigned long task, unsigned long page)
+{
+       struct wtbk_wait_task *wtbk_wait_task = (struct wtbk_wait_task *)task;
+       if (wtbk_wait_task) {
+               wtbk_wait_task->sleeping_time = running_clock();
+               wtbk_wait_task->wait_page = page;
+       }
+}
+
+unsigned long get_wtbk_wait_task(struct task_struct *p)
+{
+       struct wtbk_wait_task *tmp, *wtbk_wait_task = NULL;
+
+       if (ext2_hung_task_debug.wtbk_wait_task_cache &&
+          (ext2_hung_task_debug.wtbk_wait_task_cache->task == p)) {
+               wtbk_wait_task = ext2_hung_task_debug.wtbk_wait_task_cache;
+       } else {
+               spin_lock(&ext2_hung_task_debug.wait_lock);
+               list_for_each_entry_reverse(tmp, &ext2_hung_task_debug.wtbk_wait_list, node) {
+                       if (tmp->task == p) {
+                               wtbk_wait_task = tmp;
+                               break;
+                       }
+               }
+               spin_unlock(&ext2_hung_task_debug.wait_lock);
+       }
+
+       return  (unsigned long)wtbk_wait_task;
+}
+
+unsigned long record_wtbk_task_wait_info(void)
+{
+       struct wtbk_wait_task *wtbk_wait_task;
+
+       wtbk_wait_task = kzalloc(sizeof(struct wtbk_wait_task), GFP_KERNEL);
+
+       wtbk_wait_task->task = current;
+
+       INIT_LIST_HEAD(&wtbk_wait_task->node);
+       spin_lock(&ext2_hung_task_debug.wait_lock);
+       list_add_tail(&wtbk_wait_task->node, &ext2_hung_task_debug.wtbk_wait_list);
+       wait_num++;
+       spin_unlock(&ext2_hung_task_debug.wait_lock);
+       ext2_hung_task_debug.wtbk_wait_task_cache = wtbk_wait_task;
+
+       return (unsigned long) wtbk_wait_task;
+}
+
+int wake_num = 0;
+void release_wtbk_task_wake_info(void)
+{
+       struct wtbk_wake_task *tmp, *wtbk_wake_task = NULL;
+
+       if (ext2_hung_task_debug.wtbk_wake_task_cache &&
+          (ext2_hung_task_debug.wtbk_wake_task_cache->task == current)) {
+               wtbk_wake_task = ext2_hung_task_debug.wtbk_wake_task_cache;
+               while(wrdb_flag == 1)
+                       msleep(100);
+               ext2_hung_task_debug.wtbk_wake_task_cache = NULL;
+       } else {
+               spin_lock(&ext2_hung_task_debug.wake_lock);
+               list_for_each_entry_reverse(tmp, &ext2_hung_task_debug.wtbk_wake_list, node) {
+                       if (tmp->task == current) {
+                               wtbk_wake_task = tmp;
+                                       break;
+                       }
+               }
+               spin_unlock(&ext2_hung_task_debug.wake_lock);
+       }
+
+       if (wtbk_wake_task) {
+               spin_lock(&ext2_hung_task_debug.wake_lock);
+               wake_num--;
+               list_del(&wtbk_wake_task->node);
+               spin_unlock(&ext2_hung_task_debug.wake_lock);
+               while(wrdb_flag == 1)
+                       msleep(100);
+               kfree(wtbk_wake_task);
+       }
+}
+
+void set_wtbk_wake_task_flag(unsigned int flag)
+{
+       struct wtbk_wake_task *tmp, *wtbk_wake_task = NULL;
+
+       if (ext2_hung_task_debug.wtbk_wake_task_cache &&
+          (ext2_hung_task_debug.wtbk_wake_task_cache->task == current)) {
+               wtbk_wake_task = ext2_hung_task_debug.wtbk_wake_task_cache;
+
+       } else {
+               spin_lock(&ext2_hung_task_debug.wake_lock);
+               list_for_each_entry_reverse(tmp, &ext2_hung_task_debug.wtbk_wake_list, node) {
+                       if (tmp->task == current) {
+                               wtbk_wake_task = tmp;
+                               break;
+                       }
+               }
+               spin_unlock(&ext2_hung_task_debug.wake_lock);
+       }
+
+       if (wtbk_wake_task) {
+               wtbk_wake_task->running_flag |= flag;
+       }
+}
+
+void record_wtbk_task_real_wake_time(void)
+{
+       struct wtbk_wake_task *tmp, *wtbk_wake_task = NULL;
+
+       if (ext2_hung_task_debug.wtbk_wake_task_cache &&
+          (ext2_hung_task_debug.wtbk_wake_task_cache->task == current)) {
+               wtbk_wake_task = ext2_hung_task_debug.wtbk_wake_task_cache;
+       } else {
+               spin_lock(&ext2_hung_task_debug.wake_lock);
+               list_for_each_entry_reverse(tmp, &ext2_hung_task_debug.wtbk_wake_list, node) {
+                       if (tmp->task == current) {
+                               wtbk_wake_task = tmp;
+                               break;
+                       }
+               }
+               spin_unlock(&ext2_hung_task_debug.wake_lock);
+       }
+
+       if (wtbk_wake_task) {
+               wtbk_wake_task->wku_usb_strg_real_time = running_clock();
+       }
+}
+
+void record_wtbk_task_wake_info(void)
+{
+       struct wtbk_wake_task *wtbk_wake_task;
+
+       wtbk_wake_task = kzalloc(sizeof(struct wtbk_wake_task), GFP_KERNEL);
+
+       wtbk_wake_task->task = current;
+       wtbk_wake_task->running_flag = 0x1;
+       wtbk_wake_task->wku_usb_strg_time = running_clock();
+
+       INIT_LIST_HEAD(&wtbk_wake_task->node);
+       spin_lock(&ext2_hung_task_debug.wake_lock);
+       list_add_tail(&wtbk_wake_task->node, &ext2_hung_task_debug.wtbk_wake_list);
+       wake_num++;
+       spin_unlock(&ext2_hung_task_debug.wake_lock);
+       ext2_hung_task_debug.wtbk_wake_task_cache = wtbk_wake_task;
+}
+
+void init_ext2_hung_task_debug(void)
+{
+       ext2_hung_task_debug.wtbk_wake_task_cache = NULL;
+       ext2_hung_task_debug.wtbk_wait_task_cache = NULL;
+       INIT_LIST_HEAD(&ext2_hung_task_debug.wtbk_wake_list);
+       INIT_LIST_HEAD(&ext2_hung_task_debug.wtbk_wait_list);
+
+       ext2_hung_task_debug.cnt = 0;
+}
+
+static void print_wtbk_wake_info(unsigned long task)
+{
+       struct wtbk_wake_task *tmp, *wtbk_wake_task = NULL;
+       struct wtbk_wait_task *wtbk_wait_task = (struct wtbk_wait_task *)task;
+       struct task_struct *p = wtbk_wait_task->task;
+
+       if (ext2_hung_task_debug.wtbk_wake_task_cache &&
+          (ext2_hung_task_debug.wtbk_wake_task_cache->task == p)) {
+               wtbk_wake_task = ext2_hung_task_debug.wtbk_wake_task_cache;
+       } else {
+               spin_lock(&ext2_hung_task_debug.wake_lock);
+               list_for_each_entry_reverse(tmp, &ext2_hung_task_debug.wtbk_wake_list, node) {
+                       if (tmp->task == p) {
+                               wtbk_wake_task = tmp;
+                                       break;
+                       }
+               }
+               spin_unlock(&ext2_hung_task_debug.wake_lock);
+       }
+       if (wtbk_wake_task) {
+               pr_emerg("Writeback wake task:(%d), running_flag:0x%x,\n"
+                               "\t\t wku_usb_strg_time:%lld\n"
+                               "\t\t wku_usb_strg_real_time:%lld\n",
+                               wtbk_wake_task->task->pid,
+                               wtbk_wake_task->running_flag,
+                               wtbk_wake_task->wku_usb_strg_time,
+                               wtbk_wake_task->wku_usb_strg_real_time);
+       }
+}
+
+static void print_wtbk_wait_info(unsigned long task)
+{
+       struct wtbk_wait_task *wtbk_wait_task = (struct wtbk_wait_task *)task;
+       pr_emerg("Writeback wait task:(%d), wait_page:0x%lx,\n"
+                       "\t\t sleeping_time:%lld\n",
+                       wtbk_wait_task->task->pid,
+                       wtbk_wait_task->wait_page,
+                       wtbk_wait_task->sleeping_time);
+}
+
+static void print_usb_strg_info(void)
+{
+       struct usb_strg_task *usb_strg_task = &ext2_hung_task_debug.usb_strg_task;
+       unsigned int i;
+       unsigned int cnt = usb_strg_task->cnt;
+       pr_emerg("USB storage info: pid:%d, cnt:%d\n",
+                       usb_strg_task->task->pid, cnt);
+       for (i = cnt + 1; i < USB_STRG_TASK_CNT; i++) {
+               pr_emerg("\ti:%d, running_flag:0x%x, done_time:%lld,\n"
+                               "\t\t wakeup_time:%lld, sleeping_time:%lld\n",
+                               i, usb_strg_task->usb_strg_task_back[i].running_flag,
+                               usb_strg_task->usb_strg_task_back[i].done_time,
+                               usb_strg_task->usb_strg_task_back[i].wakeup_time,
+                               usb_strg_task->usb_strg_task_back[i].sleeping_time);
+       }
+       for (i = 0; i <= cnt; i++) {
+               pr_emerg("\ti:%d, running_flag:0x%x, done_time:%lld,\n"
+                               "\t\t wakeup_time:%lld, sleeping_time:%lld\n",
+                               i, usb_strg_task->usb_strg_task_back[i].running_flag,
+                               usb_strg_task->usb_strg_task_back[i].done_time,
+                               usb_strg_task->usb_strg_task_back[i].wakeup_time,
+                               usb_strg_task->usb_strg_task_back[i].sleeping_time);
+       }
+}
+
+static void print_blk_sftirq_raise_info(void)
+{
+       unsigned int i;
+       unsigned int cnt = ext2_hung_task_debug.cnt;
+       pr_emerg("Raise BLK softirq info:\n");
+       for (i = cnt + 1; i < BLK_SFTIRQ_RAISE_CNT; i++) {
+               pr_emerg("i:%d, pid:%d, raise_softirq_cpu:%d, raise_softirq_time:%lld\n",
+                               i, ext2_hung_task_debug.blk_sftirq_raise[i].task->pid,
+                               ext2_hung_task_debug.blk_sftirq_raise[i].raise_softirq_cpu,
+                               ext2_hung_task_debug.blk_sftirq_raise[i].raise_softirq_time);
+       }
+       for (i = 0; i <= cnt; i++) {
+               pr_emerg("i:%d, pid:%d, raise_softirq_cpu:%d, raise_softirq_time:%lld\n",
+                               i, ext2_hung_task_debug.blk_sftirq_raise[i].task->pid,
+                               ext2_hung_task_debug.blk_sftirq_raise[i].raise_softirq_cpu,
+                               ext2_hung_task_debug.blk_sftirq_raise[i].raise_softirq_time);
+       }
+}
+
+static void print_blk_sftirq_info(void)
+{
+       unsigned int i;
+       pr_emerg("BLK softirq info:\n");
+       for (i = 0; i < NR_CPUS; i++) {
+               unsigned int j;
+               if (!ext2_hung_task_debug.blk_sftirq[i].wakeup_time)
+                       continue;
+               pr_emerg("\tcpu:%d,flags:0x%x,running_flag:0x%x\n"
+                               "\t\tpage_cnt:%d,sleeping_time:%lld,wakeup_time:%lld\n",
+                               i, ext2_hung_task_debug.blk_sftirq[i].flags,
+                               ext2_hung_task_debug.blk_sftirq[i].running_flag,
+                               ext2_hung_task_debug.blk_sftirq[i].page_cnt,
+                               ext2_hung_task_debug.blk_sftirq[i].sleeping_time,
+                               ext2_hung_task_debug.blk_sftirq[i].wakeup_time);
+               pr_emerg("\t\tpage_node info:\n");
+               for (j = ext2_hung_task_debug.blk_sftirq[i].page_cnt + 1; j < PAGE_NODE_CNT; j++) {
+                       if (!ext2_hung_task_debug.blk_sftirq[i].page_node[j].page)
+                               continue;
+                       pr_emerg("\t\tj:%d, page:0x%lx, wku_wtbk_time:%lld\n",
+                                       j, ext2_hung_task_debug.blk_sftirq[i].page_node[j].page,
+                                       ext2_hung_task_debug.blk_sftirq[i].page_node[j].wku_wtbk_time);
+               }
+               for (j = 0; j <= ext2_hung_task_debug.blk_sftirq[i].page_cnt; j++) {
+                       if (!ext2_hung_task_debug.blk_sftirq[i].page_node[j].page)
+                               continue;
+                       pr_emerg("\t\tj:%d, page:0x%lx, wku_wtbk_time:%lld\n",
+                                       j, ext2_hung_task_debug.blk_sftirq[i].page_node[j].page,
+                                       ext2_hung_task_debug.blk_sftirq[i].page_node[j].wku_wtbk_time);
+               }
+       }
+}
+static void print_ext2_hung_task_debug_info(unsigned long wrbk_wait_task)
+{
+       pr_emerg("wake_num:%d,wait_num:%d\n",wake_num, wait_num);
+       print_wtbk_wake_info(wrbk_wait_task);
+       print_wtbk_wait_info(wrbk_wait_task);
+       print_usb_strg_info();
+       print_blk_sftirq_raise_info();
+       print_blk_sftirq_info();
+}
+
+#include <linux/sched.h>
+
+static void print_hung_task_debug_info(struct task_struct *t)
+{
+       unsigned long wrbk_wait_task;
+       pr_emerg("=========WRDB:DEBUG PATCH V2=================\n");
+       wrdb_flag = 1;
+       wrbk_wait_task = get_wtbk_wait_task(t);
+       if (wrbk_wait_task) {
+               if (ext2_hung_task_debug.usb_strg_task.task)
+                       sched_show_task(ext2_hung_task_debug.usb_strg_task.task);
+               print_ext2_hung_task_debug_info(wrbk_wait_task);
+       }
+       wrdb_flag = 0;
+       pr_emerg("\n\n");
+}
+
+extern void patch_test(void);
+void patch_test(void)
+{
+       pr_emerg("=========WRDB:DEBUG PATCH V2 test=================\n");
+       if (ext2_hung_task_debug.wtbk_wait_task_cache && ext2_hung_task_debug.wtbk_wait_task_cache->task)
+               print_hung_task_debug_info(ext2_hung_task_debug.wtbk_wait_task_cache->task);
+}
+
+struct task_struct *usb_storage_task = NULL;
 static void check_hung_task(struct task_struct *t, unsigned long timeout)
 {
        unsigned long switch_count = t->nvcsw + t->nivcsw;
@@ -116,7 +672,10 @@ static void check_hung_task(struct task_struct *t, unsigned long timeout)
                init_utsname()->version);
        pr_err("\"echo 0 > /proc/sys/kernel/hung_task_timeout_secs\""
                " disables this message.\n");
+       print_hung_task_debug_info(t);
        sched_show_task(t);
+       if (usb_storage_task)
+               sched_show_task(usb_storage_task);
        debug_show_held_locks(t);

        touch_nmi_watchdog();
@@ -245,7 +804,6 @@ static int __init hung_task_init(void)
 {
        atomic_notifier_chain_register(&panic_notifier_list, &panic_block);
        watchdog_task = kthread_run(watchdog, NULL, "khungtaskd");
-
        return 0;
 }
 subsys_initcall(hung_task_init);
diff --git a/mm/filemap.c b/mm/filemap.c
index 1ffef05..1f371d8 100644
--- a/mm/filemap.c
+++ b/mm/filemap.c
@@ -267,6 +267,7 @@ static int filemap_check_errors(struct address_space *mapping)
  * these two operations is that if a dirty page/buffer is encountered, it must
  * be waited upon, and not just skipped over.
  */
+extern void set_wtbk_wake_task_flag(unsigned int flag);
 int __filemap_fdatawrite_range(struct address_space *mapping, loff_t start,
                                loff_t end, int sync_mode)
 {
@@ -278,8 +279,10 @@ int __filemap_fdatawrite_range(struct address_space *mapping, loff_t start,
                .range_end = end,
        };

-       if (!mapping_cap_writeback_dirty(mapping))
+       if (!mapping_cap_writeback_dirty(mapping)) {
+               set_wtbk_wake_task_flag(0x10);
                return 0;
+       }

        ret = do_writepages(mapping, &wbc);
        return ret;
@@ -326,6 +329,8 @@ EXPORT_SYMBOL(filemap_flush);
  * Walk the list of under-writeback pages of the given address space
  * in the given range and wait for all of them.
  */
+extern unsigned long get_wtbk_wait_task(struct task_struct *p);
+extern void record_wtbk_task_wait_page(unsigned long task, unsigned long page);
 int filemap_fdatawait_range(struct address_space *mapping, loff_t start_byte,
                            loff_t end_byte)
 {
@@ -334,10 +339,13 @@ int filemap_fdatawait_range(struct address_space *mapping, loff_t start_byte,
        struct pagevec pvec;
        int nr_pages;
        int ret2, ret = 0;
+       unsigned long task;

        if (end_byte < start_byte)
                goto out;

+       task = get_wtbk_wait_task(current);
+
+
        pagevec_init(&pvec, 0);
        while ((index <= end) &&
                        (nr_pages = pagevec_lookup_tag(&pvec, mapping, &index,
@@ -352,6 +360,7 @@ int filemap_fdatawait_range(struct address_space *mapping, loff_t start_byte,
                        if (page->index > end)
                                continue;

+                       record_wtbk_task_wait_page(task, (unsigned long)page);
                        wait_on_page_writeback(page);
                        if (TestClearPageError(page))
                                ret = -EIO;
@@ -421,21 +430,30 @@ EXPORT_SYMBOL(filemap_write_and_wait);
  * Note that `lend' is inclusive (describes the last byte to be written) so
  * that this function can be used to write to the very end-of-file (end = -1).
  */
+extern void record_wtbk_task_wake_info(void);
+extern void release_wtbk_task_wake_info(void);
+extern unsigned long record_wtbk_task_wait_info(void);
+extern void release_wtbk_task_wait_info(unsigned long task);
 int filemap_write_and_wait_range(struct address_space *mapping,
                                 loff_t lstart, loff_t lend)
 {
        int err = 0;

        if (mapping->nrpages) {
+               record_wtbk_task_wake_info();
                err = __filemap_fdatawrite_range(mapping, lstart, lend,
                                                 WB_SYNC_ALL);
                /* See comment of filemap_write_and_wait() */
                if (err != -EIO) {
+                       unsigned long task;
+                       task = record_wtbk_task_wait_info();
                        int err2 = filemap_fdatawait_range(mapping,
                                                lstart, lend);
                        if (!err)
                                err = err2;
+                       release_wtbk_task_wait_info(task);
                }
+               release_wtbk_task_wake_info();
        } else {
                err = filemap_check_errors(mapping);
        }
diff --git a/mm/page-writeback.c b/mm/page-writeback.c
index eb59f7e..3da09a9ba 100644
--- a/mm/page-writeback.c
+++ b/mm/page-writeback.c
@@ -2026,12 +2026,15 @@ int generic_writepages(struct address_space *mapping,
 EXPORT_SYMBOL(generic_writepages);

+extern void set_wtbk_wake_task_flag(unsigned int flag);
 int do_writepages(struct address_space *mapping, struct writeback_control *wbc)
 {
        int ret;

-       if (wbc->nr_to_write <= 0)
+       if (wbc->nr_to_write <= 0) {
+               set_wtbk_wake_task_flag(0x100);
                return 0;
+       }
        if (mapping->a_ops->writepages)
                ret = mapping->a_ops->writepages(mapping, wbc);
        else
--
1.8.3.1

```
