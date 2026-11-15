# AGENTS.md

Geosite2Domain is a data repository that generates and maintains domain lists organized by category and service name from v2fly's domain-list-community, used for network filtering, proxy configuration, and firewall rules. See [README.md](README.md) for the Chinese overview.

## HANDOVER.md

When a multi-step task is interrupted, write progress to `HANDOVER.md` so the next session can pick up where you left off. Include: what was done, what remains, key decisions made, and the next concrete step. Delete the file when the task is complete.

## Behavioral Guidelines

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

**Additional rules:**
- Be concise in output but thorough in reasoning.
- Do not re-read files you have already read.
- User instructions always override this file.
- Don't wrestle with indentation in the Edit tool — focus on logic, let the project's formatter fix formatting. Use `sed` or `python3` freely when the Edit tool struggles with whitespace.

## Git commit convention

Every commit subject starts with a type prefix — `feat` / `fix` / `chore` / `docs` / `revert` / `refactor` / `perf` / `test` / `build` — optionally scoped: `fix(toolbar): ...`. Subject: imperative, lowercase start, no trailing period, aim ≤ 72 chars; detail goes in the body after a blank line (why, not what); breaking changes take `!` before the colon or a `BREAKING CHANGE:` body footer. Never rewrite pushed history.

- Commit incrementally — one logical change per commit, after lint and tests pass. Present changes for review before committing.
- PRs: link the issue, summarize the change and verification (tests/benchmarks), and include config/log/pprof evidence for bug fixes.
- Use clear commit messages describing what domains were added/removed
- Reference related services or categories in commit messages
- Avoid commits that only change timestamps
- Group related changes in single commits

### Co-Author Format

- Always add co-author information. If closing an issue in commit text, verify against `main` first: `gh issue view <issue-id>`.
- Only ONE co-author line is allowed. If multiple agents contributed, aggregate into ONE entry.

Format: `Co-authored-by: <AgentName> <Email>`

Valid examples (choose EXACTLY ONE):

```
Co-authored-by: ZCode <noreply@z.ai>
```

## Documentation format

Family-wide Markdown rules, shared verbatim across all ZJ projects:

- **Language**: AGENTS.md, AUDIT.md, DESIGN.md and other agent-facing docs are English; README.md is Chinese (human-facing). Keep literal UI strings and code identifiers verbatim.
- **AGENTS.md skeleton** — generic sections first, then project; omit sections that don't apply, keep the order:
  `# AGENTS.md` → 1–2 sentence intro (what the project is + key doc links) → Behavioral Guidelines → Git commit convention → Documentation format → Project → Commands → Architecture → Conventions (Gotchas as a subsection) → project-specific sections → Key Docs.
- **README.md skeleton**: `# <project>` → badge block → one-line positioning + stack + key links → 特性 → 快速开始 → 配置 / 用法 → 文档 → License (omit as needed).
- **README badges** (shields.io, after the title, in order): Version (only where a real version exists, linked to the repo's GitHub releases), License (Apache 2.0 with Commons Clause — every project), primary-language badge (Go / TypeScript / Bash / Python / Docker), Biome for TypeScript projects; pure data/config repos get License only. Version links use the real GitHub repo name.
- **Formatting**: ATX headings (`#`), never skip a level; code fences always carry a language tag (`bash`, `go`, `json`, `text` for trees); tables for structured reference; no decorative emoji in headings or bullets; bold for key terms only.

## Project

Geosite2Domain is a data repository that generates and maintains domain lists organized by category and service name. It processes domain data from v2fly's domain-list-community repository and outputs structured domain list files used for network filtering, proxy configuration, and firewall rules.

**Repository Structure:**
- `/category` - Contains category-specific domain lists (e.g., `category-ads-all.txt`, `category-ai-!cn.txt`)
- `/non-category` - Contains service-specific domain lists (e.g., `115.txt`, `18comic.txt`)
- `/.github/workflows` - Contains CI/CD workflow configuration

## Commands

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

## Conventions

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
