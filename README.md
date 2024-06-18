# Custom pfSense kernel build with bbr, epair, vxlan and fusefs support.
I mostly follow [the guide of Augustin-FL](https://github.com/Augustin-FL/building-pfsense-iso-from-source), except I only tend to build a working kernel with bbr support instead of the whole ISO.
Currently only tested in pfSense CE 2.7.2

<details>
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

#### pfSense GUI


</details>

<details>
<summary>

## Prepare FreeBSD build machine

</summary>

</details>

<details>
<summary>

## Build & install the custom kernel

</summary>
</details>

<details>
<summary>

## Enable BBR in pfSense

</summary>
</details>

<details>
<summary>

## Speedtest benchmark

</summary>
</details>

<details>
<summary>

## Difference between official kernel and custom kernel

</summary>
</details>

<details>
<summary>

## Buy me a coffee

</summary>
</details>
