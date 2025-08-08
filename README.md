**Gaming Is a Right — Even on a Virtual Machine.**

# Other Project
For QEMU ANTIDETECTION, see https://github.com/zhaodice/qemu-anti-detection
# Proxmox VE(PVE 8.4-1)
See the old https://github.com/zhaodice/proxmox-ve-anti-detection/blob/main/readme-8.1.5-3.md

# Proxmox VE(PVE 8.4-1) Anti Detection
 | Type       | Engine | Bypass |
 |------------|--------|--------|
 | AntiCheat | Mhyprot | ☑️   |
 | AntiCheat | Anti Cheat Expert(ACE) | ☑️   |
 | AntiCheat | Easy Anti Cheat(EAC) | ☑️   | 
 | AntiCheat | nProtect GameGuard(NP) | ☑️   | 
 | AntiCheat | Vanguard | ‼️(1: Incorrect function) | 
 | AntiCheat | Roblex | ‼️(The application encountered an unrecoverable error) | 
 | AntiCheat | Gepard Shield | ☑️ (But need to patch host kernel: https://github.com/WCharacter/RDTSC-KVM-Handler ) |
 | Encrypt | VMProtect | ☑️   | 
 | Encrypt | VProtect | ☑️   |  
 | Encrypt | Themida | ☑️   |  
 | Encrypt | Enigma Protector | ☑️   |  
 | Encrypt | Safegine Shielden | ☑️   |  

Flaws :
```
These commands could DETECT THIS VM (Shows "No instance available"), and NO SOLUTION CURRENTLY (I don't know how to simulate that..).

---------------------------

wmic path Win32_Fan get *

wmic path Win32_CacheMemory get *

wmic path Win32_VoltageProbe get *

wmic path Win32_PerfFormattedData_Counters_ThermalZoneInformation get *

wmic path CIM_Memory get *

wmic path CIM_Sensor get *

wmic path CIM_NumericSensor get *

wmic path CIM_TemperatureSensor get *

wmic path CIM_VoltageSensor get *
```

# Build deb

!!! Create Proxmox VE Virtual Machine as a compile environment. !!!

1. Login to System


2. Remove the old sources:
```
mv /etc/apt/sources.list /etc/apt/sources.list.deleted
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.deleted
```


3. Add to `/etc/apt/sources.list`
```
deb http://download.proxmox.com/debian bookworm pve-no-subscription

deb http://ftp.us.debian.org/debian bookworm main contrib non-free

deb http://ftp.us.debian.org/debian bookworm-updates main contrib non-free

# security updates
deb http://security.debian.org bookworm-security main contrib non-free
```

4. Upgrade
```
apt update && apt -y full-upgrade
```

5. PVE 8.4-1 ISO is not actually quite PVE 8.4.1, so downgrade
```
apt install pve-manager=8.4.1
```

6.
.
(This patch is made at commit e0969989ac8ba252891a1a178b71e068c8ed4995)
```
git clone git://git.proxmox.com/git/pve-qemu.git
cd pve-qemu
git reset --hard e0969989ac8ba252891a1a178b71e068c8ed4995
apt install  devscripts
mk-build-deps --install
wget "https://raw.githubusercontent.com/Ethorbit/proxmox-ve-anti-detection/refs/heads/main/001-anti-detection.patch" -O qemu/001-anti-detection.patch
```

7.

`nano debian/rules`

FIND LINE:

```
	# guest-agent is only required for guest systems
	./configure \
	--with-git-submodules=ignore \
	--docdir=/usr/share/doc/pve-qemu-kvm \
	--localstatedir=/var \
	--prefix=/usr \
	--sysconfdir=/etc \
	--target-list=$(ARCH)-softmmu,aarch64-softmmu \
	--with-suffix="kvm" \
	--with-pkgversion="${DEB_SOURCE}_${DEB_VERSION_UPSTREAM_REVISION}" \
	--audio-drv-list="alsa" \
	--datadir=/usr/share \
	--libexecdir=/usr/lib/kvm \
	--disable-capstone \
	--disable-gtk \
	--disable-guest-agent \
	--disable-guest-agent-msi \
	--disable-libnfs \
	--disable-libssh \
	--disable-sdl \
	--disable-smartcard \
	--disable-strip \
	--disable-xen \
	--enable-curl \
	--enable-docs \
	--enable-glusterfs \
	--enable-gnutls \
	--enable-libiscsi \
	--enable-libusb \
	--enable-linux-aio \
	--enable-linux-io-uring \
	--enable-numa \
	--enable-opengl \
	--enable-rbd \
	--enable-seccomp \
	--enable-slirp \
	--enable-spice \
	--enable-usb-redir \
	--enable-virglrenderer \
	--enable-virtfs \
	--enable-virtiofsd \
	--enable-zstd
```


Inject a line :

```
	# [Inject]Surprised Detector's Mother Fucker !!!
	patch -p1 < 001-anti-detection.patch
	
	# guest-agent is only required for guest systems
	...
```

8. Current folder is `git's root path`.
```
make clean
make
```

9. You will see a `ERROR` like follows

```
dpkg-source: info: unapplying 001-anti-detection.patch
dpkg-source: error: diff pve-qemu-kvm-9.2.0/debian/patches/001-anti-detection.patch modifies file pve-qemu-kvm-9.2.0/subprojects/libvduse/standard-headers/linux/qemu_fw_cfg.h through a symlink: pve-qemu-kvm-9.2.0/subprojects/libvduse/standard-headers/linux
dpkg-buildpackage: error: dpkg-source --after-build . subprocess returned exit status 25
make: *** [Makefile:36: pve-qemu-kvm_9.2.0-5_amd64.deb] Error 25
```

This error occurs because you've edited the rules file to apply a patch, but it is not able to remove the patch cleanly.

However, there is NO PROBLEM, because you should still get the patched .deb file: pve-qemu-kvm_9.2.0-5_amd64.deb.

# Install deb

copy `pve-qemu-kvm_??_amd64.deb` into your real PVE system.(to use this deb , you should install `librbd-dev=16.2.11-pve1` first)
```
apt install librbd-dev
dpkg -i pve-qemu-kvm_??_amd64.deb
```

# VM Show

`/etc/pve/qemu-server/100.conf`
```
args: -cpu host,+kvm_pv_unhalt,+kvm_pv_eoi,hv_spinlocks=0x1fff,hv_vapic,hv_time,hv_reset,hv_vpindex,hv_runtime,hv_relaxed,kvm=off,hv_vendor_id=intel,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off -smbios type=0,version=UX305UA.201 -smbios type=1,manufacturer=ASUS,product=UX305UA,version=2021.1 -smbios type=2,manufacturer=Intel,version=2021.5,product='Intel i9-12900K' -smbios type=3,manufacturer=XBZJ -smbios type=17,manufacturer=KINGSTON,loc_pfx=DDR5,speed=4800,serial=114514,part=FF63 -smbios type=4,manufacturer=Intel,max-speed=4800,current-speed=4800
audio0: device=ich9-intel-hda,driver=none
balloon: 0
bios: ovmf
boot: order=sata0
cores: 12
cpu: host,flags=+pcid
efidisk0: local-lvm:vm-100-disk-0,efitype=4m,pre-enrolled-keys=1,size=4M
hostpci0: 0000:01:00,pcie=1,x-vga=1
machine: q35
memory: 32768
meta: creation-qemu=9.2.0,ctime=1679627202
name: Windows11
net0: rtl8139=CA:09:F3:97:56:0B,bridge=vmbr0,firewall=1
numa: 1
ostype: win11
sata0: HugeSSD:100/vm-100-disk-1.qcow2,discard=on,size=64G,ssd=1
sata1: HugeHDD:100/vm-100-disk-0.qcow2,backup=0,size=128G
smbios1: uuid=24c326dd-3cec-48fc-bb9f-87aa3984e2c9,manufacturer=QVNVUw==,product=VVgzMDVVQQ==,version=MjAyMS4x,serial=MTI0NjY3,sku=MTM0NDY4,family=Ng==,base64=1
sockets: 1
tpmstate0: local-lvm:vm-100-disk-1,size=4M,version=v2.0
usb0: host=4e53:5406
usb1: host=040b:2000
vga: none
vmgenid: 020229e3-2cdb-4a91-8e77-1d04cf1f060f
```

![Screenshot_20230329_124628](https://user-images.githubusercontent.com/63996691/228429821-443a3a9f-8d24-4712-9760-bbfb1f10aa8d.png)
