# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## Unreleased

Upcoming changes will be listed here. Please check this section before filing a
feature request.

- SSD baseline comparison showing percentage of "typical" SSD performance

## 2.1.0 - 2018-06-17
### Added
- HDD baseline comparison showing percentage of "typical" HDD performance
- Repeating a specific test now shows raw fio command for reference
- Repeating very important tests as "Take 2", showing possible inconsistencies
### Changed
- Running a specific test no longer outputs all test group headers
### Removed
- Misleading hint "To run only a specific test, add its name in quotes
  as argument" as there is better explanation in --help

## 2.0.0 - 2018-06-02
### Changed
- First portable release, adding FreeBSD and OpenIndiana to the test suite
- Many tests replaced with new ones to gain portability, thus creating
  backwards-incompatible release
