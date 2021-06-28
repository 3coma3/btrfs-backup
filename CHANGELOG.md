# Changelog

All notable changes to this project will be documented in this file. See [standard-version](https://github.com/conventional-changelog/standard-version) for commit guidelines.

## [1.5.0](https://github.com/3coma3/btrfs-backup/compare/v1.4.3...v1.5.0) (2021-06-28)


### Features

* choose progress meter configuration ([e3d5a37](https://github.com/3coma3/btrfs-backup/commit/e3d5a3792a26ddb70d593881e033b46112f14d28))
* configuration helper commands ([32075b0](https://github.com/3coma3/btrfs-backup/commit/32075b0af56ac190ecb0b7bffdef7e69209d80f8))


### Bug Fixes

* force naked file names to omit path ([0824731](https://github.com/3coma3/btrfs-backup/commit/0824731c2214265d2bd7475dfa5bf11e64988248))
* quotes in command line arguments ([d933c2e](https://github.com/3coma3/btrfs-backup/commit/d933c2ed7a8a496823b37f9ad6d45d068fdd3ef5))
* reset init arrays ([2c70992](https://github.com/3coma3/btrfs-backup/commit/2c70992d46cffc3c6530a1bb9507ccea864406ae))


### Improvements

* add long options ([1ff248c](https://github.com/3coma3/btrfs-backup/commit/1ff248c1e5cb8500f34fc7e5a0dd9412b1062a8d))

### [1.4.3](https://github.com/3coma3/btrfs-backup/compare/v1.4.2...v1.4.3) (2021-05-30)


### Bug Fixes

* race condition in cmd_snap() ([33cde61](https://github.com/3coma3/btrfs-backup/commit/33cde614deb576cc2e1330d433d3b822d5e86c41))
* remove duplicates in cmd_test() ([37bb051](https://github.com/3coma3/btrfs-backup/commit/37bb051180994087e48514c12f09d91d6266a01f))

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
