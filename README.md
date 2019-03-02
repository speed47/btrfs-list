# Overview

Btrfs is a great filesystem, but its userland tools are not very user-frienfly yet.
As a long-time user, I've developed `btrfs-list` as a wrapper to make sense out of the `btrfs sub list` and `btrfs qgroup show` commands.

You need `btrfs-list` if either:
- You'd like to have a nice overview of your subvolumes/snapshots
- You've already used ZFS before and you're missing the _zfs list_ command
- You're looking for exactly which snapshot to destroy to regain some space
- You're looking for a more accurate estimation of how much space is remaining on your FS

Basically it turns this:
![btrfs_sub_list](https://user-images.githubusercontent.com/218502/53362053-99564e00-3939-11e9-9072-1d9ef617971f.PNG)
into this:
![btrfs_list](https://user-images.githubusercontent.com/218502/53362048-965b5d80-3939-11e9-8e2f-8f92c7db79e4.PNG)

# Prerequisites
- `btrfs-progs` v3.18 at least (Dec 2014)
- The _quota_ feature enabled on your Btrfs filesystems (optional, to get space usage for subvolumes and snapshots)

# Usage

```
Usage: btrfs-list [options] [mountpoint]

If no [mountpoint] is specified, display info for all btrfs filesystems.

  -h, --help                 display this message
  -d, --debug                enable debug output
      --color=WHEN           colorize the output; WHEN can be 'never', 'always',
                               or 'auto' (default, colorize if STDOUT is a term)
      --no-color             synonym of --color=never

  -s                         hide all snapshots
      --snap-min-used SIZE   hide snapshots taking less space than SIZE
      --snap-max-used SIZE   hide snapshots taking more space than SIZE

      --show-all             show all information for each item
      --show-gen             show generation of each item
      --show-cgen            show generation at creation of each item
      --show-id              show id of each item
      --show-uuid            show uuid of each item

SIZE can be a number (in bytes), or a number followed by k, M, G, T or P.
```
