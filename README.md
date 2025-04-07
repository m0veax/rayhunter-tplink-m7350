![Rayhunter Logo - An Orca taking a bite out of a cellular signal bar](https://www.eff.org/files/styles/media_browser_preview/public/banner_library/rayhunter-banner.png)
# Rayhunter for TP-Link M7350

![Tests](https://github.com/EFForg/rayhunter/actions/workflows/check-and-test.yml/badge.svg)

Rayhunter is an IMSI Catcher Catcher originally developed for the Orbic mobile hotspot (available in U. S. market) and now ported to TP-Link M7350 mobile hotspot (available in European market and worldwide).

**THIS CODE IS PROOF OF CONCEPT, IS BEING ACTIVELY DEVELOPED AND SHOULD NOT BE RELIED UPON IN HIGH RISK SITUATIONS**

## The Hardware

Rayhunter has been originally built and tested for the Orbic RC400L mobile hotspot, however Orbic works only on North American LTE Bands (B2/B4/B5/B13/B66) and can not be used in Europe.

TP-Link M7350 mobile hotspot has on the other side been developed for European and Asian markets (it uses LTE Bands B1/B3/B7/B8/B20) and currently we are porting Rayhunter to TP-Link M7350. TP-Link M7350 should also work in Africa, Oceania and South America, but with limited compatibility, because some network in those areas use Bands that are not supported by it.

TP-Link M7350 mobile hotspot has several hardware revisions - you can see your version on a sticker behind the battery - and our version of Rayhunter already works on the several versions of TP-Link M7350.

TP-Link M7350 needs to be rooted first (we are developing rooting scripts), then Rayhunter can be installed on it. Please note that **original EFF version** of Rayhunter is currently not working on TP-Link devices, so you will need to install our modified version (see below for detailed instructions).

| HW revision | rooting  | Rayhunter  |
| :---:   | :---: | :---: |
| v1 | no info               | no info            |
| v2 | no info               | no info            |
| v3 | yes, with [open-v3.sh](https://github.com/m0veax/tplink_m7350/blob/main/open-v3.sh) script  | yes              |
| v3.2 | no info               | no info            |
| v4 | yes, with [open-v4.sh](https://github.com/m0veax/tplink_m7350/blob/main/open-v4.sh) script or [tpown](https://github.com/m0veax/rayhunter-tplink-m7350/blob/installer/PoC.md#v4) | yes |
| v5 | yes, with [tpown](https://github.com/m0veax/rayhunter-tplink-m7350/blob/installer/PoC.md#v4) | yes |
| v6.2 | yes (no further info)              | yes |
| v7 | no info               | no info |
| v8 | yes, with Chrome hack | yes |
| v9 | yes, with Chrome hack | yes, but with small ioctl changes |

Rayhunter may work on other Linux/Qualcomm devices (Orbic and M7350 both have Qualcomm MDM9225 chipset), but has not been tested on them (and you would need to root them first).

## Installing Rayhunter

TP-Link M7350 mobile hotspot needs to be rooted first. For rooting use one of the methods described in the table above. Also, you will need to install `ADB` on your computer first.

After that, Rayhunter binary can be installed on the device.

Manual procedure is the following (we assume you are using (Ubuntu/Debian) Linux environment for installation).

### Prepare the files

**First**, download and extract [release v0.1](https://github.com/m0veax/rayhunter-tplink-m7350/releases/tag/test-release-v0.1):
- download file `release.tar.zip`,
- *unzip* it, and you will get file `release.tar`,
- *untar* that file and there will be a folder `release`.

Inside `release/` folder are two files you will need:
- `config.toml.example`
- `rayhunter-daemon`

`config.toml.example` is a text file and you need to edit it to use SD card instead of internal memory. Open the file with any plain text editor, for instance with `nano config.toml.example`.

Now change path for logs to `/media/card/qmdl`. **You must have SD card (SDHC, 32 GB max!) inserted in the device**. Path should be:
```
qmdl_store_path = "/media/card/qmdl"
```
`rayhunter-daemon` is a 32-bit ARM binary file that can run on a TP-Link M7350 device. `file` command shows this:
```
rayhunter-daemon: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, BuildID[sha1]=6d10da1d0c9f1994a4e0b6d29a59e0d5446dd73c, for GNU/Linux 3.2.0, stripped
```

Make this file executable:
``` 
chmod +x rayhunter-daemon
```

### Copy files to the device

Open two terminals:
- terminal 1 (we will call it `adb shell`)
- terminal 2 (we will call it `local`)

**Terminal 1 (adb shell)**, type these commands:
```
adb shell
mkdir /data/rayhunter
```
**Terminal 2 (local)**, go to the folder containig `config.toml.example` and `rayhunter-daemon` files, and copy those two files to the device:
```
adb push config.toml.example /data/rayhunter/config.toml
adb push rayhunter-daemon /media/card/rayhunter-daemon
```

**Terminal 1 (adb)**, type this command:
```
/media/card/rayhunter-daemon /data/rayhunter/config.toml
```
On your computer open web browser and go to `http://192.168.0.1:8080`.

If you can not access this URL, disconnect your computer from VPN first, if that does not help, try to disconnect from WiFi and/or Ethernet, so your only connection should be with USB cable to the TP-Link M7350 device. You also need to insert SIM card to the TP-link device if you want Rayhunter to work.

![Image](https://github.com/user-attachments/assets/ce6df40c-c87d-4adf-ac91-24082643bdeb)

### Autostart Rayhunter on the device

Since you want that Rayhunter is autostarted at each boot of the TP-Link device, you need to set up `init.d` script. Currently we are using a "hack" - we modify `lighttpd2` script - but we are working on a proper Rayhunter `init.d` script.

First copy [this script from the repository](https://github.com/m0veax/rayhunter-tplink-m7350/blob/installer/dist_tplink/lighttpd) to your clipboard.

Connect to the device:
```
adb shell 
```

Copy the content of the file to a new file (named `lighttpd2`): 
```
cd /etc/init.d/
vi lighttpd2
```
In `vi` editor press `esc`, `:`,  `i` and then paste the text of the script with right click of the mouse.  `esc`, `w`,  `q` - write and save the file.

Now remove the old file and replace it with the new one:
```
rm lighttpd
mv lighttpd2 lighttpd
reboot
```

After reboot, Rayhunter should be autostarted automatically. You can visit Rayhunter WebUI with your web browser at `http://192.168.0.1:8080`.

### Update to a new version

For updating (when there will be new release), you just need to copy new `rayhunter-daemon` binary file to the device. Basically you need to `adb shell` to a device, kill currently running `rayhunter-daemon` process, delete old binary from `/media/card/` and `adb push` new binary to the device.

## Usage

Once installed, Rayhunter will run automatically whenever your device is running. It serves a web UI that provides some basic controls, such as being able to start/stop recordings, download captures, and view heuristic analyses of captures. You can access this UI in one of two ways:

1. **Over WiFi**: Connect your phone/laptop to the TP-Link M7350 WiFi network and visit `http://192.168.0.1:8080` (click past your browser warning you about the connection not being secure, Rayhunter doesn't have HTTPS yet!)

2. **Over USB**: Connect the Orbic device to your laptop via USB, then visit `http://192.168.0.1:8080`.

Please note that if you are using VPN, access to Rayhunter's URL may not work.
 
## Frequently Asked Questions

### Do I need an active SIM card to use Rayhunter?
**It Depends**. Operation of Rayhunter does require the insertion of a SIM card into the device, but whether that SIM card has to be currently active for our tests to work is still under investigation. If you want to use the device as a hotspot in addition to a research device an active plan would of course be necessary, however we have not done enough testing yet to know whether an active subscription is required for detection. If you want to test the device with an inactive SIM card, we would certainly be interested in seeing any data you collect, and especially any runs that trigger an alert!
 
### Help, Rayhunter's line is red! What should I do?
Unfortunately, the circumstances that might lead to a positive CSS signal are quite varied, so we don't have a universal recommendation for how to deal with the a positive signal. You might also want to turn off your phone until you are out of the area (or put it on airplane mode,) and tell your friends to do the same!

 Please feel free to contact an EFF technologist with more information & a copy of the QMDL in question at [info@eff.org](mailto:info@eff.org) or to [Signal](https://signal.me/#eu/HZbPPED5LyMkbTxJsG2PtWc2TXxPUR1OxBMcJGLOPeeCDGPuaTpOi5cfGRY6RrGf) (preferred way). Please note that this file may contain sensitive information such as your IMSI and the unique IDs of cell towers you were near which could be used to ascertain your location at the time so for sending through e-mail consider using encryption.
 
### Does Rayhunter work outside of the US or Europe?
**Yes**. TP-Link M7350 has been successfully tested in several European countries. The TP-Link M7350 is designed for European and Asian markets because it supports LTE bands used in those countries. However it will probably not work in North and Latin America, Australia and New Zealand, parts of Africa and on some networks in Japan.

## Development (compiling Rayhunter binary)

Install `Rust` and cross compiling dependences:
```
sudo apt install curl build-essential libc6-armhf-cross libc6-dev-armhf-cross gcc-arm-linux-gnueabihf rustup cargo
rustup default stable
rustup target add x86_64-unknown-linux-gnu
rustup target add armv7-unknown-linux-gnueabihf
```

Clone the repository:
```
git clone https://github.com/NAME_OF_REPOSITORY
cd NAME_OF_REPOSITORY
```

Compile binary:
```
cargo build --target armv7-unknown-linux-gnueabihf --release
```
Compiled binaries are then in `target/armv7-unknown-linux-gnueabihf/release/`:
```
cd target/armv7-unknown-linux-gnueabihf/release/
file rayhunter-daemon

rayhunter-daemon: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, BuildID[sha1]=be93b0df122db6d577a2dd2da231d01ddb351ba9, for GNU/Linux 3.2.0, stripped
```

Building documentation locally:
`RUSTDOCFLAGS="--cfg docsrs" cargo doc --no-deps --all-features  --open`

Documentation is then in `target/doc/rayhunter/index.html`.


## LEGAL DISCLAIMER
Use this program at your own risk. We believe running this program does not currently violate any laws or regulations in the United States or in Europe.

The reason for that is, that this software uses Qualcomm DIAG kernel driver (`DIAG_CHAR`) to analyze **your own network traffic** that is processed by baseband chip on your device. So, there is **no interception of traffic** of other mobile subscribers and **no unauthorized firmware modifications** to the baseband chip which would normally require a new certification. Rayhunter just enables you to see and analyse all network traffic from the mobile network which is usually hidden from you.

However, we are not responsible for civil or criminal liability resulting from the use of this software. Please consult with an attorney in your country to help you assess the legal risks of running this program.

*Good Hunting!*
