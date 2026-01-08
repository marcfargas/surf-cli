# Changelog

## [Unreleased]

### Added
- **Perplexity AI integration** - Query Perplexity using browser session via `surf perplexity "query"`. Supports `--with-page` for context, `--mode` for search modes, and `--model` for model selection (Pro features).
- **`surf read` now includes visible text by default** - Reduces agent round-trips by returning both accessibility tree and page text content in one call. Use `--no-text` to get only interactive elements.

### Changed
- `surf read` behavior changed: now includes `--- Page Text ---` section by default
- Added `--no-text` flag to `surf read` to exclude text content (previous default behavior)

### Fixed
- Fixed text content not being included when screenshots were also present in page read responses

### Removed
- Removed non-functional base64 image output from CLI (was not being interpreted by agents)
