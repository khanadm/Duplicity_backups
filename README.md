**What is Duplicity?**

Duplicity is a network backup program.

It can save snapshots of directories and files to a remote GnuPG encrypted tar file, which acts as a backup repository.




**How to install install Duplicity on CentOS ?**



1 Use the package manager yum to install Duplicity:


```sh
sudo yum install duplicity
```


**In my case, I am using Duply. You can also continue with Duplicity.**


```sh
sudo yum install duply
```


**How to create profile ?**


```sh
duply <profile_name> create
```

Example

Initialize a new Duply profile using the duply command
Replace my_backup_profile with your desired profile name:


`duply my_mysql_backup create`


Then:

Go to home directory of duply

```sh
cd USER_HOME/.duply/PROFILE_NAME
```



`cd /home/cloud_user/.duply/my_mysql_backup`


Here you find **conf** file under that you have to change some parameters


**conf** This is the main configuration file for the profile. It contains various settings like source and destination, encryption, compression, etc.



```sh
GPG_KEY="your-gpg-key-id"
GPG_PW="your_passphrase_password"
TARGET="boto3+s3://your-bucket-name/path/to/backup"
SOURCE="/path/to/source"
MAX_FULLBKP_AGE=1M
```

Here is Explanation


**GPG_KEY**  Specify the GPG key ID that will be used for encryption and signing of the backup data


**GPG_PW**  Define passphrase passoword


 **TARGET** The destination where your backup data will be stored. It's the location where Duply will save your backup files. 


**SOURCE** The source data you want to back up. This could be a local directory, remote server, or other location. Set the SOURCE option accordingly.


**MAX_FULLBKP_AGE=1M** This sets the maximum age of a full backup. It specifies that if the most recent full backup is older than 1 month (1M), Duply should create a new full backup instead of an incremental backup. In other words, if it's been more than a month since the last full backup, Duply will create a new full backup to ensure that you have a fresh copy of all your data.



**exclude** This file specifies patterns for files and directories to exclude from the backup.



Create a file named **pre** (no file extension) using your preferred text editor. 

**pre** A script that is executed before running the backup.


```sh
sudo vim pre
```

```sh
#!/bin/bash
WORKDIR=/home/cloud_user/
SUBDIR=backup

mkdir -p "${WORKDIR}/${SUBDIR}"
touch "${WORKDIR}/${SUBDIR}/dump"
chmod 0600 "${WORKDIR}/${SUBDIR}/dump"

DBNAME=ongraph
MYSQL_PASSWORD="ongraph"
mysqldump -u ongraph -p"${MYSQL_PASSWORD}" --databases "${DBNAME}" > "${WORKDIR}/${SUBDIR}/dump"
```


To save the file:

Press `Esc` then `:` write  `wq` Press Enter



- Creates a backup directory (if it doesn't exist) inside the **WORKDIR** directory.

- Creates an empty file named **dump** inside the backup directory and sets its permissions to read and write for the owner.


- Defines the MySQL database name ( **DBNAME** ) and MySQL password ( **MYSQL_PASSWORD** ).


- Uses the **mysqldump** command to dump the specified MySQL database with the given username and password.


- Redirects the output of the **mysqldump** command to the **dump** file inside the backup directory.




Create **post** file Represents the directory where your backup files are stored. 

**post** A script that is executed after a successful backup.


```sh
sudo vim post
```


```sh
#!/bin/bash
WORKDIR=/home/cloud_user/backup/
```

To save the file:

Press `Esc` then `:` write  `wq` Press Enter



Create **restore** file


```sh
sudo vim restore
```

```sh
#!/bin/bash
WORKDIR=/home/cloud_user/backup/
DBNAME=ongraph
MYSQL_PASSWORD="ongraph"
mysql -u ongraph -p"${MYSQL_PASSWORD}" $DBNAME < ${WORKDIR}/dump
```


To save the file:

Press `Esc` then `:` write  `wq` Press Enter



**WORKDIR** Specifies the working directory where the backup files are located.


**DBNAME** Specifies the name of the MySQL database that you want to restore.


**MYSQL_PASSWORD** Specifies the MySQL password for the user ongraph.


mysql -u ongraph -p"${MYSQL_PASSWORD}" $DBNAME < ${WORKDIR}/dump

This command uses the mysql client to connect to the MySQL server as the user ongraph with the provided password. It then specifies the database name using the -u flag, and it redirects the contents of the backup file (dump) into the mysql command using input redirection (<). This effectively restores the database using the backup data.



**Incremental Backup:**

```sh
duply <profile_name> backup
```


Example


`duply my_mysql_backup backup`


**Full Backup:**


```sh
duply <profile_name> full
```

Example



`duply my_mysql_backup full`



**Restore Backup:**


```sh
duply <profile_name> restore /path/to/restore/destination
```


Example



`duply my_mysql_backup restore //home/cloud_user/backup.sql`




**Restore of backups in specific Timimg**



```sh
duplicity restore "SOURCE_URL" "DESTINATION_PATH" --time "RESTORE_TIME"
```


Exapmle 


`duplicity restore "boto3+s3://masterbackup1/dbbackup/" /home/cloud_user/back.sql --time 2023-08-06T14:02:42`


**Note:** 
* The "boto3+" prefix is not a requirement for Duplicity; it's specific to the use of the Boto3 backend for cloud storage, such as Amazon S3. If you're not using Boto3 for cloud storage, you don't need to include this prefix.
 
* If you are using Amazon S3, you must set up an AWS profile.[AWS CLI Configuration Files documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html). Click [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) to access the documentation.

