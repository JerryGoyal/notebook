# linux

List of Contents:

* [Useful Networking Commands](#networking)
* [Improve Performance](#improve-performance)
* [Mount Swap Partition](#mount-swap-partition)
* [Arch Linux Installation](#arch-linux-installation)
* [Building Custom Arch ISO](#building-custom-arch-iso)
* [How to submit a package to AUR?](#how-to-submit-a-package-to-aur)
* [Irssi](#irssi)
* [Xmonad](#xmonad)
* [How to build a AUR package?](#how-to-build-aur-package)
* [Useful Commands](#useful-commands)
  * [chroot](#chroot)
  * [dialog](#dialog)
  * [fc-list](#fc-list)
  * [nl](#nl)
  * [powertop](#powertop)
  * [yes](#yes)
  * [du](#du)
  * [ncdu](#ncdu)
  * [expac](#expac)
  * [ip](#ip)
  * [pkill](#pkill)
  * [sed](#sed)
  * [awk](#awk)
  * [tmux](#tmux)
  * [htop](#htop)
  * [iotop](#iotop)
  * [systemctl](#systemctl)
  * [systemd-analyze](#systemd-analyze)
  * [tee](#tee)
  * [file](#file)
  * [date](#date)
  * [find](#find)
  * [pidof](#pidof)
  * [tree](#tree)
  * [xev](#xev)
  * [nethogs](#nethogs)
  * [setxkbmap](#setxkbmap)



## Networking

#### Get the name of the actively used network interfaces

```bash
route | grep '^default' | grep -o '[^ ]*$'
```

#### Get Internal IP Address:

```bash
ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1'
```

#### Get Sent/Received Bytes

```
cat /sys/class/net/eth0/statistics/rx_bytes
cat /sys/class/net/eth0/statistics/rx_packets
cat /sys/class/net/eth0/statistics/tx_packets
cat /sys/class/net/eth0/statistics/tx_bytes
```

#### Check If You're Online

```
ping -q -w 1 -c 1 $gateway> /dev/null && echo 1 || echo 0
```

#### Get Gateway IP

```
ip r | grep default | cut -d ' ' -f 3
```

## Mount Swap Partition

Create a Swap partition using partitioning tools like `cfdisk` or `parted`. Then format the partition as swap;

```
$ mkswap /dev/sdXY
```

Enable it;

```bash
$ swapon /dev/sdXY
```

Get UUID of the partition:

```bash
$ sudo blkid /dev/sdXY
```

And add following line into `/etc/fstab`

```fstab
UUID=? none   swap    sw      0       0
```

List swap devices and their sizes:

```bash
$ swapon -s
```

See which swap devices are being used:

```bash
$ cat /proc/swaps
```

See total, used and free swap space:

```
$ cat /proc/meminfo | grep Swap
```

Or:

```
$ cat /proc/swaps
```

### Swapiness

The swappiness sysctl parameter represents the kernel's preference (or avoidance) of swap space. Swappiness can have a value between 0 and 100, the default value is 60. A low value causes the kernel to avoid swapping, a higher value causes the kernel to try to use swap space. Using a low value on sufficient memory is known to improve responsiveness on many systems.
To check the current swappiness value:

```bash
$ cat /proc/sys/vm/swappiness
```

To temporarily set the swappiness value:

```bash
$ sysctl vm.swappiness=10
```

To set the swappiness value permanently, edit a `sysctl` configuration file:

```bash
echo "vm.swappiness=10" > /etc/sysctl.d/99-sysctl.conf
```

## Irssi
* Auto-connect to a network: `/server ADD -auto -network NetworkName irc.host.com 6667`
* Auto-join to channels: `/channel ADD -auto #channel NetworkName password`
* Switch to a window: `M-[number]` use letters when for windows beyond 9: `M-[q|w|e|r|t|y]`
* Close window: `/wc`
* Save config: `/save`
* Load a script: `/script load awm`
* Set theme: `/set theme weed`

## Xmonad

* After making a change hit `xmonad --recompile` to check errors, then `mod+r` to reload the config.


## How to submit a package to AUR ?

## Arch Linux Installation

Download ISO and create a bootable USB stick;

In Linux:
```bash
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync
```

In OSX:

```bash
sudo dd if=path/to/arch.iso of=/dev/rdiskX bs=1m
```

After booting the USB, follow these steps;

* [Create Partitions](#partitioning)
* Make sure boot (/mnt/boot) & root (/mnt) mounted.
* Run `pacstrap` to install the base: `pacstrap /mnt`
* Update FS Tab: `genfstab -U /mnt >> /mnt/etc/fstab`
* Switch to the new root: `arch-chroot /mnt`
* Install GRUB
* Localize:
  * [Select timezone](#timezones)
  * Generate /etc/adjtime: `hwclock --systohc`
  * Uncomment en_US lines in /etc/locale.gen: `sed -i -e '/^#en_US/s/^#//' /etc/locale.gen`
  * Generate locales with `locale-gen`
  * `echo "LANG=en_US.UTF-8" >> /etc/locale.conf`
  * `echo "FONT=Lat2-Terminus16" >> /etc/vconsole.conf`
* [Create users](#creating-users)
* Done

### Partitioning

  * Command-line tools: parted, fdisk, cfdisk (with UI)
  * List disks and partitions by `fdisk -l` or `lsblk`
  * The simple and popular layout is boot & root;

  ```bash
  parted /dev/sda --script mklabel msdos \
      mkpart primary ext4 1MiB 512MiB \
      set 1 boot on \
      mkpart primary ext4 512MiB 100%
  ```

  * Don't forget formatting them as ext4: `yes | mkfs.ext4 /dev/sda1`

### Creating Users

  This bash function I wrote for happy-hacker-linux installer shows the steps of creating a user.

```bash
createUser () {
    useradd -m -s /usr/bin/zsh $1
    echo "$1:$2" | chpasswd

    echo "$1 ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
    echo $1 > /etc/hostname
    echo "127.0.1.1	$1.localdomain	$1" >> /etc/hosts
}
```

### Timezones

View your time info:

```bash
timedatectl
```

List all regions:

```bash
find /usr/share/zoneinfo/. -maxdepth 1 -type d | cut -d "/" -f6 | sed '/^$/d'
```

Or, select your timezone interactively:

```bash
tzselect
```

Then set your timezone:

```bash
timedatectl set-timezone Asia/Makassar
```

### date

Print current date formatted:

```
date '+%d %h %H:%M'
```

Set current date:

```
date --set="23 June 1988 10:00:00"
```

Or time:

```
date --set="10:00:00"
```


## Building Custom Arch ISO

#### Step 1: Create Chroot Base System

```bash
$ pacman -S devtools git make --needed
$ mkarchroot /tmp/chroot base
$ git clone git://projects.archlinux.org/archiso.git
$ make -C archiso/archiso DESTDIR=/tmp/chroot install
$ arch-chroot /tmp/chroot
```

#### Step 2: Customize

```
mknod /dev/loop0 b 7 0
echo 'Server = http://ftp.osuosl.org/pub/archlinux/$repo/os/x86_64' >> /etc/pacman.d/mirrorlist
pacman -S devtools libisoburn squashfs-tools
```

And install necessary packages.

#### Step 3: Build The ISO

```
cp -r /usr/share/archiso/configs/baseline /tmp
cd /tmp/baseline
./build.sh
exit
```

Now, you will be out of your chroot, with the built ISO at the file path of:

```
/tmp/chroot/tmp/baseline/out/<iso>
```

## How to build AUR package?

* Obtain `PKGBUILD`
* Run `makepkg -Acs`
* Install from the tarball running `sudo pacman -U x.pkg.tar.xz`

## Useful Commands

#### chroot
Opens a new TTY on given root directory.

#### dialog
You can create UI dialogs with it on command-line easily. For example;

  ```bash
  usernameDialog () {
      username=$(dialog --stdout \
                        --title "Creating Users" \
                        --backtitle "Happy Hacking Linux" \
                        --ok-label "Done" \
                        --nocancel \
                        --inputbox "Choose your username" 8 50)
  }
  ```

#### fc-list
Lists fonts available in the system.

```bash
fc-list | grep Monaco
```

#### nl
Adds line numbers to beginning of each line.

```bash
cat foobar.txt | nl
```

#### powertop
Lists processes by their energy consume.

#### yes
Approve all confirmations

  ```bash
  yes | pacman -S yolo
  ```

#### du
Checks size of a folder.

  ```bash
  du -sh /
  ```

#### ncdu
Disk usage analyzer with CLI UI with ncurses.

#### expac
Data extraction tool for ALPM (Arch Linux Package Management).

To list packages by their size;
  ```bash
  expac "%n %m" -l'\n' -Q $(pacman -Qq) | sort -rhk 2 | less
  ```

  ```
  yes | pacman -S yolo
  ```

#### ip

List network interfaces in the system:

```bash
ip link show
```

Manage routing tables:

```
ip r
```

#### pkill

Kill a process partially matching given pattern;

```
pkill -f pattern
```

#### sed

Uncomment matching line:

```bash
sed -i -e '/^#en_US/s/^#//' /etc/locale.
```

Slugify a string:

```bash
sed -e 's/[^[:alnum:]]/-/g' | tr -s '-' | tr A-Z a-z
```

#### awk

Select particular columns and print out;
```bash
lsblk | awk '{print $1,$4}'
# sda 8GB
```

#### file

Returns file info. It's especially useful on images;

```bash
file logo.png
# PNG image data, 16 x 16, 8-bit/color RGBA, non-interlaced
```

#### date

Get unix timestamp:

```
date +%s
```

Get date & time nicely formatted;

```
date '+%d %h %H:%M'
# 22 Jun 01:42
```

#### Tmux

On command-line:
* `tmux a -t [name]` Open a session
* `tmux ls` List available sessions

On session:
* `C-z $` Rename session
* `C-z d` Detach session

Windows

* `S-[left]` Select the next window
* `S-[right]` Select the previous window
* `C-[left]` Move window to left
* `C-[right]` Move window to right
* `C-z c` Open new window
* `C-z x` Close current window
* `C-z C-z`  Open last window
* `C-z :swap-window -t -1` Move current window to left
* `C-z ,` Rename window
* `C-z n` Next window
* `C-z p` Previous window

Status Bar

* `C-z b` Toggle status bar

Other

* `C-z :source-file ~/.tmux.conf` Reload config

#### htop

Interactive process viewer. Useful keybindings:

* `/` Search
* `\` Filter
* `,` Choose the sorting criteria
* `k` Send kill signal to selected process
* `u` Filter results by user
* `t` Open/close tree mode
* `-` or `+` Collapse/uncollapse process trees
* `H` Turn off displaying threads

#### iotop

Sorts processes by disk writes, and show how much and how frequently programs are writing to the disk.

#### nethogs

htop for network. lists processes by their network traffic.

#### systemctl

Use `systemctl` command to control the services. Example;

```bash
systemctl restart slim.service
```

Some useful `systemctl` commands:
* start
* stop
* restart
* reload
* status
* is-enabled
* enable
* disable
* mask
* unmask
* list-units

Restart `systemd` scanning for new or changed units;

```bash
systemctl daemon-reload
```

Toggle a service with one-liner:

```
if [[  "`systemctl is-active NetworkManager`" != "active" ]]; then sudo systemctl start NetworkManager; else sudo systemctl stop NetworkManager; fi

```

See which units are failing;

```bash
systemctl list-units --state=failed
```

See logs of a specific unit;

```bash
sudo journalctl -xu dhcpcd@enp0s3.service --since today
```

#### systemd-analyze

Analysize the boot time:

```bash
$ systemd-analyze
```

List the started unit files, sorted by the time each of them took to start up:

```bash
$ systemd-analyze blame
```

See boot units as chain:

```bash
$ systemd-analyze critical-chain
```

#### tee

It's used for splitting the output of a program so we can both display it and also save it.

For example, add a new entry to hosts file;

```
echo "127.0.0.1 foobar" | sudo tee -a /etc/hosts
```

#### find

List files by extension;

```
find . -type f -name *.strings
```

Using `-or`:

```
find . -type f \( -name "*.strings" -or -name "*.txt" \)
```

Executing a command per file;

```
find . -type f -name *.strings -exec sed -i '' -e "s/foo/bar/" {} \;
```

### tree

Lists contents of a directory in tree-like format.

```
tree
```

Show hidden files:

```
tree -a
```

Show only directories:

```
tree -d
```

### xev

Creates a window and lets you see the keyboard events. Useful when you modify keybindings.

### setxkbmap

Sets the keyboard layout:

```
setxkbmap tr -variant alt -option lv3:ralt
```


### pidof

Find process id of a running program:

```
pidof nginx
```

It might return multiple pids (e.g nginx have worker processes). Specify `-s` parameter to get only one pid:

```
pidof -s nginx
```
