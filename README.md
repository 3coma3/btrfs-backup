# btrfs-backup
A simple, yet flexible script for versioned backups of non-COW filesystems to btrfs backends using rsync.

In around 50 lines of code (omitting config, blank lines and comments), it provides the following:

* Incremental, in place, COW powered snapshots
* Custom GFS rotation scheme with arbitrary per level copies to maintain at the snapshot, day, week, month and year levels
* File globs for source specification in a compact form
* Dry-run mode

Its goals are to be easy to use, understand, modify and adapt to work as a cron job or manually, and be extended with btrfs / rsync features like deduplication, compression, network transports and miscellaneous options. Security may be added by stacking btrfs over LUKS.
