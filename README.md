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

Prerequisites:
- `btrfs-progs` v3.18 at least (Dec 2014)
- The _quota_ feature enabled on your Btrfs filesystems (optional, to get space usage for subvolumes and snapshots)
