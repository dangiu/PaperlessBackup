# Paperless Backup
This is a simple guide on how to create an automatic, incremental, encrypted backup solution of a Paperless inscance using Duplicity and Cron/Anacron.

This guide assumes that Paperless-ng is installed in the docker container form and that the path to the volumes is the default one (`/var/lib/docker/volumes/`). If this is not the case, modify the backup script (`paperless_backup`) accordingly.

In this example [MEGA] is used as cloud storage provide. Duplicity supports many different cloud storage services, read its [documentation] to learn how to configure different ones.

The backup script stops the Paperless instance (if running), performs the backup and optionally re-enables it.
The script is also configured to create a full backup if the last full backup available is older than 1 month, otherwise the backup created will be incremental. Backups older than 2 years will be automatically deleted to make space for more recent ones.

## Requirements
* For Duplicity to function correctly with MEGA you will need to install [MEGAcmd] the official MEGA CLI.
* The backup script must be executed as root since the docker volume we are trying to backup require root permission to access.

## Configuration
* Download the script `paperless_backup`.
* Edit the script by adding your MEGA acount information and your GPG key that will be used for encrypting the backup.
* Change the ownership of the script to root by using:

    `sudo chown root paperless_backup`

* Make the script executable and accessible only to root (since it contains sensitive information) with:
    
    `sudo chmod 700 paperless_backup`
* Run the script as root and verify that everything is working as expected.

## Automation
To automate the backup you will need to add a job to Cron/Anacron to execute the script periodically.
Whether to use Cron or Anacron depends on your system. If your Paperless instance is executing on a server running 24/7 you will want to use Cron. Othewise if you are running it on your personal desktop (which is not always on) using Anacron will allow you to setup jobs that are triggered periodically regardless of the specific time. Anacron jobs are not executed at a specific minute/hour but as soon as the set period has expired. In this way you don't risk missing a scheduled job because your computer was turned off when the job was due, which can happen with Cron.

### Anacron
Open the Anacron configuration file:

`sudo nano /etc/anacrontab`

Add a new job for the script:

`7	10	PaperlessBackup.weekly	/bin/bash	/PATH/TO/SCRIPT/paperless_backup`

Save and exit.

### Cron
If you are using cron the process is similar.
Remember to ensure that you are editing the root user Cron configuration:

`sudo crontab -e`

Othewise the script won't execute correctly.

If your instance is running 24/7 on a dedicated server, rememember to edit the backup script and uncomment the line the re-enables the execution of Paperless in detached mode after the backup:

`docker-compose up -d`

### Attention
Cron/Anacron does not support scripts containing dots in the name. E.g. `script.sh` will not work but `script` will.

## GPG Keys
You will need to generate a pair of public-private GPG keys to encrypt the backup. Remember to store them in the root user keyring otherwise they won't be available when executing the script as root.




[MEGA]: https://mega.nz/
[MEGAcmd]: https://mega.nz/cmd
[documentation]: https://duplicity.us/