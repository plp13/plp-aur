Thea
====

Thea is a simple backup tool for desktop computers. It aims to provide
functionality similar to that of Apple's Time Machine, but does so primarily
through the command line (though some GUI tools are also provided). It allows
you to keep multiple backup versions of every file on your system in a
space-efficient way, and to compare with and recover from backup with ease.

Thea is written in shell language (zsh) and uses rsnapshot as its back-end.

Features
--------

Functional:

* Mount and unmount the backup volume from the GUI or the CLI
* Backup from the CLI, the GUI, or cron
* Summarize the backup status of any file or directory (CLI only)
* View the differences between a file/directory and one of its backed up
  versions (CLI only)
* Restore a file or directory from backup (CLI only)

Non-functional:

* Color output facilitates readability

History
-------

Thea began its life in November 2012 as a humble 10-line backup script for my
Arch Linux desktop computer.  It would mount my external USB drive, run
rsnapshot to back my system up, and then unmount it. Its functionality grew as
I kept customizing my desktop, and soon it was augmented by another pair of
scripts that would respectively mount and unmount the backup drive at the touch
of a button in my XFCE panel.

I soon realised that my scripts could easily become a full fledged backup
solution. All that was needed was to give them the ability to compare files
with their backed up versions, and to restore.  So, I gave my set of scripts a
name, organized them, and coded the required functionality.

The result was this very early version of Thea.

Requirements
------------

* rsnapshot (for backing up)
* zsh (the program is coded in shell language)
* yad (for displaying GUI dialogs)
* sudo (for gaining root privileges when needed)
* gksudo (GUI front-end to sudo)
* colordiff (for highlighting differences when comparing files with their
  backed up versions)

The program also assumes the existence of basic GNU/Linux command line
utilities, such as cp, mv, ls, column, etc.

Installation
------------

Simply get the sources from GitHub and run "make install" as root.

    % git clone git://github.com/plp13/thea.git
    Cloning into 'thea'...
    % sudo make install
    install -D -m 644 README.md /usr/local/share/doc/thea/README.md
    install -D -m 644 config /usr/local/share/thea/config
    install -D -m 644 functions /usr/local/share/thea/functions
    install -D -m 755 thea-config /usr/local/sbin/thea-config
    install -D -m 755 thea-backup /usr/local/bin/thea-backup
    install -D -m 755 thea-mount /usr/local/bin/thea-mount
    install -D -m 755 thea-umount /usr/local/bin/thea-umount
    install -D -m 755 thea-diff /usr/local/bin/thea-diff
    install -D -m 755 thea-summary /usr/local/bin/thea-summary
    install -D -m 755 thea-restore /usr/local/bin/thea-restore
    install -D -m 755 thea-gui-backup /usr/local/bin/thea-gui-backup
    install -D -m 755 thea-gui-mount /usr/local/bin/thea-gui-mount
    install -D -m 755 thea-gui-umount /usr/local/bin/thea-gui-umount

Configuration
-------------

1. Create your backup partition and create an entry in /etc/fstab for it. You
   can use any device or file system you want, provided it supports basic Unix
   file attributes and hard links (ie, don't expect NTFS to work correctly).
   Also, make sure you add "noauto" to your mount options, so that the backup
   volume won't be mounted on boot.

   As an example, let's assume that your backup volume is /dev/sdc1, uses btrfs
   (with LZO compression), and that want it to be mounted on /mnt/snapshots.

   The fstab entry will thus look like this:

        /dev/sdc1	/mnt/snapshots	btrfs	noauto,rw,relatime,compress=lzo

   You can make sure it works by typing:

        % mount /mnt/snapshots
        % df
        Filesystem  1K-blocks      Used  Available Use% Mounted on
        .
        .
        .
        /dev/sdc1   524288000 245872076  277345828  47% /mnt/snapshots

2. Make sure that you are a sudoer. Thea uses the sudo mechanism to gain root
   privileges in order to perform backup and restore operations.

   For more information on setting up sudo, see "man sudo" or consult your
   system's documentation.

3. Edit Thea's configuration file:

        % nano /usr/local/share/thea/config

   The config file sets a number of variables necessary for the operation of
   Thea's scripts. It also includes comments that explain each variable's
   purpose, so editing it should be self-explanatory.

   Most variables can be left to their default values, but make sure you set
   the following correctly:
   
   * BACKUP\_VOLUME: Set it to your backup partition's mount point, as defined
     in /etc/fstab.
   * RSNAPSHOT\_CONFIG: Set it to the real path of rsnapshot's configuration
     file on your system.
   * SUSPEND\_CMD, SUSPEND\_CMD\_ARGS, POWEROFF\_CMD, POWEROFF\_CMD\_ARGS: The
     default values for these will work fine for systems using systemd, but
     will need to be changed for systems that use sysv init. Check your
     system's documentation to find out which commands to use to suspend and
     shut down your computer.

   You may also want to set BACKUP\_PRE\_CMD and BACKUP\_POST\_CMD. These
   specify a command to run before the backup starts and after it's finished,
   respectively. In many cases, this is necessary for guaranteeing the
   integrity of backed up data; for instance, running "mysqldump
   --all-databases > /data.sql" before backing up will save a snapshot of all
   MySQL databases in /data.sql.

   Similarly, variables MOUNT\_PRE\_CMD and UMOUNT\_POST\_CMD let you execute
   custom commands before mounting and after unomunting the backup volume,
   respectively. This may be necessary for more complicated setups (i.e. ones
   that use LVM, Luks, or NFS over VPN).

   TIP: If you need to run multiple commands, put them in a script and have
   Thea execute it.

4. Run "thea-config" as root. This will automatically configure rsnapshot to work with
   Thea:

        % sudo thea-config
        * Moving original configuration file to /etc/rsnapshot.conf.orig
        * Writing configuration to /etc/rsnapshot.conf
        * Configuration written to /etc/rsnapshot.conf

   Note: this is the only one of Thea's commands you need to run as root, and
   you only need to run it once, after editing your configuration file.

That's all!

Usage
-----

Thea commands are quite simple and easy to understand. If you get lost, you can
always use "--help". For example:

    % thea-backup --help
    * Make a backup
    * Usage: thea-backup [options] [postact]
    * If postact is 'andsleep', the computer will be suspended after backing up
    * If postact is 'andpoweroff', the computer will be powered off
    * If postact is 'andpause', the user will be prompted to press ENTER to exit
    * Valid options are:
    * --help		Display this help message
    * --color		Use color output
    * --nocolor		Do not use color output
    * --quiet		Be quiet; do not print anything to standard output

### Backing Up

To back your system up, use "thea-backup":

    % thea-backup
    02/03/13 20:44:46: Starting backup
    02/03/13 20:44:46: Backup volume /mnt/snapshots is not mounted
    02/03/13 20:44:46: Attempting to mount
    02/03/13 20:44:47: Backup volume is now mounted
    02/03/13 20:44:47: Executing backup command (rsnapshot)
    echo 5054 > /var/run/rsnapshot.pid 
    /usr/bin/rm -rf /mnt/snapshots/daily.14/     
    mv /mnt/snapshots/daily.13/ /mnt/snapshots/daily.14/ 
    mv /mnt/snapshots/daily.12/ /mnt/snapshots/daily.13/ 
    mv /mnt/snapshots/daily.11/ /mnt/snapshots/daily.12/ 
    mv /mnt/snapshots/daily.10/ /mnt/snapshots/daily.11/ 
    mv /mnt/snapshots/daily.9/ /mnt/snapshots/daily.10/ 
    mv /mnt/snapshots/daily.8/ /mnt/snapshots/daily.9/ 
    mv /mnt/snapshots/daily.7/ /mnt/snapshots/daily.8/ 
    mv /mnt/snapshots/daily.6/ /mnt/snapshots/daily.7/ 
    mv /mnt/snapshots/daily.5/ /mnt/snapshots/daily.6/ 
    mv /mnt/snapshots/daily.4/ /mnt/snapshots/daily.5/ 
    mv /mnt/snapshots/daily.3/ /mnt/snapshots/daily.4/ 
    mv /mnt/snapshots/daily.2/ /mnt/snapshots/daily.3/ 
    mv /mnt/snapshots/daily.1/ /mnt/snapshots/daily.2/ 
    mv /mnt/snapshots/daily.0/ /mnt/snapshots/daily.1/ 
    mkdir -m 0755 -p /mnt/snapshots/daily.0/ 
    /usr/bin/rsync -a --delete --numeric-ids --relative --delete-excluded \
        --exclude=/mnt --exclude=/media --exclude=/proc --exclude=/sys \
        --exclude=/tmp --exclude=/run/user --exclude=mnt/snapshots \
        --link-dest=/mnt/snapshots/daily.1/mauretania/ /. \
        /mnt/snapshots/daily.0/mauretania/ 
    touch /mnt/snapshots/daily.0/ 
    rm -f /var/run/rsnapshot.pid 
    02/03/13 20:53:59: Backup complete
    02/03/13 20:53:59: Backup volume /mnt/snapshots is mounted
    02/03/13 20:53:59: Attempting to unmount
    02/03/13 20:54:12: Backup volume has been unmounted

The backup command takes one optional argument that specifies what to do when
backup is completed:

* "thea-backup andsleep" will suspend the computer
* "thea-backup andpoweroff" will shut the computer down
* "thea-backup andpause" will wait for the user to press ENTER before exiting
  (useful when running the backup command in its own X terminal)

### Mounting and Unmounting

To mount or unmount the backup volume, use "thea-mount" and "thea-umount"
respectively:

    % thea-mount
    * Backup volume /mnt/snapshots is not mounted
    * Attempting to mount
    * Backup volume is now mounted

    % thea-umount
    * Backup volume /mnt/snapshots is mounted
    * Attempting to unmount
    * Backup volume has been unmounted

You should always run "thea-mount" before using any backup recovery commands,
such as "thea-diff" and "thea-restore". And since it's preferable - for safety
reasons - to keep your backup volume unmounted except when needed, you should
also run "thea-umount" after you are done with them.

### Searching your Backups

To summarize the backup status of a file or directory, use "thea-summary
\<filename\>":

    % thea-summary README.md
    * File: /home/plp/Documents/Development/Other/Present/Thea/README.md
    * Size: 7190 bytes
    * File type: file
    * Status summary:
    Snapshot  Size in backup        Status
    --------  --------------------  ---------------------------------
    0         4544 bytes            different in backup
    1         0 bytes               not in backup
    2         0 bytes               not in backup
    3         0 bytes               not in backup
    4         0 bytes               not in backup
    5         0 bytes               not in backup
    6         0 bytes               not in backup
    7         0 bytes               not in backup
    8         0 bytes               not in backup
    9         0 bytes               not in backup
    10        0 bytes               not in backup
    11        0 bytes               not in backup
    12        0 bytes               not in backup
    13        0 bytes               not in backup
    14        0 bytes               not in backup

To view the differences between a file and one of its backed up versions, use
"thea-diff \<filename\> \<snapshot number\>":

    % thea-diff README.md 0
    * File: /home/plp/Documents/Development/Other/Present/Thea/README.md
    * Size: 8753 bytes
    * File type: file
    * Snapshot no: 0
    * Status: different in backup
    * Size in backup: 8750 bytes
    * Differences:
    121,225c121,123
    <     *  Moving original configuration file to /etc/rsnapshot.conf.orig
    <     *  Writing configuration to /etc/rsnapshot.conf
    <     *  Configuration written to /etc/rsnapshot.conf
    ---
    >     \*  Moving original configuration file to /etc/rsnapshot.conf.orig
    >     \*  Writing configuration to /etc/rsnapshot.conf
    >     \*  Configuration written to /etc/rsnapshot.conf

### Restoring from Backup

To restore a file to one of its backed up versions, use "thea-restore
\<filename\> \<snapshot number\>":

    % thea-restore thea-gui-backup-interactive 14
    * File: /home/plp/Documents/Development/Other/Present/Thea/thea-gui-backup-interactive
    * Size: 0 bytes
    * File type: weird file
    * Snapshot no: 14
    * Copying backed up version of file to disk
    * Restoration successful

### GUI Integration

Thea provides a number of commands that can be launched by buttons or menu
entries in your GUI's panel. They use yad and xterm to provide a simple
graphical interface to the backup, mount, and unmount operations.

Quick reference:
* "thea-gui-mount" will mount the backup volume (showing a progress dialog in
  the process)
* "thea-gui-mount andopen" will mount the backup volume and open it in the file
  manager
* "thea-gui-umount" will unmount the backup volume
* "thea-gui-backup" will perform a backup (after giving the user a short chance
  to abort)
* "thea-gui-backup andsleep" will perform a backup and then suspend the
  computer
* "thea-gui-backup andpoweroff" will perform a backup and then shut the
  computer down

Status
------

I've been using Thea for almost a year now. During this time I've found and
fixed several bugs, and none of them was threattening to the integrity of my
data. Though I don't consider the program to be stable, I think it's now mature
enough to be used by others who want to contribute to its development. 

Todo
----

* Test and fix bugs
* Add a new command, "thea-get", that returns information on the backup version
  of a file (filename, size, status, diffs) without extra information or
  colorization; useful for scripting
* Write a GUI program for restoring files (this will probably need to be done
  using a real language, such as C or Python)
* Integrate said GUI program with Thunar, Nautilus, etc.

Contact Me
----------

You can email me at p.panayiotou@gmail.com. Contributions are welcome.

License
-------

Copyright (c) 2013, Pantelis Panayiotou
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met: 

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer. 
2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution. 

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

(simplified BSD license)
