# Custom pfSense kernel build with bbr, epair, vxlan and fusefs support.
I mostly follow [the guide of Augustin-FL](https://github.com/Augustin-FL/building-pfsense-iso-from-source), except I only tend to build a working kernel with bbr support instead of the whole ISO.
Currently only tested in pfSense CE 2.7.2

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
`options         FUSEFS
options         VIMAGE
makeoptions     WITH_EXTRA_TCP_STACKS=1
options         TCPHPTS
options         RATELIMIT`

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

</details>

<details open>
<summary>

## Build & install the custom kernel

</summary>
</details>

<details open>
<summary>

## Enable BBR in pfSense

</summary>
</details>

<details open>
<summary>

## Speedtest benchmark

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
