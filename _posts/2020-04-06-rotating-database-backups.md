---
title: Rotating database backups with node.js and S3
date: 2020-04-06 02:00:00
lastmod: 2020-04-06 02:00:00
description: >-
  Use node.js to store MySQL dumps in S3
layout: post
---

RDS provides it's own in built database backup system, but if you are not using RDS you can roll your own with a small node.js script and an S3 policy.
{: .article-headline}

In this guide I'll walk through setting up a database backup where hourly dumps are stored for a day, daily for a week, weekly for a month and monthly for a year.

## Setup

If you have an existing node project you can just add this script and some dependencies to it. If you don't then set up a new project with `npm init`. Then add the AWS SDK and mysqldump libraries:

```
npm install --save mysqldump aws-sdk
```

## Generating the MySQL dump

With the project set up we need a script to trigger the backup every hour. Create a file called `backup.js`

```javascript

async function backup() {
  const now = new Date();
  const dateString = now.toJSON().substring(0, 16).replace(":", "");
  const filename = `db-${dateString}.sql.gz`;
  const path = "/tmp/" + filename;

  await mysqldump({
    connection: {
      user: "root",
      host: "localhost",
      password: "",
      database: "mydb"
    },
    dumpToFile: path,
    compressFile: true,
  });
}

// run the backup function every hour
setInterval(backup, 3600 * 1000);

```

Be sure to plug in the correct database credentials to the `mysqldump` call. One thing to note is that the library does not accept null passwords, but it does treat an empty string as null.

If you run the script with `node backup.js` a backup file will be created in the `/tmp` folder every hour.

## Setting up an S3 bucket

Log in to your AWS console and navigate to S3. Select create a bucket:

![create-bucket](/asset/img/rotating-database-backup/create-bucket.png){: .img-responsive }

Make note of the name as it will be needed later.

Now we need to create the S3 lifecycle policy to remove the old backup files. Click on the bucket and go to the Management tab:

![add-lifecycle](/asset/img/rotating-database-backup/add-lifecycle.png){: .img-responsive }

Select add lifecycle and then enter the name RemoveDaily and apply it to the hourly/ folder

![setup-lifecycle](/asset/img/rotating-database-backup/setup-lifecycle.png){: .img-responsive }

Next, set the policy to expire files older than 1 day:

![expire](/asset/img/rotating-database-backup/expire.png){: .img-responsive }

Now repeat this process so that items in the daily folder expire after 7 days, weekly expire after 31 days and monthly expire after 365 days.

## Uploading to S3

Now the dumps are generated and the S3 bucket has been set up we can upload the dumps. You'll also need to generate an API key to use with the AWS SDK. Typically these are stored as the environment variables: `AWS_ACCESS_KEY` and `AWS_SECRET_ACCESS_KEY`.

```javascript
async function upload(filename, path, date) {
  const s3 = new S3({
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  });

  const s3directory = getS3Directory(date);

  return new Promise((resolve, reject) => {
    s3.upload({
      Bucket: "yourbucketnamehere",
      Key: s3directory + filename,
      Body: fs.createReadStream(path),
    }, (err, res) => err ? reject(err) : resolve());
  });
}

function getS3Directory(date) {
  if (date.getHours() === 0) {
    if (date.getDate() === 1) {
      return "monthly/";
    }

    if (date.getDay() === 1) {
      return "weekly/";
    }

    return "daily/";
  }

  return "hourly/";
}
```

The `getS3Directory` function will categorize the current backup by classing it as "monthly", "weekly", "daily" or "hourly" depending on the date given.

Now we just need to update our backup function so that it uploads the file and then deletes it afterwards:

```javascript
async function backup() {
  const now = new Date();
  const dateString = now.toJSON().substring(0, 16).replace(":", "");
  const filename = `db-${dateString}.sql.gz`;
  const path = "/tmp/" + filename;

  await mysqldump({
    connection: {
      user: "root",
      host: "localhost",
      password: "",
      database: "mydb"
    },
    dumpToFile: path,
    compressFile: true,
  });    

  await upload(filename, path, now);

  fs.unlink(path, () => {});
}
```

Now when you run `node backup.js` it will take a dump of the database every hour and upload it to your S3 bucket.

One thing to note about the S3 policy is that it is only executed once a day, which means your hourly backups won't expire exactly when they a supposed to but they won't be more than 24 hours out, and it never hurts to have an extra hourly backups lying around.
