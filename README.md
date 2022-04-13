# How to install Rhel8 on linux running on old macbook pro (Mid 2010 6,2)

I spend two days about running Rhel 8 on virtual box, but it always ended up with errors which i could not find on internet, so i decided to try KVM,
here is my step by step guide.

## Install QEMU & KVM
`$ sudo apt install qemu-system libvirt-clients libvirt-daemon-system`

## Install QEMU & KVM without graphical packages
`$ sudo apt install --no-install-recommends qemu-system libvirt-clients libvirt-daemon-system`

## Add your user to the libvirt group (Probably added)
`$ adduser <YourUser> libvirt`

## Reload updated group membership, if any password is asked, enter your login password.
`$ exec su -l $USER`

## run virsh as a regular user. It will show a list of available VMs (currently none). If you do not encounter a permission error, it means everything is okay so far.
`$ virsh list`

## Configure Bridged Networking
`$ sudo apt-get install bridge-utils`

## Check if there is available bridged nic
`$ brctl show`

## If there is any available birdged nic and if you want to use one of them ex: virbr0
`$ sudo brctl stp virbr0 off`

`$ sudo brctl setfd virbr0 0`

## If you want to create new Bridged Network- For this step please follow online documents below is my guess
`$ sudo brctl addbr br0`

`$ sudo brctl stp br0 off`

`$ sudo brctl setfd br0 0`

## May not be necessary
`$ sudo brctl addif <ActiveInterfaceName>`

## Take ip address of Bridged nic (we will use it to connect kvm via VNC)
`$ ifconfig`

## Xml for creating new Virtual device
===================================================================
- 4GB memory, 2 vCPU and one hard drive.
- Disk image: /home/`<YourUserName>`/images/YourDisk.img. Change `<YourUserName>` with you User Name
- Boot from CD-ROM (/home/`<YourUserName>`/iso/CentOS-6.5-x86_64-minimal.iso). Change `<YourUserName>` with you User Name. iso/ is where you keep .ISO image
- Networking: one network interface bridged to virbr0 (in my case)
- Remote access via VNC.
- Give it a name by changing AlmaLinux in between name tags `<name>AlmaLinux</name>`
- Save the xml (AlmaLinux.xml in my case)
-You can change string between `<uuid></uuid>` tags by
  
  `$ sudo apt-get install uuid`
  
  `$ uuid`
  
==================================================================
```xml
<domain type='kvm'>
  <name>AlmaLinux</name>
  <uuid>a6ee1e58-b8d4-11ec-b06d-0b2b5e77716c</uuid>
  <memory>4194304</memory>
  <currentMemory>4194304</currentMemory>
  <vcpu>2</vcpu>
  <os>
    <type>hvm</type>
    <boot dev='cdrom'/>
  </os>
  <features>
    <acpi/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <devices>
    <emulator>/usr/bin/kvm</emulator>
    <disk type="file" device="disk">
      <driver name="qemu" type="raw"/>
      <source file="/home/<YourUserName>/images/YourDisk.img"/>
      <target dev="vda" bus="virtio"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x04" function="0x0"/>
    </disk>
    <disk type="file" device="cdrom">
      <driver name="qemu" type="raw"/>
      <source file="/home/<YourUserName>/iso/AlmaLinux-8.5-x86_64-dvd.iso"/>
      <target dev="hdc" bus="ide"/>
      <readonly/>
      <address type="drive" controller="0" bus="1" target="0" unit="0"/>
    </disk>
    <interface type='bridge'>
      <source bridge='virbr0'/>
      <mac address="00:00:A3:B0:56:10"/>
    </interface>
    <controller type="ide" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x1"/>
    </controller>
    <input type='mouse' bus='ps2'/>
    <graphics type='vnc' port='-1' autoport="yes" listen='0.0.0.0'/>
    <console type='pty'>
      <target port='0'/>
    </console>
  </devices>
</domain>
```
## Create 100 Gb Disk increases automatically does not occupy full size
`$ sudo qemu-img create -f raw /home/<YourUserName>/images/YourDisk.img 100G`

## Create and Start Vm
`$ virsh create AlmaLinux.xml`

## Check if Vm is started
`$ virsh list`

## if not started
`$ virsh start AlmaLinux.xml`

## Access KVM
With any VNC client connect to KVM
`<KVM_HOST_IP:5900>`
`192.168.122.1:5900` in my case

## Note: 
After installation if Vm keeps booting from ISO, edit xml file to comment ISO disk part, by doing this ISO image wont be mounted on next boot.

## But you can use Virtual Machine Manager too

`$ yum install virt-manager (Fedora)`

`$ apt-get install virt-manager (Debian)`

`$ emerge virt-manager (Gentoo)`

`$ pkg_add virt-manager (OpenBSD)`

  
### Sources:
- https://acikkaynakfikirler.com/sanallastirma-yaparken-vmwareye-mecbur-degilsiniz/  MUTLAKA OKUYUN
- https://www.fosslinux.com/48755/top-opensource-virtualization-software-for-linux.htm
- https://www.xmodulo.com/use-kvm-command-line-debian-ubuntu.html
- https://qemu-project.gitlab.io/qemu/system/images.html
- https://www.tecmint.com/create-network-bridge-in-ubuntu/
- https://tldp.org/HOWTO/BRIDGE-STP-HOWTO/set-up-the-bridge.html
