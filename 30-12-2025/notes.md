# ✅ Parrot Core (Minimal / No Bloat)

**Docker image**

```
parrotsec/core
```

This is **true Parrot Core**.

---

# ✅ Goal

Create a **single command**:

```
parrot
```

---

# ✅ Step-by-Step (from creating the file)

## 1️⃣ Create the command file

Create a system-wide executable:

```sh
sudo nano /usr/local/bin/parrot
```

---

## 2️⃣ Paste this script (CORE ONLY)

```bash
#!/usr/bin/env bash
set -e

CONTAINER="parrot-core"
IMAGE="parrotsec/core"

# Check Docker exists
if ! command -v docker >/dev/null 2>&1; then
  echo "[!] Docker not installed"
  exit 1
fi

# Pull Parrot Core image if missing
if ! docker image inspect "$IMAGE" >/dev/null 2>&1; then
  echo "[*] Pulling Parrot CORE (no bloat)..."
  docker pull "$IMAGE"
fi

# Create container once (if not exists)
if ! docker container inspect "$CONTAINER" >/dev/null 2>&1; then
  echo "[*] Creating minimal Parrot CORE container..."
  docker create -it \
    --name "$CONTAINER" \
    --hostname parrot \
    --network host \
    -v "$HOME:/host" \
    "$IMAGE" \
    bash
fi

# Start and attach
exec docker start -ai "$CONTAINER"
```

Save and exit.

---

## 3️⃣ Make it executable (once)

```sh
sudo chmod +x /usr/local/bin/parrot
```

---

## 4️⃣ Run Parrot Core

```sh
parrot
```

You are now inside **Parrot Security OS (core only)**.

---
