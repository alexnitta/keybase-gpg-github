This is a fork of [https://github.com/pstadler/keybase-gpg-github] and also includes info from: [https://www.ahmadnassri.com/blog/github-gpg-keybase-pgp/]

# Setup Keybase.io, GPG & Git to sign commits on GitHub

This is a step-by-step guide on how to create a GPG key on [keybase.io](https://keybase.io), adding it to a local GPG setup and use it with Git and GitHub.

Although this guide was written for OS X, most commands should work in other operating systems as well.

[Discussion](https://news.ycombinator.com/item?id=12289481) on Hacker News.

## Requirements

```sh
$ brew install gpg keybase
```

You should already have an account with Keybase and be signed in locally using `$ keybase login`. In case you need to set up a new device first, follow the instructions provided by the keybase command during login.

Make sure your local version of Git is at least 2.0 (`$ git --version`) to automatically sign all your commits. If that's not the case, use Homebrew to install the latest Git version: `$ brew install git`.

## Create a new GPG key on keybase.io

```sh
$ keybase pgp gen --multi
# Enter your real name, which will be publicly visible in your new key: Patrick Stadler
# Enter a public email address for your key: patrick.stadler@gmail.com
# Enter another email address (or <enter> when done):
# Push an encrypted copy of your new secret key to the Keybase.io server? [Y/n] Y
# ▶ INFO PGP User ID: Patrick Stadler <patrick.stadler@gmail.com> [primary]
# ▶ INFO Generating primary key (4096 bits)
# ▶ INFO Generating encryption subkey (4096 bits)
# ▶ INFO Generated new PGP key:
# ▶ INFO   user: Patrick Stadler <patrick.stadler@gmail.com>
# ▶ INFO   4096-bit RSA key, ID CB86A866E870EE00, created 2016-04-06
# ▶ INFO Exported new key to the local GPG keychain
```

## Set up Git to sign all commits

```sh
$ gpg --list-secret-keys
# /Users/pstadler/.gnupg/secring.gpg
# ----------------------------------
# sec   4096R/E870EE00 2016-04-06 [expires: 2032-04-02]
# uid                  Patrick Stadler <patrick.stadler@gmail.com>
# ssb   4096R/F9E3E72E 2016-04-06

$ git config --global user.signingkey E870EE00
$ git config --global commit.gpgsign true
```

## Add public GPG key to GitHub

```sh
$ open https://github.com/settings/keys
# Click "New GPG key"

$ keybase pgp export -q CB86A866E870EE00 | pbcopy # copy public key to clipboard
# Paste key into GitHub UI, save
```

## Import key to GPG on another host

```sh
$ keybase pgp export
# ▶ WARNING Found several matches:
# user: Patrick Stadler <patrick.stadler@gmail.com>
# 4096-bit RSA key, ID CB86A866E870EE00, created 2016-04-06

# user: keybase.io/ps <ps@keybase.io>
# 4096-bit RSA key, ID 31DBBB1F6949DA68, created 2014-03-26

$ keybase pgp export -q CB86A866E870EE00 | gpg --import
$ keybase pgp export -q CB86A866E870EE00 --secret | gpg --allow-secret-key-import --import
```

## Optional: Set as default GPG key

```sh
$ $EDITOR ~/.gnupg/gpg.conf
# Add line: default-key E870EE00
```

## Optional: Disable TTY 
If you have problems with making autosigned commits from IDE or other software add no-tty config
```sh
$ $EDITOR ~/.gnupg/gpg.conf
# Add line: no-tty
```

## Optional: Setting up TTY  
Depending on your personal setup, you might need to define the tty for gpg
whenever your passphrase is prompted. Otherwise, you might encounter an `Inappropriate
ioctl for device` error.
```sh
$ $EDITOR ~/.profile # or other file that is sourced every time
# Paste these lines
GPG_TTY=$(tty)
export GPG_TTY
```

## Optional: Don't ask for password every time

### Tell GitHub about your GPG key
```sh
$ git config user.signingkey <github_email> # per repository, or:
$ git config --global user.signingkey <github_email> # global
```
You can now simply use -S or --gpg-sign to commit without having to provide the Key ID:
```sh
$ git commit -S
```

### Set all commits to be signed by default, no further need for -S or --gpg-sign per commit. (Git v2.0.0 and above):
```sh
$ git config commit.gpgsign true # per repository, or:
$ git config --global commit.gpgsign true # global
```
