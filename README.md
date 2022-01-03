# Overview

Btrfs is a great filesystem, but its userland tools are not very user-frienfly yet.
As a long-time user, I've developed `btrfs-list` as a wrapper to make sense out of the `btrfs sub list` and `btrfs qgroup show` commands.

You need `btrfs-list` if either:
- You'd like to have a nice tree-style overview of your subvolumes/snapshots
- You've already used ZFS before and you're missing the _zfs list_ command
- You're looking for exactly which snapshot to destroy to regain some space
- You're using `btrfs-progs` < v5.7 and you're looking for a more accurate estimation of how much space is remaining on your FS for all btrfs supported data profiles,
  as corner cases of raid1/raid10 are not handled well by older versions of `btrfs-progs`, and raid5/raid6 were not implemented at all

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
      --debug                enable debug output
  -q, --quiet                silence the quota disabled & quota rescan warnings
      --version              display version info
      --color=WHEN           colorize the output; WHEN can be 'never',
                               'always', or 'auto' (default is:
                               colorize if STDOUT is a term)
  -n, --no-color             synonym of --color=never
      --bright               use bright colors (better for dark terminals)
  -H, --no-header            hide header from output
  -r, --raw                  show raw numbers instead of human-readable
      --btrfs-binary BIN     path to the btrfs binary to use instead of using
                               the first binary found in the PATH

  -s, --hide-snap            hide all snapshots
  -S, --snap-only            only show snapshots
  -d, --deleted              show deleted parents of orphaned snapshots
      --snap-min-excl SIZE   hide snapshots whose exclusively allocated extents
                               take up less space than SIZE
      --snap-max-excl SIZE   hide snapshots whose exclusively allocated extents
                               take up more space than SIZE
  -f, --free-space           only show free space on the filesystem

  -p, --profile PROFILE      consider data profile as 'dup', 'single', 'raid0',
                               'raid1', 'raid10', 'raid5' or 'raid6', for
                               free space calculation (default: autodetect)

  -a, --show-all             show all information for each item
      --show-gen             show generation of each item
      --show-cgen            show generation at creation of each item
      --show-id              show id of each item
      --show-parent          show parent id of each item
      --show-toplevel        show top level of each item
      --show-uuid            show uuid of each item
      --show-puuid           show parent uuid of each item
      --show-ruuid           show received uuid of each item
      --show-otime           show snap creation time

  -w, --wide                 don't truncate uuids on output (this is the
                               default if STDOUT is NOT a term)
      --no-wide              always truncate uuids on output (useful to
                               override above default)

SIZE can be a number (in bytes), or a number followed by k, M, G, T or P.
```

# Examples

## Quick view

```
root@nas:~# btrfs-list /git
NAME                                  TYPE    REFER     EXCL  MOUNTPOINT
git                                     fs       -    58.96G (single, 17.27G free)
   [main]                          mainvol   60.61G  292.00k /git
   .beeshome                        subvol   56.64M   56.64M
   .snaps/20220103_005415_daily.0   rosnap   60.54G  144.00k
   .snaps/20220101_004815_daily.2   rosnap   60.42G  160.00k
   .snaps/20211229_004815_daily.5   rosnap   60.42G  160.00k
   .snaps/20220102_005203_daily.1   rosnap   60.42G  160.00k
   .snaps/20220103_114214_hourly.6  rosnap   60.54G  144.00k
   .snaps/20220103_154814_hourly.2  rosnap   60.61G  144.00k
   .snaps/20220103_175114_hourly.0  rosnap   60.61G  144.00k
   .snaps/20211230_004815_daily.4   rosnap   60.42G  160.00k
   .snaps/20211231_004815_daily.3   rosnap   60.42G  160.00k
   .snaps/20220103_144814_hourly.3  rosnap   60.61G  144.00k
   .snaps/20220103_164815_hourly.1  rosnap   60.61G  144.00k
   .snaps/20220103_124215_hourly.5  rosnap   60.54G    2.23M
   .snaps/20220103_134556_hourly.4  rosnap   60.60G  480.00k
   .snaps/20220103_104215_hourly.7  rosnap   60.54G  144.00k
```

## Detailed view

```
root@nas:/mnt/b# btrfs-list -qad .
NAME                                 ID PARENT TOPLVL GEN CGEN       UUID PARENTUUID  RCVD_UUID                OTIME    TYPE     EXCL  MOUNTPOINT
7cb8325f                              -      -      -   -    -          -          -          -                    -      fs  157.00M (raid5, 3.59G free, 2.74G unallocatable)
   [main]                             5      -      -   -    -          -          -          -                    - mainvol       -  /mnt/b
   sub1                             258      5      5  21    8 ae9c..6cae          -          -                    -  subvol       -
      sub1/.snap1                   259    258    258   9    9 2d50..2094 ae9c..6cae          -  2022-01-03 18:39:48    snap       -
      sub1/.snap2                   260    258    258  10   10 a2e9..1431 ae9c..6cae          -  2022-01-03 18:39:48    snap       -
      sub1/.snap3                   270    258    258  20   20 b054..6ce2 ae9c..6cae          -  2022-01-03 18:41:26    snap       -
      sub1/.snap4                   271    258    258  21   21 ae07..cc69 ae9c..6cae          -  2022-01-03 18:41:27    snap       -
   sub1/subsub1                     261    258    258  14   11 bdf1..e7fe          -          -                    -  subvol       -
      sub1/subsub1/.snapA1          262    261    261  12   12 ab9b..e6df bdf1..e7fe          -  2022-01-03 18:40:09    snap       -
      sub1/subsub1/.snapA2          263    261    261  13   13 407c..1a14 bdf1..e7fe          -  2022-01-03 18:40:09    snap       -
   sub2                             265      5      5  19   15 bc35..8104          -          -                    -  subvol       -
   sub3                             266      5      5  17   16 eb52..da06          -          -                    -  subvol       -
   sub3/subsub3                     267    266    266  32   17 eb81..00af          -          -                    -  subvol       -
      sub3-snaps/.snapK             280    279    279  30   30 41b9..442a eb81..00af          -  2022-01-03 18:43:25    snap       -
      sub3-snaps/.snapL             281    279    279  31   31 3308..f953 eb81..00af          -  2022-01-03 18:43:25    snap       -
   sub2/subsub2                     268    265    265  28   18 2189..08b4          -          -                    -  subvol       -
      sub2/subsub2/.snapB           269    268    268  19   19 259d..0afd 2189..08b4          -  2022-01-03 18:41:15    snap       -
      sub2/subsub2/.snapC           272    268    268  24   22 8ae4..1313 2189..08b4          -  2022-01-03 18:41:49    snap       -
         sub2/subsub2/.snapB-backup 294    268    268  24   24 fe27..2169 259d..0afd          -  2022-01-03 18:42:05    snap       -
         sub2/subsub2/.snapC-backup 274    268    268  24   24 da86..1964 8ae4..1313          -  2022-01-03 18:42:03    snap       -
      sub2/subsub2/.snapD           273    268    268  23   23 713f..6a28 2189..08b4          -  2022-01-03 18:41:50    snap       -
      sub2-snaps/.snapX             276    275    275  26   26 97ef..7187 2189..08b4          -  2022-01-03 18:43:04    snap       -
      sub2-snaps/.snapZ             278    275    275  28   28 4c08..f7fb 2189..08b4          -  2022-01-03 18:43:06    snap       -
   sub2-snaps                       275      5      5  28   25 3ce4..2ae3          -          -                    -  subvol       -
   sub3-snaps                       279      5      5  32   29 05c9..0620          -          -                    -  subvol       -
   sub4-snaps                       284      5      5  37   34 2366..b451          -          -                    -  subvol       -
   (deleted)                          -      -      -   -    - 05eb..d578          -          -                    - deleted       -
      sub4-snaps/sub4-bkp1          285    284    284  35   35 0134..77cb 05eb..d578          -  2022-01-03 18:45:11    snap       -
      sub4-snaps/sub4-bkp2          286    284    284  36   36 587b..e3e0 05eb..d578          -  2022-01-03 18:45:11    snap       -
      sub4-snaps/sub4-bkp3          287    284    284  37   37 55eb..e32e 05eb..d578          -  2022-01-03 18:45:12    snap       -
```

Note that the hierarchy here is the hierarchy between the subvolumes and snapshots, not the folder hierarchy.
This is why for example `sub3-snaps/.snapK` is under `sub3/subsub3`, because it is a snapshot of this subvolume,
even if in the folder hierarchy, it is under `sub3-snaps`.

Same goes for `.snapD` and `.snapX`, these are at a different spot in the folder hierarchy, but both are snapshots
of the `sub2/subsub2` subvolume, hence are placed under it.

We also have 3 snapshots of a `(deleted)` subvolume, these ghosts subvolumes are shown with the option `-d`.

## View free space of all btrfs filesystems at a glance

```
root@nas:/tmp/md5# btrfs-list -f
NAME     TYPE     EXCL  MOUNTPOINT
var        fs   18.09G (single, 5.62G free)
root       fs  950.39M (single, 36.61M free)
newtank    fs   15.61T (raid1, 764.30G free)
git        fs   58.96G (single, 17.27G free)
opt        fs    1.11G (single, 668.23M free)
incoming   fs   26.18T (single, 1.07T free)
7cb8325f   fs  157.00M (raid5, 3.59G free, 2.74G unallocatable)
home       fs   13.64G (single, 5.99G free)
slash      fs   19.80G (single, 11.70G free)
varlog     fs   12.26G (single, 3.96G free)
```

## Display heavy snapshots only

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

## Get accurate free space amount

Note: this is fixed with recent versions of `btrfs-progs` (v5.7 onwards), but we'll keep this feature to continue
supporting older releases of `btrfs-progs` if for some reason you're stuck with older versions.

For RAID5/6 setups, old versions of `btrfs filesystem usage` always display 0 bytes in the *Free (estimated)* section,
and you have no way to know the free space of your filesystem. `btrfs-list` handles this transparently by doing
the calculations needed to report the proper amount of free space, even in RAID5/6 setups.
