# Custom pfSense kernel build with bbr, epair, vxlan and fusefs support.
I mostly follow [the guide of Augustin-FL](https://github.com/Augustin-FL/building-pfsense-iso-from-source), except I only tend to build a working kernel with bbr support instead of the whole ISO.

Currently only tested in pfSense CE 2.7.2

```
BBR : Google TCP BBR congestion control.
epair and vxlan : FreeBSD jail vnet support.
fusefs : For the ability to mount external hard drive.
```

<details open>
<summary>

## Fork pfSense repo and make changes

</summary>

### Fork pfSense repo
You will then have to fork 3 repositories: 
- [pfSense GUI](https://github.com/pfsense/pfsense) 
- [FreeBSD Ports](https://github.com/pfsense/freebsd-ports) 
- [FreeBSD Kernel Source](https://github.com/pfsense/freebsd-src) 

### Make changes
Choose a custom name for your firewall (because of trademark). I'll use libreSense for this tutorial. 

You will also need to apply the following changes: 

#### FreeBSD Source
- Checkout branch `RELENG_2_7_2`.
- In the folder `/release/conf/`, rename files starting with `pfSense` to `libreSense`

| Original Name    | New Name |
| -------- | ------- |
| pfSense_build_src.conf | libreSense_src.conf |
| pfSense_install_src.conf | libreSense_install_src.conf |
| pfSense_installer_make.conf | libreSense_installer_make.conf |
| pfSense_installer_src.conf | libreSense_installer_src.conf |
| pfSense_make.conf | libreSense_make.conf |
| pfSense_src-env.conf | libreSense_src-env.conf |

- Rename the file `/sys/amd64/conf/pfSense` to `/sys/amd64/conf/libreSense`
- Add the following options to file `sys/amd64/conf/DEFAULTS`
```
options         FUSEFS
options         VIMAGE
makeoptions     WITH_EXTRA_TCP_STACKS=1
options         TCPHPTS
options         RATELIMIT
```

#### pfSense GUI
- Go to the folder `/tools/templates/pkg_repos/` and rename the file `pfSense-repo.conf` to `libreSense-repo.conf`
- Edit the file `/src/etc/inc/globals.inc` : replace the content of `product_name` by `libreSense`, and the content of `pkg_prefix` by `libreSense-pkg-`.
- In the folder `/src/usr/local/share/`, rename the folder `pfSense` to `libreSense`.
- In the folder `/src/etc/`, rename the files `pfSense-ddb.conf` and `pfSense-devd.conf` to `libreSense-ddb.conf` and `libreSense-devd.conf`.
- Edit the file `/tools/builder_defaults.sh`:
  - Remove`drm2` and `ndis` from the variable `MODULES_OVERRIDE_amd64`.
  - Add `if_epair if_vxlan tcp/bbr tcp/rack accf_http accf_data accf_dns if_wg linuxkpi_hdmi pchtherm fusefs` to variable `MODULES_OVERRIDE_base`.
  - Change variable `PKG_REPO_BRANCH_RELEASE` to `v2_7_2`.
  - Change variable `PFSENSE_DEFAULT_REPO` to `"${PRODUCT_NAME}-repo"`.

</details>

<details open>
<summary>

## Prepare FreeBSD build machine

</summary>

### Install FreeBSD
Since we only need to build kernel, high amount of disk, CPU or memory isn't needed. Insatll Freebsd on ZFS is optional but recommended.  
For easy build, rent a [VPS from hetzner](https://hetzner.cloud/?ref=aY3GsPMrFDw9), choose CX22, this will give us 2 CPU + 4 G Ram + 40 SSD, which is more than enough for building a kernel.   

![hetzner vps](https://github.com/mikehu404/pfsense-bbr/blob/main/Image/VPS.png?raw=true)

![Freebsd ISO](https://github.com/mikehu404/pfsense-bbr/blob/main/Image/ISO.png?raw=true)

Plus their VPS service is hourly billed, and our entire build time should be under 1 hour. So this won't cost us any money.  

### Configurate the build server
Login as root and execute following commands:

```
# Make sure system is up to date
pkg update -f && pkg upgrade && pkg autoremove

# Allow SSH using root, if you want it.
echo PermitRootLogin yes >> /etc/ssh/sshd_config
service sshd restart

# Required for configuring the server
pkg install -y pkg vim nano emacs

# Required for building kernel
pkg install -y git-lite curl xmlstarlet pkgconf openssl

# not required but advised for building/monitoring/debugging
pkg install -y htop screen wget mmv 

# Only install this if your FreeBSD is a virtual machine, eg. VPS
pkg install -y open-vm-tools
```

### Configure how pfSense will be built
Clone your fork of pfSense GUI, checkout to the branch that will be built.

```
cd /root
git clone https://github.com/{your username}/pfsense.git
cd pfsense
git checkout RELENG_2_7_2 # Replace by the branch of pfSense GUI to build.
```

Create a file called build.conf in the folder of pfSense GUI.

```
# uncomment this for debugging  
#set -x  

export PRODUCT_NAME="libreSense" # Replace with your product name
export FREEBSD_REPO_BASE=https://github.com/{your username}/FreeBSD-src.git # Location of your FreeBSD sources repository
export POUDRIERE_PORTS_GIT_URL=https://github.com/{your username}/FreeBSD-ports.git # Location your FreeBSD ports repository

export FREEBSD_BRANCH=RELENG_2_7_2 # Branch of FreeBSD sources to build

# The branch of FreeBSD ports to build is set automatically based on pfSense GUI branch.
# If you would like to build a specific branch of FreeBSD ports, the variable to set is POUDRIERE_PORTS_GIT_BRANCH
# Also, if you are building a FreeBSD port branch that does not respect Netgate conventions (devel or RELENG_*),
# You will also have to update .conf files in "tools/templates/pkg_repos/" because repositories names are partially
# hardcoded there.

# Netgate support creation of staging builds (pre-dev, nonpublic version)
unset USE_PKG_REPO_STAGING # This disable staging build

# The kind of ISO that will be built (stable or development) is defined in src/etc/version in pfSense GUI repo
export DEFAULT_ARCH_LIST="amd64.amd64" # We only want to build an x64 ISO, we don't care of ARM versions
```

</details>

<details open>
<summary>

## Build & install the custom kernel

</summary>

First, start a screen on your build server, using command `screen -S build`. You may leave this screen using ctrl+A then D, and you may enter this screen again using command `screen -r build`. The purpose of the screen is to keep your work running if you disconnect from the build server.

### Build the kernel
```
# Enter the folder of pfSense GUI 
cd /root/pfsense

# Clean up the build
./build.sh --clean-builder

# Build the kernel
./build.sh --build-kernels
```
You can find the kernel in `./tmp/libreSense_v2_7_2_amd64-core/` after build complete.

### (Optional) Convert PKG to tar
```
# move the pkg to temporary work space
mv ./tmp/libreSense_v2_7_2_amd64-core/.*/All/libreSense-kernel-libreSense-2.7.2.pkg /tmp/

# extract the pkg
cd /tmp/ && tar -xvf libreSense-kernel-libreSense-2.7.2.pkg

# extract the kernel
gzip -d boot/kernel/kernel.gz

# repack the pkg to tar
cd boot && tar -cpvf pfSense-bbr-vnet-fuse-kernel-2.7.2.tar kernel/
```

### Install the kernel
[Enable SSH](https://docs.netgate.com/pfsense/en/latest/recipes/ssh-access.html) on pfSense box
```
# Copy the kernel to pfSense box
scp /path/to/your/package admin@pfsense_box_IP:/tmp/   

# SSH to pfSense box
ssh admin@pfsense_box_IP

# Direct install via PKG
pkg add -f /tmp/libreSense-kernel-libreSense-2.7.2.pkg

# Or Install the kernel via tar
mv /boot/kernel /boot/kernel.default
cd /tmp && tar -xvf pfSense-bbr-vnet-fuse-kernel-2.7.2.tar && mv kernel /boot/kernel

# Load bbr on boot
sysrc kld_list+="tcp_rack tcp_bbr"
```

[Reboot the pfSense box](https://docs.netgate.com/pfsense/en/latest/diagnostics/system-reboot.html)

</details>

<details open>
<summary>

## Enable BBR in pfSense

</summary>

### Load BBR
Add the following values to [System Tunables](https://docs.netgate.com/pfsense/en/latest/config/advanced-tunables.html)
```
# set to at least 16MB for 10GE hosts
kern.ipc.maxsockbuf=16777216

# set autotuning maximum to at least 16MB too
net.inet.tcp.sendbuf_max=16777216
net.inet.tcp.recvbuf_max=16777216

# enable send/recv autotuning
net.inet.tcp.sendbuf_auto=1
net.inet.tcp.recvbuf_auto=1

# increase autotuning step size
net.inet.tcp.sendbuf_inc=16384
net.inet.tcp.recvbuf_inc=524288

# set this on test/measurement hosts
#net.inet.tcp.hostcache.expire=1

# Set bbr
net.inet.tcp.functions_default=bbr
net.inet.tcp.functions_inherit_listen_socket_stack=0
```

After reboot verify BBR is loaded
```
sysctl net.inet.tcp.cc.algorithm
sysctl net.inet.tcp.functions_available
sockstat -SPtcp
```

</details>

<details open>
<summary>

## Speedtest benchmark
Install `py311-speedtest-cli` on pfSense box  
Here is my result:
| Configuation | Download | Upload |
| -------- | -------- | ------- |
| Default | 1294.78 Mbit/s | 975.52 Mbit/s |
| Default + htcp | 1285.62 Mbit/s | 1041.47 Mbit/s |
| BBR | 1317.20 Mbit/s | 1301.95 Mbit/s |

</summary>
</details>

<details open>
<summary>

## Difference between official kernel and custom kernel

</summary>
</details>

<details open>
<summary>

## Buy me a coffee

</summary>
</details>
