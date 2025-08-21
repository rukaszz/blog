---
layout: post
title:  "Linuxお引越しログ"
date:   2025-08-16T14:25:52-05:00
author: rukaszz
categories: [linux]
tags: [linux]
permalink: /posts/linux
---

# Linuxお引越しログ

## 概要

学生時代から使っていたノートPCのファンがイカれました．愛着があったので修理でもしようかと思いましたが，壊したら嫌ですしノートPCの基盤というか中身の修理はハードルが高いイメージなので，いい機会ですし買い換えることにしました．

スペックは高めなノートPCで，購入の数ヶ月前に出た新作でした．新規のPCへデータを移すわけですが，環境の再構築がめんどくさいです．私の理想としては，新PCへデータを移したら旧PCでやっていた作業を即座に再現できることが理想です．そのためにclonezillaというソフトウェアを用いて，クローンを新PCへ移植することにしました．clonezillaで旧PCのクローンを作成→クローンを新PCのへ移植→新PCを起動，というプランを立てて始めました．

結局プラン通りに行かず非常に苦労しましたので，備忘録というかログとして残しておき，過去に学ぶ賢者のために少しでも役に立つような記事にしたいと思います．

## clonezillaでクローンを作る

### clonezillaとは

オープンソースのディスククローンソフトウェアです．HDDなどのストレージデバイスをまるごとコピーしたりイメージとしてコピーして保存したりすることができます．これによってシステムのバックアップやドライブの交換が効率的にできます．端末だけでなくサーバで起動すれば複数のクライアントの処理もできますが，今回は個人の端末1台をクローニングします．

いろんなファイルシステムに対応しているのも嬉しいポイントで，ext4(Linux系)やNTFS，exFAT(Windows)に対応してます．USFドライブなどに焼くことで起動し，表示されるメニューに従ってクローンを作っていきます．

基本的には

1. clonezillaをメディアに焼く
2. bootメニューから起動するように設定を変更
3. Clonezillaを起動
4. 言語選択〜初心者モードとか選ぶ
5. ソースドライブとターゲットドライブを選んでクローンを作成
6. 終了

表示される文言に従って操作すれば問題ないかと思います．

そんなclonezillaですが，私の場合は購入したPCが最新すぎて復旧時，安定版では起動しないというトラブルがありました．その辺りはもう柔軟に対応するしか無いですね．


lsblk -f 
でパーティションを確認

以下を実行
sudo mount /dev/nvme0n1p5 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

chroot環境で次を実行
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu


EFI variables are not supprted on this system

というエラーがでた。

対処するために、
exit
で一度chroot環境から抜けて、
sudo mount --bind /sys/firmware/efi/efivars /mnt/sys/firmware/efi/efivars
を実行した。
するとコマンドが通ったので、update-grubまで実施した。

sudo chroot /mnt
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu --recheck
update-grub

エントリの確認：efibootmgr -v
BootCurrent: 0002 ←これはLive USBのもの
Timeout: 1 seconds
BootOrder: 0001, 0002
Boot0001* ubuntu        HD(1,MBR,0x2961deb,0x800,0x100000)/File(\EFI\ubuntu\grubx64.efi)
Boot002* UEFI: USB DISK 3.0 PMAP, Partition 2 PciRoot(0x0)/Pci(0x14,0x0)/USB(12,0)/HD(2,GPT,b0ef6ca3-60d1-4a21-9898db-17c385a8b6f8,0xbce0b4,0x27a0)..BO

どうやら通常のEFIシステムであまり使っていない、MBR形式らしい。
MBRなのでEFIがブートしない可能性がある？

sudo parted /dev/nvme0n1 print

partition table: msdos

ただEFIエントリとして正しく認識されているようなので、現状問題なさそう

ls -R /mnt/boot/efi/EFI
でEFIのパーティションを確認：

/mnt/boot/efi/EFI:
BOOT ubuntu

/mnt/boot/efi/EFI/BOOT
BOOTX64.EFI

/mnt/boot/efi/EFI/ubuntu:
grub.cfg grubx64.efi

構成としては問題なし。
セキュアブートは無効化している：
mokutil --sb-state: 
SecureBoot diabled

現状を再確認してみる：
sudo blkid: 
/dev/nvme0n1p1: UUID="EABB-E396" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="02961deb-01"
/dev/nvme0n1p5: UUID="2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="02961deb-05"

sudo chroot /mnt
nano /etc/fstab:
...
# / was on /dev/sda5 during installation
<file system>	<mount point>	<type>	<options>	<dump>	<pass>
UUID=2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30	/		ext4	errors=remount-ro 0	1
# /boot/efi was on /dev/sda1 during installation
UUID=EABB-E396	/boot/efi	vfat	umask=0077	0	1
/swapfile		none	swap	sw	0	0

UUIDなどは一致しており、問題なさそう
→念のため、initramfsを再生成
sudo chroot /mnt
update-initramfs -u
update-grub

cat /boot/grub/grub.cfg
を確認し、UUIDがnvmee0n1p5のものであることを確認した。

また、/bootに
initrd.imgやinitrd.img-XXXがあることを確認し、さらにvmlinuzやvmlinuz-xxxも存在していることを確認した。

ひとまずシャットダウンして、起動するか確認してみる。
poweroff

PC起動→grubメニューが起動→ubuntuを選んでeキー押下→rootdelay=10
それでもinitramfsへ落ちた

initramfsでは、
blkid
ls /dev
の2つを実施しても、nvme0n1やUUIDなどが表示されなかった。

現状のカーネル(nvme0n1のもの？)がnvmeをサポートしていない可能性が高い。


再度live usbで確認していく

上記の手順でchrootへ入る

ls /lib/modules/$(uname -r)/kernel/drivers/nvme/

でカーネルのドライバーを確認。
→/lib/modules/6.11.0-17-generic/kernel/drivers/nvme/
は無いといわれた。
一番数字が大きいディレクトリで調べる
ls /lib/modules/5.15.0-139-generic/kernel/drivers/nvme/

host target

しか表示されなかった。

6.11.0.17-genericへドライバーをインストールする。
E: Unable to package云々が表示された
6.11系は安定版ではなく、公式レポジトリからダウンロードできない可能性が高い？


modprobe nvme
を実行すると、
modprobe: FATAL: Module nvme not found in directory /lib/modules/6.11.0-17-generic

と表示された。

initramfsで、nvme.koを持っているカーネルのバージョンを調べる。

find /lib/modules -type f -name 'nvme.ko'：
/lib/modules/5.4.0-216-generic/kernel/drivers/nvme/nvme.ko
/lib/modules/5.15.0-138-generic/kernel/drivers/nvme/nvme.ko
/lib/modules/5.15.0-139-generic/kernel/drivers/nvme/nvme.ko

が表示された。

最も新しいであろう最新版、5.15.0-139を使う。

ls /boot | grep 5.15.0-139
で確認した結果、イメージがあることを確認。

initramfsを再生成：
update-initramfs -c -k 5.15.0-139-generic
update-grub

それでもinitramfsへ落ちた。

ドライバの再確認を行う。
デバイスの確認：
lsblk -f
blkid
ls /dev/nvme*
のすべてでnvmeデバイスの存在が確認できた

ドライバのロード確認：
modprobe nvme
modprobe nvme_core

この2点は出力が返ってこなかった

sudo dmesg | grep nvme
[0.997106] nvme nvme0: pci function 10000:e1:00.0
[0.997147] nvme 10000:e1:00.0 PCI INT A: no GSI
[1.020683] nvme nvme0: allocated 64MiB host memory buffer.
[1.035462] nvme nvme0: 16/0/0 default/read/poll queues
[1.265959] EXT4-fs (nvme0n1p5): unmounting filesystem 2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30 ro with ordered data mode. Quota mode; none.
[1.266858] EXT4-fs (nvme0n1p5): unmounting filesystem 2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30
[19.889591] block nvme0n1: No UUID available providing old NGUID
[80.579242] block nvme0n1: the capability attribute has been deprecated. 
[149.078262] nvme nvme0: I/O tag 641 (d281) QID 4 timeout, completion polled
が返ってきた。

内容を確認すると、nvmeデバイスの認識はされてそうである。
[149.078262] nvme nvme0: I/O tag 641 (d281) QID 4 timeout, completion polled
にあるように、I/Oで問題があるように見える。

nvmeデバイスの初期化が遅れている、initramfsの起動タイミングが早すぎる、I/O遅延やエラー、カーネルとnvmeの相性などがある

grubを編集して起動に遅延を設ける。grubメニューでcを押下。
rootdelay=20 nvme_core.default_ps_max_latency_us=0

nvme_core.default_ps_max_latency_us=0
は省電力モードによる不安定性に対処するコマンド

上記のrootdelay=20...でもダメ

rootwaitで行けたという記事を見つけたのでやってみた：
https://gihyo.jp/lifestyle/serial/01/ganshiki-soushi-2/0001

rootwait nvme_core.default_ps_max_latency_us=0

これでもinitramfsに落ちた


initramfsでモジュールの読み込みを確実にする方法を実施してみる

1. live USBで起動
2. chrootで入るところまでやる
3. nano /etc/initramfs-tool/modulesを実行
modulesはテキストファイルで、起動時のinitramfs、つまり初期RAMに組み込むモジュールを選択する
4. /etc/initramfs-tool/modulesに追記
nvme-core
nvme
5. ctrl + O -> enter -> ctrl + Xで上書き保存

これで確実にnvmeのドライバが組み込まれ、デバイスが検知されるかもしれない。

ダメだったので、状況を整理するためにいくつかのコマンドをinitramfsで実行する：

1. lsmod | grep nvme
→lsmodが見つからなかった
2. modprobe nvme_core
→出力無し
3. modprobe nvme
→出力無し
4. ls /dev/nvme*
→No such file or directory
5. cat /proc/partitions
→major minor #blocks name
6. dmesg | grep -i nvme
→出力無し
7. cat /proc/cmdline
→BOOT_IMAGE=/boot/vmlinuz-5.15.0-139-generic root=UUID=2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30 ro quiet splash rootdelay=20 nvme_core.default_ps_max_latency_us=0
8. cat /etc/initramfs-tools/modules
→initramfs-tools/modulesディレクトリが無い
9. chroot環境で次のコマンドを実行
lsinitramfs /boot/initramfs-*.img | grep nvme
→/usr/bin/unmkinitramfs: 64: cannot open /boot/initramfs-*.img: No such file
/usr/bin/unmkinitramfs: 57: cannot open /boot/initramfs-*.img: No such file
/usr/bin/unmkinitramfs: 38: cannot open /boot/initramfs-*.img: No such file
cpio: premature end of archive
10. chroot環境外で次を実行した：
ls /mnt/boot | grep initrd
initrd.img
initrd.img-5.15.0-138-generic
initrd.img-5.15.0-139-generic
initrd.img-5.4.0-216-generic
initrd.img.old
11. lsinitramfs /mnt/boot/initrd.img-5.15.0-139-generic | grep nvme
→/usr/bin/unmkinitramfs: 64: cannot open /boot/initrd.img-5.15.0-139-generic: No such file
/usr/bin/unmkinitramfs: 57: cannot open /boot/initrd.img-5.15.0-139-generic: No such file
/usr/bin/unmkinitramfs: 38: cannot open /boot/initrd.img-5.15.0-139-generic: No such file
12. chrootの環境で次のコマンドを実施：
cat /etc/initramfs-tools/modules
→# List of 
  ...
  # sd_mod
nvme_core
nvme
・cat /etc/initramfs-tools/initramfs.conf | grep -MODULES=
→MODULES=most
13. BusyBox上(initramfs上)で次を実施：
・cat /proc/modules | grep nvme
→nvme 49152 0 - Live 0xffffffffc0341000
nvme_core 135168 1 nvme, Live 0xffffffffc37c000

・find /lib/modules -type f | grep nvme
→/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvmet.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvmet-tcp.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvmet-rdma.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvmet-fc.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvmet-loop.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme-tcp.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme-rdma.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme-fc.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme-fabrics.ko
/lib/modules/5.15.0-139-generic/kernel/drives/nvme/target/nvme-core.ko

・cat /proc/partitions
→major minor #blocks name

・cat /proc/cmdline
→BOOT_IMAGE=/boot/vmlinuz-5.15.0-139-generic root=UUID=2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30 ro quiet splash

14. Live USBでハードウェア確認
・sudo lspci -nnk | grep -i nvme -A3
→
0000:00:0e.0 RAID bus controller [0104]: Intel Corporation Volume Management Device NVMe RAID Controller Intel Corporation [8086:7d0b]
    DeviceName: Onboard - Other
    Subsystem: Micro-Star International Co., Ltd. [MSI] Volume Management Device NVMe RAID Controller Intel Corporation [1462:144f]
    Kernel driver in use: vmd
    Kernel modules; vmd
0000:00:14.0 USB controller [0c03]: Intel Corporation Device [8086:777d]
--
    Kernel driver in use: nvme
    Kernel modules: nvme

・lsblk -o NAME,HCTL,SIZE,MODEL
nvme0n1    953.9G Micron_2500_MTFDKBA1T0QGN
├nvme0n1p1    512M
├nvme0n1p2    1k
└nvme0n1p5    238G

・sudp apt update && sudo apt install -y nvme-cli
sudo nvme list
→
Node    Generic    SN    Model    Namespace    Usage    Format    FW Rev
----    ----    ----    ----    ----    ----    ----    ----
/dev/nvme0n1    /dev/ng0n1    24504CEB9A7A    Micron_2500_MTFDKBA1T0QGN    0x1    1.02 TB / 1.02 TB    512 B + 0 B    V8MA000

15. chrootへ
# マウント例
sudo mount /dev/nvme0n1p5 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi   # EFI が別なら
# └ /boot が別パーティションなら /mnt/boot にマウント

# 必要な仮想ファイルシステムをバインド
for d in dev proc sys run; do
  sudo mount --bind /$d /mnt/$d
done

# chroot
sudo chroot /mnt

16. grubの調整
# 編集: rootdelay と NVMe パラメータを追加
sed -i \
  -e 's/^GRUB_CMDLINE_LINUX="/&rootdelay=20 nvme_core.default_ps_max_latency_us=0 /' \
  /etc/default/grub

# 反映
update-grub

・
# /etc/initramfs-tools/modules に nvme 系を追記
grep -v '^#' /etc/initramfs-tools/modules
→nvme_core
nvme

# MODULES 設定が “most” か確認
grep '^MODULES=' /etc/initramfs-tools/initramfs.conf
→MODULES=most

update-initramfs -u -k all
→update-initramfs: Generating /boot/initrd.img-5.15.0-139-generic
update-initramfs: Generating /boot/initrd.img-5.15.0-138-generic
update-initramfs: Generating /boot/initrd.img-5.4.0-216-generic

17. 16までやってもbusyboxへ落ちたので、いろいろ確認する
・cat /proc/modules | grep -i nvme
→nvme 49152 0 -Live 0xffffffffc02d6000
nvme_core 135168 1 nvme, Live 0xffffffffc277000
・grep -R "nvme" /sys/bus/pci/devices/*/class 2>/dev/null
→結果の表示なし

・grep nvme /proc/devices
→237 nvme-generic
238 nvme

18. これでmajor番号がわかった。nvmeは238という数字であり、これはブロックデバイス用のmajor番号である。
mknod <デバイスファイル> b <major> <minor>
なので、
# NVMe デバイス本体
mknod /dev/nvme0n1 b 238 0

# 1番目パーティション
mknod /dev/nvme0n1p1 b 238 1

# 5番目パーティション（ルート）
mknod /dev/nvme0n1p5 b 238 5

・ls /dev
→nvme0n1, nvme0n1p1, nvme0n1p5が確認できた

・mkdir /rootfs
→ / に /rootfsが作られた
・mount /dev/nvme0n1p5 /rootfs
→ [948.092713] /dev/nvme0n1p5: Cant't open blockdev
[948.099783] /dev/nvme0n1p5: Cant't open blockdev

・ls /rootfs
→何も表示されなかった

19. 一時的な対処
・mount -t proc proc  /proc
→mountできた
・mount -t sysfs sysfs /sys
→mount: mounting sysfs on /sys failed: Device or resource busy
・mount -t devtmpfs devtmpfs /dev
→mount: mounting sysfs on /dev failed: Device or resource busy

となった。

・ls /lib/modules/$(uname -r)/kernel/drivers/pci/
→controllerしかない

・ls /lib/modules/$(uname -r)/kernel/drivers/pci/controller
→pci-hyperv-initf.ko vmd.ko
が表示された

20. もう一度live 環境へ
・chroot環境へ
sudo mount /dev/nvme0n1p5 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
for d in dev proc sys run; do
  sudo mount --bind /$d /mnt/$d
done
sudo chroot /mnt

・uname -r
→6.11.0-17-generic

なので直接指定した。

・ls /lib/modules/5.15.0-139-generic/kernel/drivers/nvme/host
→nvme-core.ko nvme-fc.ko nvme-tcp.ko nvme-fabrics.ko nvme-rdma.ko nvme.ko

21. nvmeモジュールを早期ロードさせる
chroot環境で、次のファイルを作成する

```Bash
#!/bin/sh
set -e

PREREQ=""
prereqs() { echo "$PREREQ"; }
case "$1" in
  prereqs) prereqs; exit 0 ;;
esac

. /usr/share/initramfs-tools/hook-functions

copy_modules_dir nvme

# 必要なモジュールを全部コピー
# copy_moduleが無いので、copy_modules_dirを使う
# copy_module nvme-core
# copy_module nvme
# copy_module nvme-fabrics
# copy_module nvme-fc
# copy_module nvme-rdma
# copy_module nvme-tcp
```

・権限付与
sudo chmod +x /etc/initramfs-tools/hooks/force-nvme

・initramfsの再生成
sudo update-initramfs -u -k all
sudo update-grub

22. udevによるデバイスノードの生成を確認する
・mkdir /tmp/initrd-test && cd /tmp/initrd-test
・zcat /boot/initrd.img-5.15.0-139-generic | cpio -idmv
→gzip: /boot/initrd.img-5.15.0-139-generic: not in gzip format
cpio: premature end of archive

23．改めてdmesg | grep nvmeを実行

[0.973320] nvme nvme0: pci function 10000:e1:00.0
[0.973334] nvme 10000:e1:00.0: PCI INT A: no GSI
[0.996176] nvme 0: allocated 64 MiB host memory buffer. 
[0.996176] nvme 0: 16/0/0default/read/poll queues
[1.011196] nvme0n1: p1 p2 < p5 >
[1.241015] EXT4-fs (nvme0n1p5): mounted filesystem 2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30 ro with ordered data mode. Quota mode; none.
[1.241375] EXT4-fs (nvme0n1p5): unmouting filesystem 2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30
[19.575770] block nvme0n1: No UUID available providing old NGUID
[19.575770] block nvme0n1: the capability attribute has been depricated
[215.241015] EXT4-fs (nvme0n1p5): mounted filesystem 2a92b7e7-dcc9-4cac-a9f4-b52d715f2b30 r/w with ordered data mode. Quota mode; none.
[13537.651864] nvme nvme0: I/O tag 449 (51c1) QID 1 timeout, completion polled
[13540.403918] nvme nvme0: I/O tag 276 a114) QID 5 timeout, completion polled

























