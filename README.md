# VM-dualboot
Instruction For Dual-booting Arch and a VM with GPU passthrough

This will guide you into installing a VM with GPU passthrough that start automatically when selecting it from the bootloader, and stays off when booting the main system with the GPU
(I Use grub as a bootloader but should work with any other bootloader)


## 0. Prerequisite
Your CPU and motherboard must support hardware virtualization and IOMMU 

Your GPU ROM must support UEFI. (All GPUs from 2012 and later should support this)

The following packages : Qemu Ovmf Libvirt Dnsmasq Virt-manager (last one is optionnal if you want a gui to make your VM)

On Arch you can install them with the following command :
```
pacman -S qemu-desktop libvirt edk2-ovmf dnsmasq virt-manager
```

## 1. Setting up the VM
Setup your VM with the ovmf firmware

If using Virt Manager

Tick the option to customise before install

![image](https://github.com/K-arch27/VM-dualboot/assets/98610690/4a52f965-2dbe-4b89-b750-47a677ed6e2d)

Choose the Ovmf firmware you need (in doubt take the base x64 one in blue)
![image](https://github.com/K-arch27/VM-dualboot/assets/98610690/1a5d397b-181c-4db7-b2f1-4886a258bfa6)

Then proceed to the installation as normal, setup what you want inside your VM



## 3. Setting up the bootloader


## 4. Setting up GPU passthrough



### windows note

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


## 5. Setting up script to start VM when booting the passthrough entry


