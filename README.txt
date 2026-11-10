NETSCALER PRE-UPGRADE TOOLS  (disk cleanup + upgrade jobs)
Update of 10 Nov 2026


WHAT'S NEW IN THIS UPDATE
==============================================================================
1. Disk cleanup now does what your team asked:
     - /var/nsinstall      deletes ALL .tgz files (and old build folders)
     - /var/log            deletes the old logs (rotated copies such as
                           ns.log.0.gz); the live log files are never touched
     - /var/ns_sys_backup  removes the old backup and creates a new one.
                           (/var/ns_sys/backup in your team's note; on
                           NetScaler the folder is /var/ns_sys_backup.)
                           For safety the new backup is created FIRST and the
                           old ones are deleted after, so there is always a
                           backup.
2. NEW: Upgrade jobs tool. It checks your HA pairs, creates the scheduled
   upgrade job in NetScaler Console, and checks the result after the upgrade.
3. Both tools are now in one package, NetScalerTools.zip. It replaces the
   earlier NetScalerCleanup.zip; you can delete the old folder.

This is a continuation of the disk cleanup tool you already have:

  Part 1  Disk cleanup    Run NetScaler Cleanup.bat
  Part 2  Upgrade jobs    Run NetScaler Upgrade.bat

Each part opens on its own. You can run Part 2 alone: if the cleanup is
already done, go straight to Part 2.


WHERE EVERYTHING IS
After you unzip NetScalerTools.zip you get one folder:

  NetScalerTools\
      Run NetScaler Cleanup.bat     <- Part 1: double-click to open disk cleanup
      NetScalerCleanup.ps1             (the cleanup tool itself, leave it here)
      Run NetScaler Upgrade.bat     <- Part 2: double-click to open upgrade jobs
      NetScalerUpgrade.ps1             (the upgrade tool itself, leave it here)
      reports\                         created on the first run; every run of
                                       either tool saves its .log and .csv here

Keep the files together in the same folder. Nothing needs to be installed.


GETTING STARTED
1. Unzip NetScalerTools.zip on your management PC.
   (If Windows says it blocked a file: delete the unzipped folder,
   right-click NetScalerTools.zip > Properties > tick "Unblock" > OK,
   then unzip again.)
2. Open the NetScalerTools folder.
3. Double-click the .bat file for the part you want (see below).


==============================================================================
PART 1  DISK CLEANUP           (Run NetScaler Cleanup.bat)
==============================================================================
Frees space in /var on each appliance, then checks it against the free space
the upgrade needs. It never starts an upgrade.

It cleans three places:

  /var/nsinstall      every installer file (.tgz/.tar/.gz) and build folder
                      (adc.version and installns_state files are always kept)
  /var/log            old logs: the rotated copies such as ns.log.0.gz,
                      ns.log.2 or messages.3.gz. The log files NetScaler is
                      writing to right now (ns.log, messages, ...) are never
                      deleted.
  /var/ns_sys_backup  the NetScaler system backup folder (the folder your team
                      called /var/ns_sys/backup). It first creates a NEW
                      system backup, then deletes the old ones. If the new
                      backup cannot be created, the old ones are kept, so
                      the appliance is never left without a backup.

How to run it:
1. Enter your NetScaler IPs, one per line (both nodes of each HA pair),
   and your NetScaler username and password (the password is never saved).
2. Under "What to clean", "/var/log: delete old rotated logs" and
   "/var/ns_sys_backup: replace backup" are ticked. Untick
   either one to skip it. The backup level can be "full" (default) or "basic".
3. Click "Test run" first. It lists every file it would delete in each folder
   and deletes nothing. Its /var estimate allows for the size of the new
   backup.
4. If the list looks right, click "Real run" and confirm.

Options:
  Keep the running build .tgz (rollback)
                      off by default, so all .tgz files go. Tick it to keep
                      the installer of the build running now and any newer
                      build (e.g. an upgrade image already copied there).
  Also keep these builds   builds to keep, e.g. 14.1-73.49
  Never delete             file names or patterns that must never be deleted
Only folders under /var are touched. /flash is never touched.
Note: if the upgrade image was copied into /var/nsinstall it is removed too;
the upgrade job uploads it again when it runs ("Upload at the time of
execution").

Results:  PASS   enough free space for the upgrade
          FAIL   still below the minimum, needs manual attention
          ERROR  could not connect or log in, see the details box


==============================================================================
PART 2  UPGRADE JOBS           (Run NetScaler Upgrade.bat)
==============================================================================
Does the same steps as Console > Infrastructure > Upgrade Jobs > Create Job,
for the HA pairs you tick.

1. Enter your NetScaler Console address (e.g. 10.150.10.104) and your Console
   username and password (never saved). Click "Load from Console".
2. Tick the HA pairs to upgrade and pick the upgrade image from the list.
   If the image is not on Console yet, click "Upload..." and choose the .tgz.
3. Click "Check". It only reads and creates nothing. Each HA pair shows:
     READY         good to upgrade
     NOT READY     something must be fixed first, the reason is in the details
     ALREADY DONE  already on that build or newer
4. Enter a job name and the time the upgrade should run, then click
   "Create job" and confirm. Only pairs that are READY go into the job.
5. After the upgrade window, click "Check after upgrade":
     PASS     both nodes on the new build, one Primary and one Secondary
     WAITING  the job has not finished yet
     FAIL     see the details box and Console > Upgrade Jobs for the reason

The job options match the Console screen: back up the instances before the
upgrade and remove the image from NetScaler after success are on by default;
save the configuration and keep primary/secondary roles are optional.

SAFETY
  - The job is always scheduled. The tool never upgrades right away.
  - Right after creating the job, the tool reads it back from Console for
    about 30 seconds. If the job is not waiting for the time you picked (for
    example it shows In Progress, or a different time), the tool deletes the
    job straight away and tells you. This is the "SAFETY STOP" message.
  - You can always stop or delete the job yourself in Console under
    Infrastructure > Upgrade Jobs before it runs.

PICK THE RIGHT IMAGE TYPE (important for NetScaler02)
Linux-based VPX appliances need the "_lx_" upgrade image. FreeBSD-based ones
need the "_nc_" image. NetScaler02 (10.150.10.101 / .102) is Linux-based (it
holds Update-NSVPXbuild-14.1-73.37_lx_64.tgz). For 14.1-73.49 please download
the _lx_ upgrade image for your hypervisor and use that, not
build-14.1-73.49_nc_64.tgz. The _nc_ image on a Linux-based appliance is the
most likely cause of Console's message "NetScaler upgraded from freeBSD
variants is required for downgrade back to freeBSD". "Check" warns about it.

GOOD TO KNOW
  - If a node is short on space, "Check" says so. Run Part 1, then check again.
  - If possible, run "Check" before 13 Nov. Your scheduled job
    FIPS13.1-37.286 is still in Console until then, and the tool copies the
    job format from it.
  - Console and the appliance run their own checks again when the job runs.
    If they stop the job, "Check after upgrade" shows FAIL and Console shows
    the reason under Upgrade Jobs.


REPORTS
Both tools save a .log and a .csv for every run in the "reports" folder.
Click "Open reports folder" (Part 1) or "Open reports" (Part 2).
If anything looks wrong, send us the reports folder.


HOW IT CONNECTS
Both tools use the built-in connection of NetScaler (the same one the web pages
use): Part 1 talks to the appliances, Part 2 talks to NetScaler Console. They
log in with your normal username and password, work only inside your network,
need no internet, and send nothing anywhere else.
