# Overview

Btrfs is a great filesystem, but its userland tools are not very user-frienfly yet!
As a long-time user, I've developed `btrfs-list` as a wrapper to make sense out of the `btrfs sub list` and `btrfs qgroup show` commands.

You need `btrfs-list` if either:
- You'd like to have a nice overview of your subvolumes/snapshots
- You've already used ZFS before and you're missing the _zfs list_ command
- You're looking for exactly which snapshot to destroy to regain some space
- You're looking for a more accurate estimation of how much space is remaining on your FS for all btrfs supported data profiles, as corner cases of raid1/raid10 are not handled well by _btrfs-progs_, and raid5/raid6 are not implemented at all yet

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
  -q, --quiet                silence the quota disabled & quota rescan warnings
      --color=WHEN           colorize the output; WHEN can be 'never', 'always',
                               or 'auto' (default, colorize if STDOUT is a term)
  -n, --no-color             synonym of --color=never
  -H, --no-header            hide header from output

  -s, --hide-snap            hide all snapshots
  -S, --snap-only            only show snapshots
      --snap-min-excl SIZE   hide snapshots whose exclusively allocated extents
                               take up less space than SIZE
      --snap-max-excl SIZE   hide snapshots whose exclusively allocated extents
                               take up more space than SIZE
  -f, --free-space           only show free space on the filesystem

  -p, --profile PROFILE      consider data profile as 'dup', 'single', 'raid0',
                               'raid1', 'raid10', 'raid5' or 'raid6', for
                               realfree space calculation (default: autodetect)

      --show-all             show all information for each item
      --show-gen             show generation of each item
      --show-cgen            show generation at creation of each item
      --show-id              show id of each item
      --show-uuid            show uuid of each item

SIZE can be a number (in bytes), or a number followed by k, M, G, T or P.
```

# Examples

## Display heavy snapshots

```
root@nas:~# btrfs-list --snap-min-excl 4G --snap-only /tank
NAME                                                          TYPE     REFER      EXCL MOUNTPOINT
      backups/.snaps/skyline/20130213_231649_lastskyline    rosnap    22.52G    19.58G
      backups/.snaps/box/20171231_221207_monthly.12         rosnap    88.73G     4.96G
      backups/.snaps/box/20180130_221209_monthly.11         rosnap    91.25G     4.90G
      backups/.snaps/box/20180307_154215_monthly.10         rosnap    96.28G    10.72G
      backups/.snaps/box/20190120_193004_weekly.3           rosnap    56.45G     4.25G
      backups/.snaps/nasroot/20180122_091325_monthly.12     rosnap    34.65G    10.79G
      backups/.snaps/nasroot/20180221_092311_monthly.11     rosnap    31.96G     4.98G
      backups/.snaps/nasroot/20180323_092734_monthly.10     rosnap    33.69G     7.05G
      backups/.snaps/nasroot/20180820_205559_monthly.5      rosnap    31.74G     5.37G
      .syncthing-bkp                                        rosnap    40.48G     8.15G
```

## Display verbose information

```
root@nas:~# btrfs-list --show-all /mnt/a
NAME                             ID     GEN    CGEN                                 UUID     TYPE     REFER      EXCL MOUNTPOINT
203be355                         -1       -       -                                    -       fs         -   134.09M (868.78M free)
   [main]                         5       -       -                                    -  mainvol    16.00k    16.00k
   sub1                         256      23       6 7e5e30e0-4e68-da46-aa38-381048b0a794   subvol    33.02M    16.00k /mnt/a/test
      sub1/.snap1               257       7       7 c8b1b7c9-26e0-b346-858f-a37d346ce8b6     snap    16.00k    16.00k
      sub1/.snap2               259       8       8 dbf3e766-0a47-d24b-914e-246cc24f6b6a     snap    10.02M    16.00k
      sub1/subsub1              260      29       9 f3dfc28f-84a4-6a43-8e2c-c5163d134e52     snap    16.02M    16.00k /mnt/a
         sub1/subsub1/.snapA1   263      12      12 728b929f-8c33-d541-806d-94871eaa75a6     snap    47.02M    31.02M
         sub1/subsub1/.snapA2   265      14      14 45281588-176e-7e4c-9d9b-79375cfcb6f3     snap    47.02M    16.00k
         subroot                266      15      15 66a0de92-f6c3-c441-8b18-1110b5b89a0e     snap    47.02M    16.00k
         sub1/subsub1/.snapA3   268      17      17 6e19b614-e6b6-a945-9ea6-1857329d8d06     snap    16.02M    16.00k
      sub1/.snap3               261      10      10 196f1987-161e-a644-b9f9-abcf406bdabd     snap    26.02M    16.00k
      sub1/.snap4               262      11      11 41c86ef5-09ec-2449-aa70-d963430c7b1b     snap    49.02M    23.02M
      sub1/.snap5               264      13      13 7c53673b-c078-834e-97f3-1e07a2a6cf77     snap    49.02M    16.00k
      sub1/.snap6               267      16      16 6bfd05d5-bc7e-5d40-aeda-81764c6c5b2b     snap    33.02M    16.00k
      snapq222                  269      22      21 12005af8-92b4-0646-a4fa-4ff1480797ee     snap    33.03M    32.00k
      snapmulti                 270      23      22 628a9b53-003c-fb40-be89-eee5bbbbeb42     snap    33.03M    32.00k
```

## Get accurate free space amount

For some configurations, `btrfs filesystem usage` *Free (estimated)* section is misleading, for example in a 5-devices RAID1 setup with 4 devices of 133M and 1 device of 500M:

```
root@nas:~# btrfs-list /mnt/a
NAME          TYPE     REFER      EXCL MOUNTPOINT
8c4ca5e5        fs         -     0.00  (456.44M free) (418.00M realfree)
   [main]  mainvol    16.00k    16.00k /mnt/a
```

The *free* amount is reported by `btrfs filesystem usage`, the *realfree* is the adjusted amount by `btrfs-list`. Let's verify that we got it right:

```
root@nas:~# dd if=/dev/urandom of=/mnt/a/big
dd: writing to '/mnt/a/big': No space left on device
853114+0 records in
853113+0 records out
436793856 bytes (437 MB, 417 MiB) copied, 3.55206 s, 123 MB/s

root@nas:~# ls -lh /mnt/a/big
-rw-r--r-- 1 root root 417M Mar  2 19:31 /mnt/a/big

root@nas:~# btrfs-list /mnt/a
NAME          TYPE     REFER      EXCL MOUNTPOINT
8c4ca5e5        fs         -   416.56M (39.88M free) (320.00k realfree)
   [main]  mainvol   416.58M   416.58M /mnt/a
```
