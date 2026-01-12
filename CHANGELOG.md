# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Deprecated

### Removed

### Fixed

## [v1.1.1] - 2026-01-12

### Fixed

- Updated **Relation types** section to explicitly define `merkle-proof`, resolving a 
contradiction where the spec previously stated no new relation types were introduced.
- Clarified in "Linking to a Proof" that the **Root Catalog** acts as the trust anchor and therefore does not require a proof link.

## [v1.1.0] - 2026-01-12

### Added

- **"Deep Integrity"** specification: defined how to bind metadata hashes to physical file checksums using the [File Info Extension](https://github.com/stac-extensions/file).
- `Dependencies` field to the extension header.
- **Merkle Proofs**: introduced the `merkle-proof` link relation and the JSON schema for proof objects to enable efficient verification.

### Changed

- **Hash Computation** rules to strongly recommend including `assets` (specifically `file:checksum`) 
in the `merkle:object_hash` to ensure data integrity.
- Item, Collection, and Catalog examples to demonstrate usage with the `File Info Extension` and `Merkle Proof` links.

## [v1.0.0] - 2024-11-05

- move extension to stacchain org

## [v1.0.0-beta.2] - 2024-10-18

- update schema links via template

## [v1.0.0-beta.1] - 2024-10-17

- first release

[Unreleased]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.1.1...main
[v1.1.1]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.1.0...v1.1.1
[v1.1.0]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.0.0...v1.1.0
[v1.0.0]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.0.0-beta.2...v1.0.0
[v1.0.0-beta.2]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.0.0-beta.1...v1.0.0-beta.2
[v1.0.0-beta.1]: https://github.com/stacchain/merkle-tree-stac-extension/tree/v1.0.0-beta.1
