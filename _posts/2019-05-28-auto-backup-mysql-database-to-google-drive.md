---
title: "Auto Backup your Mysql Database to Google Drive using Cron"
layout: post
date: 2019-05-28 23:02
tag:
- mysql
category: blog
author: taranjeet
description: This post is about setting up a cron to periodically upload backup of mysql database to Google Drive.
---

Let's assume that you have an instance running Ubuntu and mysql is installed in it. We will also assume that a `ubuntu` user is setup and there is a database called `products`, for which we need to create backup and upload to Google Drive. We will assume taking hourly backups. This can be updated as per the requirements.

First, we will install [gdrive](https://github.com/gdrive-org/gdrive) which will be used to access google drive from command line

```sh

# find installation file version specifiv to your version
wget -O gdrive "https://drive.google.com/uc?id=1Ej8VgsW5RgK66Btb9p74tSdHMH3p4UNb&export=download"
chmod +x gdrive
sudo mv gdrive /usr/local/bin/

# authenticate
gdrive about

# enter verification code here to authenticate

# on success, we will get the following as output
User: Taranjeet Singh, someemail@gmail.com
Used: 190.5 MB
Free: 15.9 GB
Total: 16.1 GB
Max upload size: 5.2 TB
```

Just to be sure that gdrive is working we will run `gdrive list` to see the list of folders present in drive.

We will now create a new folder named `dbbackups` using gdrive

```sh
gdrive mkdir dbbackups
Directory 3EDlQZRuPdyAG91lhVhsl5gLFeX68PG3n created.
```

We will copy the id (here `3EDlQZRuPdyAG91lhVhsl5gLFeX68PG3n`) and here in refer to as `GOOGLE_DRIVE_FOLDER`

We will create a folder named `mysqlbackup` in the home directory of `ubuntu` user. We will also make sure that `ubuntu` user ownws this directory.

```sh
pwd
# /home/ubuntu
mkdir mysqlbackup

# just to be sure
chown -R ubuntu:ubuntu mysqlbackup/
```

Next we will create a script called `uploadbackup.sh` and place it in the home directory of ubuntu user.

```sh
pwd
# /home/ubuntu

vim uploadbackup.sh
```

We will write the following content in `uploadbackup.sh`

```sh
#!/bin/sh
set -e
set -x

DUMP_DIR="/home/ubuntu/mysqlbackup"
DB_NAME=products
DB_USER=DB_USER_NAME_HERE
DB_PASSWORD=DB_PASSWORD_HERE
DATE_FORMAT="%Y-%m-%d_%H"
GOOGLE_DRIVE_FOLDER=ID_OF_GOOGLE_FOLDER_COPIED_ABOVE

file_name="$(date +$DATE_FORMAT)"
sql_file="$file_name.sql"
gz_file="$file_name.tar.gz"

# make sure that the folder exists
mkdir -p $DUMP_DIR
cd $DUMP_DIR

# run mysqlbackup, tar gz and delete sql file
mysqldump -u$DB_USER -p$DB_PASSWORD $DB_NAME > $sql_file
tar -zvcf $gz_file $sql_file
rm $sql_file

/usr/local/bin/gdrive upload --parent $GOOGLE_DRIVE_FOLDER $gz_file
rm $gz_file
```

Now we will test this script by running it once.

```sh
sh uploadbackup.sh
```

A `.tar.gz` file should be created in the `dbbackups` google drive folder.

Now, we will perform the last step, which is adding this script on crontab. We will use the ubuntu user to add an hourly crontab.

```sh
crontab -e
```

In the editor, that open, we will add the following line

```sh
* * * * * sh /home/ubuntu/uploadbackup.sh >> /var/log/cron/mysqlbackup.log 2>&1
```

Here we have `/var/log/cron/mysqlbackup.log` as the output of log file. In case we want to take some other folder, we need to make sure that the directory is owned and write-able by `ubuntu` user, since crontab is being run via `ubuntu` user. To own a directory(lets say `/var/log/cron`), we will execute the following steps

```sh
cd /var/log
sudo mkdir cron
sudo chown -R ubuntu:ubuntu cron
```

References

* [This tutorial](https://andypalmer.me/blog/linux/mysql-database-backups-google-drive/) helped me a lot in writing this blog post. Thanks
