# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Git credential helper that enables `git push` using Nostr (NIP-98) authentication. When git requests credentials, this helper generates a signed NIP-98 token (kind 27235) and returns it as a Basic Auth password with username `nostr`.

## Architecture

Single-file CLI tool (`bin/git-credential-nostr`) implementing the git credential helper protocol:

- **`get` command**: Called by git. Reads stdin for credential request params, reads private key from git config (`nostr.privkey`) or keyfile (`nostr.keyfile`), generates NIP-98 event, outputs `username=nostr` and `password=<base64-token>`
- **`generate` command**: Creates new Nostr keypair and prints setup instructions
- **`store`/`erase` commands**: No-ops (tokens are generated on-the-fly)

The NIP-98 token includes:
- `kind: 27235`
- `["u", "<repository-base-url>"]` - URL binding
- `["method", "*"]` - wildcard for git's multiple HTTP methods

## Dependencies

Uses `nostr-tools` for Nostr event creation and signing. No build step - runs directly via Node.js ESM.

## Testing Locally

```bash
# Generate test keypair
./bin/git-credential-nostr generate

# Simulate git credential get (type input then empty line)
echo -e "protocol=https\nhost=example.com\npath=/repo\n" | ./bin/git-credential-nostr get
```

## Git Config Options

- `nostr.privkey` - 64-char hex private key
- `nostr.keyfile` - path to file containing private key
- `nostr.hosts` - space-separated list to restrict which hosts use nostr auth
