# Hypervisors
## KVM
Kernel-based Virtual Machine (KVM) is a module in the Linux kernel that allows the Linux OS to operate as a hypervisor. This enables a native install of Linux to act as the host for a virtual machine. In combination with QEMU and libVirt, a KVM virtual machine emulates a completely independent system that can be limited to a subset of the host hardware, and that can also run completely different operating systems from the host.

An overview of KVM is at [https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine).

The benefit of a virtual machine is strong isolation. A virtual machine is isolated from other virtual machines and from the host. This isolation provides a high level of security because one application in a virtual machine may not be aware that it is in a virtual machine, much less that there may be other virtual machines with other applications. Also, a virtual machine provides separation from the host.

Even if a driver in the virtual machine crashes the entire virtual machine, that crash will not take down the host and, therefore, will allow other services in other virtual machines to continue operating.

The cost of these benefits is additional overhead to set up the system. Also, additional processing may be required at runtime between the virtual machine and the native hardware.

AIC100 only supports PCIe passthrough to a virtual machine. This means that the virtual machine completely owns the AIC100 device. The AIC100 device cannot be shared between virtual machines, or between a virtual machine and the native host.

The generic setup and operation of a KVM virtual machine is outside the scope of this document. This document assumes that the reader is familiar with those operations and, thus, only explains the elements directly related to assigning an AIC100 device to a KVM virtual machine.

AIC100 supports only the operating systems listed [here](../index.md#supported-operating-systems-hypervisors-and-platforms) as the operating system running within the virtual machine as the guest OS.

Within a virtual machine, AIC100 still requires the assignment of 32 message signal interrupts (MSI) to operate, which requires the virtual machine to emulate an IOMMU. During the creation of a virtual machine, the virtual machine must be configured to emulate a system that can emulate an IOMMU.
 
One such system is the “q35” system. If using “virt-install”, a q35 based virtual machine can be created by adding “—machine=q35” to the virt-install command.

The remainder of this section assumes that the virtual machine is created with virt-install and the –machine=q35 option. Other systems may require different configurations than what is described to obtain the same end effect.

After the virtual machine is created, it must be configured. This can be done with the “virsh edit” command while the virtual machine is not running. See the [virsh man page](https://linux.die.net/man/1/virsh) for more details.

The virtual machine configuration must have the emulated IOMMU presented to the guest OS in the virtual machine, and the virtual machine must have the AIC100 device passed in.

First, to present the IOMMU to the guest OS, add the following elements to the configuration XML:

  1. Configure the ioapic driver in split mode for interrupt remapping. This is done by adding “<ioapic driver='qemu'/>” under the “features” section of the XML.
  2. Configure the emulated IOMMU to be present with the interrupt remapping functionality enabled. This is done by adding the following snippet under the “features” section of the XML:
  ```
  <iommu model='intel'>
  <driver caching_mode='on' intremap='on'/>
  </iommu>
  ```

Second, configure what device to pass through to the virtual machine guest OS.

  1. Obtain the PCIe SBDF address of the AIC100 device using “lspci” in the host.
  2. Add the device address obtained from Step 1 to the “devices” section of the XML, as follows, but change the address tag values to match that of your specific system.

  ```
  <hostdev mode='subsystem' type='pci' managed='yes'>
  <source>
  <address domain='0x0' bus='0x0' slot='0x19' function='0x0'/>
  </source>
  </hostdev>
  ```
After making these edits, save the configuration. The next time the virtual machine is booted, you will observe the AIC100 device in lspci output in the virtual machine. It will have a different PCIE SBDF address than the host. Install the AIC100 Platform SDK and Apps SDK in the same manner as if you were operating a native system.

## Hyper-V
Hyper-V is a virtualization hypervisor commonly used with Microsoft Windows Server. This enables a native install of Windows Server to act as the host for a virtual machine. The virtual machine emulates a completely independent system that can be limited to a subset of the host hardware, and also can run completely different operating systems from the host.
  
An overview of Hyper-V is at [https://en.wikipedia.org/wiki/Hyper-V](https://en.wikipedia.org/wiki/Hyper-V).

The benefit of a virtual machine is strong isolation. A virtual machine is isolated from other virtual machines and also from the host. This isolation provides a high level of security because one application in a virtual machine may not be aware that it is in a virtual machine, much less that there may be other virtual machines with other applications. Also, a virtual machine provides separation from the host. Even if a driver in the virtual machine crashes the entire virtual machine, that crash will not take down the host and, therefore, will allow other services in other virtual machines to continue operating.

The cost of these benefits is additional overhead to set up the system. Also, additional processing may be required at runtime between the virtual machine and the native hardware.

AIC100 only supports PCIe passthrough to a virtual machine, which means that the virtual machine completely owns the AIC100 device. The AIC100 device cannot be shared between virtual machines, or between a virtual machine and the native host.

The generic setup and operation of a Hyper-V virtual machine is outside the scope of this document. This document assumes that the reader is familiar with those operations and, thus, only explains the elements directly related to assigning an AIC100 device to a Hyper-V virtual machine.

AIC100 supports only the operating systems listed in [this table](../index.md#operating-systems) as the operating system running within the virtual machine as the guest OS.

Within a virtual machine, AIC100 still requires the assignment of 32 message signal interrupts (MSI) to operate. Hyper-V does not support emulating an IOMMU in the guest OS of the virtual machine.

However, Hyper-V supports a paravirtualized PCIe root controller that has a driver in the Linux kernel. This driver uses the Hyper-V hypervisor to configure the host IOMMU for the purposes of supplying MSIs for devices like AIC100.

AIC100 requires the following fixes to the Hyper-V PCIe root controller driver within the Linux kernel to operate properly:

  * [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=08e61e861a0e47e5e1a3fb78406afd6b0cea6b6d](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=08e61e861a0e47e5e1a3fb78406afd6b0cea6b6d)
  * [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=455880dfe292a2bdd3b4ad6a107299fce610e64b](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=455880dfe292a2bdd3b4ad6a107299fce610e64b)
  * [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=b4b77778ecc5bfbd4e77de1b2fd5c1dd3c655f1f](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=b4b77778ecc5bfbd4e77de1b2fd5c1dd3c655f1f)
  * [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=a2bad844a67b1c7740bda63e87453baf63c3a7f7](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.19-rc7&id=a2bad844a67b1c7740bda63e87453baf63c3a7f7)

Consult the provider of your Linux distribution to confirm that those fixes are present in the Linux kernel used for your Linux distribution.
Once the Hyper-V virtual machine is created, the AIC100 device must be assigned to the virtual machine. Hyper-V calls this direct device assignment (DDA) and it is the same as PCIe passthrough.
Assign the AIC100 device to the Hyper-V virtual machine as follows:

  1. Configure the virtual machine stop action.
  2. Disable the device in Windows.
  3. Unmount the device from Windows.
      This step requires the “-force” option to be used. AIC100 currently does not have a partitioning driver. This may be provided in the future.
  4. Add the device to the virtual machine.

Details on these steps can be found at the following DDA resources:

  * [https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda)
  * [https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-storage-devices-using-dda](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-storage-devices-using-dda)
 
To identify the AIC100 device to disable and dismount, look for a device that has the following in the “Hardware Ids” property:
```
PCI\VEN_17CB&DEV_A100&SUBSYS_A10017CB&REV_00
```
Once the AIC100 device is assigned to the virtual machine, it should appear in lspci output in the virtual machine the next time the virtual machine is booted. It will have a different PCIE SBDF address than the host. Install the AIC100 software in the same manner as if you were operating a native system.


## ESXi
ESXi is the virtualization hypervisor from VMWare. Refer to ESXi documentation for instructions on how to create a virtual machine (VM).

Before powering-on the VM and installing the guest OS, a few setting changes are required to assign one or more AI 100 cards to the VM. To activate passthrough, follow these steps.

  1. Go to “Host” -> ‘Manage’ tab on the left bar. Then click on the ‘Hardware’ tab.
  2. Search for “Qualcomm”. All the installed AI 100 cards should show up.
  3. Check the cards and click “Toggle Passthrough”. Each card should then list “Active” under the Passthrough attribute.
  4. Create a VM per instructions in the ESXi documentation. Under "virtual hardware", "add other device", and select "pci device". This will add a new entry for the PCI device. Verify that the correct AI 100 card is selected here. Repeat this process for every AI 100 card that should be assigned to the VM.
  5. Setup is now complete and the VM can be powered ON. It should automatically boot the guest OS ISO and start the installer. A preview of the console is shown in the virtual machine tab when the concerned VM is selected. The preview can be clicked to be expanded and used as an interface for the VM. Walk through the OS installer like any other system.

## Xen
Xen is an open source virtualization hypervisor that is maintained by the Xen community. Please refer to the community documentation for Xen hypervisor and virtual machine setup/operation instructions.

Only HVM type virtual machines are supported with Qualcomm Cloud AI products. PV type virtual machines are not supported.

The desired Cloud AI device must be assigned to the desired VM via PCI passthrough. The Xen documentation details this process, but for reference:

Assume the Cloud AI device is at PCI address 0000:12:00.0. 
A driver (xen-pciback) to support device assignment needs to be loaded in Dom0 every time Dom0 is booted - modprobe xen-pciback
The desired PCI device needs to be configured for passthrough in Dom0 every time Dom0 is booted - xl pci-assignable-add 12:00.0
The passthrough configuration can be verified - xl pci-assignable-list
The VM config needs to be edited to assign the device to the VM - pci         = [ '0000:12:00.0,permissive=1' ]
The VM will need to be booted after the config change. The device will appear in lspci output within the VM.

## 
