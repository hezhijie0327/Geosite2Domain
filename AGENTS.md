# AGENTS.md

This file contains guidelines and commands for agentic coding agents working in the Geosite2Domain repository.

## Project Overview

Geosite2Domain is a data repository that generates and maintains domain lists organized by category and service name. It processes domain data from v2fly's domain-list-community repository and outputs structured domain list files used for network filtering, proxy configuration, and firewall rules.

**Repository Structure:**
- `/category` - Contains category-specific domain lists (e.g., `category-ads-all.txt`, `category-ai-!cn.txt`)
- `/non-category` - Contains service-specific domain lists (e.g., `115.txt`, `18comic.txt`)
- `/.github/workflows` - Contains CI/CD workflow configuration

## Build/Update Commands

Since this is a data repository, building refers to generating the domain lists from source.

```bash
# Clone the source repository
git clone https://github.com/v2fly/domain-list-community.git

# Process category domain lists
cd domain-list-community
export EXPORT_LISTS=$(ls -1 data | grep 'category-' | tr '\n' ',' | sed 's/,$//')
go mod download
go run ./ --datapath=./data --outputdir=../category --exportlists=${EXPORT_LISTS}

# Process non-category domain lists
export EXPORT_LISTS=$(ls -1 data | grep -v 'category-' | tr '\n' ',' | sed 's/,$//')
go run ./ --datapath=./data --outputdir=../non-category --exportlists=${EXPORT_LISTS}

# Cleanup (optional)
cd ..
rm -rf domain-list-community
```

## Testing

This repository doesn't contain traditional unit tests as it primarily stores generated data. To verify changes:

```bash
# Validate file format (example validation script)
for file in category/*.txt; do
  # Check that each line follows the expected format
  grep -E '^(domain:|full:|regexp:)' "$file" > /dev/null || echo "Invalid format in $file"
done

for file in non-category/*.txt; do
  # Check that each line follows the expected format
  grep -E '^(domain:|full:|regexp:)' "$file" > /dev/null || echo "Invalid format in $file"
done

# Count generated files
echo "Category files: $(ls category | wc -l)"
echo "Non-category files: $(ls non-category | wc -l)"
```

## Code Style Guidelines

### File Naming Conventions
- Category files: `category-[name].txt` (e.g., `category-ads-all.txt`)
- Service files: `[servicename].txt` (e.g., `115.txt`, `18comic.txt`)
- Country/region suffixes: `-cn`, `-!cn`, `-jp`, etc.

### Domain List Format
All entries must follow one of these formats:
- `domain:example.com` - Standard domain entries
- `domain:example.com:@ads` - Advertisement-related domains
- `full:example.com/path` - Full URL paths
- `regexp:^pattern$` - Regular expression patterns

### Organization Rules
- Category files group domains by purpose (ads, AI, banking, etc.)
- Non-category files group domains by service/platform
- Each file should contain domains related to a single category or service
- Files are UTF-8 encoded with Unix line endings
- No blank lines at the end of files

### Import Style
Since this is a data repository, import guidelines apply to the Go toolchain in `.github/workflows/main.yml`:
- Keep dependencies minimal
- Use standard library packages when possible
- Document any external dependencies

### Formatting Standards
- All text files use UTF-8 encoding
- Unix line endings (LF)
- No trailing whitespace
- Domain entries are sorted alphabetically (when applicable)

### Naming Conventions
- Files: lowercase with hyphens for category files, exact service names for non-category files
- Country codes use ISO 3166-1 alpha-2 (e.g., `cn`, `jp`, `us`)
- Negative country indicators use prefix `!` (e.g., `!cn` for non-China domains)

### Error Handling
When processing domain data:
- Validate format before adding entries
- Handle malformed entries gracefully
- Log errors to stderr for debugging
- Continue processing other entries if one fails

### Commit Guidelines
- Use clear commit messages describing what domains were added/removed
- Reference related services or categories in commit messages
- Avoid commits that only change timestamps
- Group related changes in single commits

## CI/CD Workflow

The repository updates automatically via GitHub Actions:
- Scheduled to run twice daily at 04:00 and 16:00 UTC+8
- Workflow defined in `.github/workflows/main.yml`
- Builds both category and non-category domain lists
- Commits changes automatically

## Working with Domain Data

When modifying domain lists:
1. Always validate the format before committing
2. Ensure entries are placed in the correct files (category vs. non-category)
3. Use appropriate country/region modifiers when needed
4. Test regex patterns for correctness
5. Verify that full path entries include the protocol and path

## Environment Variables

The build process uses these environment variables:
- `EXPORT_LISTS` - Comma-separated list of data files to process
- `DATAPATH` - Path to the data directory (default: `./data`)
- `OUTPUTDIR` - Directory for generated lists (default: `../category` or `../non-category`)

## Common Operations

### Adding a new domain list
1. Create appropriate file in category or non-category directory
2. Follow naming conventions
3. Add domains using correct format
4. Validate the file format

### Modifying existing lists
1. Locate the correct file based on category or service
2. Make changes following the format rules
3. Validate entries
4. Test if using regex patterns

### Batch processing
When processing multiple files:
- Use shell scripts with proper error handling
- Log processing status for each file
- Validate all generated files
- Maintain consistent formatting across files

## License and Terms

This project is licensed under Apache License 2.0 with Commons Clause v1.0. Any modifications or additions must comply with these terms.