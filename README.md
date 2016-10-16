# How to import a big mbox file to Gmail

## Scenario

My hoster allows to create a zip file of all IMAP folders to be downloaded from my ftp webspace. This zip file is 3GB (~70000 mails).

Importing this mail to Gmail takes very long (TODO link) and there is no resume feature, so importing is not feasible from a local machine. Instead I use a EC2 instance.

## Setup EC2 instance

Use Ubuntu 14.04, I tried Amazon Linux but failed to install PyOpenSSL (no idea why). A micro instance is fine but use 32GB HD.

## Download the archive from FTP

SSH into your EC2 instance.

```bash
$ ftp
ftp> open <your_ftp_server> # will prompt to username and password
ftp> passive # enter passive mode, otherwise stuff broke for be
ftp> dir # show directory
ftp> get <your_archive_filename>
```

That'll take some time. But still way fater then downloading it to your local machine.

# Copy to S3

Copy your mailbox archive to S3 so you don't need to redownload from FTP if stuff goes wrong (copying to S3 is very fast).

### Install aws-cli

```bash
sudo apt-get install awscli
```

### Configure aws-cli

```bash
aws configure
```

Will prompt for AccessID and Secret, make sure that this user has read/write access from/to your S3 backup bucket. Will also prompt for the region.

### Start copy

```bash
aws s3 cp <your_mailbox_archive> s3://<your_bucket>/<your_mailbox_archive>
```

To later copy archive to machine use:

```bash
aws s3 cp s3://<your_bucket>/<your_mailbox_archive> <your_mailbox_archive> 
```

# Install [`import-mailbox-to-gmail`](https://github.com/google/import-mailbox-to-gmail)

I suggest you make yourself familar with `import-mailbox-to-gmail` on your local machine.

```bash
sudo apt-get update
sudo apt-get install python-pip python-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev libjpeg8-dev zlib1g-dev # otherwise PyOpenSSL install failed
sudo pip install --upgrade google-api-python-client PyOpenSSL
```
