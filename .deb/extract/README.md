## Extracting a .deb file without opening it on Debian or Ubuntu Linux
https://www.cyberciti.biz/faq/how-to-extract-a-deb-file-without-opening-it-on-debian-or-ubuntu-linux/

Downloaded a .deb Debian file. How do I extract deb package without installing it on
my Debian or Ubuntu Linux based system? How do I list and extract the contents of a
Debian package?

### A Debian or Ubuntu .deb package is nothing but old good Unix ar archive format

The ar command is used to keep together groups of files as a single archive and .deb
includes the following three files:

	- debian-binary – A text file indicating the version of the .deb package format.
	- control.tar.gz – A compressed file and it contains md5sums and control directory for building package.
	- data.tar.xz – A compressed file and it contains all the files to be installed on your system.

Let see how to list and extract the contents of a .deb package file on Debian Linux
using various command line options.

### Step 1 – Download .deb package

Use the apt-get command/apt command as follows to download a file named nginx*.deb:

	$ apt download nginx
	OR
	$ aptitude download nginx
	OR
	$ apt-get download nginx

Sample outputs:

	Get:1 http://in.archive.ubuntu.com/ubuntu xenial-updates/main amd64 nginx all 1.10.0-0ubuntu0.16.04.4 [3,498 B]
	Fetched 3,498 B in 0s (4,460 B/s)

To list file use the ls command:

	$ ls *.deb
		nginx_1.10.0-0ubuntu0.16.04.4_all.deb

### Step 2 – Extract .deb package using ar command

The syntax is:

	$ ar x ${file.deb}

#### Install ar command

	You can install ar command using the following apt-get command/apt command:

	$ sudo apt install binutils
	OR
	$ sudo apt-get install binutils

Sample outputs:

	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	Suggested packages:
	  binutils-doc
	The following NEW packages will be installed:
	  binutils
	0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
	Need to get 2,310 kB of archives.
	After this operation, 13.6 MB of additional disk space will be used.
	Get:1 http://in.archive.ubuntu.com/ubuntu xenial-updates/main amd64 binutils amd64 2.26.1-1ubuntu1~16.04.3 [2,310 kB]
	Fetched 2,310 kB in 6s (343 kB/s)                                                                                                                                           
	Selecting previously unselected package binutils.
	(Reading database ... 92869 files and directories currently installed.)
	Preparing to unpack .../binutils_2.26.1-1ubuntu1~16.04.3_amd64.deb ...
	Unpacking binutils (2.26.1-1ubuntu1~16.04.3) ...
	Processing triggers for libc-bin (2.23-0ubuntu5) ...
	Processing triggers for man-db (2.7.5-1) ...
	Setting up binutils (2.26.1-1ubuntu1~16.04.3) ...
	Processing triggers for libc-bin (2.23-0ubuntu5) ...

To extract nginx_1.10.0-0ubuntu0.16.04.4_all.deb, run:

	$ ar vx nginx_1.10.0-0ubuntu0.16.04.4_all.deb
	$ ls -al

Sample outputs:

Extract files from control.tar.gz and data.tar.gz

Type the following tar command:

	$ tar xvf control.tar.gz
	$ tar data.tar.gz
	$ ls -al

All files are extracted into the current directory.

### dpkg-deb command

the dpkg-deb command COULD BE USED to extract .deb file too. The syntax is:

	$ dpkg-deb -xv {file.deb} {/path/to/where/extract}

To extract htop_2.0.1-1ubuntu1_amd64.deb in the /tmp/ directory run:

	$ dpkg-deb -xv htop_2.0.1-1ubuntu1_amd64.deb /tmp/

To extract htop_2.0.1-1ubuntu1_amd64.deb in the current directory run:

	$ dpkg-deb -xv htop_2.0.1-1ubuntu1_amd64.deb .

The syntax is:

	$ dpkg -c {file.deb}
	OR
	$ apt-file list {packageName}

For example to view contents of a Debian package named htop_2.0.1-1ubuntu1_amd64.deb, run:

	$ dpkg -c htop_2.0.1-1ubuntu1_amd64.deb

Sample outputs:

The apt-file is not installed by default. So install it and use it as follows:

	$ sudo apt-get install apt-file ## install ##
	$ sudo apt-file update ## update package cache ##
	$ apt-file list htop ## list htop package contents ##
