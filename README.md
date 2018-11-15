# btrfs-backup
A simple, yet flexible script for versioned backups of non-COW filesystems to btrfs backends using rsync.

In around 50 lines of code (omitting config, blank lines and comments), it provides the following:

* Incremental, in place, COW powered snapshots
* Custom GFS rotation scheme with arbitrary per level copies to maintain at the snapshot, day, week, month and year levels
* File globs for source specification in a compact form
* Dry-run mode

Its goals are to be easy to use, understand, modify and adapt to work as a cron job or manually, and be extended with btrfs / rsync features like deduplication, compression, network transports and miscellaneous options. Security may be added by stacking btrfs over LUKS.


### but I've seen already a million other scripts for backup, why another one?
Because there is not a single one out there that has the same characteristics, so I had to wrote it.

All the scripts out there either:
* Are made to manage snapshots / backups for native btrfs filesystems

This script just needs something that rsync can read from (including networked sources). Of course, it needs a btrfs *destination* to store and manage the backups.

* Have too many impositions/assumptions about how you should structure your backups. Either you have to name or initialize your destinations in some specific way, or you should cron the executions in another way, or learn yet another config file syntax. 

This script will run whenever you want, as many times as you want, called from command line or from cron jobs, in any way you want. The configuration is strictly:

1) What you want to backup
2) Where will you store it
3) How many snapshots do you want to exist there per (discrete snapshot | different day | different week | different month | different year).

**That's it**

* Are too restrictive in the granularity of snapshots. For example, you can have only N yearly backups, or X monthly, etc.

See above.

* Require you to understand new syntaxes to use them.

Heh. Guess what there's only two subcommands: "snap" and "clean", and a "-d" switch if you want to test what is going to happen in dry mode. The three parameters can be specified in any order.

**That's it**

The only syntax you'll be benefited to know is that of bash globs, for a more efficient way of expressing what you want to backup. However this is not mandatory. You can just write each file or directory in its own line and it will work, as simple strings are also valid globs.

* Have dependencies like a database in which to store metadata, or some extra language interpreter.

Here you have three dependencies: rsync (doh), bash (doh) and btrfs (doooooh). No extra software, just 50 lines of bash. Ok, to be honest it uses also "sed" "mkdir" and "rm" too, so you'll need to download them ;-)

### mmkay, so how are you saying I should use it?
1) Copy the file "backup" in this repo, give it execution permissions, and have it in your $PATH (or invoke it with some path, as you prefer)

2) Edit the file, between the lines that read ```CONFIG START``` and ```CONFIG END```. Modify ```sources```, ```destination```, ```retention``` and ```weekstart``` to suit your needs. They are self-explanatory and commented in the script.

3) Invoke the script:

```backup snap```
Will setup the base location for the snapshots if this is the first run, and then diligently rsync the files to the first snapshot. Subsequent invocations will base the new snapshots on the last one available.

```backup clean```
Will scan the snapshot storage, applying your retention policy as expressed in the ```retention``` and ```weekstart``` variables. Details of the algorithm are below in the **retention algorithm** section.

You can chain as many invocations of those two subcommands and they will be executed in order:

```backup clean snap```
Will first apply the retention policy and then take a snapshot.

```backup snap clean```
Will do the opposite, first take a snapshot and then "clean" the snapshot repository.

etc.

```backup -q action(s)```
To see what would be done without actually making any modifications. Use it to test your sources and retention policy (although for this last task there's another script in this repository made to that specific effect).

### Backup storage structure
As you take snapshots, each one will be stored in a mixed structure composed of btrfs subvolumes, snapshots and standard directories that'll look like this:

***base***    <--- a btrfs subvolume as specified in the ```destination``` configuration variable

***|***

***|--year/month/day/hhmmss*** <- year/month/day are subdirectories, hhmmss is the snapshot for this backup. All the time related items in the structure are taken from the time you make the snapshot, so the full path to the snapshot identifies it and also *when* you made it.

This also means that a directory representing a year will have at most 12 subdirectories beneath it, a month something between 28 and 31, and a day can have theoretically 86400 snapshots, provided you don't clean it :-)

And since we're talking about cleaning...

### Retention algorithm
The algorithm is based on the classic GFS (Grandfather-Father-Son) scheme, which lets you have some number of items according to some form of nesting or leveling. Here the nesting is given by the subdirectory structure, and the leveling is given by each node in that structure: the directories for years, months and days when backups were made, and the "leaf" folders named with the hour, minute and second when the backups were made.

Actually the leafs aren't subdirectories but btrfs snapshots, but from the user perspective there's little difference when you interact with the tree.

The gist of it is that you tell the script just how many items of each level you want to exist *at the minimum*:

You might want to keep the last 15 snapshots you made, and after that save at least 1 for the last 7 days, and then at least the last one for the last 4 weeks, and so on for the months and years. You then go on making backups, in an ordered or unordered fashion, and when you apply the cleaning procedure the script will try to respect those numbers, and delete everything else, starting from the oldest snapshots available.

If you want to test how it would work for different scenarios, I recommend to use the companion script, aptly named ```test-retention```. You can tune the configuration in the same fashion as for the backup script, and then invoke it as:

```test-retention test N``` where N is the number of snapshots to simulate. It will print out a list of virtual snapshots and how would the configured retention policy affect their deletion.

### Restore procedure
The restore procedure consists of two basic steps:

1) Navigate to the snapshot desired

2) Copy the files to whenever you want them to be

***AND... THAT'S IT***



