## User manual

### Installation

The script is contained in the file `backup` in this repo. You'll want to give it proper ownership, execution permissions, and have it in your `PATH` (or invoke it with some path, as you prefer). Running from a symbolic link is supported, the script will detect its real location.


### Configuration

Configuration variables are declared in the `CONFIG DECLARATION AND DEFAULTS` section at the top of the script. This is also where any defaults may be set.

Custom configuration files can be used to provide the variables, and will override any defaults defined above. The configuration file to use can be specified in the `config` variable or by passing the `-c` option to the script (see below for details).

The configuration files are useful to define different backup applications. As examples, I provide two that I've used at my laptop, you can find them under the `/config` subdirectory:

`full.conf`
Defines a full system backup stored on an external drive, intended to be issued manually

`homesnaps.conf`
Defines home directory snapshots (also stored under $HOME) that I issue hourly via cron



> **Note**
>
> Configuration files override the variables defined in the script, and command line options (where applicable) take precedence over both the configuration files and the script defaults.



**Configuration variables:**

`config_locations`
This array holds the locations where a configuration file (specified by the `config` variable or by using the `-c` option), is to be searched when only a file name was specified, with no directory information.

It defaults to these locations:

* The directory where the script is stored
* A `/config` subdirectory at the above location
* `'.'` the current directory

`config_suffixes`

Just as `config_locations` controls where and in what order to look for configuration files, this array controls *which file endings* will be attempted to match (and in what order), so you can also omit the extension when specifying a configuration.

It defaults to these suffixes:

* `''` (null string, ie: no suffix is added, just attempt to match exactly what was passed)
* `'.config'`
* `'.cf'`

Together these two arrays allow to choose a specific configuration file by path, or to have instead a controlled way in we can use "shortcuts" and let the script locate the configuration for us. To tune them, head to the `init()` function near the end of the script.

`config`
This can hardcode the configuration file to use. It can be a full path, a relative path or a "naked file name", without path information, to be searched as described above. 

The `-c` option overrides this variable

`destination`
This is an (absolute) path to a btrfs subvolume. The script will attempt to create it if it doesn't exist. Note that by default this location is omitted from backups, although this can be overridden with an rsync filter rule consisting of a ! (bang) character, ie, the List Clearing Filter Rule.

`sources`
This is an array where each entry is a bash "glob" path expansion expression, pointing to a filesystem portion you want to back up.

Examples of valid glob expressions are:

```bash
*/home*
```
Simple paths are valid globs. This would include the /home directory in the list of files to backup

```bash
*/home/*
```
This would include every subdirectory or file under /home

```bash
*/home/user/{b,d,m,p,s,t,v,w}*
```
This would include any file or subdirectory under /home/user, that starts with the letters b, d, m, p, s, t, v or w

```bash
*/home/user/{bash,cinnamon,config,dmrc,fzf,g{conf,i{mp,t},n{ome,upg},tkrc},icons,l{inuxmint,ocal},m{ozilla,ysql},p{rofile,utty},ssh,th{emes,underbird},vim,znc}*
```
This is a complex expression that illustrates nesting

Full documentation of the syntax is available in the bash man page, or easily found around the web (I recommend the reference at the [Wooledge Wiki](https://mywiki.wooledge.org/glob), or the Bash Hacker's Wiki).

Similar examples are already included in the provided configurations (and those are in fact paths I regularly make backups of). It isn't necessary though to put each and every item you want to backup in the `sources` list. It's better to list there only the *top level* items, and then use the `filters` array to refine what's included and excluded.

`filters`
This is an associative array, whose keys may include any *value* in the `sources` array.

When backing up a `sources` entry, the matching `filters` entry is passed to rsync. The value portion of a `filters` entry is a multiline string where each line begins with "-" to exclude an item or "+" to force inclusion. The full documentation for the filters behavior is available in the rsync manual. You can also check out some filters in the included sample configurations, they are pretty simple to use.

`retention`
This associative array describes your *retention policy*. Basically you say how many snapshots you want to keep per discrete snapshot, different day, different week, different month and different year.

The retention policy sets the rules to follow when you have to determine whether it is not necessary anymore to keep a specific snapshot. Say for example that your policy is a simple "keep the last 10 snapshots". When you get to make more than that amount of snapshots, the policy checker will mark the oldest snapshot for rotation.

What specifically is done when something is marked for rotation, depends on the *rotation handler* selected. Currently the only ones defined are "remove" and "test". The latter only used to "test policies" (doh).

More on this on the section dedicated to retention policy down this document.

`rotate_handler`
As stated above, this variable specifies what to do when a retention check decides to mark a specific snapshot for rotation for it to comply with the current policy. The only useful value here as of now is 'remove'. Rotation handlers are just bash functions with names like rotate_`name`, where `name` is the part that can be selected in the `rotate_handler` variable.

`weekstart`
This is used by the retention policy checks and should be set to whatever is the first day of the week where you live. Here 1 means monday, and 7 means sunday.

`dryrun`
You'll probably be using this variable dynamically, through the `-d` command line switch. Its value is prefixed to any line in the script where an operation that "changes things" is to be performed, (ie: that writes anything in any way). The `-d` switch sets it to something similar to `'echo'`. Here you can hardcode it to something in particular, should you need that.

`rsyncflags`
These are the recommended rsync flags used when updating a snapshot. Tune as needed.

`flags_verbose`

These are the flags that control verbosity for most commands that are used by the script. They are turned off with the `-q` switch.

`rsyncflags_verbose`
Same as above, but specifically for rsync (as it has many options for verbosity). This is good to control the script output when using long rsync updates. It's also turned off with `-q`.

`header_char`

The character used as filler for the header lines that are output by the script.


###  Invocation

#### Actions
`help`
It will print a quick help for reference

`snap`
Will setup the base location for the snapshots if this is the first run, and then diligently rsync the files to the first snapshot. Subsequent invocations will base the new snapshots on the last one available.

`rotate`
Will scan the snapshot storage, applying your retention policy as expressed in the `retention` and `weekstart` variables. Snapshots that get marked for rotation are then sent to the configured `rotation_handler` to do its job. 

`test n`
Will generate `n` fake snapshots with random dates, and check them against your retention policy. This is useful to build a policy that behaves exactly how you want, and also to clear up any doubts you may have about the retention algorithm behavior.

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
Configuration file to use, same logic as with the `config` variable.

`-d`
It prevents the script to perform any actual operation, making it to print the actions that would be performed on the screen instead.

`-q`
Currently, it restricts a bit the rsync output in normal situations and some other commands. In future updates the verbosity controls may become more powerful. Note that most script messages are output to standard error, so they affect its "pipeable" output to a minimum.

`-y`
Assume 'yes' for questions. This is useful for automated invocations. Note that the script also accepts piped output from the "yes" command.



### Backup storage structure
As you take snapshots, each one will be stored in a mixed structure composed of btrfs subvolumes and standard directories that'll look like this:

***`base`***
***`|--year/month/day/hhmmss`***

**base** is a btrfs subvolume as specified in the ```destination``` configuration variable

Now `year/month/day` are standard subdirectories, and the `hhmmss` "leaf" is an actual btrfs snapshot with the backed up contents. All the time related tokens in the structure are taken from the time the snapshots are made, so *the path to a snapshot is simultaneously, its name and time stamp*.

This also implies that a directory representing a year will have at most 12 subdirectories beneath it, a month something between 28 and 31, and a day can have theoretically 86400 snapshots, provided you don't clean it :-)

And since we're talking about cleaning...



### Retention algorithm
The algorithm is based on the classic GFS (Grandfather-Father-Son) scheme, which lets you have some number of items according to some form of nesting or leveling. Here the nesting is given by the subdirectory structure, and the leveling is given by each node in that structure: the directories for years, months and days when backups were made, and the "leaf" folders named with the hour, minute and second when the backups were made.

Even when the leafs aren't subdirectories but btrfs snapshots, from the user perspective there's little difference in the interaction with the tree.

The gist of this is expectedly, you tell the script just how many snapshots to keep at each level, and the older ones are progressively phased out. **A snapshot will be kept as long as one retention rule marks it as valid**.

Let us see an example of a simple snapshot and retention policy in action:

Let's say that at the first minute each hour, as long as your computer is running, you take a snapshot of your home directory. Just after you take a snapshot, you apply the `rotate` action, in order to remove the old snapshots so they don't pile up.

But, you still want access to *some* older snapshots.

Let's say: 

* Keep the last **48** snapshots made ("leaf snapshots"), which would theoretically support having an hourly sample of `$HOME` for the last two days.

* Before removing the last snapshot for some day, keep it as a "day snapshot". You want to do this for the last **14** days. Now you have also a daily $HOME sample for the last two weeks.

You extend this logic to promote days to weeks, weeks to months, and months to years, keeping the last **4** weeks, **12** months and **5** years. This policy keeps a sample "density" that is more sparse as time passes.

You begin to use this policy, and two full days are about to pass. You now have **48** snapshots with this directory structure:

`/BKP_ROOT/2020/01/01/`**`000100` <- first snapshot of the first day**
`/BKP_ROOT/2020/01/01/010100`
`/BKP_ROOT/2020/01/01/020100`
`...`
`/BKP_ROOT/2020/01/01/`**`230100` <- last snapshot of the first day**
`/BKP_ROOT/2020/01/02/`**`000100` <- first snapshot of the second day**
`/BKP_ROOT/2020/01/02/010100`
`/BKP_ROOT/2020/01/02/020100`
`...`
`/BKP_ROOT/2020/01/02/`**`230100` <- last snapshot of the second day**

Let's focus first on what happens to the very first snapshot: `/BKP_ROOT/2020/01/01/000100` .



**`2020/01/03/000100`**
The next snapshot in the sequence is the **first of the third day**.

With **49** snapshots, the retention is evaluated, and now for the first time your **leaf** retention policy marks a snapshot for rotation: "`/BKP_ROOT/2020/01/01/000100` is the 49th snapshot in existence and thus should be rotated".

Your **day** retention policy instead, says that it should be *kept*, because it's the last one that represents 2020/01/01, only three days have passed, and we are still far from 14 days. So the daily retention policy prevents the rotation. At this point, we can say that the snapshot was *promoted* to daily. It will still be kept even as newer snapshots are marked for deletion by the leaf policy, at least for now.



**`2020/01/15/000100`**
After another 12 days, the next snapshot in the sequence is the **first of the 15th day**. The leaf policy keeps tagging the first snapshot as ready for rotation, but now the daily policy agrees, as we don't keep more than 14 snapshots in daily promotion.

The **week** policy won't mark it as valid either. Why? Because it keeps the *last* snapshot for a week. Since 01/01/2020 is Wednesday, we have still daily snapshots for that same week, and hence this is not the one to promote to weekly.

Monthly and yearly policies don't even apply here, as not even one month has passed. And so the snapshot gets rotated.



**`2020/01/20/000100`**
Four more days pass, and (assuming you have selected Monday as the first day of the week in the configuration) it's now time to evaluate the **last snapshot of the first week**: `/BKP_ROOT/2020/01/05/230100`.

All previous snapshots are outside of what the hourly and daily policies cover, and are already out of the snapshot tree. The leaf and daily policies have been tagging this snapshot for rotation already, but in this case the weekly retention does prevent its rotation... as the snapshot is **still younger than four weeks**... so it's still kept, promoted to weekly snapshot.



**`2020/02/03/000100`**
We need to get to this point to get beyond four Sunday weekly snapshots, and are going to see what happens at this point with  `/BKP_ROOT/2020/01/05/230100`. 

Will the monthly retention policy keep the snapshot? Not at all, as the policy will look only at the *last* snapshot of a month for promotion. Yearly? Still doesn't apply, but still, it will likely have newer snapshots to promote eventually.



**Conclusion**

In practice, because a rotation can be prevented by some retention level other than the one you are interested in, you may end up storing more snapshots at each level than what you might have expected, depending on your policy. This can be observed easily with the leaf category.

On the other hand, a lower retention for days and "upper" categories, would get you closer to actually having the leaf policy more closely followed.

For these reasons the `test` comes really handy in testing and modeling the behavior of different retention policies. It generates as many fake snapshots as you want (only in names, nothing is actually created anywhere), then sends them to the policy checker, reporting back which snapshots would be marked for rotation and which ones not. You can get an outlook of how your backup store would actually look like after some time of using it.

The test generator uses random years in the near range of the current year minus the yearly retention, with random months, days, and leaf time data. Because of this it's better oriented at modeling snapshot stores created with manual invocations of the script, or with custom schedules that not regular cron jobs.

I intend to add more controls for the simulation behavior, such as fixed intervals at each level, enabling to simulate long-time snapshot stores as they would be actually constructed with cron jobs.




### Restore procedure
The restore procedure consists of the following deceptively simple steps:

1. Locate the desired snapshot
2. Copy the files to wherever you want them to be

Sorry! Doesn't get more complicated than that.
