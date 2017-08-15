This is a fork of [https://github.com/pstadler/keybase-gpg-github] and also includes info from: [https://www.ahmadnassri.com/blog/github-gpg-keybase-pgp/]

# Set up Keybase.io, GPG & Git to sign commits on GitHub

This is a step-by-step guide on how to create a GPG key on [keybase.io](https://keybase.io), adding it to a local GPG setup and use it with Git and GitHub.

Although this guide was written for OS X, most commands should work in other operating systems as well.

[Discussion](https://news.ycombinator.com/item?id=12289481) on Hacker News.

## Requirements

```sh
$ brew install gpg keybase
```

You should already have an account with Keybase and be signed in locally using `$ keybase login`. In case you need to set up a new device first, follow the instructions provided by the keybase command during login.

Make sure your local version of Git is at least 2.0 (`$ git --version`) to automatically sign all your commits. If that's not the case, use Homebrew to install the latest Git version: `$ brew install git`.

## 1. Install gpg if you don't have it yet
```sh
brew install gnupg gnupg2
```

## 2. Create a new GPG key on keybase.io

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

If you happened to do step 2 before step 1, you can do:
```sh
curl https://keybase.io/<username>/pgp_keys.asc | gpg --import
```
to import the keys from your keybase account to your local gpg.

## 3. Set up Git to sign all commits

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

## 4. Add public GPG key to GitHub

```sh
$ open https://github.com/settings/keys
# Click "New GPG key"

$ keybase pgp export -q CB86A866E870EE00 | pbcopy # copy public key to clipboard
# Paste key into GitHub UI, save
```

## 5. Import key to GPG on another host

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

## Optional: Don't ask for password every time I commit

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

## Optional: Don't ask for password every time I pull or push

If prompted for your passphrase when pushing or pulling, like this:
```sh
git pull origin master
Enter passphrase for key '<path_to_your_id_rsa>':
```

There are two options:

A) You can add the SSH key to your keychain:
```
ssh-add -K <path_to_your_id_rsa>
Identity added: <path_to_your_id_rsa> (<path_to_your_id_rsa>)
```
If you're on OSX Sierra, you will have to enter the passphrase again after a reboot. You can change this configuration by doing the following:

B) You can set up a config file to tell `ssh` to use your key.
[Original post] (http://apple.stackexchange.com/questions/48502/how-can-i-permanently-add-my-ssh-private-key-to-keychain-so-it-is-automatically)

1. If you haven't already, create an ~/.ssh/config file. In other words, in the .ssh directory in your home dir, make a file called config.
2. In that .ssh/config file, add the following lines:
    ```
    Host *
      UseKeychain yes
      AddKeysToAgent yes
      IdentityFile ~/.ssh/id_rsa
    ```
    Change ~/.ssh/id_rsa to the actual filename of your private key. If you have other private keys in your ~.ssh directory, also add an IdentityFile line for each of them. For example, I have one additional line that reads IdentityFile ~/.ssh/id_ed25519 for a 2nd private key.

    The UseKeychain yes is the key part, which tells SSH to look in your OSX keychain for the key passphrase.

    Note that you will still need to ssh-add -K with the key the first time you use it to put the passphrase in the keychain in the first place, but not if you already have at least once.
3. That's it! Next time you load any ssh connection, it will try the private keys you've specified, and it will look for their passphrase in the OSX keychain. No passphrase typing required.
