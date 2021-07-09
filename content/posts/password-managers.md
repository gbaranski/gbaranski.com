+++
title = "Comparison of password managers and my setup"
date = 2021-07-09
tags = ["password-manager", "gopass"]
description = """
This post describes my password manager setup, use of [gopass](https://github.com/gopasspw/gopass) on PC, and [Android-Password-Store](https://github.com/android-password-store/Android-Password-Store) on Android.
"""
showFullContent = false
+++

# Introduction
For a long time I was looking for a password manager which would meet all the following requirements:

1. Open-source,
2. CLI/TUI application must not be written in any scripting language. I don't want slow startup time, I might use a password manager in scripts and 0.5s startup time is unacceptable.
3. Android & Linux support,
4. Option for self-hosting,
5. Must be relatively easy to synchronize between multiple computers, must work on Linux and Android.

## What I've tried so far

### Bitwarden

1. ✅ Open-source, under GNU GPLv2 License.
2. ❌ There is CLI app, but it's written in JS which makes it horribly slow to start up, launching `bw --help` took 544ms; just for comparsion [gopass](https://github.com/gopasspw/gopass) help page takes 66ms, retrieving a specific password takes 200ms, huge difference.
3. ✅ Android & Linux is fully supported, but the desktop app is written in Electron, which is slow; moreover, by enabling Wayland's fractional scaling everything gets blurred.
4. ✅ Self-hosting is possible via [vaultwarden](https://github.com/dani-garcia/vaultwarden).
5. ✅ Super simple to synchronize, probably the easiest of all other options mentioned here.

### KeepassXC

1. ✅ Open-source, under GNU GPLv3 License.
2. ✅ There is `keepassxc-cli`.
3. ✅ Android is supported by [KeepassDX](https://www.keepassdx.com/), I personally don't like the UI of the app; Linux is supported.
4. ✅ Self-host by storing database on computer.
5. ❌ Complicated synchronization between Linux and Android.

### gopass

1. ✅ Open-source, under MIT License.
2. ✅ `gopass` by itself is CLI/TUI Application.
3. ✅ Android is supported by [Android-Password-Store](https://github.com/android-password-store/Android-Password-Store), which is very nice, looks the best of all other apps mentioned here; Linux is supported.
4. ✅ Self-host is possible by storing git repo on my computer.
5. ✅ As soon as you get GPG keys working, synchronization is achieved by synchronizing git repository.

### Verdict

As you can see, gopass meets all of my requirements.

The rest of the post covers
- GPG Keys for safe encrypting/decrypting stored keys.
- Git repository to store passwords.
- Synchronizing passwords between Android and Linux.
- Setting up gopass password store.

# Prerequisites

`GPG_TTY` variable must be set to get GPG working, check if exists by `echo $GPG_TTY`, if it's not set `GPG_TTY` to output of `tty` command.

Bash/ZSH: 
```bash
# ~/.bashrc or ~/.zshrc
export GPG_TTY=$(tty)
```

Fish:
```bash
# ~/.config/fish/config.fish
export GPG_TTY=(tty)
```

# Git repository

Git repository is required to store passwords, however gopass also [supports](https://github.com/gopasspw/gopass/blob/master/docs/setup.md#storing-and-syncing-your-password-store-with-google-drive--dropbox--syncthing--etc) other ways to store passwords, but Git Repo seems best option for me, I store them in Github private repository, although it could be even self-hosted. 

Keep in mind that password database is not the top secret, of course it will be better if it will be private, but the password database by itself will be encrypted with password.

Create Github repository using [Github CLI](https://github.com/cli/cli/)
```bash
$ gh repo create pass
? Visibility Private
? This will create the "pass" repository on GitHub. Continue? Yes
✓ Created repository gbaranski/pass on GitHub
? Create a local project directory for "gbaranski/pass"? No
```

# GPG keys

## GPG Primary key

***If you already have GPG Primary Key that you can use, you can skip this step***
   
Generate new GPG Key using `gpg --full-generate-key --expert`, in this example we're using RSA because I'm not sure about ECC keys compatibility.

The primary key won't be able to encrypt/sign, there will have sub-keys for that, primary key will be used only for creating new sub-keys if needed, however for password managament purposes we're going to use only one sub-key for all computers.

```
$ gpg --full-generate-key
gpg (GnuPG) 2.2.28; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
(ins)Your selection? 
RSA keys may be between 1024 and 4096 bits long.
(ins)What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
(ins)Key is valid for? (0) 
Key does not expire at all
(ins)Is this correct? (y/N) Y

GnuPG needs to construct a user ID to identify your key.

(ins)Real name: Grzegorz Baranski
(ins)Email address: root@gbaranski.com
(ins)Comment: 
You selected this USER-ID:
    "Grzegorz Baranski <root@gbaranski.com>"

(ins)Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 6C771D81228409BA marked as ultimately trusted
gpg: directory '/home/gbaranski/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/gbaranski/.gnupg/openpgp-revocs.d/EA0AEFD44D1EAFA05B3ED6BD6C771D81228409BA.rev'
public and secret key created and signed.

pub   rsa4096 2021-07-09 [SC]
      EA0AEFD44D1EAFA05B3ED6BD6C771D81228409BA
uid                      Grzegorz Baranski <root@gbaranski.com>
sub   rsa4096 2021-07-09 [E]
```

In this example root@gbaranski.com will used as GPG Key ID identifier, of course replace it with your own email when setting it up.
If you have few GPG Key IDs with the same email, check your GPG Key ID by using `gpg --list-secret-keys --keyid-format 0xLONG` and then use it instead email.

Check if the keypair has been properly created
```
$ gpg --list-secret-key --keyid-format 0xLONG root@gbaranski.com
sec   rsa4096/0x6C771D81228409BA 2021-07-09 [SC]
      EA0AEFD44D1EAFA05B3ED6BD6C771D81228409BA
uid                   [ultimate] Grzegorz Baranski <root@gbaranski.com>
ssb   rsa4096/0x6FC346E99EA085F4 2021-07-09 [E]
```

### Backing up primary secret key

Copy `~/.gnupg` to some safe place, such as USB Stick in case something happens.

## Setting up Gopass

First off, you need to have a Git repository, in my case it's Github private repo named `passwords`.

```none
$ gopass setup

   __     _    _ _      _ _   ___   ___
 /'_ '\ /'_'\ ( '_'\  /'_' )/',__)/',__)
( (_) |( (_) )| (_) )( (_| |\__, \\__, \
'\__  |'\___/'| ,__/''\__,_)(____/(____/
( )_) |       | |
 \___/'       (_)

🌟 Welcome to gopass!
🌟 Initializing a new password store ...
🌟 Configuring your password store ...
🎮 Please select a private key for encrypting secrets:
[0] gpg - 0x6C771D81228409BA - Grzegorz Baranski <root@gbaranski.com>
Please enter the number of a key (0-0, [q]uit) (q to abort) [0]: 
Please enter an email address for password store git config []: root@gbaranski.com
❓ Do you want to add a git remote? [y/N/q]: Y
Configuring the git remote ...
Please enter the git remote for your shared store []: git@github.com:gbaranski/passwords.git
✅ Configured
```

### Add first password

```
$ gopass create
🌟 Welcome to the secret creation wizard (gopass create)!
🧪 Hint: Use 'gopass edit -c' for more control!
[ 0] Website Login
[ 1] PIN Code (numerical)
[ 2] Generic

Please select the type of secret you would like to create (q to abort) [0]: 
0
🧪 Creating Website login
  [1] URL                                    []: example.com
  [2] Login                                  []: gbaranski
  [3] Generate Password?                     [Y/n/q]: 
    [a] Human-pronounceable passphrase?      [y/N/q]: 
    [b] How long?                            (q to abort) [24]: 
    [c] Include symbols?                     [y/N/q]: 
    [d] Strict rules?                        [y/N/q]: 
  [4] Comments                               []: 
✅ Credentials saved to "websites/example.com/gbaranski"
✔ Copied websites/example.com/gbaranski to clipboard. Will clear in 45 seconds.
$ gopass show websites/example.com/gbaranski
⚠ Parsing is enabled. Use -n to disable.
Secret: websites/example.com/gbaranski

Ro0EWV3Q8H2oGlWTR8XUHanu
comment: 
url: example.com
username: gbaranski
```

### Synchronizing with Android phone

First off, we need to export the GPG Private Key, I use croc for sending and receiving files between PC and Phone.

Send the private key to the device using croc
```bash
$ gpg --export-secret-keys --armor root@gbaranski.com | croc send
Sending 'stdin' (6.6 kB)         
Code is: 4733-quick-eclipse-forum
On the other computer run

croc 4733-quick-eclipse-forum

Sending (->192.168.1.12:49309)
 100% |████████████████████| (6.6/6.6 kB, 3.214 MB/s)
```

Accept the file on Android using [Croc Android](https://f-droid.org/en/packages/com.github.howeyc.crocgui/)

Use OpenKeychain app to import PGP Key, available in [`F-Droid`](https://f-droid.org/en/packages/org.sufficientlysecure.keychain/).
Import private key by selecting  and then "Import from File".

Open [Android-Password-Store](https://github.com/android-password-store/Android-Password-Store), generate SSH keys, clone repository from Github and view your passwords.

# Conclusions

gopass is awesome password manager for people who like using things in terminal.

I hope this article was useful and saved you some time. 👋


# Updated 09.07.2021
No longer using sub-keys, that complicated this setup by a lot, and for some reason my phone suddenly stopped decrypting passwords encrypted on my laptop.
