# Hibiki

## General software configuration:
- macOS Catalina 10.15.x
- [OpenCore](https://github.com/acidanthera/OpenCorePkg/releases) Bootloader

## Prepared by [Mild Ideas](https://www.mildideas.com) for a Hackintosh video build using these parts:
* Intel Core i9-9900K
* ASRock Z390 Phantom Gaming-ITX/AC w/ TB3 port on `v4.20` bios firmware
* Powercolor Radeon VII
* Dell DW1560 (BCM94352Z) Wifi/BT
* Intel I219V GbE NIC
* Realtek ALC1220

_The rest of the Hibiki parts that don't affect this configuration at all:_
* 64GB (2x32GB) Corsair Vengeance DDR4-3000 Memory
* (2) 2TB Adata XPG 8200 Pro NVMe M.2 SSDs
* (1) 500MB Samsung 850 Pro SATA SSD
* Dancase A4-SFX v4 ITX case
* Corsair SF600


## How to set up your macOS installer USB drive and OpenCore bootloader

This guide assumes you're on a Mac, you have `git` installed, and you have a 16GB+ USB drive handy.

* Open `Terminal.app`, located in `/Applications/Utilities/` (almost everything in this guide requires copy/pasting terminal commands)
* Download macOS installer from the App Store
```
# Catalina:
/usr/bin/open "macappstores://itunes.apple.com/app/id1466841314"
```
```
# Mojave:
/usr/bin/open "macappstores://itunes.apple.com/app/id1398502828"
```

* Format USB Drive in Disk Utility to GUID
* Create macOS installer USB drive:
```
sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --nointeraction --downloadassets --volume /Volumes/Hibiki/
```
* Open the EFI partition on the USB drive
  * Use [MountEFI](https://github.com/corpnewt/MountEFI) and select the `Hibiki` drive
```
  git clone https://github.com/corpnewt/MountEFI
  cd MountEFI
  chmod +x MountEFI.command
  ./MountEFI.command
```
* Download [`fangf2018`](https://github.com/fangf2018/ASRock-Z390-Phantom-ITX-OpenCore-Hackintosh)'s Z390 OpenCore EFI configuration
```
cd /Volumes/EFI
git init
git remote add origin https://github.com/fangf2018/ASRock-Z390-Phantom-ITX-OpenCore-Hackintosh.git
git pull origin master
```
* If you prefer the macOS installer to be in English, run this to remove the preset Simplified Chinese language setting in the NVRAM of the just-downloaded EFI `config.plist`
```
plutil -replace NVRAM.Add.7C436110-AB2A-4BBB-A880-FE41995C9F82.prev-lang:kbd -data ''  EFI/OC/config.plist
```
* Use `macserial` against an `iMac19,1` Model ID to generate and replace `SystemSerialNumber`, `MLB`, `SystemUUID`, and `ROM`
```
# Download `macserial` and extract it
mkdir -p hibiki_tmp
curl https://github.com/acidanthera/MacInfoPkg/releases/download/2.0.8/macinfo-2.0.8-mac.zip -LO && unzip macinfo-2.0.8-mac.zip -d hibiki_tmp && rm macinfo-2.0.8-mac.zip
chmod a+x ./hibiki_tmp/macserial
# Generate SystemSerialNumber and MLB, takes the first output of `macserial`
HIBIKI_SERIAL=$(./hibiki_tmp/macserial -m iMac19,1 | head -n1)
rm -rf ./hibiki_tmp
IFS=' | ' read -r -a HIBIKI_SERIAL_ARRAY <<< "$HIBIKI_SERIAL"
# Generate SystemUUID
HIBIKI_UUIDGEN=$(uuidgen)
# Generate ROM (via a randomly generated MAC address)
HIBIKI_ROM=$(openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//')
# Replace `config.plist` with the appropriate generated serials
plutil -replace PlatformInfo.Generic.SystemSerialNumber -string ${HIBIKI_SERIAL_ARRAY[0]} EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.MLB -string ${HIBIKI_SERIAL_ARRAY[1]} EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.SystemUUID -string $HIBIKI_UUIDGEN EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.ROM -data $HIBIKI_ROM EFI/OC/config.plist
# Lint the `config.plist` to make sure we didn't mess anything up
plutil -lint EFI/OC/config.plist
```
* (Optional) Additional `config.plist` modifications per my preferences
```
# Clear out `ScanPolicy` so you can select a partition upon boot
plutil -replace Misc.Security.ScanPolicy -integer '0' EFI/OC/config.plist
# Other minor cleanups for my specific setup
plutil -replace NVRAM.Add.7C436110-AB2A-4BBB-A880-FE41995C9F82.boot-args -string 'dart=0 debug=0x100 keepsyms=1 darkwake=0 -v' EFI/OC/config.plist
plutil -replace Misc.Boot.Resolution -string '3840x2160@32' EFI/OC/config.plist
```
* (Optional for future maintenance) Because we pulled the EFI from an active modder's git repo, it's possible to commit your current changes and whenever you want to refresh what [`fangf2018`](https://github.com/fangf2018/ASRock-Z390-Phantom-ITX-OpenCore-Hackintosh) has been up to by `git fetch origin && git rebase origin master`

* __TODO__: Add additional kexts for Dell DW1560 (BCM94352Z) Wifi/BT card, maybe as a post macOS-install step cause we need access to `/Library/Extensions`?
* __TODO__: Carry over additional modifications to `config.plist` since
* __TODO__: Suggest how to configure bios settings?
