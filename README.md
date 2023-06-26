# VM-dualboot
Instruction For Dual-booting Arch and a VM with GPU passthrough

This will guide you into installing a VM with GPU passthrough that start automatically when selecting it from the bootloader, and stays off when booting the main system with the GPU
(I Use grub as a bootloader but should work with any other bootloader)


## 0. Prerequisite
- Your CPU and motherboard must support hardware virtualization and IOMMU 

- Enable needed virtualization options in the BIOS the setting may be called VT-d or AMD-V also Enable Intel VT-x or AMD IOMMU if the options are available.

- Your GPU ROM must support UEFI. (All GPUs from 2012 and later should support this)

- The following packages : Qemu Ovmf Libvirt Dnsmasq Virt-manager (last one is optionnal if you want a gui to make your VM)

On Arch you can install them with the following command :
```
pacman -S qemu-desktop libvirt edk2-ovmf dnsmasq virt-manager
```

## 1. Setting up the VM

- Setup your VM with the ovmf firmware

If using Virt Manager

Tick the option to customise before install

![image](https://github.com/K-arch27/VM-dualboot/assets/98610690/4a52f965-2dbe-4b89-b750-47a677ed6e2d)

Choose the Ovmf firmware you need (in doubt take the base x64 one in blue)
![image](https://github.com/K-arch27/VM-dualboot/assets/98610690/1a5d397b-181c-4db7-b2f1-4886a258bfa6)

- Then proceed to the installation as normal, setup what you want inside your VM



## 2. Setting up the bootloader

- Duplicate the entry to boot your system

  For grub in can be found inside /boot/grub/grub.cfg after the line ### BEGIN /etc/grub.d/10_linux ###
  
```
menuentry 'Arch Linux' --class arch --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-2cebc8e6-54aa-4f18-b669-a509413594e0' {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_gpt
	insmod btrfs
	set root='hd2,gpt3'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd2,gpt3 --hint-efi=hd2,gpt3 --hint-baremetal=ahci2,gpt3  2cebc8e6-54aa-4f18-b669-a509413594e0
	else
	  search --no-floppy --fs-uuid --set=root 2cebc8e6-54aa-4f18-b669-a509413594e0
	fi
	echo	'Loading Linux linux-zen ...'
	linux	/@/.snapshots/1/snapshot/boot/vmlinuz-linux-zen root=UUID=2cebc8e6-54aa-4f18-b669-a509413594e0 rw rootflags=subvol=@/.snapshots/1/snapshot  loglevel=3 quiet
	echo	'Loading initial ramdisk ...'
	initrd	/@/.snapshots/1/snapshot/boot/intel-ucode.img /@/.snapshots/1/snapshot/boot/initramfs-linux-zen.img
}
```
copy this entire part from your own boot entry to a new file in /etc/grub.d/21_arch_passthrough


- Then add the needed kernel parameters iommu=pt and intel_iommu=on if using intel to the linux line inside the new file

```
linux	/@/.snapshots/1/snapshot/boot/vmlinuz-linux-zen root=UUID=2cebc8e6-54aa-4f18-b669-a509413594e0 rw rootflags=subvol=@/.snapshots/1/snapshot  loglevel=3 quiet intel_iommu=on iommu=pt
```

## 3. Setting up GPU passthrough


---
### windows note
---

Windows should detect and install a basic version of the drivers needed when booting with the passthrough but can fail to do so

Happenned to me with an edited version of windows and I had to install the nvidia drivers manually without a screen (yay muscle memory) before being able to see

Boot back to your OS without Gpu passthrough ( if using udev hit both ctrl keys to control the Linux Os, then switch to tty2 with ctrl+alt+2 login and then type reboot)

remove the GPU pci from the VM and put back the spice display and start it 

download the drivers and extract them 

make a shortcut of the installer in a corner of the desktop to be able to launch it once back in the dark

test out the installer to see how it work and what keyboard key you need to use to proceed (so it can be done without seeing a mouse)

turn the vm off and add back the GPU pci and remove the spice display

reboot back in passthrough mode

login and lauch the installer from the corner

Wait a couple seconds to make sure it loaded and proceed to the installation with the test made before rebooting

for Nvidia Default option is next so press Space and wait again a couple seconds, Space again to start the install and then if made properly the screen should eventually appear (5 to 10 min) 

---

## 4. Setting up script and service to start VM when booting the passthrough entry

- Create a script that will start your VM and copy it to somewhere in $path (ex : /usr/local/bin/ )

start_vm.sh :
```
#!/bin/bash

if grep -q "vfio-pci.ids" /proc/cmdline; then
    virsh start MY-VM
else
    echo "iommu not active, Skipping VM start"
fi
```
this will check if the kernel parameters contain the vfio-pci instruction and start the VM if so

change "MY-VM" for the name of your VM

- Give excution permission to your script

```
chmod +x /usr/local/bin/start_vm.sh
```

- Create a new systemd service to start your script with the following :
start_vm.service :

```
[Unit]
Description=Start VM with iommu
After=multi-user.target

[Service]
ExecStart=/bin/bash /usr/local/bin/start_vm

[Install]
WantedBy=multi-user.target
```

- Move it to /etc/systemd/system/start_vm.service

- Enable it with systemctl enable start_vm.service
