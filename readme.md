# Linux Cheatsheet

## Table of Contents

1. Filesystem
2. Peripherals
3. System Resources
4. Git
5. Networking
6. User Configuration
7. Logs
8. Docker
9. System Services
10. Yum
11. Ubuntu

## Filesystem

- Filesystem Hierarchy Standard (FHS)

  > The Filesystem Hierarchy Standard (FHS) defines the directory structure and directory contents in Linux distributions. [Source](https://en.m.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

---

- List 5 largest files in folder and sort by size

  ```sh
  du --human-readable --summarize * | sort --reverse --human-numeric-sort | head --lines 5
  ```

- Report file system disk space usage

  ```sh
  df --human-readable
  ```

- Create filesystem on external disk

  ```sh
  # Create filesystem
  mkfs.ext4 /dev/vdb

  # Create mountpoint directory
  mkdir --parents /data

  # Mount external disk to /data
  mount /dev/vdb /data
  ```

- Mount from `/etc/fstab`

  ```sh
  mount --all
  ```

- List all partitions and their Universally Unique Identifier (UUID)

  ```sh
  sudo blkid
  ```

## Peripherals

As a general rule device shown in `/dev/sd*` are storage devices as opposed to the ones shown in `/dev/bus`.

- List connected system devices

  ```sh
  find /dev/sd*
  ```

- List available USB ports

  - Simple

    ```sh
    find /dev/bus/
    ```

  - Advanced

    ```sh
    ls /sys/bus/usb/devices/*
    ```

- Display info for specific USB device

  ```sh
  sudo lsusb -D /dev/bus/usb/001/005
  ```

- List block devices

  ```sh
  lsblk
  ```

- List the partition tables for the specified devices

  ```sh
  sudo fdisk --list
  ```

- List hardware

  ```sh
  sudo lshw
  ```

- List hardware (only input devices)

  ```sh
  sudo lshw -class disk -class input -short
  ```

- List USB devices

  - `usb-devices`

    > usb-devices is a (bash) shell script that can be used to display details of USB buses in the system and the devices connected to them.

    ```sh
    usb-devices
    ```

    [Source](https://linux.die.net/man/1/usb-devices)

  - `lsusb`

    > lsusb is a utility for displaying information about USB buses in the system and the devices connected to them

    ```sh
    lsusb
    ```

    [Source](https://linux.die.net/man/1/usb-devices)

  - `usbview`

    > usbview provides a **graphical** summary of USB devices connected to the system. Detailed information may be displayed by selecting individual devices in the tree display.

    ```sh
    usbview
    ```

    [Source](https://linux.die.net/man/1/usb-devices)

### Further Reading

- [Interpreting the output of lsusb](https://diego.assencio.com/?index=1363692dafeabeff8e3f975077f92dfe)
- [Find USB device details in Linux/Unix using LSUSB command](https://www.linuxnix.com/find-usb-device-details-in-linuxunix-using-lsusb-command/)
- [USB Descriptors](https://www.beyondlogic.org/usbnutshell/usb5.shtml)

## System Resources

### Disk Space

- Check available disk space

  ```sh
  df --human-readable / | awk 'END{print $4}' | cut --delimiter='G' --fields=1
  ```

### RAM

- Check available RAM

  ```sh
  free --total --gibi | awk '/Total:/ { print $2 }'
  ```

### Swap Memory

- Configure swap memory

  ```sh
  configure_swap_memory() {
    local -r swap_file_path="$1"
    local -r swap_memory_size_in_gb="$2"

    echo 'Disable all swap memory temporarily'
    swapoff --all

    echo "Create $swap_file_path of size ${swap_memory_size_in_gb}GB"
    fallocate --length "${swap_memory_size_in_gb}G" "$swap_file_path"

    echo "Set up a Linux swap area in file $swap_file_path"
    mkswap "$swap_file_path"

    echo "Enable file $swap_file_path for paging and swapping"
    swapon "$swap_file_path"

    echo "Setting file permissions to 0600 of $swap_file_path"
    chmod 600 "$swap_file_path"

    echo 'Verify increased size of swap memory'
    grep SwapTotal /proc/meminfo
  }

  configure_swap_memory '/swapfile' 8
  ```

## Git

- List biggest files in `.git` folder

  ```sh
  git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | sed --quiet 's/^blob //p' \
  | sort --numeric-sort --key=2 \
  | tail --lines 10 \
  | cut --characters=1-12,41- \
  | $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest
  ```

  [Source](https://stackoverflow.com/questions/9456550/how-to-find-the-n-largest-files-in-a-git-repository/46085465#46085465)

## Networking

- Connect to network on interface `eth0` on boot

  ```sh
  sudo systemctl status network
  sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0
  # Set ONBOOT=yes for desired network
  ```

- Set DNS server

  ```sh
  vim /etc/resolv.conf
  # Add DNS server
  #   Example:
  #     nameserver 8.8.8.8
  ```

  [Source](https://developers.google.com/speed/public-dns/docs/using)

- List kernel routing tables

  ```sh
  route --numeric
  ```

### SSH

- Regenerate remote host identification entry in `known_hosts` file

  ```sh
  ssh-keygen -f "~/.ssh/known_hosts" -R <IP_ADDRESS>
  ```

## User configuration

- Set locale

  > perl: warning: Setting locale failed.
  >
  > perl: warning: Please check that your locale settings:
  >
  > LANGUAGE = (unset),
  >
  > LC_ALL = (unset),
  >
  > LC_CTYPE = "UTF-8",
  >
  > LANG = "en_US.UTF-8"
  >
  > are supported and installed on your system.
  >
  > perl: warning: Falling back to the standard locale ("C").

  Append the following lines to `~/.bashrc`:

  ```sh
  export LANGUAGE=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  export LANG=en_US.UTF-8
  export LC_CTYPE=en_US.UTF-8
  ```

- Preserve env when executing `sudo`

  ```sh
  sudo --preserve-env <command>
  ```

## Logs

- Recover logs from failed `/etc/fstab` during boot

  - Search by **date**

    ```sh
    journalctl --since today
    ```

  - Search by **keyword**

    ```sh
    grep --ignore-case --regexp=KEYWORD --files-with-matches --dereference-recursive /var/log 2> /dev/null
    ```

  - Search by **file**

    ```sh
    journalctl --file /var/log/FILENAME.journal
    ```

## Docker

- Watch status of Docker containers

  You may need to install the `watch` command depending on your Linux distribution.

  ```sh
  watch --interval 1 'docker ps -a'
  ```

- Enter a crashing Docker container

  ```sh
  docker commit <container_id> my-broken-container
  docker run --interactive --tty my-broken-container /bin/bash
  ```

- Check cause of container crash

  ```sh
  dmesg --ctime | grep --extended-regexp --ignore-case --before-context=100 'killed process'
  ```

### Docker in Docker (dind)

The trick is to mount `/var/run/docker.sock` as a volume. The Docker container can then access Docker on the host.

[Source](https://itnext.io/docker-in-docker-521958d34efd)

## System Services

- Print service definition

  ```sh
  systemctl cat SERVICE_NAME.service
  ```

## Yum

- [Yum Command Cheat Sheet](https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf)

## Ubuntu

- Reinstall Unity on Ubuntu

  ```sh
  sudo apt-get update
  sudo apt-get install --reinstall ubuntu-desktop
  sudo apt-get install --reinstall unity
  ```

- Fix `mesg: ttyname failed: Inappropriate ioctl for device`

  Comment out `mesg n || true` in `/root/.profile`:

  ```sh
  # mesg n || true
  test -t 0 && mesg n
  ```

  [Source](https://superuser.com/a/1277604)
