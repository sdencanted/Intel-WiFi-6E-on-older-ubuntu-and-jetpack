# Intel-WiFi-6E-on-older-ubuntu-and-jetpack
Instructions on getting Wi-Fi 6E to work on older Ubuntu systems, specifically on NVIDIA Jetson devices

-   Tested with an amd64 Ubuntu 20.04 system and a NVIDIA Jetson Orin NX/Nano ont JetPack 5.1.1, all using AX210 Wi-Fi 6E. 
-   These instructions might apply to other Wi-Fi cards / computers running Ubuntu 20.04 and under with modifications.
-   Thanks to 

## Preparation
-   Existing non Wi-Fi 6E internet (Wi-Fi 6 and below, Ethernet cable, etc)
-   Wi-Fi 6E connection for testing
-   For headless Jetson devices:
    -   USB cable to connect to your Jetson device for SSH over USB (`ssh username@192.168.55.1`)

## Downloading necessary files
Get iwlwifi-backport from https://launchpad.net/ubuntu/mantic/arm64/backport-iwlwifi-dkms. The package tested is http://launchpadlibrarian.net/676942313/backport-iwlwifi-dkms_11289-0ubuntu1_all.deb.

```bash
cd ~ 
wget http://launchpadlibrarian.net/676942313/backport-iwlwifi-dkms_11289-0ubuntu1_all.deb
```

iwlwifi-backport requires dkms 3, get it from https://launchpad.net/ubuntu/lunar/arm64/dkms. Lunar was chosen as Mantic dkms requires gcc-12 which is not included in JetPack 5. The package tested is http://launchpadlibrarian.net/681873109/dkms_3.0.10-7ubuntu2.1_all.deb.

```bash
wget http://launchpadlibrarian.net/681873109/dkms_3.0.10-7ubuntu2.1_all.deb
```

Get the wifi drivers from https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/. For AX210 `iwlwifi-ty-a0-gf-a0-83.ucode` and `iwlwifi-ty-a0-gf-a0.pnvm` were used.

```bash
wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/iwlwifi-ty-a0-gf-a0-83.ucode
wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/iwlwifi-ty-a0-gf-a0.pnvm
```

## Installation

```bash
sudo apt update
sudo apt upgrade
sudo apt install dpkg-dev
sudo dpkg -i dkms_3.0.10-7ubuntu2.1_all.deb
sudo dpkg -i backport-iwlwifi-dkms_11289-0ubuntu1_all.deb
sudo mv iwlwifi-ty-a0-gf-a0-83.ucode /lib/firmware
sudo mv iwlwifi-ty-a0-gf-a0.pnvm /lib/firmware
```

Restart
```bash
sudo init 6
```

## Connecting to a Wi-Fi 6E network

1. Open the text UI for NetworkManager
    ```
    sudo nmtui
    ```
2. Select `Edit a connection`
3. Select `<Add>`
4. Choose `Wi-Fi`
5. Fill in `Profile name`, `Device` (default seems to be `wlan0`),`SSID`,`Mode` (Leave as Client), `Security`(`WPA3` if you have a password), and `Password`
6. Select `<OK>`
7. Replace `<profilename>` with your `Profile name` and run 
    ```bash
    sudo nmcli con up <profilename>
    ```

- It may take a while for the wifi device to show up after rebooting, so try step 7 again if it fails. Personally it took around 5 minutes.
