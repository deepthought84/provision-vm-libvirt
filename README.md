# provision-vm-libvirt
bash script for provisioning Debian Based VMs for libvirt with debootstrap

install to /usr/local/bin/

change variables to your fitting (release,vg,adminuser,network,authorized_key)

# prerequisites
you need a working libvirtd installation with an lvm pool and a network with dhcp configured

replace vg00 with your volume group

prepare shared partition for refind bootloader
  lvcreate vg00 -n vm_refind -L 1M
  mkfs.vfat /dev/vg00/vm_refind
  mkdir -p /mnt/vg00/vm_refind
  mount /dev/vg00/vm_refind /mnt/vg00/vm_refind
copy EFI to vm_refind
  umount

# structure
 - /mnt/vg00/vm_refind -> /dev/vda -> refind boot manager (read only / shared between vms)  - loads boots vm via efistub
 - /mnt/vg00/vm_<hostname>_root -> /dev/vdb -> rootfs
 - /mnt/vg00/vm_<hostname>_swap -> /dev/vdc -> swapfs

# Usage 
provision_vm <hostname> <rootsize> <swapsize> <ramsize in MiB> <vcpu_count>
e.g.: provision_vm vm01 16GiB 8GiB 4096 2
