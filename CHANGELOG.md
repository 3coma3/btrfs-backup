### [1.4.0](https://github.com/3coma3/btrfs-backup/compare/v1.4...v1.3) (2021-05-27)

* Begin to use semantic versioning starting with this release
* Enhanced config file specification and multiple-location search
* Minor updates and improvements

### [1.3](https://github.com/3coma3/btrfs-backup/compare/v1.3...v1.2) (2021-05-25)

* Bugfixes
* Minor improvements

### [1.2](https://github.com/3coma3/btrfs-backup/compare/v1.2...v1.1) (2021-05-24)

* Bugfixes
* Improve output
* Confirmation / autoconfirmation for snapshot removal

### [1.1](https://github.com/3coma3/btrfs-backup/compare/v1.1...v1.0) (2021-05-22)

* Bugfixes
* Improvements with sudo interaction and user permissions
* The destination holding the snapshots is now by default omitted from new snapshots. To revert to the previous behaviour you can use a filter with the List Reset pattern as the first rule in the ruleset.

### [1.0](https://github.com/3coma3/btrfs-backup/releases/tag/v1.0) (2020-06-07)
* Multiple config support, with global defaults and per config overrides
* Regex operations: find and remove snapshots quickly
* Pluggable rotation handlers
* Integrated retention policy tester/simulator
* Online help
* Quiet mode
