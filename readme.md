# tadycon - tadyen's config
this is a very quick and dirty, poorly thought out spec for a config system

## 0 Goal
to automate syncing custom configs / overrides for a given package on a system.
this DOES NOT have anything to do with build systems, but the inspiration is from Arch PKGBUILD
it (may) expand to include package management to quickly bootstrap my workstation faik

still subject to lots of change.

uses `rsync`

name `tadyen` is just me, and i suck at naming. Since this is my desired solution compared to many many others' dotfiles, 
it would make sense for me to call it me. If this actually takes off (haha fat chance) then it can be renamed/refactored.

## 1 Overview
- these are `TOML` files and follow the toml spec
- for specifivity of this function, they shall be strictly named `tadyconf.toml`
- only 1 or none of it can exist per dir
- 1 file only applies to 1 application

## 2 Description

tbd properly ig.
- TBD: Support for root/sudo level config

### 2.1 Described values
assume snake_case for key value naming. CAPS is just for emphasis for now..
Stuff described:
    - Longname: full name of application eg. "neovim". This will probably be unused.
    - Name: shorthand name of application used for its config eg. "nvim"
    - Entries: Array of objects that follows the spec below
        - FROM dir: FROM is always in the parent directory. 
        - TO dir: TO is where the live config should live
        - SYNC TYPE
            - Options: 
                - MASTER (read-only, sync TO system)
                - SLAVE (write-only, sync FROM system)
                - DUPLEX (2-ways allows new files to be created with deletion, Default)
                - DUPLEX_SAFE (like duplex, but no deleting of files)

### 2.2 addtl properties/assumptions
- Copy/Syncing of directories is recursive.
- Config structure agnostic. There is no check nor description for how the configs are moved about.
- Assumes compliance with linux/posix [Filesystem Heirarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)


## 3 Example

``` toml
# tadycon.toml
# This is a comment.

[tadycon]
name = "nvim"
longname = "neovim"

[[tadycon.entry]]
from = "./.config"
to = "$HOME/.config"
match = "nvim"      # can use globbing, defaults to '*' to glob all contents in <from>
sync = "duplex"

[[tadycon.entry]]
from = "./.config"
to = "/root/.config"
sync = "master"     # can still be overwritten due to entry above

```

## 4 Edge cases / discussion
since this uses rsync, and file deletions do not have receipts without a VCS attached, there is no way of syncing delete operations.

this can be UNSAFE when bi-directionally syncing.

Also, VCS is IMPLIED MISSING since modifying configs on the fly and stuff does not engage a VCS, including system actions.

Thus: TBD create a backup system with delete receipts. We can assume the deletion time is based on the most recent change in the files, as a safer estimate.

Consider `foo: 1 day ago` and `bar: a few seconds ago`. If `bar` is rm'd, then the delete receipt for bar is created on the next sync as the following: 
`bar.deleted: 1 day ago`. If the reflecting dir has a `bar: half day ago`, then the most recent of the two is picked, in this case, `bar: half day ago` even though the deletion was more recent.

This is just a safety feature, because it is implied that the most recent dir that was worked on will have the most recent update, and flat-out deleting is NOT-CONSIDERED work (otherwise rm -rf will kill all).


