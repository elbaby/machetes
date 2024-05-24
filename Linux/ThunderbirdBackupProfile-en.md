# Exporting Thunderbird profiles _without_ IMAP cached messages

Every tool, plugin and post I found about backing up all Thunderbird profile
settings end up making a full copy of the `~/.thunderird` folder.

When you have one or a few IMAP accounts with thousands (or millions) of
messages, this is impractical.

In my case, my Thunderbird profile has my Gmail account and a couple of other
IMAP accounts where I store lots of historical pre-Gmail messages that make my
`~/.thunderird` folder over 90Gb.

Even when I don't care about the disk space, when I want to export all the
settings in order to configure a new computer, I don't want to be moving that
ammount of data which is mostly cached messages which are stored and backed up
in the mail servers.

I started playing around with `tar` to find out if I could export my profiles
_without_ storing the messages that are in the servers (and that will eventually
be cached again over time in the new computer).

Mozilla Thunderbird stores the message cached from the IMAP servers together
with all the folder structre under the folder `ImapMail/<nombre-del-servidor>`.

The problem is that it is hard to identify the files and folders used to store
the messages themselves (which I want to _avoid_ backing up) from the ones used
to store metainformation about the folders and the structure (like columns, sort
order, etc).

## When all the folders are in `maildir` format

When all the folders from every configured server are stored in Thunderbird
using **`maildir`** format (which uses une file per message), I took advantadge
of the `maildir` format and found an easy way to do it (assuming you don't have
in any IMAP server, a folder named **tmp** or **cur** (in lowercase).

In this case, do the following:

En este caso, alcanza con hacer lo siguiente:
```
# COMPLETE THE ENVIRONMENT VARIABLES

# The name of the folder holding the Thunderbird profiles
# (usually .thunderbird)
PROFILES_DIR=".thunderbird"

# The folder holding the abovementioned Thunderbird profiles' folder
# (usually the user's home or, if Thunderbird was installed with flatpak
# it's ${HOME}/.var/app/org.mozilla.Thunderbird)
BASE_DIR=${HOME}

# The name of the backup file to create
BACKUP_FILE="thunderbird-backup_$(date +"%Y-%m-%d")"

# Create the backup (in the directory we're executing from)
tar --create --verbose --bzip2 --file=${BACKUP_FILE}.tar.bz2 \
    --exclude=cur/* --exclude=tmp/* --exclude=global-messages-db.sqlite \
    --directory=${BASE_DIR} ${PROFILES_DIR}

if [ $? -eq 0 ] ; then
    echo "Backup successfully generated on ${PWD}/${BACKUP_FILE}.tar.bz2"
else
    echo "Something went wrong"
fi
```

**CAUTION** this will _not_ work for folders with the **`mbox`** format (which,
regretfully, is Thunderbird's default). That is, the backup _will_ be made, but
the folders in `mbox` format will be backed up with all the cached messages.

## When some (or all) folders are in `mbox` format

This is more complex sin `mbox` folders are stored in a plain text file with
**_no filename extension_**.

The only workaround I found is to find and exclude all files which _don't_ have
a dot in the filename. The point is that if the folder has a dot in its name,
the file holding it will also have it and thus will not be excluded from the
backup. Anyway, this should reduce the size of the backup.

```
# COMPLETE THE ENVIRONMENT VARIABLES

# The name of the folder holding the Thunderbird profiles
# (usually .thunderbird)
PROFILES_DIR=".thunderbird"

# The folder holding the abovementioned Thunderbird profiles' folder
# (usually the user's home or, if Thunderbird was installed with flatpak
# it's ${HOME}/.var/app/org.mozilla.Thunderbird)
BASE_DIR=${HOME}

# The name of the backup file to create
BACKUP_FILE="thunderbird-backup_$(date +"%Y-%m-%d")"




# temp file to hold mbox folder names we found
EXCLUDE_FILE=`mktemp tbbackup.exclude.XXXXXX`

# save pwd
CURDIR=${PWD}

cd ${BASE_DIR}
# find files without a "." in the filename and store the filenames in the tempfile
for SERVER in ${PROFILES_DIR}/*/ImapMail/* ; do
    find ${SERVER} -type f ! -name "*.*" \
        | grep -v /tmp/ | grep -v /cur/ >> ${EXCLUDE_FILE}
done

# go back tp the original directory
cd ${CURDIR}


# Create the backup (in the directory we're executing from)
tar --create --verbose --bzip2 --file=${BACKUP_FILE}.tar.bz2 \
    --exclude=cur/* --exclude=tmp/* --exclude=global-messages-db.sqlite \
    --exclude-from=${EXCLUDE_FILE} \
    --directory=${BASE_DIR} ${PROFILES_DIR}


if [ $? -eq 0 ] ; then
    echo "Backup successfully generated on ${PWD}/${BACKUP_FILE}.tar.bz2"
else
    echo "Something went wrong"
fi
```

___
<!-- LICENSE -->
___
<a rel="licencia" href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
<br /><br />
Este documento está licenciado en los términos de una <a rel="licencia"
href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.
<br /><br />
This document is licensed under a <a rel="license" 
href="https://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END --> 
