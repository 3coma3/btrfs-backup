![Image](https://raw.githubusercontent.com/3coma3/btrfs-backup/master/image.png)

# btrfs-backup
I know there are already a million btrfs/rsync scripts out there, I know. I checked them out. It happened that every tool I tested was either:

* made to manage snapshots for native btrfs filesystems (based on btrfs-send)
* having too many impositions/assumptions about how you should structure your backups: you must name or initialize your destinations in some specific way, or cron the executions in another way
* too restrictive in the granularity and retention of snapshots, for example: you can have only N yearly backups, or X monthly, etc
* dependent on things like a database in which to store metadata, or some extra language interpreter to understand, or yet another config file syntax to learn

I wanted something lean and mean, that does the job and gets out of the way, and has minimal dependencies. I needed an easy to navigate snapshot structure, and I wanted a retention / rotation scheme.  I wanted ease of use and above everything, ease of understanding, so anyone can modify it.

So I wrote this little piece, and I'm pretty satisfied with the results:

* instead of new languages, the native syntax and features of rsync, bash and sed/grep regular expressions are exposed through the script, simplifying adoption and leveraging documentation and implementation already there
* the only state is the backup store itself, which holds the only metadata used so far: the timestamp for each backup, that is *also* nicely encoding the navigational structure. In other words: the database is the filesystem. If more metadata support is added in the future, it will be done through extended attributes, so if you want indexes or caches, you can add them as you need with any of the many external tools available
* internally, liberal use of stream operations allows for a high degree of code reutilization, rendering the code clean and making easy to modify and extend it

> **Update**
>
> The last couple of commits add many new features, which made me think to declare this in a "1.0 beta" state. Please test and report any issues observed. Feature requests and any other input are welcome!
> 
> Also, the code has grown from about 50 lines to 150, which is still small for what it packs, but not so minimalistic anymore as it was so far. Feel free to check the earlier versions, I will add tags soon to the best ones to aid in their use.

## Features

* Plain bash. No DSL, no complex parameters, no nothing.
* ~150 lines of code omitting config, comments and blank lines.
* No impositions. It may be used as needed, manually or via cron. It does its job and gets out of the way.
* Does Incremental, in place, COW powered snapshots out of any file system Rsync can read, courtesy of Btrfs.
* Custom GFS-style rotation algorithm to maintain arbitrary copies at the snapshot, day, week, month and year levels.
* Compact data source specification in extended bash "glob" [pattern syntax](https://mywiki.wooledge.org/glob)
* Compact per-data-source filter specification in rsync's own [pattern syntax](https://manpages.ubuntu.com/manpages/eoan/en/man1/rsync.1.html#include/exclude%20pattern%20rules)
* Queued operations
* Dry-run mode

**new in version 1.0:**
* Multiple config support, with global defaults and per config overrides
* Regex operations: find and remove snapshots quickly
* Pluggable rotation handlers
* Integrated retention policy tester/simulator
* Online help
* Quiet mode


## User guide

### Installation

The script is contained in the file `backup` in this repo. You'll want to give it proper ownership, execution permissions, and have it in your $PATH (or invoke it with some path, as you prefer). Running from a symlink is supported, as the script detects its real location.


### Configuration

Config variables are declared in the CONFIG DECLARATION AND DEFAULTS section at the top of the script. This is also where any defaults may be set.

Custom configuration files, placed in the same directory as this script, can set (or override) the configuration variables as needed, for different backup applications. I include the two that I use at my laptop:

`full.conf`
Defines a full system backup stored on an external drive, to be issued manually

`homesnaps.conf`
Defines home directory snapshots (also stored under $HOME) that are issued via cron, hourly


The configuration variables:

`destination`

This is an (absolute) path to a btrfs subvolume. The script expects that you don't create this volume, so just specify it here and on the first run it will be created.


`sources`

This is an array where each entry is a bash "glob" path expansion expression, pointing to a filesystem portion you want to back up.

Examples of valid glob expressions are:

*/home*

Simple paths are valid globs. This would include the /home directory in the list of files to backup

*/home/*

This would include every subdirectory or file under /home

*/home/user/{b,d,m,p,s,t,v,w}*

This would include any file or subdirectory under /home/user, that starts with the letters b, d, m, p, s, t, v or w

*/home/user/{bash,cinnamon,config,dmrc,fzf,g{conf,i{mp,t},n{ome,upg},tkrc},icons,l{inuxmint,ocal},m{ozilla,ysql},p{rofile,utty},ssh,th{emes,underbird},vim,znc}*

This is a complex expression that illustrates nesting

Full documentation of the syntax is available in the bash man page, or easily found around the web. Similar examples are already included in the provided configurations (these are in fact the paths I regularly make backups of).

It isn't necessary though to put each and every item you want to backup in the `sources` list. It's better to list there only the *top level* items, and then use the `filters` array to refine what's included and excluded.


`filters`

This is an associative array, whose keys may include any *value* in the `sources` array.

When backing up a `sources` entry, the matching `filters` entry is passed to rsync. The value portion of a `filters` entry is a multiline string where each line begins with "-" to exclude an item or "+" to force inclusion. The full documentation for the filters behaviour is available in the rsync manual. You can also check out some filters in the included sample configurations, they are pretty simple to use.


`retention`

This associative array describes your *retention policy*. Basically you say how many snapshots you want to keep per discrete snapshot, different day, different week, different month and different year.

The retention policy sets the rules to follow when you have to determine whether it is not necessary anymore to keep a specific snapshot. Say for example that your policy is a simple "keep the last 10 snapshots". When you get to make more than that amount of snapshots, the policy checker will mark the oldest snapshot for rotation.

What specifically is done when something is marked for rotation, depends on the *rotation handler* selected. Currently the only ones defined are "remove" and "test". The latter only used to "test policies" (doh).

More on this on the section dedicated to retention policy down this document.


`rotate_action`

As stated above, this variable specifies what to do when a retention check decides to mark a specific snapshot for rotation for it to comply with the current policy. The only useful value here as of now is 'remove'. Rotation handlers are just bash functions with names like rotate_`name`, where `name` is the part that can be selected in the `rotate_action` variable.


`weekstart`

This is used by the retention policy checks and should be set to whatever is the first day of the week where you live. Here 1 means monday, and 7 means sunday.


`dryrun`

You'll probably be using this variable dynamically, through the `-d` command line switch. Its value is prepended at any line in the script where an operation that "changes things" is to be performed, (anything that writes anything in any way). The `-d` switch sets it to something similar to 'echo'. Here you can hardcode it to something in particular, should you need that.


`config`

This variable is set by the `-c` switch, and should point to a configuration file name that exists in the same directory as the script. 


`rsyncflags`

These are the recommended rsync flags used when updating a snapshot. Tune as needed.


`rsyncflags_verbose`

These are the rsync flags that control verbosity. They are turned off with the `-q` switch to tune down the script output which is in great part affected by long rsync updates.


###  Invocation

#### Actions
`help`
It will print a quick help for reference
 
`snap`
Will setup the base location for the snapshots if this is the first run, and then diligently rsync the files to the first snapshot. Subsequent invocations will base the new snapshots on the last one available.

`rotate`
Will scan the snapshot storage, applying your retention policy as expressed in the `retention` and `weekstart` variables. Snapshots that get marked for rotation are then sent to the configured `rotation_handler` to do its job. 

`test n`
Will generate `n` fake snapshots with random dates, and check them against your retention policy. This is useful to build a policy that behaves exactly how you want, and also to clear up any doubts you may have about the retention algorithm behaviour.

`find`
Will scan the snapshot storage, filtering those snapshots whose name (also, their path) matches a regular expression and printing them on the screen.

`remove`
Like `find` but instead of printing names, it will destroy the snapshots. Use with care of course. 

You can "chain" any of the actions above, they will be queued and executed in order:

`backup rotate snap`
Will first apply the retention policy and then take a snapshot.

`backup snap rotate`
Will do the opposite, first take a snapshot and then check the snapshot repository against the policy.

`backup snap find '.*'`

Will print an updated list of the snapshots after taking the last one.

and so on.


#### Command line switches ("options"):

`-h`
An alias to `help`


`-c config`
Use it to select which configuration file to load

`-d`
It prepends an 'echo' to any modifying operation, making the script to print those actions on the screen instead of running them.

`-q`
Currently, it restricts a bit the rsync output in normal situations. In future updates the verbosity controls may become more powerful. Note that most script messages are outputted to standard error, so they affect its "pipeable" output to a minimum.



### Backup storage structure
As you take snapshots, each one will be stored in a mixed structure composed of btrfs subvolumes and standard directories that'll look like this:

***base***

a btrfs subvolume as specified in the ```destination``` configuration variable

***|--year/month/day/hhmmss***

The `year/month/day` part of a snapshot structure is made with common subdirectories, while the "leaf" directory `hhmmss` is the actual btrfs snapshot with the backed up contents. All the time related tokens in the structure are taken from the time the snapshots are made, so *the full path to the snapshot is at the same time its name, and its time stamp*.

This also means that a directory representing a year will have at most 12 subdirectories beneath it, a month something between 28 and 31, and a day can have theoretically 86400 snapshots, provided you don't clean it :-)

And since we're talking about cleaning...



### Retention algorithm
The algorithm is based on the classic GFS (Grandfather-Father-Son) scheme, which lets you have some number of items according to some form of nesting or leveling. Here the nesting is given by the subdirectory structure, and the leveling is given by each node in that structure: the directories for years, months and days when backups were made, and the "leaf" folders named with the hour, minute and second when the backups were made.

Actually the leafs aren't subdirectories but btrfs snapshots, but from the user perspective there's little difference when you interact with the tree.

The gist of this is expectedly, you tell the script just how many snapshots to keep at each level, and the older ones will be progressively phased out. To see how this behaves exactly, a good option is to think about an example, so let's imagine a simple retention policy.

Let's say that at the first minute each hour, as long as your computer is running, you take a snapshot of your home directory. You want to rotate it removing the older snapshots so they don't pile up in masses, but you still want to access some of the older snapshots. Let's say: 

* Keep the last **48** snapshots you made (leafs), which could theoretically support having an hourly sample of $HOME for the last two days.

* Before removing all the snapshots for the same day, you keep the last one made that day as the "day snapshot". You want to do this for the last **14** days. Now you have also a daily $HOME sample for the last two weeks.

You extend this logic to promote days to weeks, weeks to months, and months to years, keeping the last **4** weeks, **12** months and **5** years.

Two full days are about to pass, and you have 48 snapshots with this directory structure:

/BKP_ROOT/2020/01/01/000100 *<- first backup of the first day*

/BKP_ROOT/2020/01/01/010100

/BKP_ROOT/2020/01/01/020100

...

/BKP_ROOT/2020/01/01/230100 *<- last backup of the first day*

/BKP_ROOT/2020/01/02/000100 *<- first backup of the second day*

/BKP_ROOT/2020/01/02/010100

/BKP_ROOT/2020/01/02/020100

...

/BKP_ROOT/2020/01/02/230100 *<- last backup of the second day*

When the next snapshot is taken, your **leaf** retention policy says that the snapshot at /BKP_ROOT/2020/01/01/000100 should be rotated. Your **day** retention policy says instead that it should be kept, because it's the last one that represents 2020/01/01!

This gets to the heart of how GFS is interpreted here:

___"a snapshot is rotated ONLY if it isn't required to fullfill ANY of the retention rules"___

In practice, this means that you may end up storing more snapshots at each category than what you might have expected, depending on your policy. On the other hand, a lower retention for days and "upper" categories, would get you closer to actually having the leaf policy more closely followed.

This gets even more complicated by the fact that this script can be used manually, or with custom schedules that might not be so regular and simple as "one snapshot per hour, every hour, always".

It is for this reason that the `test` action is so useful. It generates as many fake snapshots as you want (only in names, nothing is actually created anywhere), then sends those names/timestamps to the policy checker to test your retention policy, reporting back which snapshots would be marked for rotation and which ones not. You can get an outlook of how your backup store would actually look like after some time of using it.

Note that still the test generator is somewhat harcode-y and less flexible than I would like, and it isn't actually able to mimic many specific situations. Maybe sometime I will create a better one, where one can model more closely and test real scenarios being projected.


### Restore procedure
The restore procedure consists of the following deceptively simple steps:

1. Locate the desired snapshot
2. Copy the files to wherever you want them to be

Sorry! Doesn't get more complicated than that.
