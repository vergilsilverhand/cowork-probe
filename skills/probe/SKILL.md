---
name: probe
description: Find a persistent, fresh-shell-discoverable anchor in Cowork (clone location + marker). Run manually.
disable-model-invocation: true
---

# Cowork Persistent-Anchor Probe

Run EACH block below as a SEPARATE Bash command (do NOT merge — the point is to see what survives across
separate Bash calls). Paste the FULL output of each. Core question: **is there a stable, persistent,
writable path a fresh shell can find every time — to clone the code into and to drop a root marker?**

## Block 1 — paths, env, the persistent mount folder, tools, write markers
```bash
echo "== cwd =="; pwd
echo "== CLAUDE_PLUGIN_DATA=[$CLAUDE_PLUGIN_DATA] =="
echo "== CLAUDE_PLUGIN_ROOT=[$CLAUDE_PLUGIN_ROOT] =="
echo "== path/session/mount-like env vars (hunting a stable anchor) =="
env | grep -iE 'session|mnt|home|claude|plugin|workspace|project|onedrive|stockv7|dir|root' | sort
echo "== mounted persistent folder candidates =="
ls -la /sessions/*/mnt/ 2>/dev/null | head -20
echo "== tools =="; git --version; python3 --version; (python3 -c "import yaml; print('pyyaml ok')" 2>&1 || echo "pyyaml MISSING")
echo "== ROOT read-only? =="; mkdir -p "$CLAUDE_PLUGIN_ROOT/_t" 2>&1 && echo "ROOT WRITABLE (unexpected)" || echo "ROOT read-only (expected)"
echo "== write a PERSISTENCE MARKER to CLAUDE_PLUGIN_DATA =="
D="${CLAUDE_PLUGIN_DATA:-}"; if [ -n "$D" ]; then mkdir -p "$D" && echo "marker @ $(date)" > "$D/sv7_persist_marker.txt" && echo "WROTE $D/sv7_persist_marker.txt" && cat "$D/sv7_persist_marker.txt"; else echo "CLAUDE_PLUGIN_DATA is empty"; fi
echo "== write a PERSISTENCE MARKER to the mount folder candidate =="
M=$(ls -d /sessions/*/mnt/* 2>/dev/null | head -1); echo "mount candidate = [$M]"; if [ -n "$M" ]; then echo "marker @ $(date)" > "$M/sv7_persist_marker.txt" 2>&1 && echo "WROTE $M/sv7_persist_marker.txt" || echo "could not write to mount"; fi
echo "== set a cross-step var =="; export PROBE_VAR="set_in_block1"; echo "pid=$$ PROBE_VAR set"
```

## Block 2 — does shell state / cwd survive into a NEW Bash call? + is CLAUDE_PLUGIN_DATA stable?
```bash
echo "pid=$$  (vs Block 1)"
echo "PROBE_VAR=[$PROBE_VAR]  (empty = fresh shell, cross-step state LOST)"
echo "cwd=$(pwd)"
echo "CLAUDE_PLUGIN_DATA=[$CLAUDE_PLUGIN_DATA]  (same value as Block 1?)"
```

## Block 3 — clone the code into the persistent candidate
```bash
D="${CLAUDE_PLUGIN_DATA:-$HOME/_scratch}"; echo "clone -> $D/clone-test"
git clone --depth 1 https://github.com/octocat/Hello-World.git "$D/clone-test" 2>&1 | tail -3
ls "$D/clone-test" 2>&1 | head -3
```

## THEN — the decisive cross-session test
Open a **NEW Cowork session** and run JUST this (it proves which location persists across sessions):
```bash
echo "CLAUDE_PLUGIN_DATA now=[$CLAUDE_PLUGIN_DATA]"
cat "$CLAUDE_PLUGIN_DATA/sv7_persist_marker.txt" 2>&1 || echo "plugin_data marker GONE"
for m in /sessions/*/mnt/*/sv7_persist_marker.txt; do echo "mount marker: $m"; cat "$m" 2>/dev/null; done
ls "$CLAUDE_PLUGIN_DATA/clone-test" 2>&1 | head -2 || echo "clone GONE"
```
Report which markers/clone still exist in the new session — that is the anchor.
