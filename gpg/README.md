https://www.jfry.me/articles/2015/gpg-smartcard/ - dead link?
https://github.com/drduh/YubiKey-Guide#ssh

# Ubuntu setup

Install Dependencies:

```
sudo apt-get install gnupg2 gnupg-agent libpth20 pinentry-curses libccid pcscd scdaemon libksba8
```

Import public key

```
curl "https://raw.githubusercontent.com/tdickman/dotfiles/master/gpg/tom-public-key.asc" | gpg2 --import
```

Add the following to your .gnupg/gpg-agent.conf file:

```
default-cache-ttl 600
max-cache-ttl 7200
enable-ssh-support
```

# Update bash init file

See bash_shared.sh content. You may have to kill gpg-agent to get it to restart and use the new settings:

```
killall gpg-agent
```

# Symlink gpg to gpg2 on older versions of ubuntu

```
ln -s /usr/bin/gpg2 /usr/bin/gpg
```

# Set your key to be trusted

```
gpg -K
gpg --edit-key <KEY_ID> (not sure if need to select the three keys [1-3] before the following step)
gpg> trust
gpg --card-status
```

# Sign git commits by default

https://help.github.com/articles/signing-commits-using-gpg/

```
git config --global commit.gpgsign true
```

# Getting ssh public key
ssh-add -L

# Changing yubikey device (swapping to backup)

```
rm -rf ~/.gnupg/private-keys-v1.d (on windows this is at C:\Users\USER_NAME\AppData\Roaming\gnupg\private-keys-v1.d)
If on windows, go to Kleopatra->Settings->Configure Kleopatra->GnuPG System->Smartcards and adjust the value of "Connect to reader at port N" to match that of the yubikey (different for different hardware types - logging option at bottom of that menu can be helpful for this) 
killall gpg-agent (seemingly not applicable on windows)
gpg2 --card-status
# Trust new key as shown above
```

# Renewal

https://two-wrongs.com/checklist-for-renewing-gpg-subkeys

Update key expiration:

```
gpg --import <master-key>
gpg --import <sub-keys>
gpg --list-secret-keys     // note that the only key listed may be the master key at first; once you run "--edit-key" on the master key the others will show up
gpg --edit-key 0x...
# Repeat for each subkey
key <n>
expire
save
```

Backup:

```
gpg2 -a --export-secret-key 0x2896DB4A0E427716 >> /media/BACKUP/<new-date>2896DB4A0E427716.master.key
gpg2 -a --export-secret-subkeys 0x2896DB4A0E427716 >> /media/BACKUP/<new-date>2896DB4A0E427716.sub.key
```

Import to yubikey:

Note: This 'moves' the keys from the computer to the smartcard. After they have been "moved" they are no longer able to be loaded
to a secondary card. We will need to reboot and repeat the entire process, restoring from the backed up keys we just created. This
keeps us from having to run through the key refresh cycle a second time for the backup smartcard.

Additional Note: It's not necessary to copy the updated private keys to smart cards. Just updating the public key is sufficient.

```
gpg --edit-key 0x...
# Repeat for each subkey
key <n>
keytocard
save
```

Export public key:

```
gpg -a --export 0x... > my-public-key.asc
```

Import public key to computers (yes, this does necessitate an additional USB drive in order to keep the backups air-gapped):

```
mv /media/... ~/dotfiles/gpg/
gpg --import my-public-key.asc
```
Alternatively, upload the public key to github, then run this command (same as above):

```
curl "https://raw.githubusercontent.com/tdickman/dotfiles/master/gpg/tom-public-key.asc" | gpg2 --import
```

# Enabling Touch Support

Follow [this
guide](https://ruimarinho.gitbooks.io/yubikey-handbook/content/openpgp/touch-protection/enabling-touch-protection.html)
to require touch to approve openpgp actions.
