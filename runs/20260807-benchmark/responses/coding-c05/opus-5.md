I'll check the environment first, then build the project.

`execute_bash`

```bash
uname -a; echo "---"; pwd; echo "HOME=$HOME"; echo "---"; ls -la /; echo "---"; which node npm python3 curl 2>/dev/null; node -v 2>/dev/null; python3 -V 2>/dev/null
```

`execute_bash`

```bash
mkdir -p "$HOME/projects" && cd "$HOME/projects" && pwd && touch .w && rm .w && echo writable
```

`execute_bash`

```bash
timeout 20 curl -sSI https://cdn.jsdelivr.net/npm/three@0.160.1/build/three.module.js | head -5; echo "exit=$?"
```