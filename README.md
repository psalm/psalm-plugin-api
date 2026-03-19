# psalm-plugin-api

This package must be required by [Psalm](https://psalm.dev) plugins in order to require a specific version of the Psalm plugin API.  

This is a separate, empty metapackage: its major version will be bumped every time a breaking change occurs within Psalm's plugin API, breaking changes will be listed in this README; minors will also be bumped when adding new features to Psalm's plugin API.  

This allows breaking changes within Psalm's plugin API, without requiring a major in Psalm itself (which will be reserved only for breaking changes in the CLI API or other major behavioral changes).  

Current stable release: `1.0.0`.

## Changelog

### `0.1.0`

Initial release, covers Psalm `7.0.0-beta17`.  

All breaking changes up to this point are listed in [vimeo/psalm#UPGRADING.md](https://github.com/vimeo/psalm/blob/master/UPGRADING.md).  

All breaking changes in future releases will be listed here.  

### `1.0.0`

Covers Psalm v7 (to be defined&released).