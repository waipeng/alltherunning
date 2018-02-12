---
layout: post
title: Password manager with Pass, Keybase and per device PGP keys
tag: PGP password
---

Not having a cross platform, easy to use, open sourced password manager has
always been a pet peeve of mine. I started playing with
[Keybase](https://keybase.io/) and [pass](https://www.passwordstore.org/) a while
back, and was wondering whether I could build the password manager I wanted out of
these pieces.

Firstly, I wanted a solution that is:

1. Open Source
1. Simple
1. Browser integration
1. Shareable with other users
1. Able to store more than website passwords
1. Per Device keys (details below)


# Pass

Finally got introduced to [pass](https://www.passwordstore.org/) by my
colleagues, which was a refreshing solution. It is open source and simple - each
password is merely a PGP encrypted text file. It supports
multiple PGP keys, and can organise passwords using OS filesystem hierarchy.

This means that I can encrypt different folders with different keys (work,
personal). It also allows me to share secrets with colleagues by having separate
directory that is encrypted with all team member's keys. For example, I can
have a password store that looks like this:

```
.password-store/
├── family
├── personal
└── work
```

`work` is encrypted with mine + colleagues keys, `personal` is encrypted only
with my keys, and `family` is encrypted with mine + family members' keys.

Browser integration is provided by
[browserpass](https://github.com/dannyvankooten/browserpass), which works on
Chrome on Linux / OS X.


# Why device keys?

I wanted device specified keys so that I can revoked a device key if the device
goes missing, but still retain the ability to access my passwords using other
devices in my possession. For example, if I have 3 devices (work desktop, home
desktop, laptop) and I lose my laptop, I can very quickly revoke the laptop's
key and re-encrypt my password store.


# Trust

Trust in this system is (partly) provided by Keybase. Keybase already has a
similar structure (device keys), but unfortunately they use NaCl keys which
don't work with pass. I used keybase cli to generate (and sign) device PGP keys,
which anyone can verify by going to my Keybase account. Each device PGP key is
generated locally and signed by the device NaCl key, which never leaves the
device. Anyone can [verify my keys](https://keybase.io/waipeng) by visiting
Keybase.


# Step by step instructions

1. Set up a Keybase account.
1. Install Keybase client on each device. This will ensure that your device has
   a NaCl key provisioned for it.
1. **On each device**, generate a PGP key:
   ```
   keybase pgp gen --multi
   ```
1. Keybase will generate a PGP key and send it up to keybase servers. It will
   also import it into your local keyring. Check if it is imported by running
   ```
   gpg -K --fingerprint
   ```
   and comparing it with with the secret keys in keybase
   ```
   keybase pgp list
   ```
   **NOTE:** PGP key ids shown can be in 1 of 3 formats - *short*, *long* or
   *fingerprint*. *Short* and *long* are the **last** (note last) 8 and 16 hex
   digits of the *fingerprint*, respectively.
1. If your secret key is not imported into your local keyring, bring it in with:
   ```
   keybase pgp export -q <key-id> -s | gpg --import
   ```
1. Repeat above steps for each of your device
1. On each device, now pull in all the public keys of other devices. This is
   needed so that each device can encrypt your store for other devices.
   ```
   keybase pgp export -q <key-id> | gpg --import
   ```
1. When all the devices has their own PGP keys, initialise your password store.
   Use the key-ids of all of your devices, e.g.:
   ```
   pass init -p personal <key-id-1> <key-id-2> ...
   ```
   This should create a password store encrypted with all your device keys.
1. Follow [pass documentation](https://www.passwordstore.org/) on inserting
   passwords into your store.
1. Install [browserpass](https://github.com/dannyvankooten/browserpass) to
   easily use your passwords on websites.


# Drawback

A major drawback of passwords is that they are static. An attacker gaining
control of **both** my password store and a device key can very easily brute force
the passphrase to gain access to all the passwords.

For example, if I lose my device which contains both the key and the password
store, revoking the device key does not prevent an attacker from gaining access
to all passwords in the store, which are valid up to the point I lose my
laptop.

This is an inherent drawback of passwords. The only fix is change all the
passwords encrypted in store when you revoke a key. That way, an attacker will
only gain access to the old passwords, which will be useless to him.
Unfortunately at this time there is no easy way to do that across all sites.

An alternative way is to keep the key and the store separate from each other;
they only come together when you need to decrypt the password. For example, the
store can live on a [Keybase Filesystem](https://keybase.io/docs/kbfs) (KBFS)
share (which is streamed to the device).  If you lose the device, you can very
quickly stop access by revoking the device key, revoking the device's access to
the share, and re-encrypting the store.

One way to achieve this is by moving the password store to kbfs and symlink
it back to where `pass` expects it:
```
mv ~/.password-store/personal /keybase/private/<username>/password-store/personal
ln -s /keybase/private/<username>/password-store/personal ~/.password-store/personal
```


# Improvements

Figuring this out took around an afternoon, which got me thinking:
- Why is this so hard?
- Why doesn't each device comes with a key by default?
- Why are we not using hardware devices
  (maybe [TPM](https://en.wikipedia.org/wiki/Trusted_Platform_Module)) to do the
  above, by default?
- How can a non-technical user figure all this out?
- And finally, (again) why is decent security so hard?

We already have all the technology pieces, but it seems like there is still work
to be done to 'glue' all the pieces together into something an average user will
understand.  Many solutions are closed source or silo'ed to a particular
platform, and by that limitation you can't trust data to it.

Many thanks to projects like pass, keybase, keepass, lastpass, chrome/firefox
password manager, that helps bring encryption to the masses. Hopefully one day
we can have the password nirvana I dream about.
