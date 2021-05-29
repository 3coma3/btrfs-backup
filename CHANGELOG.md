# Changelog

All notable changes to this project will be documented in this file. See [standard-version](https://github.com/conventional-changelog/standard-version) for commit guidelines.

### [1.4.2](https://github.com/3coma3/btrfs-backup/compare/v1.4.1...v1.4.2) (2021-05-29)

### [1.4.1](https://github.com/3coma3/btrfs-backup/compare/v1.4.0...v1.4.1) (2021-05-29)


### Bug Fixes

* bug in find() with --no-header ([62bb2e9](https://github.com/3coma3/btrfs-backup/commit/62bb2e97435ebcf763561850a1e96abe2252112b))


### Improvements

* configurable suffixes for file search ([54f26b8](https://github.com/3coma3/btrfs-backup/commit/54f26b89f18b866af4d6581ba8c690c74f0340b4))
* FIXME and others in cmd_test() ([9a2ae38](https://github.com/3coma3/btrfs-backup/commit/9a2ae380ad46a0b180640923f862e9e8a37c0b36))
* sudo-as-root prevention code ([f2d1bd0](https://github.com/3coma3/btrfs-backup/commit/f2d1bd0586d2c8f2aaae8ce106deee50d30a9a82))

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
