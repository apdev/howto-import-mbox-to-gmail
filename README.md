# how-to-import-mbox-to-gmail

[`import-mailbox-to-gmail`](https://github.com/google/import-mailbox-to-gmail) is the starting point. It does import your mbox file. But there are problems.

## Problem 1: The sent label

We want to properly import the sent mails, since we want to have conversation view and stuff. `import-mailbox-to-gmail` does not propertly import sent email.

Gmail determines what email is a "sent one" by reading the `From` header ([read more](https://developers.google.com/gmail/api/guides/labels)). `import-mailbox-to-gmail` uses [`import()`](https://developers.google.com/gmail/api/v1/reference/users/messages/import) to get the mails from the mbox file into Gmail. Import is generally very useful, it does some checks like spam or that duplicates are detected. However, when using `import()`, the `From` header is not read correctly.

Solution: use [`insert()`](https://developers.google.com/gmail/api/v1/reference/users/messages/insert). It is more low level (similar to `IMAP APPEND`) and header (including `From`) are read correctly. In code [`import()` can simply replaced by `insert()`](https://github.com/google/import-mailbox-to-gmail/blob/master/import-mailbox-to-gmail.py#L233). That will work.

New problem: `import-mailbox-to-gmail` has no resume import. If something goes wrong you will need to start form the beginning. Spoiler: import happens one-by-one and is very slooooow. With `import()` restarting is no problem, duplicates are detected. With `insert()` they are not, you end up with duplicates. It's didn't made it importing (or better inserting) my some-thousand sent mails.

Try this: use a EC2 instace.

*Update:* The speed issue comes from the GMail API, it's the same on EC2 and Google Compute Engine. Importing an email takes ~5sec, one API call. Importing 10000 mails take 13 hours. Still, cloud servers are the way to go.

## Problem 2: Discard imap folder

I don't want to tranfer the IMAP folder into GMail labels.

Solution: replace [this line](https://github.com/google/import-mailbox-to-gmail/blob/master/import-mailbox-to-gmail.py#L223) with `metadata_object = {}`.
