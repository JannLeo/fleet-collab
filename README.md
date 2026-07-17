# fleet-collab

A Hermes Agent skill for **multi-machine collaboration over Tailscale** using a shared kanban board and SSH remote dispatch.

Any node can create tasks; a central hub dispatches ready tasks to remote workers via SSH — no shared filesystem (NFS/SSHFS) required.

## Architecture

```
Hub (Tailscale 100.x)
├── shared kanban.db @ ~/.hermes/kanban/boards/fleet-collab/
├── fleet_dispatch.sh   (dispatches ready tasks to workers)
└── fleet_create.sh     (creates/list/shows tasks locally)

Workers (Tailscale 100.x, each running its own Hermes)
├── fleet_create.sh     (SSH callback to hub to create/list/show)
└── own hermes + model endpoint
```

The hub is the single source of truth for the board. Workers create tasks by SSH-calling back into the hub's `hermes kanban create`. The hub dispatches tasks to workers by SSH-ing into each worker's own `hermes chat` to execute the task body, then writing the result back to the board.

## Why not the SSH terminal backend?

Hermes's built-in `terminal.backend: ssh` force-syncs the hub's `~/.hermes/skills` (42 dirs) to the remote on every spawn — slow and overwrites the remote's config. This skill avoids that by SSH-ing into the *remote's own* hermes instead, with zero file sync.

## Quick start

```bash
# 1. On hub: create board + worker profiles
hermes kanban boards create fleet-collab --switch
hermes profile create worker-beefy --clone

# 2. Set up passwordless SSH both ways (hub↔workers)

# 3. Fix NO_PROXY on any node running a local HTTP proxy (Clash etc.):
#    add 100.64.0.0/10 + each node's Tailscale IP to ~/.hermes/.env

# 4. Deploy scripts, edit the HUB_* / FLEET_MAP placeholders

# 5. From any node, create a task:
~/fleet_create.sh "compile firmware" --assignee worker-beefy --body "make and report size"

# 6. On hub, dispatch:
~/fleet_dispatch.sh --task t_xxxxx
# or continuously:
~/fleet_dispatch.sh --loop 30
```

See `SKILL.md` for the full playbook, prerequisites, pitfalls, and verification.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Hermes skill metadata + usage docs |
| `scripts/fleet_create.sh` | Create / list / show tasks (any node) |
| `scripts/fleet_dispatch.sh` | Dispatch ready tasks to workers (hub only) |

## License

MIT
