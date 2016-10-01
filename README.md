# how-to-import-mbox-to-gmail

[`import-mailbox-to-gmail`](https://github.com/google/import-mailbox-to-gmail) is the starting point. It does import your mbox file. But there are problems.

## Problem 1: The sent label

We want to properly import the sent mails, since we want to have conversation view and stuff. `import-mailbox-to-gmail` does not propertly import sent email.

Gmail determines what email is a "sent one" by reading the `From` header ([read more](https://developers.google.com/gmail/api/guides/labels)). `import-mailbox-to-gmail` uses [`import()`](https://developers.google.com/gmail/api/v1/reference/users/messages/import) to get the mails from the mbox file into Gmail. Import is generally very useful, it does some checks like spam or that duplicates are detected. However, when using `import()`, the `From` header is not read correctly.

Solution: use [`insert()`](https://developers.google.com/gmail/api/v1/reference/users/messages/insert). It is more low level (similar to `IMAP APPEND`) and header (including `From`) are read correctly. In code [`import()` can simply replaced by `insert()`](https://github.com/google/import-mailbox-to-gmail/blob/master/import-mailbox-to-gmail.py#L233).

