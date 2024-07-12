---
title: Pull all your emails from Gmail using fetchmail
date: 2024-07-12 21:45:00 +0800
categories: [howtos]
tags: [fetchmail,gmail,getmail,debian]
---

---
_This is an old post from 14 years ago today. I was looking for away to download all my emails from a GMail account and I remember writing this and putting it in my [old tumblr blog](https://gari.melecio.org/post/43571126056/pull-all-your-emails-from-gmail-using-fetchma). I'm putting it here and slightly updated it._

---

The problem with `fetchmail` is that it cannot store fetched mails to a file. It needs to use an external MDA to deliver the fetched message. To store the messages to a file, we will use the `getmail_maildir` [(man1)](https://manpages.debian.org/bullseye/getmail6/getmail_maildir.1.en.html) program from the [getmail](https://tracker.debian.org/pkg/getmail6) package as MDA. 

In this example, we fetch all emails from Gmail and store it locally in qmail-style maildir format. Create first all the standard maildir folders before fetching. 

```bash
mkdir -p maildir/{cur,new,tmp}
```

In Gmail, all mails are stored or linked from the “All Mail” label. It also includes sent mails. 

```bash
fetchmail -vakn -p IMAP —ssl -u user@domain.com —auth password —folder “[Gmail]/All Mail” imap.gmail.com -m “getmail_maildir /home/user/maildir/”
```

`getmail_maildir` will fail when using relative paths. Instead, use absolute paths or use `./maildir/`. 

Enjoy.