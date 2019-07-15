---
title: "GPG Signed Commits"
date: 2019-07-15T19:16:29+03:00
draft: false
toc: false
images:
tags:
  - gpg
  - git
  - github
---

Hello everyone, how is it going? Today I will show you how to sign your commits with GPG key. First of all, we should install GPG command-line tools.

- Here is the link for download: https://www.gnupg.org/download/
- Then we should open a terminal and type `gpg --gen-key`
- Type your real name and e-mail
- Verify your inputs
- Type a secure passphrase

Use the `gpg --list-secret-keys --keyid-format LONG` command to list GPG keys. A private key is required for signing commits.

From the list of GPG keys, copy the GPG key ID you'd like to use.
```
$ gpg --list-secret-keys --keyid-format LONG
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot 
ssb   4096R/42B317FD4BA89E7A 2016-03-10
```
In this case, the GPG key ID is `3AA5C34371567BD2`

Then, use the `gpg --armor --export 3AA5C34371567BD2` command to generate GPG key, in ASCII armor format.
Add the GPG key to your GitHub account.( Settings > SSH and GPG keys )

After all these configuration steps, you should execute these two commands in your terminal.

```
git config --global commit.gpgsign true
git config --global user.signingkey 3AA5C34371567BD2
```

Thanks for reading :)
