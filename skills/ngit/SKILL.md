---
name: ngit
description: Manage Git repositories using Nostr for decentralized coordination. Use to clone/add Nostr-based repositories, manage pull requests across Nostr relays, publish repositories to Nostr, and understand the hybrid Git+Nostr architecture.
license: CC-BY-SA-4.0
---

# ngit - Nostr Git

Work with Git repositories coordinated through the Nostr protocol, enabling decentralized repository discovery, PR management, and collaboration without relying on centralized platforms.

GitHub: https://github.com/DanConwayDev/ngit-cli

## Installation

To install (or upgrade) ngit:

```bash
curl -sSL https://raw.githubusercontent.com/DanConwayDev/ngit-cli/refs/heads/main/install.sh | sh
```

Verify installation:
```bash
ngit --version
ngit --help
```

---

## Core Architecture: The Hybrid Model

**Critical understanding:** ngit is NOT a pure decentralized Git replacement. It's a **hybrid system** combining:

1. **Nostr layer** — decentralized repository metadata, discovery, and PR coordination on Nostr relays
2. **Git server layer** — traditional Git servers (GitHub, Gitea, etc.) for actual code storage and syncing

This hybrid design creates behaviors that differ significantly from pure Git workflows:

### Why Hybrid?

- **Nostr relays alone cannot store large files** → use Git servers for code
- **Nostr relays are discoverable without prior knowledge** → use for metadata/coordination
- **Multiple Git servers can serve the same repository** → redundancy without central control

### The Flow

```
User publishes to Nostr:  Repository metadata event (nip29 repository manifest)
                          ↓
User configures Git:      Clone/push to Git server (traditional Git workflow)
                          ↓
User publishes PR:        PR metadata to Nostr + patch/branch to Git server
                          ↓
Maintainer discovers:     Finds repo on Nostr, fetches from Git server
                          ↓
Maintainer reviews:       Reviews PR branch, sees status on Nostr
                          ↓
Maintainer merges:        Pulls branch from Git server, pushes to main
```

---

## Domain Knowledge: Unintuitive Differences from Git

### 1. Repository Addresses vs. HTTP/SSH URLs

**Git (familiar):**
```bash
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git
```

**ngit (unintuitive):**
```bash
git clone nostr://npub1user/repo-identifier
git clone nostr://user@domain.com/repo-identifier
```

**The difference:** 
- Nostr addresses are human-readable Nostr identities (npub, nip05) not HTTP addresses
- They resolve to Git servers via Nostr events, not DNS
- The same Nostr repository can be cloned from multiple Git servers

### 2. PR Workflow Deviates from GitHub Flow

**GitHub (familiar):**
```bash
git push origin pr-branch
# Go to GitHub UI, create PR
```

**ngit (unintuitive):**
```bash
git push -u origin pr/my-branch
# Automatically published to Nostr as PR event
# OR use: ngit send --title "..." --description "..."
```

**The difference:**
- PR metadata lives on Nostr relays (transparent, distributed)
- PR branches conventionally use `pr/` prefix (not just any branch name)
- `ngit list` shows all open PRs across all relays
- Maintainers might not see your PR if they're not monitoring the same relays

### 3. Remote Configuration is Hidden but Critical

**Git (familiar):**
```bash
# Your remote points directly to the server
git remote -v
# origin  https://github.com/user/repo.git (fetch)
```

**ngit (unintuitive):**
```bash
git clone nostr://npub1user/repo
# Git remotes are auto-populated from Nostr metadata
# Nostr events determine which Git servers are "official"
# Multiple remotes may be added automatically
```

**The difference:**
- You don't specify the Git server URL—Nostr events do
- The same repository identifier can have multiple Git server backends
- If all configured servers go offline, you can still clone from backups listed in Nostr

### 4. Publish/Init is Not Intuitive

**Git (familiar):**
```bash
# Create repo on GitHub UI
# git clone locally
# git push
```

**ngit (unintuitive):**
```bash
git init
ngit init
# Publishes repository metadata to Nostr relays
# Links to Git server(s) where code is stored
```

**The difference:**
- Publishing is explicit (`ngit init`) and publishes to Nostr
- You must already have a Git server set up (self-hosted or external)
- The Nostr event contains pointers to Git servers, not the code itself

### 5. Relay Configuration Affects Discoverability

**Git (no equivalent):**
- Repository visibility depends on which Nostr relays carry your repo metadata
- Two users on different relay sets may not see the same PRs
- Different relays can have different states (eventual consistency)

### 6. Account Management is Nostr-Native

**Git (familiar):**
```bash
# Username/password or SSH key on GitHub
git config user.email "user@example.com"
git config user.name "User Name"
```

**ngit (unintuitive):**
```bash
ngit account login
# Uses Nostr keypairs (nsec/npub)
# Your Nostr identity IS your git identity
# No separate username/password
```

---

## Key Workflows

### Publishing a Repository to Nostr

```bash
# 1. Create repo locally or have existing repo
git init
# or: git clone <server>

# 2. Publish to Nostr (creates metadata event)
ngit init

# 3. Configure which Git servers back this repo
# (via ngit metadata or editing Nostr event)
```

**Result:** Repository is discoverable on Nostr under your Nostr identity

### Cloning a Nostr Repository

```bash
# Using Nostr address (npub)
git clone nostr://npub1user/repo-identifier

# Using NIP-05 address (human-readable)
git clone nostr://user@example.com/repo-identifier
```

**Under the hood:**
1. Queries Nostr relays for repository metadata event from that pubkey
2. Reads Git server URLs from the Nostr event
3. Clones from one of the listed Git servers

### Submitting a Pull Request

```bash
# Method 1: Push branch with pr/ prefix
git push -u origin pr/feature-name

# Method 2: Use ngit send for more control
ngit send --title "My Feature" --description "Details..."
```

**Result:** PR metadata published to Nostr relays, branch available on Git servers

### Viewing Open PRs

```bash
ngit list
```

**Shows:**
- All open PRs across your subscribed relays
- PR authors, titles, descriptions
- Which branches are available on Git servers

---

## Practical Gotchas

1. **Relay Coverage:** If a PR author publishes to `relay.example.com` and you only listen to `damus.io`, you won't see their PR. Ask for relay recommendations.

2. **Eventual Consistency:** Different relays may have different versions of the repo state. This is normal.

3. **Git Server Downtime:** Code is still in Nostr metadata (via branches), but cloning/pushing requires at least one Git server online.

4. **Identity is Permanent:** Your Nostr keypair is your identity. Losing it means losing account access (no password recovery).

5. **Public by Default:** All repositories and PRs are broadcast to Nostr relays you configure. There's no "private" repo in ngit unless you use encrypted relays.

---

## Common Tasks

### Set up ngit for the first time
```bash
ngit account login
# or: ngit account create
```

### Check your current Nostr relay configuration
```bash
ngit status
# or: git config --list | grep ngit
```

### Push to an existing Nostr repository
```bash
git push origin branch-name
```

### See what Git servers are backing a Nostr repo
```bash
# Clone it and inspect remotes
git clone nostr://npub1.../repo
cd repo
git remote -v
```

### Migrate a GitHub repo to ngit
1. `git clone https://github.com/user/repo`
2. `cd repo && ngit init`
3. Add Git server(s) in ngit metadata
4. Users can then: `git clone nostr://npub1.../repo`

---

## References

- [ngit GitHub](https://github.com/DanConwayDev/ngit-cli)
- [Nostr Protocol (NIP-01)](https://github.com/nostr-protocol/nips/blob/master/01.md)
- [NIP-29: Relay-based Groups](https://github.com/nostr-protocol/nips/blob/master/29.md) — related to repository coordination
