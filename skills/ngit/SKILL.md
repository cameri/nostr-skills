---
name: ngit
description: Manage Git repositories using Nostr for decentralized coordination. Use to clone/add Nostr-based repositories, manage pull requests across Nostr relays, publish repositories to Nostr, and understand the hybrid Git+Nostr architecture.
license: CC-BY-SA-4.0
---

# ngit - Nostr Git

Work with Git repositories coordinated through Nostr, enabling decentralized repository discovery and PR management without centralized platforms.

GitHub: https://github.com/DanConwayDev/ngit-cli

## Installation

```bash
curl -sSL https://raw.githubusercontent.com/DanConwayDev/ngit-cli/refs/heads/main/install.sh | sh
ngit --version
```

---

## Core Architecture: Hybrid Git + Nostr

**Critical understanding:** ngit is not pure decentralized Git. It's a **hybrid system**:

- **Nostr layer** — repository metadata, discovery, and PR coordination on Nostr relays (lightweight, distributed)
- **Git server layer** — traditional Git servers (GitHub, Gitea, etc.) for actual code storage and syncing

Why? Nostr relays can't store large files. Git servers provide redundancy when multiple servers back the same repo.

**The flow:** Repository metadata published to Nostr → code lives on Git servers → PRs coordinated via Nostr events → maintainers discover repos and sync from Git servers.

---

## Unintuitive Differences from Standard Git

| Concept | Git (Familiar) | ngit (Unintuitive) | Why |
|---------|---|---|---|
| **Repository URL** | `https://github.com/user/repo.git` or `git@github.com:user/repo.git` | `nostr://npub1user/repo-id` or `nostr://user@domain.com/repo-id` | Nostr addresses are identities, not HTTP URLs. They resolve to Git servers via Nostr events. |
| **Discovering Git Servers** | You specify the server | Nostr events determine which Git servers are "official" | Same repo can be cloned from multiple servers. |
| **PR Branches** | Any branch name | Use `pr/` prefix convention (`pr/feature-name`) | Nostr discovery works by convention. |
| **Publishing a Repo** | Create on GitHub UI | Run `ngit init` (requires Nostr keypair, Git server already set up) | Publishing is explicit and publishes to Nostr relays. |
| **Identity** | Username/password or SSH key per platform | Nostr keypair (nsec/npub) | Your Nostr identity IS your git identity. No password recovery. |
| **Relay Configuration** | No equivalent | Repository visibility depends on which Nostr relays you configure | Different relays = different PR visibility (eventual consistency). |

---

## Key Workflows

### Publish a Repository
```bash
git init
# or: git clone <server>
ngit init
```
Result: Repository metadata on Nostr, discoverable under your Nostr identity.

### Clone a Nostr Repository
```bash
git clone nostr://npub1user/repo-id
git clone nostr://user@domain.com/repo-id
```
Under the hood: Queries Nostr relays for metadata → reads Git server URLs → clones from one of them.

### Submit a PR
```bash
git push -u origin pr/feature-name
# OR
ngit send --title "My Feature" --description "Details..."
```
Result: PR metadata on Nostr relays, branch available on Git servers.

### View Open PRs
```bash
ngit list
```

---

## Practical Gotchas

- **Relay coverage**: If a PR author publishes to `relay.example.com` and you only listen to `damus.io`, you won't see it.
- **Eventual consistency**: Different relays may have different repo states — normal.
- **Git server downtime**: Code is in branches, but cloning/pushing needs at least one server online.
- **Identity is permanent**: Losing your Nostr keypair means losing account access.
- **Public by default**: All repos and PRs broadcast to your configured relays — no private repos unless using encrypted relays.

---

## Common Tasks

| Task | Command |
|------|---------|
| First-time setup | `ngit account login` or `ngit account create` |
| Check relay config | `ngit status` |
| Push to existing repo | `git push origin branch-name` |
| See backing Git servers | `git clone nostr://npub1.../repo && cd repo && git remote -v` |
| Migrate GitHub repo to ngit | `git clone https://github.com/user/repo && cd repo && ngit init` |

---

## References

- [ngit GitHub](https://github.com/DanConwayDev/ngit-cli)
- [Nostr Protocol (NIP-01)](https://github.com/nostr-protocol/nips/blob/master/01.md)
