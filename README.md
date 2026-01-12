# psalm-plugin-api

This package must be required by [Psalm](https://psalm.dev) plugins in order to require a specific version of the Psalm plugin API.  

This is a separate, empty metapackage: its major version will be bumped every time a breaking change occurs within Psalm's plugin API; minors will also be bumped when adding new features to Psalm's plugin API.  

This allows breaking changes within Psalm's plugin API, without requiring a major in Psalm itself (which will be reserved only for breaking changes in the CLI API or other major behavioral changes).  

Current release: `1.0.0`.

## Changelog

### `1.0.0`

First release.