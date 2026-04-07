this is from the picoGym's Forensics playlist, section II

all of these commands were run on a kali linux VM

i had to look up instructions for the one before this because i absolutely could not get the partitioning concept, every time i tried to use the plain filesystem commands in the kali linux sleuthkit documentation i was STRUGGLING
so, with this one, i knew to start with mmls 

```
$ mmls disk.flag.img
```
 which gives us
 
```
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000360447   0000153600   Linux Swap / Solaris x86 (0x82)
004:  000:002   0000360448   0000614399   0000253952   Linux (0x83)
```

now, as embarrassing as it sort of is to admit, because i knew the steps to search individual filesystems by offsetting but didn't quite grasp why they worked the way that they did, i spent a good amount of time rooting through the wrong filesystem
(semi-related side note, these kinds of challenges are a real lesson in remembering to use grep.....)

so, next step was to go to the kali linux docs. it seemed like fiwalk was the next step. that command gives you a very...very long list, since it works with all partitions
but there's a way around that :-)

```
$ fiwalk disk.flag.img | grep flag -B 1 -A 5
```
(flag -B is for lines before, flag -A is for lines after, because the inode info would not be in the same line as the filesystem name
and....

```
parent_inode: 3981
filename: root/my_folder/flag.txt
partition: 2
id: 4423
name_type: r
filesize: 42
alloc: 1
--
parent_inode: 3981
filename: root/my_folder/flag.uni.txt
partition: 2
id: 4424
name_type: r
filesize: 60
alloc: 1

```
i thought yay! home stretch!
except i really had no grasp of how inodes worked, so there was a lot of trial and error. i spent a while trying to icat with the parent inode. i eventually ran
```
$ fls -o 2048 disk.flag.img
```
```
d/d 11: lost+found
r/r 12: ldlinux.sys
r/r 13: ldlinux.c32
r/r 15: config-virt
r/r 16: vmlinuz-virt
r/r 17: initramfs-virt
l/l 18: boot
r/r 20: libutil.c32
r/r 19: extlinux.conf
r/r 21: libcom32.c32
r/r 22: mboot.c32
r/r 23: menu.c32
r/r 14: System.map-virt
r/r 24: vesamenu.c32
V/V 25585:      $OrphanFiles
```
and i was very confused, because wasn't root in the filepath? except there's no root here??
then i remembered that the mmls output was a few lines longer than the one of the previous challenge. i was just looking in the wrong partition!

```
$ fls -o 360448 disk.flag.img
```
```
d/d 451:        home
d/d 11: lost+found
d/d 12: boot
d/d 1985:       etc
d/d 1986:       proc
d/d 1987:       dev
d/d 1988:       tmp
d/d 1989:       lib
d/d 1990:       var
d/d 3969:       usr
d/d 3970:       bin
d/d 1991:       sbin
d/d 1992:       media
d/d 1993:       mnt
d/d 1994:       opt
d/d 1995:       root
d/d 1996:       run
d/d 1997:       srv
d/d 1998:       sys
d/d 2358:       swap
V/V 31745:      $OrphanFiles
```
look, there's root!!!
and, we know that there are two flag files. just to get the inodes for both, run it again with -r (so it searches recursively, aka recursively moves through each directory) (i know that's obvious for anyone with a science background but it was not for me lol)


```
$ fls -r -o 360448 disk.flag.img | grep 'flag' 
```
```
++ r/r * 2082(realloc): flag.txt
++ r/r 2371:    flag.uni.txt
```

with the inode number (the numbers after r/r) you can run icat, with offset, then the inode number
```
 icat -o 360448 disk.flag.img 2082 
```
```
           3.449677            13.056403
```
hm, that's strange. i'm going to be honest, i don't exactly know why that is, but i suspect it has something to do with the fact that the file was deleted (indicated by *)
no worries though, there was a second hit! ran it again, swapped out the inode, and there i was!
