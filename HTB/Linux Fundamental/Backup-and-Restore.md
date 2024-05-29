# Backup and Restore

Linux systems offer a variety of software tools for backing up and  restoring data. These tools are designed to be efficient and secure,  ensuring that data is protected while also allowing us to easily access  the data we need.

When backing up data on an Ubuntu system, we can utilize tools such as:

- Rsync
- Deja Dup
- Duplicity

Rsync is an open-source tool that allows us to quickly and securely  back up files and folders to a remote location. It is particularly  useful for transferring large amounts of data over the network, as it  only transmits the changed parts of a file. It can also be used to  create backups locally or on remote servers. If we need to back up large amounts of data over the network, Rsync might be the better option.

Duplicity is another graphical backup tool for Ubuntu that provides  users with comprehensive data protection and secure backups. It also  uses Rsync as a backend and additionally offers the possibility to  encrypt backup copies and store them on remote storage media, such as  FTP servers, or cloud storage services, such as Amazon S3.

Deja Dup is a graphical backup tool for Ubuntu that simplifies the  backup process, allowing us to quickly and easily back up our data. It  provides a user-friendly interface to create backup copies of data on  local or remote storage media. It uses Rsync as a backend and also  supports data encryption.

#### Rsync - Backup a local Directory to our Backup-Server

```shell
DixLan@htb[/htb]$ rsync -av /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

This command will copy the entire directory (`/path/to/mydirectory`) to a remote host (`backup_server`), to the directory `/path/to/backup/directory`. The option `archive` (`-a`) is used to preserve the original file attributes, such as permissions, timestamps, etc., and using the `verbose` (`-v`) option provides a detailed output of the progress of the `rsync` operation.

We can also add additional options to customize the backup process,  such as using compression and incremental backups. We can do this like  the following:

```shell
DixLan@htb[/htb]$ rsync -avz --backup --backup-dir=/path/to/backup/folder --delete /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

With this, we back up the `mydirectory` to the remote `backup_server`, preserving the original file attributes, timestamps, and permissions, and enabled compression (`-z`) for faster transfers. The `--backup` option creates incremental backups in the directory `/path/to/backup/folder`, and the `--delete` option removes files from the remote host that is no longer present in the source directory.

If we want to restore our directory from our backup server to our local directory, we can use the following command:

#### Rsync - Restore our Backup

```shell
DixLan@htb[/htb]$ rsync -av user@remote_host:/path/to/backup/directory /path/to/mydirectory
```

## Encrypted Rsync

To ensure the security of our `rsync` file transfer  between our local host and our backup server, we can combine the use of  SSH and other security measures. By using SSH, we are able to encrypt  our data as it is being transferred, making it much more difficult for  any unauthorized individual to access it. Additionally, we can also use  firewalls and other security protocols to ensure that our data is kept  safe and secure during the transfer. By taking these steps, we can be  confident that our data is protected and our file transfer is secure.  Therefore we tell `rsync` to use SSH like the following:

#### Secure Transfer of our Backup

```shell
DixLan@htb[/htb]$ rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

The data transfer between our local host and the backup server occurs over the encrypted SSH connection

## Auto-Synchronization

To enable auto-synchronization using `rsync`, you can use a combination of `cron` and `rsync` to automate the synchronization process. 

Therefore we create a new script called `RSYNC_Backup.sh`, which will trigger the `rsync` command to sync our local directory with the remote one.

#### RSYNC_Backup.sh

```bash
#!/bin/bash

rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

Then, in order to ensure that the script is able to execute properly, we must provide the necessary permissions. Additionally, it's also  important to make sure that the script is owned by the correct user, as  this will ensure that only the correct user has access to the script and that the script is not tampered with by any other user.

```shell
DixLan@htb[/htb]$ chmod +x RSYNC_Backup.sh
```

After that, we can create a crontab that tells `cron` to  run the script every hour at the 0th minute. We can adjust the timing to suit our needs. To do so, the crontab needs the following content:

#### Auto-Sync - Crontab

```shell-session
0 * * * * /path/to/RSYNC_Backup.sh
```

With this setup, `cron` will be responsible for executing the script at the desired interval, ensuring that the `rsync` command is run and the contents of the local directory are synchronized with the remote host.