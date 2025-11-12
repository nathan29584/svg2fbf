# Changelog

All notable changes to svg2fbf will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial public release
- Core frame-by-frame animation generation
- Support for SVG transformation and viewBox handling
- Duplicate element detection and deduplication
- Gradient optimization
- Path optimization and precision control
- SMIL animation generation

### Changed
- Migrated to Apache License 2.0
- Simplified dependencies (numpy only)
- Organized project structure

### Fixed
- Fixed viewBox transformation bug for frames with negative coordinates
- Fixed XML serialization for self-closing tags
- Improved error handling and user feedback

## [0.1.0] - 2024-11-06

### Added
- Initial development version
- Basic FBF animation functionality
- Command-line interface
- SVG preprocessing and optimization
- Test suite with pixel-perfect validation

---

## Version History Notes

This project was previously developed privately and is now being prepared for open source release. The version history prior to 0.1.0 is not publicly documented.
