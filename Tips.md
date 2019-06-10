
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
