# Linux Cheatsheet

**Highly recommended:** [tldr-pages/tldr](https://github.com/tldr-pages/tldr) ([website](https://tldr.sh))

> collection of community-maintained help pages for command-line tools, that aims to be a simpler, more approachable
> complement to traditional man pages.

---

- [Linux Cheatsheet](#linux-cheatsheet)
  - [OS-specific](#os-specific)
    - [CentOS](#centos)
    - [Ubuntu](#ubuntu)
  - [System-wide configuration](#system-wide-configuration)
    - [System Services](#system-services)
    - [Logs](#logs)
    - [Networking](#networking)
      - [SSH](#ssh)
    - [Env vars](#env-vars)
    - [Filesystem](#filesystem)
    - [Peripherals](#peripherals)
      - [Further Reading](#further-reading)
    - [Default editor](#default-editor)
    - [Disk Space](#disk-space)
    - [RAM](#ram)
    - [Swap Memory](#swap-memory)
  - [User-specific configuration](#user-specific-configuration)
  - [Git](#git)
  - [GPG](#gpg)
  - [Docker](#docker)
    - [Docker in Docker (DinD)](#docker-in-docker-dind)
    - [Update a Docker image](#update-a-docker-image)
  - [Image Manipulation](#image-manipulation)
  - [PDF manipulation](#pdf-manipulation)
  - [Optimization](#optimization)
    - [Parallel](#parallel)
    - [Time](#time)
    - [Copy dirs and files](#copy-dirs-and-files)
  - [String manipulation](#string-manipulation)
  - [Shell Scripting](#shell-scripting)
  - [Related Projects](#related-projects)

## OS-specific

### CentOS

- [Yum Command Cheat
  Sheet](https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf)

### Ubuntu

- Unattended reboot

  ```sh
  sudo editor /etc/gdm3/custom.conf

  # Uncomment the following lines and change user1 to the value of $USER:
  #   AutomaticLoginEnable = true
  #   AutomaticLogin = user1
  ```

- [Debian Unattended Upgrades](https://wiki.debian.org/UnattendedUpgrades)
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

## System-wide configuration

### System Services

- Print service definition

  ```sh
  systemctl cat SERVICE_NAME.service
  ```

### Logs

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

### Networking

- Connect to network on interface `eth0` on boot

  ```sh
  sudo systemctl status network
  sudo editor /etc/sysconfig/network-scripts/ifcfg-eth0
  # Set ONBOOT=yes for desired network
  ```

- Set DNS server

  ```sh
  editor /etc/resolv.conf
  # Add DNS server
  #   Example:
  #     nameserver 8.8.8.8
  ```

  [Source](https://developers.google.com/speed/public-dns/docs/using)

- List kernel routing tables

  ```sh
  route --numeric
  ```

#### SSH

- Generate SSH key

  ```sh
  # -t specifies the type of key to generate
  # -C provides a comment for the key
  ssh-keygen -t ed25519 -C "your_email@example.com"
  ```

  [Source](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

- Test passphrase for SSH key

  ```sh
  # -y "This option will read a private OpenSSH format file and print an OpenSSH public key to stdout."
  ssh-keygen -y -f <your_private_ssh_key>
  ```

- Regenerate remote host identification entry in `known_hosts` file

  ```sh
  ssh-keygen -f "~/.ssh/known_hosts" -R <IP_ADDRESS>
  ```

- [SSH Message Numbers](https://www.iana.org/assignments/ssh-parameters/ssh-parameters.xhtml#ssh-parameters-1)

  Examples:

  > send packet: type 50 receive packet: type 51

### Env vars

- Given a file named `example.env` with contents:

  ```sh
  a='jane'
  b='john'
  c='doe'
  ```

  - Source all variables:

    ```sh
    source example.env
    ```

  - Export all variables in file

    ```sh
    # After sourcing all variables, we can export them
    export $(cut --delimiter= --fields=1 example.env)
    ```

    [Source](https://unix.stackexchange.com/a/79065)

### Filesystem

- Filesystem Hierarchy Standard (FHS)

  > The Filesystem Hierarchy Standard (FHS) defines the directory structure and directory contents in Linux
  > distributions. [Source](https://en.m.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

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

- Create sequence of numbered directories

  ```sh
  # Creates directories 1/ ... 10/
  mkdir -p $(seq 1 10)
  ```

### Peripherals

- Cheatsheet from `opensource.com`: [Linux commands to display your hardware
  information](https://opensource.com/article/19/9/linux-commands-hardware-information)

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

    > usb-devices is a (bash) shell script that can be used to display details of USB buses in the system and the
    > devices connected to them.

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

    > usbview provides a **graphical** summary of USB devices connected to the system. Detailed information may be
    > displayed by selecting individual devices in the tree display.

    ```sh
    usbview
    ```

    [Source](https://linux.die.net/man/1/usb-devices)

#### Further Reading

- [Interpreting the output of lsusb](https://diego.assencio.com/?index=1363692dafeabeff8e3f975077f92dfe)
- [Find USB device details in Linux/Unix using LSUSB
  command](https://www.linuxnix.com/find-usb-device-details-in-linuxunix-using-lsusb-command/)
- [USB Descriptors](https://www.beyondlogic.org/usbnutshell/usb5.shtml)

### Default editor

- Set default editor for `sudo systemctl edit --full SERVICE_NAME`

  ```sh
  sudo update-alternatives --config editor
  ```

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

- Configure swap memory (run as `root` user)

  ```sh
  configure_swap_memory() {
    local -r swap_file_path="$1"
    local -r swap_memory_size_in_gb="$2"

    set -o errexit
    set -o pipefail
    set -o nounset

    echo 'Disable all swap memory temporarily'
    swapoff --all

    # ATTENTION: fallocate is faster than dd but some systems require dd usage.
    # If the fallocate command below fails, use the following instead:
    #
    # size=$((swap_memory_size_in_gb * 1024))
    # sudo dd if=/dev/zero of="$swap_file_path" count=$size bs=1MiB
    #
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

- [Recommended size of swap space](https://opensource.com/article/18/9/swap-space-linux-systems)

## User-specific configuration

- Set locale

  Follow the instructions below to fix the following warnings:

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

- Print non-root username

  ```sh
  # Returns name of user with UID 1000 (defaults to non-root user)
  uid=1000
  getent passwd "$uid" | cut -d: -f1
  ```

  [Source](https://unix.stackexchange.com/a/36582)

  Alternatively:

  ```sh
  # "SUDO_USER Set to the login name of the user who invoked sudo."
  echo $SUDO_USER
  ```

  [Source](https://man7.org/linux/man-pages/man8/sudo.8.html)

## Git

- Delete local and remote tag

  ```sh
  # Delete local tag
  git tag --delete <TAG_NAME>
  # Delete remote tag
  git push --delete origin <TAG_NAME>
  ```

- Rename local and remote branch

  ```sh
  # Placeholders: old_name new_name

  # If old_name not checked out already
  git checkout old_name

  # Rename local branch
  git branch -m old_name new_name

  # Delete old_name remote branch and push new_name local branch
  git push origin :old_name new_name

  # Reset upstream branch fo new_name local branch
  git push origin --set-upstream new_name
  ```

  [Source](https://stackoverflow.com/questions/30590083/how-do-i-rename-both-a-git-local-and-remote-branch-name/45561865#45561865)

- List changed files on branch with regard to main branch

  ```sh
  git --no-pager diff --name-only FETCH_HEAD $(git merge-base FETCH_HEAD main)
  ```

  [Source](https://stackoverflow.com/questions/25071579/list-all-files-changed-in-a-pull-request-in-git-github/25071749#25071749)

- Improve performance of `git clone`

  Read GitHub's [performance
  comparison](https://github.blog/2020-12-22-git-clone-a-data-driven-study-on-cloning-behaviors/) for `git clone` and
  [introduction to shallow and partial
  clone](https://github.blog/2020-12-21-get-up-to-speed-with-partial-clone-and-shallow-clone/#:~:text=git%20clone%20%2D%2Dfilter%3Dtree,need%20access%20to%20commit%20history.)

- Solve merge conflict

  ```sh
  git mergetool
  ```

- Check performance metrics and recommendations for git repo using [git-sizer](https://github.com/github/git-sizer)

  ```sh
  git-sizer --verbose
  ```

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

- Git pull all repos in a directory

  ```sh
  find . -type d -depth 1 -exec git --git-dir={}/.git --work-tree=$PWD/{} pull \;
  ```

  [Source](https://stackoverflow.com/a/12495234/2227405)

## GPG

- List all keys in long-format

  ```sh
  gpg --list-secret-keys --keyid-format LONG
  ```

- Generate a new GPG key pair

  ```sh
  gpg --full-generate-key
  # Follow instructions from link below
  ```

  [Source](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key)

- Delete GPG key pair

  ```sh
  # Delete secret key
  gpg --delete-secret-key <key-ID>
  # Delete public key
  gpg --delete-key <key-ID>
  ```

- Renew expired yarn GPG key

  ```sh
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
  ```

  [Source](https://github.com/yarnpkg/yarn/issues/7866#issue-558663837)

## Docker

- Watch status of Docker containers

  You may need to install the `watch` command depending on your Linux distribution.

  ```sh
  # Interval unit is seconds
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

- Find absolute path to Docker volume in file system

  ```sh
  docker volume inspect --format '{{ .Mountpoint }}' VOLUME_NAME
  ```

- Search local Docker images by name

  ```sh
  # Replace placeholder IMAGENAME by Docker image name you are searching for
  docker images '*IMAGENAME*'
  ```

### Docker in Docker (DinD)

The trick is to mount `/var/run/docker.sock` as a volume. The Docker container can then access Docker on the host.

[Source](https://itnext.io/docker-in-docker-521958d34efd)

### Update a Docker image

The following image uses the image `my_image:my_tag` as a placeholder.

```sh
# Replace placeholder image_name with the name of the image you want to update
image_name=my_image:my_tag
container_name=update-image-container

# Start a container with the image to update
docker run --rm --tty --detach --name "$container_name" "$image_name"

# Log into the image to update it
docker exec --interactive --tty "$container_name" bash
# Update the image (the following command is executed inside the image). Example:
touch /tmp/new_file.txt
# Exit the image
exit

# Commit the image, i.e., create a new image from the container’s changes
docker commit "$container_name" "$image_name"

docker stop "$container_name"
```

## Image Manipulation

[Source](https://tldr.ostera.io/convert)

- Horizontally append images

  ```sh
    convert *.png +append horizontal-image.png
  ```

- Vertically append images

  ```sh
    convert *.png -append vertical-image.png
  ```

- Create image with white background

  ```sh
  convert -size 32x32 xc:white empty.jpg
  ```

- Export multiple PNG images to PDF using [img2pdf](https://gitlab.mister-muffin.de/josch/img2pdf)

  ```sh
  # Enter directory containing PNGs
  cd directory-with-images/

  # Remove alpha from images and convert colorspace to RGB (img2pdf does not support ICC)
  for i in *.png; convert $i -colorspace rgb -alpha off $i; end

  # Merge images into PDF
  # (Optional) Add --pagesize A4 to normalize page size
  img2pdf --output merged.pdf *.png

  # (Optional) To add OCR text layer, execute the following line
  # ocrmypdf --force-ocr merged.pdf merged.pdf
  ```

## PDF manipulation

- Add OCR text layer to PDF using [ocrmypdf](https://github.com/jbarlow83/OCRmyPDF)

  ```sh
  ocrmypdf --force-ocr input.pdf output.pdf
  ```

- Merge PDFs

  ```sh
  pdfunite in-1.pdf in-2.pdf out.pdf
  ```

- Reduce PDF file size

  Requires [`gs`](https://www.ghostscript.com/) and [`shrinkpdf`](http://www.alfredklomp.com/programming/shrinkpdf/) installation

  ```sh
  shrinkpdf in.pdf out.pdf
  ```

- Search for text in PDF using [pdfgrep](https://pdfgrep.org/)

  ```sh
  pdfgrep pattern file.pdf
  ```

## Optimization

### Parallel

- Run command in parallel using GNU parallel

  ```sh
  parallel --halt-on-error now,fail=1 'set -o errexit; set -o pipefail; set -o nounset; echo {}' ::: 1 2 3'
  ```

  [Parallel manual](https://www.gnu.org/software/parallel/parallel_tutorial.html)

### Time

- Schedule command execution at specific time and date

  ```sh
  at hh:mm
  ```

- Store output of time command in variable without output of argument

  ```sh
  # Format output of time command
  # Source: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-TIMEFORMAT
  TIMEFORMAT='Elapsed time in seconds: %lR'

  # Quoting from http://mywiki.wooledge.org/BashFAQ/032:
  #   Keep stdout untouched.
  #   The shell's original file descriptor (FD) 1 is saved in FD 3, which is inherited by the subshell.
  #   Inside the innermost block, we send the time command's stdout to FD 3.
  exec 3>&1
  # Captures stderr and time.
  elapsed_time=$( { time ls 1>&3; } 2>&1 )
  exec 3>&-
  echo "$elapsed_time"
  ```

  [Source](http://mywiki.wooledge.org/BashFAQ/032)

### Copy dirs and files

- Display overall progress in rsync

  ```sh
  # Use rsync options --info=progress2 and --no-inc-recursive
  # Example:
  cd /tmp/bar && find . -type -f | parallel --halt-on-error now,fail=1 -X rsync --relative--no-inc-recursive --info=progress2 --human-readable './{}' /tmp/bar/ ; }
  ```

- Tansfer large files efficiently

  ```sh
  # Options
  # WARNING: Add --compress to RSYNC_OPTS only if transferring over a slow connection.
  # For local transfers --compress actually slows down the operation
  readonly RSYNC_OPTS=(--hard-links --archive --relative --partial '--info=progress2' --human-readable)
  readonly RSYNC_EXCLUDES=(--exclude=archive
    --exclude=.git
    --exclude=.gitignore
    --exclude=.idea
    --exclude=.vagrant
    --exclude=__pycache__
    --exclude=*.swp
    --exclude=.vscode)
    # 0 jobs in parallel translates to as many as possible
    readonly NUMBER_OF_JOBS=0

  cd src-dir && find . -type f | parallel --jobs "$NUMBER_OF_JOBS" --halt-on-error now,fail=1 -X rsync "${RSYNC_OPTS[@]}" "${RSYNC_EXCLUDES[@]}" ./{} dest-dir/
  ```

  [Source](https://www.gnu.org/software/parallel/man.html#EXAMPLE:-Parallelizing-rsync)

## String manipulation

- Replace characters in filenames found in current working directory

  ```sh
  # Replace all occurrences of underscores in filenames of files in current working directory with a hyphen
  for f in *; do mv $f ${f//_/-}; done
  ```

## Shell Scripting

- Script template:

  - Robust and [portable](https://www.cyberciti.biz/tips/finding-bash-perl-python-portably-using-env.html) shebang
  - Recommended bash/shell options for scripts
  - Inherit shell options in subshells

    [Example](https://stackoverflow.com/a/20832592/2227405) [`SHELLOPTS`
    docs](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)

  ```sh
  #!/usr/bin/env bash
  #
  # Script description

  # Bash options
  # Inherit errexit in subshells (requires Bash 4.4+)
  shopt -s inherit_errexit

  # Shell options
  set -o errexit
  set -o pipefail
  set -o nounset

  # Export enabled shell options to subshells
  # Documentation:
  # https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html
  export SHELLOPTS
  ```

- [Bash FAQ](http://mywiki.wooledge.org/BashFAQ)

- [Bash Study Guide](https://fvue.nl/wiki/Bash)

- [Bash variables](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)

- Conditionally pass arguments to command

  ```sh
  declare conditional_rename=()
  if [ true ]; then
    # Including value of variable in case it's not empty
    conditional_rename=("${conditional_rename[@]}" --before-context=3)
  fi

  # Converts to grep --before-context=3 'sh' readme.md
  grep "${conditional_rename[@]}" 'sh' readme.md
  ```

## Related Projects

- [macos-cheatsheet](https://github.com/rodrigobdz/macos-cheatsheet)
- [modern-unix](https://github.com/ibraheemdev/modern-unix)
