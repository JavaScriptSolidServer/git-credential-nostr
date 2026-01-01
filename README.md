# git-credential-nostr

A git credential helper that enables `git push` using Nostr (NIP-98) authentication.

## Installation

```bash
npm install -g git-credential-nostr
```

## Quick Start

```bash
# Generate a keypair
git-credential-nostr generate

# Configure git
git config --global credential.helper nostr
git config --global nostr.privkey <your-64-char-hex-privkey>

# That's it! git push now uses NIP-98
git push
```

## How It Works

```
git push
    │
    ▼
git calls: git-credential-nostr get
    │
    ▼
reads: git config nostr.privkey
    │
    ▼
generates NIP-98 token (signed, time-bound, URL-bound)
    │
    ▼
outputs: username=nostr, password=<token>
    │
    ▼
git sends via Basic Auth → Server verifies → Push succeeds
```

## Configuration

### Required: Private Key

```bash
# Option 1: Store in git config
git config --global nostr.privkey <64-char-hex>

# Option 2: Store in file (more secure)
echo "<64-char-hex>" > ~/.nostr/privkey
chmod 600 ~/.nostr/privkey
git config --global nostr.keyfile ~/.nostr/privkey
```

### Optional: Restrict to Specific Hosts

```bash
# Only use nostr auth for these hosts
git config --global nostr.hosts "localhost solid.example.com"
```

## Commands

### Generate a new keypair

```bash
$ git-credential-nostr generate

Generated new Nostr keypair:

  Private key: a1b2c3...
  Public key:  d4e5f6...
  WebID:       did:nostr:d4e5f6...

Setup:

  git config --global nostr.privkey a1b2c3...

Add this to your ACL for write access:

  acl:agent <did:nostr:d4e5f6...>
```

## Server-Side ACL

Add your Nostr identity to the repository's `.acl` file:

```turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#nostr-writer>
    a acl:Authorization;
    acl:agent <did:nostr:YOUR_64_CHAR_HEX_PUBKEY>;
    acl:accessTo <./>;
    acl:default <./>;
    acl:mode acl:Read, acl:Write.
```

## Security

- **Private key never transmitted** - only signed tokens
- **Time-bound tokens** - 60 second validity window
- **URL-bound tokens** - only valid for the target repository
- **Use HTTPS in production** - tokens are sent via Basic Auth

### Private Key Storage

| Method | Security | Convenience |
|--------|----------|-------------|
| `nostr.privkey` in git config | ⚠️ Plaintext in ~/.gitconfig | ✅ Easy |
| `nostr.keyfile` pointing to file | ✅ Can restrict with chmod 600 | ✅ Easy |

## Compatible Servers

Works with any server that supports NIP-98 authentication via Basic Auth:

- [JavaScript Solid Server (JSS)](https://github.com/JavaScriptSolidServer/JavaScriptSolidServer)

## Protocol Details

This helper implements [NIP-98](https://nips.nostr.com/98) HTTP authentication, transmitted via Basic Auth for git compatibility:

1. Git requests credentials for a URL
2. Helper generates a signed NIP-98 event with:
   - `kind: 27235`
   - `["u", "<repository-base-url>"]`
   - `["method", "*"]` (wildcard for git's multiple requests)
3. Token is base64-encoded and sent as password with username `nostr`
4. Server decodes Basic Auth, extracts NIP-98 token, verifies signature

## License

AGPL-3.0-or-later
