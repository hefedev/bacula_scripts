## bacula_scripts

This package comes with various bacula and bareos compatible scripts. Scripts were mainly tested
with bareos.

#### Install
You can install this package with `pip3 install bacula_scripts`
Python dependencies: helputils, gymail, psycopg2, lark-parser
Distro dependencies: Both bacula and bareos come with the tool `bls`, i.e. install
bacula-/bareos-tools on your distro.

#### Configuration
See the example configs in etc/bacula_scripts and modify for your needs. general_conf.py is
used by multiple scripts.


#### Usage

##### usage: bacula_stats [-h] [-a] [-r] [--version]

bacula_stats 0.1.1 - Display recent and all backups.

optional arguments:
  -h, --help    show this help message and exit
  -a, --all     Return all backups.
  -r, --recent  Return recent backups
  --version     show program's version number and exit


##### usage: bacula_del_jobs [-h] [-d] [-dry]

 bacula-del-jobs.py

WARNING! Use with caution.

Delete all catalog entries that are associated to the given storage_name and their volume file
on the disk. Delete volumes that match the storage name or the job names.
Run this script only when you really want to delete something specifically.
This script doesn't work with remote storage devices.

If you need to remove backups from your dropbox encfs storage, then mount the encfs
storage. Use: `/usr/bin/bacula_encfs_backup mount`, which mounts it for example to /mnt/b01,
see /etc/bacula-scripts/bacula_encfs_backup_conf.py. bacula-del-jobs.py will then also remove
backups from /mnt/b01. Important! Unmount it afterwards, because the bacula user can't unmount
other users mountpoints.

If you use passphrases for your remote clients, run `ssh-add -t 10m /path/to/your/ssh/key`
before this script, else you'd get prompted repeatedly for the passphrase.

CONFIG: /etc/bacula_scripts/bacula_del_jobs_conf.py

optional arguments:
  -h, --help  show this help message and exit
  -d          Delete jobs and storage files
  -dry        Simulate deletion


##### usage: bacula_del_media_orphans [-h] [-d] [-dry]

 bacula-del-media-orphans.py

Delete all catalog entries, which backup volume doesn't exist anymore.

Removes catalog entries of inexistent volumes, you should run this better manually and not
recurring in cron, because if you accidently remove a volume and want to migrate from an offsite
backup, then the job entry would also be gone.

CONFIG: /etc/bacula-scripts/bacula_del_media_orphans_conf.py 

optional arguments:
  -h, --help  show this help message and exit
  -d          Delete jobs and storage files
  -dry        Simulate deletion


##### usage: bacula_offsite_backup_age_watch [-h] [-c]

 bacula-offsite-backup-age-watch.py

Check when the last offsite backup was performed and send a warning notification mail if the
backup is too old. Add a symlink to this script for example to cron.weekly.

CONFIG: /etc/bacula-scripts/bacula_offsite_backup_age_watch_conf.py

optional arguments:
  -h, --help  show this help message and exit
  -c          Check backup age


##### usage: bacula_del_failed_jobs [-h] [-d] [-dry]

 bacula-del-failed-jobs.py

Delete all volumes that are associated to failed jobs, to be the catalog cleaner.

Developing notes:
Issuing delete twice, because running it just once some entries persisted.
Eventually redo tests by comparing catalog entries between each deletion.

Job Status Code meanings:
A Canceled by user
E Terminated in error

NO CONFIG NEEDED

optional arguments:
  -h, --help  show this help message and exit
  -d          Delete all failed jobs associated volumes
  -dry        Dry run, simulates deletion


##### usage: bacula_add_client [-h] [-r] [-fd_fqdn FD_FQDN]
                         [-os_type {linux,windows}]
                         [-create_client_job CREATE_CLIENT_JOB]
                         [-create_client_copy_job CREATE_CLIENT_COPY_JOB]
                         [-dry_run]


##### usage: host_uptime_server [-h] [-r]

 host_uptime_server.py

Listen on a TCP socket for a host's uptime echo, packed into a json dumps. The json dumps
contains optionally a 'last_backup' json key with the seconds of the last performed backup
as its value.

Send an email notification, if the host's last uptime echo or performed backup is too long ago.

Open the configured TCP port in your firewall. E.g:
`iptables -A INPUT -p tcp --dport 11337 -j ACCEPT`
Provided you have `iptables-persistent` installed:
`iptables-save > /etc/iptables/rules.v4`

CONFIG /etc/bacula-scripts/host_uptime_conf.py

optional arguments:
  -h, --help  show this help message and exit
  -r          Run host_uptime server


##### usage: host_uptime_client [-h] [-r]

 host_uptime_client.py

Connect to the host_uptime server and send a json dictionary to the echo server containing this
hosts FQDN and the date of the latest performed bacula backup.

CONFIG: /etc/bacula-scripts/host_uptime_client_conf.py

optional arguments:
  -h, --help  show this help message and exit
  -r          Run host_uptime client


##### usage: disk_full_notifier [-h] [-d]

 disk-full-notifier.py

Scan all devices in /dev and use the given default_limit, that is % of the used disk space, to
determine whether a disk the disk limit is reached and a warning mail should be send out.
You can use the list of custom_disk tuples (diskname, limit-number) to define individual limits
for the devices.
You can define the configuration values "default_limit" and "custom_disk" in the config.

CONFIG: /etc/bacula_scripts/disk_full_notifier.conf

optional arguments:
  -h, --help  show this help message and exit
  -d          Look for full disks and eventually send out warning mail


---
Project initially created in 05/2016
