# Automatic Robust Backup

- [Automatic Robust Backup](#automatic-robust-backup)
  - [About](#about)
    - [Goals](#goals)
    - [Tools](#tools)
  - [Considerations](#considerations)
    - [Security](#security)
    - [For each their own](#for-each-their-own)
    - [Cloud synchronization](#cloud-synchronization)
    - [Granularity](#granularity)
    - [System backup](#system-backup)
    - [Task automation](#task-automation)
  - [Setup](#setup)
    - [Install files](#install-files)
    - [Configure ARB](#configure-arb)
      - [Setup tools](#setup-tools)
      - [Init Git in System repository](#init-git-in-system-repository)
      - [Init Borg repository](#init-borg-repository)
      - [Insert a password in pass](#insert-a-password-in-pass)
      - [Init gocryptfs in reverse mode in a repository](#init-gocryptfs-in-reverse-mode-in-a-repository)
      - [Configure rclone streams](#configure-rclone-streams)
  - [Utils](#utils)
    - [Borg](#borg)
      - [Show info](#show-info)
      - [List archives](#list-archives)
      - [Mount an archive in FUSE](#mount-an-archive-in-fuse)

## About
Automatic Robust Backup or A.R.B. is a group of scripts and systemd files that automate archiving and synchronization tasks done through external programs. The philosophy behind it is that it should make things sucks less by keeping things simple stupid, making good use of available mature open-source tools and avoiding the feature creep.

### Goals
- **Redundancy**: data should not be lost even if a catastrophic failure happens.
- **Automation**: after configuration it should not require intervention.
- **Performance**: tasks should be done in a timely manner and conserve limited resources like size when possible.
- **Encryption**: man-in-the-middle or server should not be able to read the data content.

### Tools
- **borg** is the most popular chunk-based deduplicated backup manager for home users.
- **gocryptfs** is a spiritual successor to ecryptfs, it is a mature,audited and has active development.
- **rclone** is a mature command line cloud storage manager that supports most if not all common providers.
- **rsync** is the best way to sync a directory to another local or network directory.
- **git** is the popular version control system.
- **pass** is the standard unix password manager.

## Considerations
### Security
Security is a big concern. While rclone and borg have their own encryption features and they probably good I see as a safer approach to use instead gocryptfs because encryption is it's main feature and it's audited. Although the repeated header pattern of borg files may be a vulnerability.

### For each their own
There is no ideal backup method. But for most users their data can be classified in an ABC fashion: few files that they really can't lose; data with average volume and importance; voluminous but not important data. And each of these categories will have their own ideal methods.

### Cloud synchronization
Cloud providers are a cheap way to have an off-site copy replicated in data centers globally. But some people may have a limited connection so they may find useful to instead sync a secondary repo with higher compression and heavy use of exclusion patterns while syncing a full repo to external disk or NAS.

### Granularity
File-based are a simpler solution but the controlled granularity of chunk-based is ideal. To sync a few large size files would be a pita because any modification would require a re-upload of the whole file. On the other side to sync a great number of small files directly would congest the API requests quota. While the size of borg files can be optimized for a seamless cloud sync.

### System backup
Arch Linux download and install packages fast so it's more practical to only backup the system configuration files and package list. After a fresh minimal install the user can run a script to recover the system settings. The advantages are: no need to restart; instantly done; no voluminous disk images or tar archives; versioned history of the system.

### Task automation
Why use systemd instead of simple tools like cron or anacron? Because desktops generally don't stay on 24/7 so there's a need for a tool that will reschedule missed tasks. Anacron does that but unfortunately it would require the scripts to run as root. While systemd allows the scripts to run in the user environment and provides it's own logging feature through journalctl.

## Setup
The scripts should work in any Arch based distros and also any other distro with little adaptations. Although it shouldn't be used by users that are unable to review and adapt the scripts to their needs and system.

### Install files
There's a script included that will automate part of the installation but there's no configuration helper and some programs should be manually configured.

`./setup.sh install`

### Configure ARB
You should edit and rename the template files adapting them to your needs.

#### Setup tools
To make things easier and more reliable you can open a terminal and source the configuration variables you set above.

`source $HOME/.local/share/arb/arb.config`

#### Init Git in System repository
`git -C "$system_repo" init`

#### Init Borg repository
`borg init -e none "$name_repo"`

#### Insert a password in pass
`pass insert $name`

#### Init gocryptfs in reverse mode in a repository
`gocryptfs -extpass pass -extpass $name_pass -init -reverse "$name_repo"`

#### Configure rclone streams
`rclone config`

## Utils
### Borg
`export BORG_REPO="$name_repo"`

#### Show info
`borg info`

#### List archives
`borg list`

#### Mount an archive in FUSE
`borg mount ::archiveName mountPoint`