## Managing and Installing Virtual Machine
```bash
# virsh - tool to manage virtual machine on command line
yum install libvirt qemu-kvm -y

# creating and configuring machine
vim firstvm.xml
    <domain type="qemu">
        <name>FirstVM</name>
        <memory unit="GiB">1</memory>
        <vcpu>1</vcpu>
        <os>
            <type arch="x86_64">hvm</type>
        </os>
    </domain>

# hvm - hyper virtual machine
virsh define firstvm.xml
virsh help
virsh list --all
virsh start FirstVM
virsh list
virsh reboot FirstVM
virsh reset FirstVM
virsh shutdown FirstVM
virsh destroy FirstVM                           # just like unplugging the power
virsh list --all
virsh undefine FirstVM                          # to delete the VM but not the data
virsh undefine FirstVM --remove-all-storage     # to delete the VM with data
virsh list --all
virsh autostart FirstVM                         # to start VM automatically after base OS boot
virsh autostart --disable FirstVM
virsh dominfo FirstVm
virsh setvcpus FirstVM 2 --config --maximum
virsh setvcpus FirstVM 2 --config 
virsh dominfo FirstVM
virsh shutdown FirstVM
virsh start FirstVM
virsh dominfo FirstVM
virsh destroy FirstVM

# set maximum memory
virsh setmaxmem FirstVM 2048M --config
virsh start FirstVM
virsh destroy FirstVM
virsh dominfo FirstVM
```
