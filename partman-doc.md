<!--
SPDX-FileCopyrightText: 2020 Loetkolben
SPDX-License-Identifier: MIT
-->

# partman documentation

partman is the thing responsible for partitioning in the debian installer
with the help of so called recipes, which describe how the partitions should
be layed out.

Upstream documentation is available at
https://salsa.debian.org/installer-team/debian-installer/blob/master/doc/devel/partman-auto-recipe.txt

What follows below is an expansion / correction / summary of the above.
It is NOT complete, but only a snapshot of what I needed / though important
from my quest.

## Recipes
General Format (quoted from partman-auto-recipe.txt, original Copyright
2001-2009 by Joey Hess <joeyh@debian.org> and the d-i team, licensed under the
terms of the GNU GPL 2 or any later version)
```
In the following rules we denote spaces by "_".

<recipe>::=<header>_<partitions>

<header>::=<simple name>|<debconf name>

<simple name>::=<name>_:

<name> can be for example "Multi user system".

<debconf name>::=<debconf template>_::

The purpose of <debconf name> is to allow translation of the names of
the recipes into different languages.

<partitions>::=<partition>|<partition>_<partitions>

<partition>::=<limits>_<specifiers>_.

<limits>::=<minimal size>_<priority>_<maximal size>_<parted fs>
```

Sizes are all given in decimal MB, so 1 MB = 1000 * 1000 Byte and may be
rounded to "cylinder size". Max size may be -1 for "no limit".
The installer needs at least one partition of a very large / unlimited size
to fill the disk.

Sizes can also be given as percentages, with 100% typically equal to the amount
of RAM the machine has. This may (and at the time of writing currently is)
clamped with the debconf parameter `partman-auto/cap-ram` so that
100% is only equal to at most 1024MB (current value).

Priority is typically between minimal and maximal size, details about
the algorithm for choosing the real size are in partman-auto-recipe.txt,
but `priority - minimal` could be considered a weight.

Example with the following partition scheme:
- Partition for Grub2 to store its stuff when in use with GPT ("BIOS Boot
  partiton").  See https://en.wikipedia.org/wiki/BIOS_boot_partition and
  https://www.gnu.org/software/grub/manual/grub/html_node/BIOS-installation.html
- /boot partition (separate, if LVM)
- /root, partman can only create one subvolume @rootfs
- swap
```
bios-gpt-btrfs-atomic ::            
    1 1 1 free                          
        $iflabel{ gpt }                 
        $reusemethod{ }                 
        method{ biosgrub } .            
    512 1024 1024 ext4                  
        $defaultignore{ }               
        $primary{ }                     
        $bootable{ }                    
        label{ deb-boot }               
        method{ format }                
        format{ }                       
        use_filesystem{ }               
        filesystem{ ext4 }              
        mountpoint{ /boot } .           
    1000 10000 -1 btrfs                 
        $lvmok{ }                       
        lv_name{ rootfs }               
        label{ deb-root }               
        method{ format }                
        format{ }                       
        use_filesystem{ }               
        filesystem{ btrfs }             
        mountpoint{ / } .               
    100% 200% 300% linux-swap           
        $lvmok{ }                       
        lv_name{ swap }                 
        method{ swap }                  
        format{ } .                     
```

## parted fs
The following is a list of values that have been observed

- `$default_filesystem`, should go together with the `$default_filesystem{ }`
  specifier. Equivalent to `FS` as `<parted fs>` and `filesystem{ FS }` as
  specifier, with `FS` from debconf param `partman/default_filesystem`.
- `btrfs`
- `ext2`
- `ext3`
- `ext4`
- `free`: no filesystem (???), also used with the EFI partition
- `linux-swap`

## Specifiers
```
<specifiers>::=<specifier>|<specifier>_<specifiers>
```

Where specifier is always of the form `name{_value_}`.

Specifier names may start with `$` which typically means they are for partman
itself. Other specifiers are created as file `name` with content `value` in
some partition-specific directory.
partman-auto-recipe.txt calls the `$` specifiers 'internal' or 'type' and
the other specifiers 'regular'.

The changelogs are typically the place where these specifiers are documented.

List of `$` specifiers:
- `$bootable{_}`: set bootable flag on msdos, no idea what it does on gpt
  (probably nothing)
- `$default_filesystem{_}`: see parted fs type `$default_filesystem`
- `$defaultignore{_}`: Ignore this partition when LVM is NOT in use.
- `$iflabel{_<label>_}`: Only consider this partition when the label (partition
  table type, like gpt) is equal to `label`.
- `$lvmignore{_}`: Ignore this partition when LVM is in use.
- `$lvmok{_}`: Partition can be on LVM
- `$primary{_}`: primary partition on msdos, no idea what it does on gpt
  (probably nothing)
- `$reusemethod{_}`: for reusing existing EFI partitions

List of regular specifiers:
- `filesystem{_<name>_}`: "Specifies the filesystem to put on the partition."
  Allowed values are probably the parted fs types.
- `format{_}`: "Also needed to make the partition be formatted."
- `label{_<name>_}`: Filesystem label, if supported by the FS. Only works if
  `method{ format }` is also specified.
- `lv_name{_<name>_}`: Name this logical volume (that is created instead of a
  partition) `name`.
- `method{_<name>_}`: Name may be
  - `biosgrub` for BIOS boot partitions on GPT disks. See
    https://en.wikipedia.org/wiki/BIOS_boot_partition and
    https://www.gnu.org/software/grub/manual/grub/html_node/BIOS-installation.html
    for more info.
  - `crypto` to create an encrypted volume, with a LVM PV within it, the LVM PV
    partition is seemingly always placed last.
  - `format` marks the partition as to be formatted.
  - `keep` for no formatting.
  - `lvm` to use as LVM PV.
  - `swap` to create a swap partition.
- `mountpoint{_<where>_}`: "Where to mount the partition."
- `options/<name>{_<name>_}`: Mount options, for `nodev,ro` use
  `options/nodev{ nodev } options/ro{ ro }`.
- `use_filesystem{_}`: "Specifies that the partition has a filesystem on it."
