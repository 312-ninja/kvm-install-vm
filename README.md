## kvm-install-vm

A bash wrapper around virt-install to build virtual machines on a local KVM
hypervisor.  You can run it as a normal user which will use `qemu:///session` to
connect locally to your KVM domains.

Tested on Fedora 25/26.

### Prerequisites

You need to have the KVM hypervisor installed, along with a few other packages:

- genisoimage
- virt-install
- libguestfs-tools-c
- qemu-img
- libvirt-client

To install the dependencies, run:

```
sudo dnf -y install genisoimage virt-install libguestfs-tools-c qemu-img libvirt-client wget
```

If you want to resolve guests by their hostnames, install the `libvirt-nss` package:

```
sudo dnf -y install libvirt-nss
```

Then, add `libvirt` and `libvirt_guest` to list of **hosts** databases in
`/etc/nsswitch.conf`.  See [here](https://libvirt.org/nss.html) for more
information.

### Usage

```
$ kvm-install-vm help
NAME
    kvm-install-vm - Install virtual guests using cloud-init on a local KVM
    hypervisor.

SYNOPSIS
    kvm-install-vm COMMAND [OPTIONS]

DESCRIPTION
    A bash wrapper around virt-install to build virtual machines on a local KVM
    hypervisor. You can run it as a normal user which will use qemu:///session
    to connect locally to your KVM domains.

COMMANDS
    help    - show this help or help for a subcommand
    create  - create a new guest domain
    list    - list all domains, running and stopped
    remove  - delete a guest domain
```

```
$ kvm-install-vm help create
NAME
    kvm-install-vm create [OPTIONS] VMNAME

DESCRIPTION
    Create a new guest domain.

OPTIONS
    -b          Bridge              (default: virbr0)
    -c          Number of vCPUs     (default: 1)
    -d          Disk Size (GB)      (default: 10)
    -f          CPU Model / Feature (default: host)
    -h          Display help
    -i          Custom QCOW2 Image
    -k          SSH Public Key      (default: /home/torresgi/.ssh/id_rsa.pub)
    -l          Location of Images  (default: /home/torresgi/virt/images)
    -m          Memory Size (MB)    (default: 1024)
    -M mac      Mac address         (default: auto-assigned)
    -t          Linux Distribution  (default: centos7)
    -T          Timezone            (default: US/Eastern)

DISTRIBUTIONS
    NAME            DESCRIPTION                         LOGIN
    centos7         CentOS 7                            centos
    centos7-atomic  CentOS 7 Atomic Host                centos
    centos6         CentOS 6                            centos
    debian9         Debian 9 (Stretch)                  debian
    fedora26        Fedora 26                           fedora
    fedora26-atomic Fedora 26 Atomic Host               fedora
    ubuntu1604      Ubuntu 16.04 LTS (Xenial Xerus)     ubuntu

EXAMPLES
    kvm-install-vm create foo
        Create VM with the default parameters: CentOS 7, 1 vCPU, 1GB RAM, 10GB
        disk capacity.

    kvm-install-vm create -c 2 -m 2048 -d 20 foo
        Create VM with custom parameters: 2 vCPUs, 2GB RAM, and 20GB disk
        capacity.

    kvm-install-vm create -t debian9 foo
        Create a Debian 9 VM with the default parameters.

    kvm-install-vm create -T UTC foo
        Create a default VM with UTC timezone.
```

### Notes

1. This script will download a qcow2 cloud image from the respective
   distribution's download site.  See script for URLs.

2. If using libvirt-nss, keep in mind that DHCP leases take some time to
   expire, so if you create a VM, delete it, and recreate another VM with the
   same name in a short period of time, there will be two DHCP leases for the
   same host and its hostname will likely not resolve until the old lease
   expires.

### Testing

Tests are written using [Bats](https://github.com/sstephenson/bats).  To
execute the tests, run `./test.sh` in the root directory of the project.

### Use Cases

If you don't need to use Docker or Vagrant, don't want to make changes to a
production machine, or just want to spin up one or more VMs locally to test
things like:

- high availability
- clustering
- package installs
- preparing for exams
- checking for system defaults
- anything else you would do with a VM

...then this wrapper could be useful for you.
