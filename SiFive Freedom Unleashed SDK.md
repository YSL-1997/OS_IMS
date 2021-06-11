## Setup for RISC-V Architecture Layer for OpenEmbedded/Yocto
https://github.com/riscv/meta-riscv

Note: this is the only way that works well! Although the build image step would cost a long time...

Before start, make sure that your host machine has an SSH key. If not, follow [this](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) to generate a new SSH key, or you may not be able to execute the following commands.

#### Install dependencies:
```
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install python3-distutils
$ sudo apt-get install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev
```

#### Create a new workspace:
```
$ mkdir riscv-yocto && cd riscv-yocto
```

#### Install repo inside workspace

The website can be found [here](https://source.android.com/setup/develop#installing-repo).

1. Ensure that you have a ```bin/``` directory in your home directory and that it's included in your path:
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH
```
2. Download the Repo Launcher and ensure that it's executable:
```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```

3. To complete the full Repo Tool installation, need to [initialize a repo client](https://source.android.com/setup/build/downloading#initializing-a-repo-client).

+ Create an empty directory to hold your working files. Give it any name you like:
```
$ mkdir WORKING_DIRECTORY
$ cd WORKING_DIRECTORY
```
+ Configure Git with your real name and email address:
```
$ git config --global user.name Your Name
$ git config --global user.email you@example.com
```
+ Run repo init to get the latest version of Repo with its most recent bug fixes. 
```
$ repo init -u https://android.googlesource.com/platform/manifest
```
+ To check out the master branch:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b master
```

A successful initialization ends with a message stating that Repo is initialized in your working directory. Your client directory now contains a .repo directory where files such as the manifest are kept.


#### Create workspace:
```
$ mkdir riscv-yocto && cd riscv-yocto
$ repo init -u git://github.com/riscv/meta-riscv  -b master -m tools/manifests/riscv-yocto.xml
$ repo sync
$ repo start work --all
```

#### Setup:
```
$ . ./riscv-yocto/meta-riscv/setup.sh
```

#### Build images:
A console-only image for the 64-bit QEMU machine:
```
$ MACHINE=qemuriscv64 bitbake core-image-full-cmdline
```

#### Run in QEMU:
Run the 64-bit machine in QEMU using the following command:
```
$ MACHINE=qemuriscv64 runqemu nographic
```

Login: root
Password: riscv

#### Proxy problem
There might be an error saying that: Fatal Error : Can't resolve host github.com.

Possible Solution: Restart your network-manager using 
```
sudo service network-manager restart
```


## SiFive Freedom Unleashed SDK
The steps are similar to the above. For more details, please refer to https://github.com/sifive/freedom-u-sdk.

```
$ mkdir riscv-sifive && cd riscv-sifive
# install repo inside this directory
$ repo init -u git://github.com/sifive/meta-sifive -b 2021.04 -m tools/manifests/sifive.xml
$ repo sync
$ repo start work --all

# Get build tools
$ ./openembedded-core/scripts/install-buildtools -r yocto-3.2_M2 -t 20200729
$ . ./openembedded-core/buildtools/environment-setup-x86_64-pokysdk-linux

# Setup build environment
$ . ./meta-sifive/setup.sh

# Build disk image (basic command line image) running on qemuriscv64 (RISC-V 64-bit (RV64GC) for QEMU virt machine), with number of threads configured to 4
$ PARALLEL_MAKE="-j 4" BB_NUMBER_THREADS=4 MACHINE=qemuriscv64 bitbake demo-coreip-cli

# Running in QEMU
$ MACHINE=qemuriscv64 runqemu qemuparams="-m 2048" nographic slirp

# Set date and time in emulated sifive
$ date -s "11 JUN 2021 11:14:00"
```


## Another Option: Freedom Studio
Here are the links to how to use it:
+ https://www.youtube.com/watch?v=KBvAKHsHBW4


## After setting up GPG key
```
git config --global user.signingkey <YOUR_SIGNING_KEY>
git config --global commit.gpgsign true
git config --global gpg.program gpg
```
Credit to https://stackoverflow.com/questions/39494631/gpg-failed-to-sign-the-data-fatal-failed-to-write-commit-object-git-2-10-0#47087248



Below are the roundabout courses I took while setting up the environment. No need to read them, but if you are interested in installing *Boost* and *Monotone*.


## Roundabout Courses when building SiFive Freedom Unleashed SDK
The GitHub repo that describes the steps is [this](https://github.com/sifive/freedom-u-sdk).

Before we start, make sure to update and upgrade ```apt-get``` by:
```
$ sudo apt-get update && sudo apt-get upgrade
```
Then install curl:
```
$ sudo apt-get install curl
```

### Step 1: install repo
The website can be found [here](https://source.android.com/setup/develop#installing-repo).

1. Ensure that you have a ```bin/``` directory in your home directory and that it's included in your path:
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH
```
2. Download the Repo Launcher and ensure that it's executable:
```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```

3. To complete the full Repo Tool installation, need to [initialize a repo client](https://source.android.com/setup/build/downloading#initializing-a-repo-client).

+ Create an empty directory to hold your working files. Give it any name you like:
```
$ mkdir WORKING_DIRECTORY
$ cd WORKING_DIRECTORY
```
+ Configure Git with your real name and email address:
```
$ git config --global user.name Your Name
$ git config --global user.email you@example.com
```
+ Run repo init to get the latest version of Repo with its most recent bug fixes. 
```
$ repo init -u https://android.googlesource.com/platform/manifest
```
+ To check out the master branch:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b master
```

A successful initialization ends with a message stating that Repo is initialized in your working directory. Your client directory now contains a .repo directory where files such as the manifest are kept.

### Step 2: install a number of packages for BitBake (OE build tool)

### Preparation:
Note that BitBake depends on Python 3.

For Ubuntu 18.04 (or newer) install python3-distutils package.
```
$ sudo apt-get install python3-distutils
```

Essentials: Packages needed to build an image on a headless system:
```
$ sudo apt-get install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev
```


#### Other Roundabout Courses

Below are the useless/needless things I did while setting up the environment.

Next, we need to install dependency - Monotone, but before that, we need to install Boost, as Monotone requires that Boost is installed. Detailed description could be found [here](https://wiki.gumstix.com/index.php/Bitbake_on_Ubuntu). 

Execute the following commands to install dependencies for Monotone:
```
$ sudo apt-get install autoconf
$ sudo apt-get install automake
$ sudo apt-get install gettext
$ sudo apt-get install libboost-dev
$ sudo apt-get install libz-dev
$ sudo apt-get install g++
```

Create a directory called slug, then cd into it.

#### Boost

Now, we could download Boost from [here](https://www.boost.org/users/download/).

I used 1_76_0 version, and there is no ./configure file; instead, there is a shell script called bootstrap.sh - execute that script.

```
$ ./bootstrap.sh
```

Then, it would show: Bootstrapping is done. To build, run: ```./b2```. To generate header files, run: ```./b2 headers```. We execute the following command:
```
$ ./b2
# note that this command will cost a really long time (at least on my virtual machine).
```

After waiting for a long time, it would show the following:

```
The Boost C++ Libraries were successfully built!

The following directory should be added to compiler include paths:
	/home/yusen/Desktop/slug/boost_1_76_0

The following directory should be added to linker library paths:
	/home/yusen/Desktop/slug/boost_1_76_0/stage/lib
```

In order to do those, need to execute the following:
```
$ sudo ./b2 install
$ export CPPFLAGS="-I/home/yusen/Desktop/slug/boost_1_76_0"
$ export LIBS="-L/home/yusen/Desktop/slug/boost_1_76_0/stage/lib"

```
To check if Boost has been successfully installed, execute the following:
```
$ grep BOOST_VERSION /usr/include/boost/version.hpp
```
The output is:
```
#ifndef BOOST_VERSION_HPP
#define BOOST_VERSION_HPP
//  BOOST_VERSION % 100 is the patch level
//  BOOST_VERSION / 100 % 1000 is the minor version
//  BOOST_VERSION / 100000 is the major version
#define BOOST_VERSION 107100
//  BOOST_LIB_VERSION must be defined to be the same as BOOST_VERSION
```

Another way to verify the version is by writing a cpp program:
```
#include <iostream>
#include <boost/version.hpp>

int main() {
	std::cout << BOOST_VERSION << std::endl;
	std::cout << BOOST_LIB_VERSION << '\n';
}
```

However, this way, the output is 1.76.0. Not sure why it does not match with the previous method.

After successfully installed Boost, we could start working on Monotone.

#### Monotone

Before we start, need to install ```Psyco```; before which, we need to first install ```svn```:
```
$ sudo apt-get update
$ sudo apt-get install subversion
```

After that, we could follow the instructions [here](http://psyco.sourceforge.net/psycoguide/sources.html) to install Psyco.

svn co https://codespeak.net/svn/psyco/dist psyco-dist

Before we start, need to install Psychopy:
```
$ pip install psychopy
```


#### Install Bitbake
1. Clone the BitBake repository:
```
$ git clone git://git.openembedded.org/bitbake
```
2. Inside the repository, there is a file named setup.py, need to use Python2 to run such file:
```
$ python setup.py build
# if by default, your computer uses Python3, then execute: set PY_PYTHON=2, and re-try.
```
3. Install dependencies: ```xmllint```, ```xsltproc```
```
$ sudo apt install libxml2-utils
$ sudo apt install xsltproc
```

4. Re-try step 2, and install (in sudo mode if permission denied):
```
$ python setup.py build
$ python setup.py install
```
5. When we type ```$ bitbake```, there might be an error: ```ImportError: No module named ply```. The [solution](https://github.com/asl2/PyZ3950/issues/4#issuecomment-432916987-permalink) is:
```
$ sudo apt-get install python-ply     # for python2.7.x
```
6. Another possible error is: ```ImportError: No module named pysh```. This corresponds to the python shell mode. The solution is to switch your default python version to 3.x
```
$ sudo apt-get install gcc gfortran libfftw3-dev libblas-dev liblapack-dev
$ ls /usr/bin/python*      # Check all the installed Python versions in the bin directory
$ sudo update-alternatives --list python    # Check for any Python alternatives configured on the system
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1    # Configure Python Alternatives
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 2    # Configure Python Alternatives
$ sudo update-alternatives --config python    # Confirm the Python Alternatives set
Then type 2, and press enter. (to change to python 3)
$ python --version    # to check the version of python
```

7. Then, there might be another error: ```ModuleNotFoundError: No module named 'bb'```
This means bitbake is installed/used in the wrong way, together with OE, according to [this](https://stackoverflow.com/questions/36395831/importerror-no-module-named-bb)


#### Download meta-sifive
```
$ git clone https://github.com/sifive/meta-sifive.git
# if Errors - "Problem with the SSL CA cert (path? access rights?)" occur, execute in sudo mode

$ cd meta-sifive
```

Then, install repo in meta-sifive directory


#### OpenEmbedded-Core Standalone Setup
The steps are [here](https://www.openembedded.org/wiki/OE-Core_Standalone_Setup), and [here](https://stackoverflow.com/questions/36395831/importerror-no-module-named-bb).
1. Clone the repositories for OE-Core (the core metadata) and BitBake (the build tool), checking out the latest stable branches of each one in turn:
```
$ git clone git://git.openembedded.org/openembedded-core
$ cd openembedded-core
$ git clone git://git.openembedded.org/bitbake bitbake
```

2. Check out the latest stable branches of both ![OpenEmbedded-Core](https://wiki.yoctoproject.org/wiki/Releases) and [BitBake](http://git.openembedded.org/bitbake/):
```
# inside openembedded-core directory
$ git checkout master
$ cd bitbake
$ git checkout master
```
Note: must checkout the master branch, or there would be failure!!! Don't ask me why I know this XD.



