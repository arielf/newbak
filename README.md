# newbak - configurable backup utility

## TL;DR

newbak is a utility for automating multiple backups on Linux systems.

**Highlights:**

    - Highly configurable backups via human-readable config-files
    - Encrypted volumes (e.g. LUKS)
    - Incremental backups (via rsnapshot)
    - Scheduled backups (via cron)
    - Idempotency (preserve device state in the face of errors)
    - Friendly & informative error messages and usage
    - Good logging

## Usage

Usage example (fictional names and details).
The script prints the usage message when called with no parameters, like this.

```
Usage: newbak [backup] <friendly-name|UUID> [rotation]
       newbak <command> <friendly-name|UUID>

Commands:
  backup <target> [rotation]  Performs full [on], open, mount, rsnapshot, unmount, close, [off] (default command).
  open <target>               LUKS open (with optional [on]) only
  force-open <target>         Unmount and LUKS close (with optional [off])
  close <target>              Auto-unmounts the target if mounted, then LUKS close (with optional [off])
  fsck <target>               Perform file-system-check (fsck) on target
  on <target>                 Bring device online (turn on) if so configured (otherwise a no-op)
  off <target>                Take device offline (turn off) if so configured (otherwise a no-op)
  mount <target>              Mount volume (must be open)
  umount <target>             Unmount volume (must be open & mounted)
  help                        Show this guide.

Available Targets:
  pam            (UUID: 41255543-63a6-4320-aa14-738fb849187f, Mount: /media/pam)
  doggo          (UUID: 97912a84-f5d0-43d6-9b6a-626f43c77643, Mount: /media/doggo)
  sandisk64      (UUID: 1a78bd05-934b-4ad2-af63-d32f32cd828f, Mount: /media/sd64)
```

## Detailed overview

*newbak* is a single-file python script.
It relies on existing foundations (avoids reinventing the wheel) to do the heavy lifting.

The main benefit to the end user is simplicity of use.
Once a backup volume is configured, there's no longer a need to remember any of the details
that are required to do the myriad of operations needed to run an elaborate back-up to that volume.

`newbak` can be called either:

  - directly (run a backup of the current system into some specific backup volume right now)
  - automatically, via crontab, following any desired cadence

It supports incremental, periodic backups on Linux encrypted volumes.
It has extensive and very clear logging so that any issue can be easily investigated and fixed

You can configure all the basic building-blocks needed for a backup:

    - Bring a connected (but offline) physical device online (turn-on) or take it off-line (turn-off) on demand
    - Use a different encryption scheme. Tested using cryptsetup + LUKS
    - Use different mount/umount options
    - Call a different incremental-backup tool. Tested using rsnapshot.

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

## Example /etc/crypttab

For encrypted volumes, having a configured crypttab is recommended.
This way the device mapper will use your friendly names (consistent with `newbak-conf.yaml`) under `/dev/mapper`.

Here is an example.

You may keep some volumes online and pre-opened (by running `newbak open <volumename>`)
while others may remain closed (and will need a interactive-passphrase to open/unlock)

The `noauto` option is to avoid trying to open+auto-mount these volumes during boot time.

```
# <name>        <device>                                        <password>      <options>
evo860          UUID=81281072-90e1-0381-da71-238fd149100f       none            luks,noauto
evo850          UUID=ed9b8e54-5af0-3d46-bcd9-a2e8b7a15b91       none            luks,noauto
...

```

## Licence

Open source. BSD 2-clause (classic license)
Basically, do whatever you want with this code as long as you:

  - Give credit (don't claim it as your own)
  - Promise not to sue the author for damages
