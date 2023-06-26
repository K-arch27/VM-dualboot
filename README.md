# VM-dualboot
Instruction For Dual-booting Arch with a VM with GPU passthrough
(I Use grub as a bootloader but should work with any other bootloader)
This will guide you into installing a VM with GPU passthrough that start automatically when selecting it from the bootloader, and stays off when booting the main system with the GPU


## 0. Prerequisite
Your CPU and motherboard must support hardware virtualization and IOMMU 

Your GPU ROM must support UEFI. (All GPUs from 2012 and later should support this)

The following packages : Qemu Ovmf Libvirt Dnsmasq Virt-manager (last one is optionnal if you want a gui to make your VM)

On Arch you can install them with the following command :
```
pacman -S qemu-desktop libvirt edk2-ovmf dnsmasq virt-manager
```

## 1. Setting up the VM


## 3. Setting up the bootloader


## 4. Setting up GPU passthrough


## 5. Setting up script to start VM when booting the passthrough entry


