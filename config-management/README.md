## Attempt to describe Apertis configuration architecture

Here, there is an attempt to describe Apertis config architecture. Since config
management (image building) is the key part of such huge systems.

### Image building
https://www.apertis.org/guides/image_building/

The process of getting from source code to an image suitable for loading into a
target device with Apertis is a clearly defined multi-step process. The initial
step is to build the source and package the resulting artefacts into .deb
packages. For the vast majority of packages this is already done and provided
by Apertis in the package repositories. This page will not cover this step of
the process an will assume all required packages have been successfully built
and are available in the package repositories.

There are two further steps to generating an image:

	Building an Operating System package (OSpack)
	Combining the OSpack with a Hardware package (HWpack) to create an image

Both the process of building OSpacks and images from them is carried out using Debos.
It is strongly recommended to utilise the appropriate Docker image builder container
to perform these steps, though the Apertis SDK image running in a VM can be used
with reduced reliability.

### What is Debos?
https://github.com/go-debos/debos

Debos stands for Debian OS builder.

### Building OSpacks

The OSpack is a root file system containing all the generic software for a particular
image type and hardware architecture (e.g. the arm architecture). Apertis provides
stock Debos recipes in the form of .yaml files in the apertis-image-recipes gitlab
repository, with a name of the form apertis-ospack-*.yaml, each one providing a
different balance of packages for different tasks.

It is best to build these packages images from the appropriate image-builder docker
container. This is documented in the apertis-image-recipes README.md.

### Building an image with a HWpack

Unlike the OSpack, that is generic, the HWpack contains the bits needed to boot on a
specific target. The HWpack configuration typically extracts an OSpack, creates an
image containing the required partition layout for the specific hardware, writes the
extracted OSpack into it, adds extra components necessary for the target and may
perform some tweaks to the image (tweaking configuration files etc). Finally the
image is compressed to minimise storage space.

Existing Apertis examples can be found in the Apertis Image Recipes repository called
apertis-image-*.yaml. The Debos documentation also provides a simple example.

### Image creation

Image creation is the point where a set of standard packages are combined to build a
solution for a specific use case. This goal is accomplish thanks to Debos, a flexible
tool to configure the build of Debian-based operating systems. Debos uses tools like
debootstrap already present in the environment and relies on virtualisation to securely
do privileged operations without requiring root access.

Ospacks and how they should be processed to generate images are defined through YAML
files.

This is an example configuration for an ARMv7 image, image-armhf.yaml:

	architecture: armhf
	actions:
	  - action: unpack
	    file: ospack-armhf.tar.gz
	    compression: gz

	  - action: apt
	    description: Install hardware support packages
	    recommends: false
	    packages:
	      - linux-image-4.9.0-0.bpo.2-armmp-unsigned
	      - u-boot-common

	  - action: image-partition
	    imagename: "apertis-armhf.img"
	    imagesize: 4G
	    partitiontype: gpt
	    mountpoints:
	      - mountpoint: /
	        partition: root
	        flags: [ boot ]
	    partitions:
	      - name: root
	        fs: ext4
	        start: 0%
	        end: 100%

	  - action: filesystem-deploy
	    description: Deploy the filesystem onto the image

	  - action: run
	    chroot: true
	    command: update-u-boot

	  - action: run
	    description: Create bmap file
	    postprocess: true
	    command: bmaptool create apertis-armhf.img > apertis-armhf.img.bmap

	  - action: run
	    description: Compress image file
	    postprocess: true
	    command: gzip -f apertis-armhf.img

And this is the ospack-armhf.yaml configuration for the ARMv7 ospack:

	architecture: armhf

	actions:
	  - action: debootstrap
	    suite: "17.06"
	    keyring-package: apertis-archive-keyring
	    components:
	      - target
	    mirror: https://repositories.apertis.org/apertis
	    variant: minbase

	  - action: apt
	    description: Install basic packages
	    packages: [ procps, sudo, openssh-server, adduser ]

	  - action: run
	    description: Setup user account
	    chroot: true
	    script: setup-user.sh

	  - action: run
	    description: Configure the hostname
	    chroot: true
	    command: echo apertis > /etc/hostname

	  - action: overlay
	    description: Overlay systemd-networkd configuration
	    source: networkd

	  - action: run
	    description: Configure network services
	    chroot: true
	    script: setup-networking.sh

	  - action: pack
	    compression: gz
	    file: ospack-armhf.tar.gz

Additionally at this stage customizations can be applied by using overlays. This process
allows the default content of packages to be combined with custom modifications to provide
the desired solution. A common case is to apply overlays to change some default system
settings found in /etc such as default hostname or default package configuration.

As an example of this mechanism, the following section is used to customize the behaviour
of dpkg

	  - action: overlay
	    source: overlays/dpkg-exclusions

Thanks to this action, the contents of the overlays/dpkg-exclusions directory will be
applied to the image, which in this case consist of the file:

	etc/dpkg/dpkg.cfg.d/apertis-exclusions

This file will be added to the rootfs, which in this instance will change the default
behaviour of dpkg to suit the needs of the image.

Collections of images are built every night and published on the deployable image
hosting website, such that developers can always download the latest image to deploy
it to a target device and start using it immediately.
