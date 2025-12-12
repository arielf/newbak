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
It relies on strong existing foundations (avoids reinventing the wheel) to do the heavy lifting.

The main benefit to the end user is simplicity of use.
Once a device is configured, there's no longer a need to remember any of the details that are required.

`newbak` can be called either:

  - directly (run a backup of the current system into some specific backup volume right now)
  - automatically, via crontab, following any desired cadence

It supports incremental, periodic backups on Linux encrypted volumes.
It has extensive and very clear logging so that any issue can be easily investigated and fixed

You can configure it as much as you want.

Want to use a different encryption scheme?
Use a different mount/unmount scheme?
Call a different incremental-backup tool? 

Implementation details are configurable.
Just edit `newbak-conf.yaml` and/or your `rsnapshot-<volume>.conf` to your liking.


## Configuration

There is one top-level configuration file: `newbak-conf.yaml`,
plus optional \<N\> `rsnapshot-*` configuration-files, (one for each specific volume backup).

  - `newbak-conf.yaml` - top level config of backup volumes, and underlying commands
  - `rsnapshot-<somename>.conf` - backup details: what content to back-up, how many periodic backups to keep etc.
    (this is a per-volume configuration, so if you want different contents backed-up, you can have multiple of these)

See `man rsnapshot` for the `rsnapshot` configuration details.
An example is provided in [rsnapshot-pam.conf](rsnapshot-pam.conf)

## Example crontab

Assuming devices are physically connected you may schedule your backups via cron.
Here's an example for a certain schedule running various periodic backups:
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
Note: you would need to configure `sudo` (see: *`man sudoers`*) in order to not require a password
when running `sudo` jobs via cron.

## Requirements (and tested with)

  - A modern Linux system
  - `cryptsetup` / luks (for full volume/disk encryption)
  - `rsnapshot` (swiss-army-knife for periodic, incremental backups, relies on `rsync` and `cp -al`)

## Licence

Open source. BSD 2-clause (classic license)
Basically, do whatever you want with this code as long as you:

  - Give credit (don't claim it as your own)
  - Promise not to sue the author for damages
