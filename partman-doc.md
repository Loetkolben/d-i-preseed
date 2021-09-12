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
General Format (taken from partman-auto-recipe.txt)
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

Example from partman-auto-recipe.txt:
```
partman-auto/text/home_scheme ::

300 4000 7000 ext3
        $primary{ }
        $bootable{ }
        method{ format }
        format{ }
        use_filesystem{ }
        filesystem{ ext3 }
        mountpoint{ / } .

64 512 300% linux-swap
        method{ swap }
        format{ } .

100 10000 -1 ext3
        method{ format }
        format{ }
        use_filesystem{ }
        filesystem{ ext3 }
        mountpoint{ /home } .
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
- `$reusemethod{_}`: ???


List of regular specifiers:
- `filesystem{_<name>_}`: "Specifies the filesystem to put on the partition."
  Allowed values are probably the parted fs types.
- `format{_}`: "Also needed to make the partition be formatted."
- `label{_<name>_}`: Filesystem label, if supported by the FS. Only works if
  `method{ format }` is also specified.
- `lv_name{_<name>_}`: Name this logical volume (that is created instead of a
  partition) `name`.
- `method{_format|swap|keep_}`: `format` marks the partition as to be formatted.
  `swap` to create a swap partition. `keep` for no formatting.
- `mountpoint{_<where>_}`: "Where to mount the partition."
- `options/<name>{_<name>_}`: Mount options, for `nodev,ro` use
  `options/nodev{ nodev } options/ro{ ro }`.
- `use_filesystem{_}`: "Specifies that the partition has a filesystem on it."
