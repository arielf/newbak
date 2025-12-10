# newbak - configurable backup utility

## Usage

Usage example.
The script prints the usage message when called with no parameters.

```
Usage: newbak [backup] <friendly-name|UUID> [rotation]
       newbak <command> <friendly-name|UUID>

Commands:
  backup <target> [rotation]  Performs full open, mount, rsnapshot, unmount, and close (default command).
  open <target>               Only performs the LUKS open operation.
  force-open <target>         Closes any active system-created map on the device, then opens it with the configured friendly-name.
  close <target>              Auto-unmounts the target if mounted, then performs the LUKS close operation.
  help                        Shows this usage guide.

Available Targets:
  pam            (UUID: 41255543-63a6-4320-aa14-738fb849187f, Mount: /media/pam)
  doggo          (UUID: 97912a84-f5d0-43d6-9b6a-626f43c77643, Mount: /media/doggo)
  sandisk64      (UUID: 1a78bd05-934b-4ad2-af63-d32f32cd828f, Mount: /media/sd64)
```

## Overview

*newbak* is a single-file python script.
It relies on strong existing foundations (no wheel reinventions) to do the heavy lifting.

The main benefit is simplicity of use. Once a device is configured, there's no longer a need to remember any of the details that are required.

It supports incremental, periodic backups on Linux encrypted volumes.
It has extensive and very clear logging so that any issue can be easily investigated and fixed

You can configure it as much as you want.

want to use a different encryption scheme?
Use a different mount/unmount scheme?
Call a different incremental-backup tool? 

Every implementation detail is configurable.
Just edit `newbak-conf.yaml` and/or your `rsnapshot-<volume>.conf` to your liking.


## Configuration

When you use `LUKS` volume encryption, `mount/umount`, and `rsnapshot` you need 1 main configuration file,
plus another \<N\> config-files, (one for each specific volume backup):

  - `newbak-conf.yaml` - top level config of volumes
  - `rsnapshot.conf` - backup details: what to back-up, how many to keep etc. (one per volume)

See `man rsnapshot` for `rsnapshot` configuration details.  An example is provided here.

## Example crontab

Assuming devices are physically connected you may schedule your backups via cron.
Here's an example:
```
# -- 'hourly' backup: every 6 hours at xx:04
4 */6 * * * sudo newbak pam hourly
# -- 'daily' backup: @ 4:53 am
53 4 * * *  sudo newbak pam daily
# -- 'weekly' backup: Every Friday @ 3:32 am
32 3 * * 6  sudo newbak pam weekly
# -- 'monthly' backup: 1st of the month @ 2:21 am
21 2 1 * *  sudo newbak pam monthly
```

## Requirements (and tested with)

  - A modern Linux system
  - `cryptsetup` / luks (for full volume/disk encryption)
  - `rsnapshot` (swiss-army-knife for periodic, incremental backups, relies on `rsync` and `cp -al`)

## Licence

Open source. BSD 2-clause (classic license)
Basically, do whatever you want wth it as long as you:

  - Give credit (don't claim it as your own)
  - Promise not to sue the author for damages
