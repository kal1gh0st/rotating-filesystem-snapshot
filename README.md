# rotating-filesystem-snapshot
How do filesystem snapshots work on Linux?
To perform valid backups of your database, it is important to suspend the database. This prevents modifications of files during the backup process. By taking a point-in-time snapshot of your database files, your backup program will be capturing a “frozen” database instead of an “in motion” database.

Our standard backup script uses database suspension with snapshots to create point-in-time images of your database files. The snapshot script itself is located at /u2/UTILS/bin/snapsave_linux.sh (with a symbolic link at /bin/save for backwards compatibility).

The snapshot script is typically scheduled to run at regular intervals via crontab to create new filesystem snapshots.  Here’s an example of a snapshot backup script that is scheduled via crontab to run every night at 12:59 AM:

[root@eclipse ~]# crontab -l
59 0 * * * /u2/UTILS/bin/snapsave_linux.sh
After running the script, the snapshot filesystems are mounted under /snap, allowing read-only access by backup software. For example, the snapshot of the /u2/eclipse/LEDGER file would be located at /snap/u2/eclipse/LEDGER. When configuring backup software, it is recommended to backup every file under /snap/u2.

Since every change (delta) between the snapshot and the “live” filesystem must be recorded, the snapshots have a finite lifespan. By default, the snapshot script is configured to hold 1GB of changes before requiring a refresh. On busier systems, or on systems where the snapshots must be retained for a longer period of time to accommodate a slow backup process, the snapshot volume size may be increased by editing the snapshot backup script. You may check the status of the snapshots using the “lvs” command, which shows a usage percentage for each snapshot volume.

[root@eclipse ~]# lvs
  LV       VG     Attr   LSize   Origin   Snap%  Move Log Copy%  Convert
  eclipse  datavg owi-ao  26.00G
  ereports datavg owi-ao   1.00G
  lvol0    datavg swi-ao   1.00G u2        45.85
  lvol4    datavg swi-ao   1.00G uvtmp      0.00
  lvol5    datavg swi-a-   1.00G ereports   0.00
  lvol6    datavg swi-ao   1.00G eclipse    0.85
  u2       datavg owi-ao   4.00G
  uvtmp    datavg owi-ao   4.00G
  esupport rootvg -wi-ao   6.00G
  root     rootvg -wi-ao  20.00G
  swap     rootvg -wi-ao   4.00G
When the Snap% value reaches 100%, the snapshot volume has reached its maximum capacity for tracking changes and must be recreated by running the snapshot script again.

For troubleshooting purposes, a log of the snapshot backup script is kept at /tmp/snapsave.log. Information regarding the creation, removal and expiration of snapshot LVs is also recorded in the system log (/var/log/messages).
