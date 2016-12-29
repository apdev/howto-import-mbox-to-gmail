# How to import a big mbox file to Gmail

## Scenario

My hoster allows to create a zip file of all IMAP folders to be downloaded from my ftp webspace. This zip file is 3GB (~70000 mails).

Importing this amount of mails [takes very long](https://github.com/google/import-mailbox-to-gmail/issues/19) and there is no resume feature, so importing is not feasible from a local machine. Instead I use a EC2 instance.

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

## Copy to S3

Copy your mailbox archive to S3 so you don't need to redownload from FTP if stuff goes wrong (copying to S3 is very fast).

### Install aws-cli

```bash
sudo apt-get install awscli
```

### Configure aws-cli

```bash
aws configure
```

Will prompt for Access Key ID and Secret Access Key, make sure that this user has read/write access from/to your S3 backup bucket. Will also prompt for the region.

### Start copy

```bash
aws s3 cp <your_mailbox_archive> s3://<your_bucket>/<your_mailbox_archive>
```

To later copy archive to machine use:

```bash
aws s3 cp s3://<your_bucket>/<your_mailbox_archive> <your_mailbox_archive> 
```

## Install [`import-mailbox-to-gmail`](https://github.com/google/import-mailbox-to-gmail)

I suggest you make yourself familar with `import-mailbox-to-gmail` on your local machine.

```bash
sudo apt-get update
sudo apt-get install python-pip python-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev libjpeg8-dev zlib1g-dev # otherwise PyOpenSSL install failed
sudo pip install --upgrade google-api-python-client PyOpenSSL
```

### Sent label issue

We want to properly import the sent mails, since we want to have conversation view and stuff. `import-mailbox-to-gmail` does not propertly import sent email.

Gmail determines what email is a "sent one" by reading the `From` header ([read more](https://developers.google.com/gmail/api/guides/labels)). `import-mailbox-to-gmail` uses [`import()`](https://developers.google.com/gmail/api/v1/reference/users/messages/import) to get the mails from the mbox file into Gmail. Import is generally very useful, it does some checks like spam or that duplicates are detected. However, when using `import()`, the `From` header is not read correctly.

Solution: use [`insert()`](https://developers.google.com/gmail/api/v1/reference/users/messages/insert). It is more low level (similar to `IMAP APPEND`) and header (including `From`) are read correctly. In code [`import()` can simply replaced by `insert()`](https://github.com/google/import-mailbox-to-gmail/blob/master/import-mailbox-to-gmail.py#L233). That will work.

### Discard imap folder

I don't want to tranfer the IMAP folder into GMail labels.

Solution: replace [this line](https://github.com/google/import-mailbox-to-gmail/blob/master/import-mailbox-to-gmail.py#L223) with `metadata_object = {}`.
