---
name: probe
description: Inspect the Cowork plugin runtime — paths, writability, fresh-shell state. Run manually.
disable-model-invocation: true
---

# Cowork Runtime Probe

Run EACH block below as a SEPARATE Bash command (do NOT merge). Paste full output of each.

## Block 1 — env, paths, mount folders, write a persistence marker
```bash
echo "cwd=$(pwd)"; echo "CLAUDE_PLUGIN_DATA=[$CLAUDE_PLUGIN_DATA]"; echo "CLAUDE_PLUGIN_ROOT=[$CLAUDE_PLUGIN_ROOT]"
env | grep -iE 'session|mnt|home|claude|plugin|workspace|project|dir|root' | sort
ls -la /sessions/*/mnt/ 2>/dev/null | head -20
git --version; python3 --version; python3 -c "import yaml; print('pyyaml ok')" 2>&1 || echo "pyyaml MISSING"
D="${CLAUDE_PLUGIN_DATA:-}"; [ -n "$D" ] && { mkdir -p "$D" && echo "marker $(date)" > "$D/sv7_marker.txt" && echo "wrote $D/sv7_marker.txt"; } || echo "no CLAUDE_PLUGIN_DATA"
M=$(ls -d /sessions/*/mnt/* 2>/dev/null | grep -vE '/(\.claude|\.remote-plugins|\.auto-memory|outputs|uploads)$' | head -1); echo "mount=$M"; [ -n "$M" ] && echo "marker $(date)" > "$M/sv7_marker.txt" 2>&1 && echo "wrote $M/sv7_marker.txt"
export PROBE_VAR=set_in_block1; echo "pid=$$"
```

## Block 2 — does state survive into a NEW bash call?
```bash
echo "pid=$$  PROBE_VAR=[$PROBE_VAR]  cwd=$(pwd)  CLAUDE_PLUGIN_DATA=[$CLAUDE_PLUGIN_DATA]"
```

## Block 3 — clone into the persistent candidate
```bash
D="${CLAUDE_PLUGIN_DATA:-$HOME/_scratch}"; git clone --depth 1 https://github.com/octocat/Hello-World.git "$D/clone-test" 2>&1 | tail -2; ls "$D/clone-test" 2>&1 | head -2
```
