# K8sQuest: Learn Kubernetes by Fixing It

## What It Is
K8sQuest is a local, game-style Kubernetes troubleshooting trainer. You break things in a local Kubernetes cluster on purpose, then fix them with `kubectl`. It has **50 levels across 5 worlds**, covering everything from basics (pods, deployments) to advanced security and scheduling.

- **Total XP:** ~10,200 across all levels
- **Target audience:** Beginners to intermediate engineers, including DevOps/SRE interview prep
- **Cloud:** None required — runs entirely locally on your machine
- **Cluster:** Uses `kind` by default; `k3s` is also supported

## How to Run

### Prerequisites (install once)
- **Docker Desktop** (running)
- **kubectl**
- **kind** (Kubernetes in Docker)
- **jq**
- **Python 3.9+**

On macOS: `brew install kubectl kind jq python3`

### Setup (once)
From the repo root:
```bash
./install.sh
```
This checks prerequisites, creates a Python venv, installs `rich` and `pyyaml`, and creates the kind cluster `k8squest`.

### Play
Terminal 1 (keep open):
```bash
./play.sh
```

Terminal 2 (your kubectl workspace):
```bash
kubectl get pods -n k8squest
kubectl describe pod <name> -n k8squest
# ... fix things ...
```

Back in Terminal 1, type:
- `check` — live view of resources in the namespace
- `hints` — unlock progressive hints (up to 3 per level)
- `solution` — see the fixed manifest
- `validate` — check if you've fixed it
- `skip` — move to the next level
- `quit` — exit the game

Progress is saved automatically to `progress.json`, so you can resume later.

## Repository Layout

```
.
├── engine/              # Game engine
│   ├── engine.py        # Main game loop (1583 lines)
│   ├── reset.py         # Reset a level or all levels
│   ├── safety.py        # Guards on destructive operations
│   ├── retro_ui.py      # Terminal UI
│   ├── player_name.py   # DevOps-themed name generation
│   └── certificate.py   # Completion certificates
├── worlds/              # The 50 challenges
│   ├── world-1-basics/       (10 levels)
│   ├── world-2-deployments/  (10 levels)
│   ├── world-3-networking/   (10 levels)
│   ├── world-4-storage/      (10 levels)
│   └── world-5-security-scheduling/ (10 levels)
├── docs/                # Documentation
├── tools/               # Level scaffolding scripts
│   ├── create_level.sh
│   └── generate_level.py
├── utils/               # Testing and auditing
│   ├── audit_levels.sh
│   └── test-all-levels.sh
├── rbac/                # RBAC manifests to limit player access
├── certificates/        # Sample certificates
├── tests/               # Test suites (test_safety.py)
├── play.sh              # Launcher
├── install.sh           # Setup script
└── Dockerfile           # iximiuz Labs packaging
```

## Anatomy of a Level

Each level is a folder like `worlds/world-1-basics/level-8-logs/`:

| File | Purpose |
|------|---------|
| `mission.yaml` | Title, description, XP, difficulty, concepts |
| `broken.yaml` | The faulty manifest (what you start with) |
| `solution.yaml` | The fixed manifest (for reference) |
| `validate.sh` | Pass/fail check — exits 0 if the fix is correct |
| `hint-1.txt`, `hint-2.txt`, `hint-3.txt` | Progressive hints (one unlocked per `hints` command) |
| `debrief.md` | Post-win explanation: why the fix works, mental models, real incidents, interview Q&A |
| `common-mistakes.md` | (Optional) Common missteps |

Example: Level 8 (logs) breaks a postgres pod by omitting `POSTGRES_PASSWORD`, so the container crashes. `validate.sh` checks that the pod is Running, Ready, has 0 restarts, and the password is set.

## Engine Lifecycle

1. **Deploy** (`engine.py` ~line 864): Wipes the `k8squest` namespace and applies `broken.yaml`.
2. **Investigate** (your terminal): Use `kubectl` to understand the problem.
3. **Fix** (your terminal): Apply the fix (e.g., `kubectl apply -f solution.yaml` or edit in place).
4. **Validate** (`engine.py` ~line 967): Runs `bash validate.sh` in the level directory. Exit code 0 = pass.
5. **Win**: XP is added, progress is saved to `progress.json`, the debrief is displayed.
6. **Reset** (`engine/reset.py`): Deletes the namespace and re-applies `broken.yaml`.

Progress is stored in `progress.json`:
- `total_xp` — cumulative XP
- `completed_levels` — array of cleared level slugs
- `current_world` — resume point
- `player_name` — your DevOps-themed name (e.g., "SwiftSRE")

## Curriculum by World

| World | Topic | Levels |
|-------|-------|--------|
| **1. Basics** | Pods, image pull, labels, ports, logs, init containers, namespaces | 1–10 |
| **2. Deployments** | Rollbacks, liveness & readiness probes, HPA, rollouts, PDB, blue/green, canary, StatefulSet, ReplicaSet | 11–20 |
| **3. Networking** | Service selectors, NodePort, DNS, Ingress, NetworkPolicy, session affinity, cross-namespace, endpoints, LoadBalancer, headless | 21–30 |
| **4. Storage** | PVC pending, volume mounts, access modes, StorageClass, ConfigMaps, Secrets, reclaim policy, emptyDir | 31–40 |
| **5. Security & Scheduling** | RBAC, securityContext, ResourceQuota, node affinity, taints & tolerations, Pod Security standards, PriorityClass, **chaos finale** (9 simultaneous failures) | 41–50 |

## Cluster Configuration

**Default:** kind cluster named `k8squest` with context `kind-k8squest`.

**Alternatives:**
- `k3s`: Set `K8S_CLUSTER_TYPE=k3s` before running `play.sh`. Kubeconfig is `~/.kube/k3s-config` and context is `k3s-k8squest`.
- minikube is not supported.

## Safety & RBAC

The game includes safety guards:
- Destructive cluster-wide operations require confirmation.
- System namespaces are protected from accidental deletion.
- RBAC limits the player to the `k8squest` namespace (see `rbac/`).

## Notable Features

- **Debrief reviews** after each level explain the why and real-world context.
- **DevOps-themed player names** and goodbye messages.
- **Completion certificates** issued when you finish a world.
- **Hints system** progressively unlocks; also auto-unlock on failed validation.
- **Safety guards** prevent accidental cluster damage.
- **iximiuz Labs integration:** Packaged as a Docker image for `mmengineer42/k8squest` on Docker Hub.

## Troubleshooting

- **Reset a level:** `python engine/reset.py <level-name>`
- **Reset all:** `python engine/reset.py all`
- **Check the cluster:** In your second terminal, `kubectl cluster-info` and `kubectl get nodes`
- **Logs:** The game logs to the terminal; the engine prints diagnostic info on errors.
