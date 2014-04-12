# Running etcd on CoreOS Vagrant

This repo provides a template Vagrantfile to create a CoreOS virtual machine running etcd using the Virtualbox software hypervisor.
After setup is complete you will have a single CoreOS virtual machine running on your local machine.

This project is forked from [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant) and modified to run etcd.

## Streamlined setup

1) Install dependencies

* [Virtualbox][virtualbox] 4.0 or greater.
* [Vagrant][vagrant] 1.3.1 or greater.

2) Clone this project and get it running!

```
git clone https://github.com/hnakamur/coreos-etcd-vagrant/
cd coreos-etcd-vagrant
```

3) Startup and SSH

There are two "providers" for Virtualbox with slightly different instructions.
Follow one of the folowing two options:

**Virtualbox Provider**

The Virtualbox provider is the default Vagrant provider. Use this if you are unsure.

```
vagrant up
vagrant ssh
```

**VMWare Provider**

The VMWare provider is a commercial addon from Hashicorp that offers better stability and speed.
If you use this provider follow these instructions.

```
vagrant up --provider vmware_fusion
vagrant ssh
```

``vagrant up`` triggers vagrant to download the CoreOS image (if necessary) and (re)launch the instance

``vagrant ssh`` connects you to the virtual machine.
Configuration is stored in the directory so you can always return to this machine by executing vagrant ssh from the directory where the Vagrantfile was located.

3) Get started [using CoreOS][using-coreos]

[virtualbox]: https://www.virtualbox.org/
[vagrant]: http://downloads.vagrantup.com/
[using-coreos]: http://coreos.com/docs/using-coreos/

#### Provisioning with user-data

The Vagrantfile will provision your CoreOS VM(s) with [coreos-cloudinit][coreos-cloudinit] using a `user-data` file found in the project directory.
coreos-cloudinit simplifies the provisioning process through the use of a script or cloud-config document.

[coreos-cloudinit]: https://github.com/coreos/coreos-cloudinit

## Cluster Setup

After this VM started, verify etcd is running on the core-00 VM.
(This is optional. You are free to go to the next step)

```
$ vagrant ssh
   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
core@core-00 ~ $ ps auxww | grep etcd
etcd      3066  0.2  0.4 192832  4872 ?        Ssl  02:10   0:00 /usr/bin/etcd
core      3083  0.0  0.0   4380   648 pts/0    S+   02:10   0:00 grep --colour=auto etcd
```

Then run another VMs using [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant).

Run commands below at another shell on the host machine.

```
git clone https://github.com/coreos/coreos-vagrant
cd coreos-vagrant
cat <<'EOF' > user-data
#cloud-config

coreos:
  etcd:
      discovery: http://172.17.8.100:4001/v2/keys/machines
      addr: $public_ipv4:4001
      peer-addr: $public_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
      runtime: no
      content: |
        [Unit]
        Description=fleet

        [Service]
        Environment=FLEET_PUBLIC_IP=$public_ipv4
        ExecStart=/usr/bin/fleet
EOF
NUM_INSTANCES=3 vagrant up
```

After core-01, core-02, core-03 VMs started, run commands below.

```
$ vagrant ssh core-01
   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
core@core-01 ~ $ fleetctl list-machines -l
MACHINE					IP		METADATA
b4e79fb3-a093-421e-a06d-7ae2f63cf7a8	172.17.8.103	-
681af189-b016-4d25-bd1b-fba970db540a	172.17.8.102	-
e436a060-0859-46c6-8ade-a5fc2ac508e5	172.17.8.101	-
```

# [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant) version used here.

```
$ git log -1
commit 46c17fc92fd4ed9c911cdbab93ad87326f73d0f1
Merge: 858dd74 4b52c3d
Author: Brian Waldon <bcwaldon@gmail.com>
Date:   Thu Apr 3 11:31:05 2014

    Merge pull request #67 from dpintats/master

    refactor(Vagrantfile): NUM_INSTANCES variable would return 0 if not set
```

