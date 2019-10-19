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
* Remove the preset Simplified Chinese language setting in the NVRAM of the just-downloaded EFI
```
plutil -replace NVRAM.Add.7C436110-AB2A-4BBB-A880-FE41995C9F82.prev-lang:kbd -data ''  EFI/OC/config.plist
```
or
```
sed -i 's/emgtSGFuczoyNTI=//g' EFI/OC/config.plist
```
* Add serial number for this machine
  * *TODO:* Automate this generation
```
plutil -replace PlatformInfo.Generic.SystemSerialNumber -string 'SERIAL HERE' EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.SystemUUID -string 'UUID HERE' EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.MLB -string 'MLB HERE' EFI/OC/config.plist
plutil -replace PlatformInfo.Generic.ROM -data 'BASE64 ROM HERE' EFI/OC/config.plist
```

* (Optional) Clear out `ScanPolicy` so you can select a partition upon boot
```
plutil -replace Misc.Security.ScanPolicy -integer '0' EFI/OC/config.plist
```

* Other minor cleanups for my specific setup
```
sed -i 's/agdpmod=pikera//g' EFI/OC/config.plist 
plutil -replace Misc.Boot.Resolution -string '3840x2160@32' EFI/OC/config.plist
```
