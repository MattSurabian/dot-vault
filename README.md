# Dot Vault v1.0.0

## The Problem

A good backup plan is absolutely essential and there have never been more options for continuous offsite backup and file sync than we have today.

The trouble is that most of these services ignore hidden files, or "dotfiles". For the average computer user that's probably a good thing, but for developers and others in the tech field it's a real pain. Our dotfiles store valuable preferences, keys, api tokens, and more.

There are lots of [resources for managing dotfiles](http://dotfiles.github.io/), and while these are great for dealing with dotfiles across multiple machines and tracking changes with git, I didn't find one that addressed the automated backup inclusion problem *without* having to clear personal data from the files first.

## The Solution

Dot Vault is a simple shell script that spiders through a list of file paths, copies them to a working directory which is tar'd and excrypted with a passphrase using des3 via openssl. The working directory is then destroyed. Leaving the user with a single password protected .vault file.

This file can then be backed up, shared, or synced without fear of compromising any sensitive information contained within the dotfiles.

Dot Vault can also import vaults making restoring an old machine or configuring a new one from a vault file a breeze.

## Getting Started

**Dot Vault uses openssl, which should be pre-installed on most OS X and linux machines.**

1. [Download the source](https://raw.github.com/MattSurabian/dot-vault/master/dotvault) from github and add it to a directory in your path, `/usr/local/bin` is the recommended default location.
     - Alternatively clone this repo and setup a symlink to `/usr/local/bin`
1. Create a `.dotvault` file in your HOME directory, copying the one from this repo should be a good start
     - Again, you could create a symlink for this if the repo defaults work for you, or you're running on your own fork. Files that aren't found will be skipped.
1. That's it, you can now run dotvault as documented below!

## Documentation

*Scripts provided assume a bash shell.*

### .dotvault

This simple dot file (name configurable) contains a new-line delimited list of the paths to all the dotfiles you would like backed up, and just like a `.gitignore` file may contain comments prefaced with a `#`. Provided paths must be complete absolute paths, but may utilize `~` to reference the users home directory. Files that are not found are skipped.


### Export Mode

Export mode looks for the configured dotvault file in the users home directory, spiders through the list of paths and copies the files and folders into a container directory which is then tar'd and encrypted using des3 via openssl. The exported file is given a .vault extension.

````
dotvault --export EXPORTED_FILENAME PASS_ARGUMENT(optional)
`````

- *EXPORTED_FILENAME*: A valid file name/path to use when exporting the vault file, a .vault extension will be appended to this value
- *PASS_ARGUMENT*: An optional parameter, very necessary when setting up Dot Vault to run under cron where direct user input isn't possible. This is passed directly to openssl using its supported -pass param valid values can be seen in the [openssl documentation](http://www.openssl.org/docs/apps/openssl.html#PASS_PHRASE_ARGUMENTS) (though I'd expect only the forms `pass:STRING` and `env:ENV_VAR` to be useful here)

*Environment Variable Password Argument Example*

This use case would set the vault file's password to the value of the environment variable named DOTVAULTPASS. This is the most secure choice for use in an environment where you don't want your chosen password visible to anyone spying on the process list.

````
dotvault --export env:DOTVAULTPASS
````

*String Password Argument Example*

This is the least secure form of the password argument as the chosen password will appear in the process list and anywhere this action is configured.

````
dotvault --export pass:wala
````

### Import Mode
Import mode requests the vault's passphrase, decrypts the vault and copies all files and folders found to their appropriate place on the machine. Dot Vault uses very simple character replacement when generating the vault file, allowing it to know a given file's configured location as it appeared in the .dotvault file of the machine which generated it.

````
dotvault --import IMPORT_FILENAME PASS_ARGUMENT(optional)
````

- *IMPORT_FILENAME*: A valid name/path to a .vault file created by Dot Vault

- *PASS_ARGUMENT*: Same as export mode.


*The vault file replaces the following characters from the provided path in the config file and reverses the process to determine where to put them back.*

```
PATH VALUE       REPLACEMENT
    .        =>      *
    /	     =>      |
```
Importing will likely require root permissions depending on how your environment is setup and which files and folders are being restored. **Importing uses the `cp` command to copy the files back into place and will overwrite in the event of a conflict, as the cp command does.**

### Configuration

In the source for Dot Vault there are several variables setup for simple configuration adjustment. The following values are availble for custom configuration:

 - **dotvaultConfig**: The location and name of the .dotvault configuration file. Default is `${HOME}/.dotvault`
 - **tmpDirName**: Dot Vault needs a temporary directory to work in. This directory is created as needed and destroyed when work is complete. Default name is `dotvaultTMP`
 - **cipher**: If you'd like to use a different cipher than des3 this where you could specify that. The default is `des3`. Note that changing this might require modifications to the openssl calls responsible for encryption and decryption.

## Scheduling Automatic Exports
You can schedule dotvault to run with cron in order to make sure you're always backing up current data. There are several ways to interact with cron, the below command is provided for convenience and will run an export every hour. If your user account does not currently have any cron jobs setup, you may get a warning that says `crontab: no crontab for USER` that's nothing to worry about. 

```
(crontab -l ; echo '* * * * *  . ~/.bash_profile; dotvault --export EXPORTED_FILENAME PASS_ARGUMENT') |uniq - | crontab -

```

- EXPORTED_FILEPATH should be the full path and name of where the .vault file should be created.
- PASS_ARGUMENT should be of the environment variable form as noted above. The above script assumes the environment variables and path are set in `.bash_profile`, you may need to modify this to suit your environment.

To confirm your job has been scheduled use the command `crontab -l` to see a list of all jobs.

*There are a few tricks going on with the above cron job command. Cron is run by the system and as a result has no access to the user's path or environment variables. Traditionally, the proper way to approach this roadblock is to ensure every cron job brings its own environment to execute in. Typically this is accomplished by setting necessary variables in the crontab file itself, or directly in the scripts that cron is expected to run. To avoid requiring duplicate config values or forcing people to modify this script to run it under cron, I've suggested including the user's .bash_profile before running the job.*








   