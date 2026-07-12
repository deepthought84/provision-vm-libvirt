# provision-vm-libvirt
bash script for provisioning Debian Based VMs for libvirt with debootstrap

install to /usr/local/bin/

change variables to your fitting (release,vg,adminuser,network,authorized_key,packages)

default is a minimal debian installation with ssh,sudo and qemu-guest-agent

# prerequisites
you need a working libvirtd installation with an lvm pool and an dhcp-based network configured

replace vg00 with your volume group

we use refind to avoid the hassle of every vm installs it own grub - the principle is simple, it just looks for the most recent kernel image and boots it

prepare shared partition for refind bootloader:

    lvcreate vg00 -n vm_refind -L 1M
    mkfs.vfat /dev/vg00/vm_refind
    mkdir -p /mnt/vg00/vm_refind
    mount /dev/vg00/vm_refind /mnt/vg00/vm_refind

use refind.conf from this repo, obtain copy of refind_x64.efi and ext4_x64.efi (e.g. from debian package), the structure should look like this:

    ./EFI/refind/refind_x64.efi
    ./EFI/refind/drivers_x64
    ./EFI/refind/drivers_x64/ext4_x64.efi
    ./EFI/refind/refind.conf

    umount

we make sure refind is booted - so we create an OVMF_VARS template for it (install python3-virt-firmware):
    sudo virt-fw-vars  --input /usr/share/OVMF/OVMF_VARS_4M.fd --append-boot-filepath "\\EFI\\refind\\refind_x64.efi" --output /usr/share/OVMF/OVMF_VARS_4M_refind.fd

# structure 

| Host  | Guest    | Description       |
| ------ | ----- | ------- |
| /mnt/vg00/vm_refind | /dev/vda | refind boot manager (read only / shared between vms)  - loads boots vm via efistub |
| /mnt/vg00/vm_\<hostname\>_root | /dev/vdb | rootfs |
| /mnt/vg00/vm_\<hostname\>_swap | /dev/vdc | swap |


# Usage 
provision_vm <hostname> <rootsize> <swapsize> <ramsize in MiB> <vcpu_count>

e.g.: provision_vm vm01 16GiB 8GiB 4096 2
