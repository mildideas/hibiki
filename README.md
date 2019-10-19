# hibiki

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
curl https://github.com/acidanthera/MacInfoPkg/releases/download/2.0.8/macinfo-2.0.8-mac.zip -LO && unzip macinfo-2.0.8-mac.zip -d hibiki_tmp
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
```

* (Optional) Additional `config.plist` modifications per my preferences
```
# Clear out `ScanPolicy` so you can select a partition upon boot
plutil -replace Misc.Security.ScanPolicy -integer '0' EFI/OC/config.plist
# Other minor cleanups for my specific setup
sed -i 's/agdpmod=pikera//g' EFI/OC/config.plist 
plutil -replace Misc.Boot.Resolution -string '3840x2160@32' EFI/OC/config.plist
```

* (Optional for future maintenance) Because we pulled the EFI from an active modder's git repo, it's possible to commit your current changes and whenever you want to refresh what [`fangf2018`](https://github.com/fangf2018/ASRock-Z390-Phantom-ITX-OpenCore-Hackintosh) has been up to by `git fetch origin && git rebase origin master`
