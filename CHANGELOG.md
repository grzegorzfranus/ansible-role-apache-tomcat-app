# Changelog

## [1.2.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v1.1.0...v1.2.0) (2026-07-02)


### Features

* make logrotate su option optional ([#19](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/19)) ([717acfb](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/717acfb533e908f5df580317e6042c6fe54f23de))

## [1.1.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v1.0.0...v1.1.0) (2026-06-27)


### Features

* add apache tomcat app role for context and config management ([#1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/1)) ([77ebd3e](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/77ebd3e026859cf28d2eac3e4737c080a225592b))
* add optional per-app logrotate configuration ([#13](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/13)) ([d634adf](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/d634adf8add163e4c51e672c2d5182eb42185f6a))
* add support for Enterprise Linux 8 (EL8) ([#5](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/5)) ([cbf7e8a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/cbf7e8a55c139342d8401a52ed570c80c497d91f))
* add support for per-app custom writable directories ([#3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/3)) ([1ce184d](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/1ce184d692fc05384c510d6e5eec64f8a06979dc))


### Bug Fixes

* generate systemd drop-in for application custom write paths ([#9](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/9)) ([6a45dc7](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/6a45dc710a3cdb10fcfbb1a05a6f0511de9ce8c6))


### Documentation

* document systemd drop-in override for custom writable paths in README ([#11](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/11)) ([61b9e3a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/61b9e3a433b1e2d25d6df693e027781953a7a051))


### Miscellaneous

* **main:** release 0.2.0 ([#2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/2)) ([9425da8](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/9425da881d6114317caed076464d2c328bef202f))
* **main:** release 0.3.0 ([#4](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/4)) ([8b9aebd](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/8b9aebd6c339970d8b020eb259d34d0a09b0c2c8))
* **main:** release 0.4.0 ([#6](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/6)) ([2f8fad3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/2f8fad3a813e986aea490991fcabd08295392135))
* **main:** release 0.4.1 ([#10](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/10)) ([54d1561](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/54d15613e94c8e58d829f513df6ea346101a16de))
* **main:** release 0.4.2 ([#12](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/12)) ([7bab728](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/7bab728fe09cf4b0d0041858d7327696e28ac790))
* **main:** release 0.5.0 ([#14](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/14)) ([6acb380](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/6acb380c1ffb23348974dcca6d2247c3ceba13a8))
* migrate workflows to github-workflows and enable galaxy publishing ([#17](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/17)) ([75efb02](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/75efb0280a63c012211620d40c96b6ece86e9b30))

## [0.5.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.4.2...v0.5.0) (2026-06-22)


### Features

* add optional per-app logrotate configuration ([#13](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/13)) ([d634adf](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/d634adf8add163e4c51e672c2d5182eb42185f6a))

## [0.4.2](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.4.1...v0.4.2) (2026-06-22)


### Documentation

* document systemd drop-in override for custom writable paths in README ([#11](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/11)) ([61b9e3a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/61b9e3a433b1e2d25d6df693e027781953a7a051))

## [0.4.1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.4.0...v0.4.1) (2026-06-22)


### Bug Fixes

* generate systemd drop-in for application custom write paths ([#9](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/9)) ([6a45dc7](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/6a45dc710a3cdb10fcfbb1a05a6f0511de9ce8c6))

## [0.4.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.3.0...v0.4.0) (2026-06-19)


### Features

* add support for Enterprise Linux 8 (EL8) ([#5](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/5)) ([cbf7e8a](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/cbf7e8a55c139342d8401a52ed570c80c497d91f))

## [0.3.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.2.0...v0.3.0) (2026-06-19)


### Features

* add support for per-app custom writable directories ([#3](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/3)) ([1ce184d](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/1ce184d692fc05384c510d6e5eec64f8a06979dc))

## [0.2.0](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/compare/v0.1.0...v0.2.0) (2026-06-09)


### Features

* add apache tomcat app role for context and config management ([#1](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/issues/1)) ([77ebd3e](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/commit/77ebd3e026859cf28d2eac3e4737c080a225592b))

## Changelog
