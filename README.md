<div align="center">

<img src="assets/banner.svg" alt="Claude Sync" width="100%">

<br>

*Encrypted with [age](https://github.com/FiloSottile/age) • R2 / S3 / GCS supported*

[![Release](https://img.shields.io/github/v/release/tawanorg/claude-sync)](https://github.com/tawanorg/claude-sync/releases)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![npm](https://img.shields.io/npm/v/@tawandotorg/claude-sync)](https://www.npmjs.com/package/@tawandotorg/claude-sync)

[Quick Start](#quick-start) • [Setup Guide](#setup-guide) • [Commands](#commands) • [Security](#security)

</div>

---

## Features

- **Cross-device sync**: Continue Claude Code conversations on any laptop
- **Auto-sync**: Automatically pull on session start, push on session end via Claude Code hooks
- **Fast**: Concurrent uploads/downloads (10 workers) with gzip compression
- **Multi-provider storage**: Cloudflare R2, AWS S3, or Google Cloud Storage
- **End-to-end encryption**: All files compressed and encrypted with age before upload
- **Passphrase-based keys**: Same passphrase = same key on any device (no file copying)
- **Configurable excludes**: Skip large directories (plugin caches, etc.) from sync
- **Cross-platform**: Windows, macOS, and Linux binaries
- **Interactive wizard**: Arrow-key driven setup with validation
- **Self-updating**: `claude-sync update` to get the latest version

<div align="center">
<img src="assets/claude-sync.gif" alt="Claude Sync Demo" width="100%">
</div>

## Quick Start

### First Device

```bash
# Install
npm install -g @tawandotorg/claude-sync

# Set up (interactive wizard)
claude-sync init

# Push your sessions
claude-sync push
```

### Second Device

```bash
# Install
npm install -g @tawandotorg/claude-sync

# Set up with SAME storage credentials
claude-sync init
# Select same provider (R2/S3/GCS)
# Enter same bucket name and credentials
# Choose "Passphrase" for encryption
# Enter the SAME passphrase as first device
# ✓ Encryption key verified  <-- confirms passphrase matches!

# Preview what would be synced
claude-sync pull --dry-run

# Pull sessions (creates backup if you have existing files)
claude-sync pull
```

**Same passphrase = same encryption key.** The init verifies your passphrase can decrypt remote files before completing.

## Setup Guide

### Step 1: Choose a Storage Provider

| Provider | Free Tier | Best For |
|----------|-----------|----------|
| **Cloudflare R2** | 10GB storage | Personal use (recommended) |
| **AWS S3** | 5GB (12 months) | AWS users |
| **Google Cloud Storage** | 5GB | GCP users |

### Step 2: Create a Bucket

<details>
<summary><b>Cloudflare R2</b> (recommended)</summary>

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/) → R2 Object Storage
2. Click "Create bucket" → name it `claude-sync`
3. Go to "Manage R2 API Tokens" → "Create API Token"
4. Select **Object Read & Write** permission → Create

You'll need: Account ID, Access Key ID, Secret Access Key
</details>

<details>
<summary><b>AWS S3</b></summary>

1. Go to [S3 Console](https://s3.console.aws.amazon.com/s3/bucket/create) → Create bucket
2. Go to [IAM Security Credentials](https://console.aws.amazon.com/iam/home#/security_credentials)
3. Create Access Keys

You'll need: Access Key ID, Secret Access Key, Region
</details>

<details>
<summary><b>Google Cloud Storage</b></summary>

1. Go to [Cloud Storage](https://console.cloud.google.com/storage/create-bucket) → Create bucket
2. Go to [Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts) → Create service account
3. Grant "Storage Object Admin" role → Create JSON key

You'll need: Project ID, Service Account JSON file (or use `gcloud auth application-default login`)
</details>

### Step 3: Run Init

```bash
claude-sync init
```

The interactive wizard will guide you through:

1. **Select storage provider** (R2, S3, or GCS)
2. **Enter credentials** (provider-specific)
3. **Choose encryption method**:
   - **Passphrase** (recommended) - same passphrase on all devices = same key
   - **Random key** - must copy `~/.claude-sync/age-key.txt` to other devices
4. **Test the connection** to verify everything works

### Step 4: Push and Pull

```bash
# Upload local changes
claude-sync push

# Download remote changes
claude-sync pull
```

## What Gets Synced

| Path | Content |
|------|---------|
| `~/.claude/projects/` | Session files, auto-memory |
| `~/.claude/history.jsonl` | Command history |
| `~/.claude/agents/` | Custom agents |
| `~/.claude/skills/` | Custom skills |
| `~/.claude/plugins/` | Plugins |
| `~/.claude/rules/` | Custom rules |
| `~/.claude/settings.json` | Settings |
| `~/.claude/CLAUDE.md` | Global instructions |

## Limitations

### Path-Based Session Indexing

Claude Code indexes project sessions by **absolute filesystem path**. This tool syncs `~/.claude/projects/` but does not perform path remapping, which means:

```
/Users/alice/Projects/my-app → ~/.claude/projects/-Users-alice-Projects-my-app/
/Users/bob/code/my-app       → ~/.claude/projects/-Users-bob-code-my-app/
```

These are treated as **completely different projects** by Claude Code. After syncing, `claude --resume` on machine2 won't find sessions from machine1 if the project paths differ.

**Workaround:** Use consistent absolute paths across all devices. For example:

- Always clone repos to `~/Projects/` on every machine
- Use symlinks to maintain the same path structure
- Consider a standardized home directory structure

If you follow a consistent path convention, sessions will sync and resume correctly across devices.

## Commands

```bash
claude-sync init        # Set up configuration (interactive wizard)
claude-sync push        # Upload local changes to cloud storage
claude-sync pull        # Download remote changes from cloud storage
claude-sync status      # Show pending local changes
claude-sync diff        # Show differences between local and remote
claude-sync conflicts   # List and resolve conflicts
claude-sync auto        # Manage automatic sync via Claude Code hooks
claude-sync reset       # Reset configuration (forgot passphrase)
claude-sync update      # Update to latest version
claude-sync --help      # Show all commands
```

### Auto-Sync (Recommended)

Set up Claude Code to sync automatically on every session:

```bash
claude-sync auto enable    # Install hooks (pull on start, push on end)
claude-sync auto disable   # Remove hooks
claude-sync auto status    # Check if enabled
```

When enabled, Claude Code will:
- **Pull** the latest data from cloud when a session starts
- **Push** your changes to cloud when a session ends

No manual `push`/`pull` needed.

### Pull Options

```bash
claude-sync pull              # Normal pull (prompts if existing files)
claude-sync pull --dry-run    # Preview what would change
claude-sync pull --force      # Skip confirmation prompts
```

### Init Options

```bash
claude-sync init              # Full setup wizard
claude-sync init --passphrase # Re-enter passphrase only (keeps storage config)
claude-sync init --force      # Reset everything, start fresh
```

### Quiet Mode

```bash
claude-sync push -q     # No output (for scripts)
claude-sync pull -q
```

### Check for Updates

```bash
claude-sync update --check   # Check without installing
claude-sync update           # Download and install latest version
```

## Excluding Paths

Skip large or unnecessary directories from sync by adding an `exclude` list to `~/.claude-sync/config.yaml`:

```yaml
exclude:
  - plugins/marketplaces   # cached plugin registry clones
  - plugins/cache          # resolved plugin versions
  - "*.tmp"                # glob pattern support
```

Patterns support glob matching, prefix matching, and exact matches. Excluded paths are skipped during push, pull, diff, and status operations. Excluded directories are skipped entirely (not walked), which also speeds up change detection.

## Pulling with Existing Files

When you pull on a device that already has `~/.claude` files, claude-sync will:

1. **Show what would change** - files that would be overwritten, kept, or downloaded
2. **Ask for confirmation** - choose to backup, overwrite, or abort
3. **Create a backup** - saves existing files to `~/.claude.backup.{timestamp}`

```bash
# Preview first
claude-sync pull --dry-run

# Pull with prompts
claude-sync pull

# Skip prompts (for scripts)
claude-sync pull --force
```

## Conflict Resolution

When both local and remote files change, the remote version is saved as `.conflict`:

```bash
claude-sync conflicts            # Interactive resolution
claude-sync conflicts --list     # Just list conflicts
claude-sync conflicts --keep local   # Keep all local versions
claude-sync conflicts --keep remote  # Keep all remote versions
```

Interactive options:
- **[l]** Keep local (delete conflict file)
- **[r]** Keep remote (replace local)
- **[d]** Show diff
- **[s]** Skip
- **[q]** Quit

## Wrong Passphrase?

If you entered the wrong passphrase on a new device:

```bash
# Re-enter passphrase (keeps your storage config)
claude-sync init --passphrase
```

The init will verify your passphrase can decrypt remote files before completing.

## Forgot Passphrase?

The passphrase is **never stored**. If you forget it:

1. Your encrypted files cannot be recovered
2. Reset and start fresh:

```bash
claude-sync reset --remote   # Delete remote files and local config
claude-sync init             # Set up again with new passphrase
claude-sync push             # Re-upload from this device
```

## Security

- Files encrypted with [age](https://github.com/FiloSottile/age) before upload
- Passphrase-derived keys use Argon2 (memory-hard KDF)
- Passphrase is never stored - only the derived key at `~/.claude-sync/age-key.txt`
- Cloud storage is private (API key/IAM auth)
- Config files stored with 0600 permissions

## Cost

Claude sessions typically use < 50MB. Syncing is effectively **free** on any provider:

| Provider | Free Tier |
|----------|-----------|
| **Cloudflare R2** | 10GB storage, 1M writes, 10M reads/month |
| **AWS S3** | 5GB for 12 months (then ~$0.023/GB) |
| **Google Cloud Storage** | 5GB, 5K writes, 50K reads/month |

## Installation Options

### npm (recommended)

**Prerequisite:** Node.js 14+ (no Go required - downloads pre-compiled binary)

```bash
# Global install
npm install -g @tawandotorg/claude-sync

# Or one-time use
npx @tawandotorg/claude-sync init
```

### GitHub Packages

**Prerequisite:** Node.js 14+

```bash
# Add to ~/.npmrc
echo "@tawanorg:registry=https://npm.pkg.github.com" >> ~/.npmrc

# Install
npm install -g @tawanorg/claude-sync
```

### Download Binary

**Prerequisite:** None

```bash
# macOS ARM (M1/M2/M3)
curl -L https://github.com/tawanorg/claude-sync/releases/latest/download/claude-sync-darwin-arm64 -o claude-sync
chmod +x claude-sync
sudo mv claude-sync /usr/local/bin/

# Windows (download .exe from releases page)
# https://github.com/tawanorg/claude-sync/releases/latest
```

Available platforms: Windows (amd64/arm64), macOS (amd64/arm64), Linux (amd64/arm64).

See [GitHub Releases](https://github.com/tawanorg/claude-sync/releases) for all binaries.

### Go Install

**Prerequisite:** Go 1.21+ (for developers)

```bash
go install github.com/tawanorg/claude-sync/cmd/claude-sync@latest
```

### Build from Source

**Prerequisite:** Go 1.21+

```bash
git clone https://github.com/tawanorg/claude-sync
cd claude-sync
make build
./bin/claude-sync --version
```

## Development

```bash
make test          # Run tests
make fmt           # Format code
make check         # Run all pre-commit checks
make build-all     # Build for all platforms
make setup-hooks   # Enable git pre-commit hooks
```

## License

MIT
