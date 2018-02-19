This is a volume plugin for Docker, allowing Docker containers
to enjoy persistent volumes residing on ploop, either locally
or on the distributed Virtuozzo Storage file system.

## Prerequisites

To use this plugin with [Virtuozzo Storage](https://virtuozzo.com/products/virtuozzo-storage/), you need to have it up and running.

Alternatively, to use this plugin with ploop, you need to have [Virtuozzo](virtuozzo.com/products/virtuozzo/) or [OpenVZ](https://openvz.org/) up and running.

Surely, you need [Docker](https://docker.com/) v1.13+ up and running.

## Installation

This guide assumes you are using a recent version of Virtuozzo, Virtuozzo Storage, or OpenVZ, and already have Docker up and running.

The recommended way of installation is from the repository. This is as easy as:

```
docker plugin install virtuozzo/ploop vstorage.source=[VSTORAGE MOUNT POINT]
```
Alternatively, you can build the plugin from source, for details, see [INSTALL.md](INSTALL.md).

## Starting

First, you need to set home path for the plugin.

```
# Set the plugin home directory
docker plugin set virtuozzo/ploop vstorage.source=/mnt/vstorage/docker
```

You can set a default size for volumes and other options:
```
docker plugin set virtuozzo/ploop args="-size 12Gb -debug"
```

Now you can start the plugin:

```docker plugin enable virtuozzo/ploop```

## Usage

Once docker and docker-volume-ploop are running, you can create a volume:

```docker volume create -d virtuozzo/ploop -o size=512G --name MyFirstVolume```

To run a container with the volume:

```docker run -it -v VOLUME:/MOUNT alpine /bin/ash```

Here ```VOLUME``` is the volume name, and ```MOUNT``` is the path under which
the volume will be available inside a container. For example:

```docker run -it -v MyFirstVolume:/mnt alpine /bin/ash```

See ```man docker volume``` for other volume operations. For example, to list existing volumes:

 ```docker volume ls```

## Troubleshooting

### Docker with Virtuozzo/OpenVZ kernel

**For Docker to work, you need to make sure conntracks are enabled on the host.** In case it's not done, docker might complain like this:

```Error starting daemon: Error initializing network controller: error obtaining controller instance: failed to create NAT chain: iptables failed: iptables --wait -t nat -N DOCKER: iptables v1.4.21: can't initialize iptables table `nat': Table does not exist (do you need to insmod?)\nPerhaps iptables or your kernel needs to be upgraded.\n (exit status 3)```

To fix, edit ```/etc/modprobe.d/parallels.conf``` (or ```/etc/modprobe.d/openvz.conf```) to look like this:

```options nf_conntrack ip_conntrack_disable_ve0=0```

In other words, the value should be set to 0. After making the change, reboot the machine.

## Miscellaneous ploop operations

The following is the quick introduction of what operations can be performed with ploop images. For more detailed information about ploop, see [openvz.org/Ploop](https://openvz.org/Ploop).

Use ```ploop``` command line tool, and refer to an image by path to ```DiskDescriptor.xml``` file. This driver creates images under ```img``` subdirectory of its home. So, to use the following commands, you need to ```cd``` to the image directory, for example:

```cd /pcs/img/MyFirstVol/```

### Snapshots

To create a snapshot:

```ploop snapshot DiskDescriptor.xml```

To list snapshots:

```ploop snapshot-list DiskDescriptor.xml```

To delete a snapshot:

```ploop snapshot-delete -u UUID DiskDescriptor.xml```

To mount a snapshot (read-only):

```ploop mount -r -u UUID -m MOUNT_POINT DiskDescriptor.xml```

### Resizing

To resize an image (can be done while it's running):

```ploop resize -s SIZE DiskDescriptor.xml```

### Checking

In case something is wrong (ploop image can't be mounted etc.), you might want to check it.

```ploop check DiskDescriptor.xml```

If you want to run fsck on an inner filesystem, you can use the following command:

```ploop mount -F DiskDescriptor.xml```

Don't forget to unmount it:

```ploop umount DiskDescriptor.xml```

## Licensing

This software is licensed under the Apache License, Version 2.0. See
[LICENSE](https://github.com/kolyshkin/docker-volume-ploop/blob/master/LICENSE)
for the full license text.

## Demo
[![asciicast](https://asciinema.org/a/121423.png)](https://asciinema.org/a/121423)
