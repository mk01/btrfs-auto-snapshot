btrfs-auto-snapshot
===================

btrfs-auto-snapshot targets to provide all the functionality found in http://mk01.github.io/zfs-auto-snapshot. 
It is build on it's basis. Because Btrfs thinks of snapshot & volumes in a slightly different ways 
(snapshots / volumes and it's organization within Btrfs filesystem), one to one perception and style of work
(learned from ZFS) isn't directly possible. 

There btrfs-auto-snapshot scripts comes into play while having the functioniality of auto backup agent, it 
provides some kind of layer between btrfs and the user. It tries to clone the known commands of 'zfs' (related
to snapshots manipulation - list snapshots, volumes, snapshot command, destroy, clone, rename, promote, rollback).

Currently functional in testing is:

working 

1) listing volumes and snapshots
2) creating ad hoc snapshots (will not be touched by rotations) 
3) regular (planned snapshots)
4) destroy
5) snapshots rotation (rr, hanoi tower still to go)
6) rename

missing
7) promote
8) rollback
9) clone


Alltogether with the scripting itself some organization of btrfs subvolumes is needed. Btrfs is one big tree of 
namespaces and calling it subvolumes. A subvolume (B) created under namespace of (A) will not be visible after we create
snapshot (C) of (A), because (C) is lets say 'detached-copy' of (A), which is not containing (B). (B) is only
a subtree attached to a (A). 

One can think of this like 'mount' would be working. If we mount a dev2 under directory of mounted dev1, if we create 
a copy of dev1, mount dir will be empty and dev1 will not have data from dev2.

Working solutions is create empty btrfs filesystem (id=0), then create subvolumes (/, /home, /var...) and it's snapshots.
On top, copy data. Let's imagine this first snapshot as live (running) copy of the data - data we are working on.

```
btrfs-fs(id=0) -- ROOT / @running 
                        
                
                \ HOME / @running
                
                
                \ VAR / @running 
```                

New snapshot are created from @running, within the one btrfs subvolume tree which for us is making representation 
of analogue of ZFS filesystem.

```
btrfs-fs(id=0) -- ROOT / @running / [x]
                       / @snap1
                
                \ HOME / @running
                       / @snap2
                
                \ VAR  / @running 
                       / @snap_wrong_copy
```

Once we have such a structure, we can look onto the subvolumes (with name starting with '@') like onto snapshots 
and manipulate freely as with ZFS snapshots & filesystems.

Volumes must be mounted with mount option subvol=path_to/snapshot/@running (in this example).

 