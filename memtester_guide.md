# CXL Memory Benchmarking manual

**Sean Hwang**

Technical Graduate Intern  

seahwang@cisco.com 

+1 (808) 971-1295 

<br>

version 0.1 (last updated on August 10, 2022)

<br>

## Contents:

- [1. Booting](#1-booting)
  - [1.1. Booting to RHEL](#11-booting-to-rhel)
  - [1.2. Booting to Fedora 36](#12-booting-to-fedora-36)
  - [1.3. Remote access using CIMC / vKVM](#13-remote-access-using-cimc--vkvm)
- [2. Memory Benchmarking test on CXL memory](#2-memory-benchmarking-test-on-cxl-memory)
  - [2.1. Configuring FPGA as a CXL Device](#21-configuring-fpga-as-a-cxl-device)
  - [2.2. Reconfiguring CXL card memory](#22-reconfiguring-cxl-card-memory)
  - [2.3. Adding CXL to Memory Type Range Register (MTRR)](#23-adding-cxl-to-memory-type-range-register-mtrr)
  - [2.4. Memtester](#24-memtester)
- [3. System Environment Setup](#3-system-environment-setup)
  - [3.1. Accessing BMC of the server unit](#31-accessing-bmc-of-the-server-unit)
  - [3.2. Enabling advanced BIOS menu (hidden menu)](#32-enabling-advanced-bios-menu-hidden-menu)
  - [3.3. Configuring fan speed of unit](#33-configuring-fan-speed-of-unit)
  - [3.4. Pulling out the BMC log](#34-pulling-out-the-bmc-log)
  - [3.5. Changing BIOS version](#35-changing-bios-version)
- [4. Debugging Guide to potential issues](#4-debugging-guide-to-potential-issues)
  - [4.1. 'Kernel not found' issue during boot](#41-kernel-not-found-issue-during-boot)
  - [4.2. 'failed to mmap /dev/mem for physical memory'](#42-failed-to-mmap-devmem-for-physical-memory)
  - [4.3. System crash / freeze while running memtester](#43-system-crash--freeze-while-running-memtester)

<div style="page-break-after: always;"></div>

# 1. Booting

This section contains booting instructions of the server unit. The unit used for this document is Cisco Mt. Rainier M7. 

> Username: `root` , Password: `Free4All` 

## 1.1. Booting to RHEL

- Boot to BIOS by pressing F2 on boot screen. 

- Navigate to `Save & Exit > Boot Override`, and select `UEFI: Built-in EFI Shell`
  
  - It is helpful to change Boot option Priorities to EFI Shell. 

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-13-34-image.png" alt="" width="435" data-align="center">

- On EFI Shell, enter `FS4:` followed by `EFI\fedora\grubx64.efi` 

<img src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-15-57-image.png" title="" alt="" data-align="inline">

- On the **grub menu**, navigate to `M2 - Red Hat Enterprise Linux Server 9.0` 

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-33-45-image.png" alt="" data-align="center" width="483">

- Hit `E` on the keyboard to edit the commands. 

- Navigate to `linux /boot/vmlinuz-5.14.0-70.13.1.el9_0.x86_64` and add `iomem=relaxed` at the end of it .
  
  - This is all the command line parameters that will be loaded into kernel upon booting. 
  - if Automatically boot ,,, memtester mmap error 

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-36-50-image.png" alt="" data-align="center" width="517">

- Hit `F10` or `Ctrl-x` to boot to RHEL 9.0. 

## 1.2. Booting to Fedora 36

- Follow the same procedure with [1.1. Booting to RHEL](#11-booting-to-rhel) to enter the grub menu. 

- On the grub menu, navigate to `Fedora release 36`.

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-49-35-image.png" alt="" data-align="center" width="505">

- Hit `E` on the keyboard to edit the commands.

- Navigate to `linux /boot/vmlinuz-5.14.0-70.13.1.el9_0.x86_64` and add `iomem=relaxed` at the end of it .

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-15-36-50-image.png?msec=1659998212785" alt="" data-align="center" width="514">

- Hit `F10` or `Ctrl-x` to boot to Fedora 36. 

## 1.3. Remote access using CIMC / vKVM

> Network connection required. Check and see if network card is on PCIe riser 3 and has a connection made to a switch via ethernet cable. 

- CIMC 

<img title="" src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-10-52-18-image.png" alt="" width="487" data-align="center">

# 2. Memory Benchmarking test on CXL memory

Below is the summary of commands 

```shell
# Set to Root mode
sudo su 

# Configuring FPGA as CXL device 
sudo setpci -s 27:0.0 0x4.L=0x7 #For CXL card mounted on Riser 1 
sudo setpci -s 98:0.0 0x4.L=0x7 #For CXL card mounted on Riser 2

# Reconfiguring CXL memory to system memory 
sudo daxctl reconfigure-device --mode=system-ram --no-online --force dax0.0

# Adding CXL to Memory Type Range Register (MTRR)
sudo echo "base=0x2080000000 size=0x80000000 type=uncachable" > /proc/mtrr

# Running memtester sequence 
# memtester [-p PHYSADDR] <MEMORY> [ITERATIONS] 
./memtester -p 0x2080000000 1M 10 #Malloc size, Loop count
./memtester -p 0x2080000000 1M 10 > log.txt #If you wish to save log  
```

## 2.1. Configuring FPGA as a CXL Device

## 2.2. Reconfiguring CXL card memory

## 2.3. Adding CXL to Memory Type Range Register (MTRR)

## 2.4. Memtester

Path: 

## 2.5 Modified Memtesters

## 2.6 Automated Scripts

# 3. System Environment Setup

## 3.1. Accessing BMC of the server unit

## 3.2. Enabling advanced BIOS menu (hidden menu)

## 3.3. Configuring fan speed of unit

## 3.4. Pulling out the BMC log

## 3.5. Changing BIOS version

# 4. Debugging Guide to potential issues

## 4.1. 'Kernel not found' issue during boot

Booting issue : you may need to load the kernel first 

<img src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-10-53-16-image.png" title="" alt="" data-align="center">fix: disable secure boot, boot with fedora using FS4: ... 

Esteban has made partitions 

## 4.2. 'failed to mmap /dev/mem for physical memory'

Issue: mmap for physical memory: operation not permitted 

<img src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-08-08-10-54-04-image.png" title="" alt="" data-align="center">Current setting has the kernel setup with parameters to avoid memory map restriction error. 

fix : kernel parameter : iomem=relaxed 

temporary : boot menu add kernel parameter 

permanent : Modifying grub menu to include specific parameter. 

## 4.3. System crash / freeze while running memtester

Issue: Memory mismatch 

<img src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-07-12-15-13-54-image.png" title="" alt="" data-align="center">

Issue: system freeze 

<img src="file:///C:/Users/seahwang/AppData/Roaming/marktext/images/2022-07-12-15-15-12-image.png" title="" alt="" data-align="center">

```
<div style="page-break-after: always;"></div>
```

<div style="page-break-after: always;"></div>
