# lgtm-autostart

I got tired of opening my laptop and waiting two minutes before I could actually look at traces.

Every time I rebooted, I had to remember to `cd` into the right folder, start the stack, wait for containers to come up — and half the time I'd already started testing something before realizing logs weren't being collected yet. Classic.

So I did the obvious thing: made it start automatically.

---

## What's in the stack

Four containers. They start together, they talk to each other, and you never have to think about them.

| Service | Port | What it does |
|---------|------|-------------|
| OTel Collector | `4317` (gRPC), `4318` (HTTP) | Receives your app's telemetry and routes it |
| Loki | `3100` | Stores logs |
| Tempo | `3200` | Stores traces |
| Grafana | `3000` | Where you actually look at things |

Grafana runs with anonymous admin access — no login screen, you just open it and you're in.

---

## Sending telemetry from your app

Point your OTel SDK at the collector:

```
OTLP gRPC  →  localhost:4317
OTLP HTTP  →  localhost:4318
```

Logs go to Loki. Traces go to Tempo. That's it.

---

## macOS autostart

The whole point of this repo. A launchd agent that runs `docker compose up -d` on login, so the stack is just... there.

Copy the example plist from `example/` to your LaunchAgents folder:

```bash
cp example/com.user.lgtm-autostart.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.user.lgtm-autostart.plist
```

To remove it:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.lgtm-autostart.plist
rm ~/Library/LaunchAgents/com.user.lgtm-autostart.plist
```

Logs from the agent go to `/tmp/otel-stack.log` and `/tmp/otel-stack.error.log` if something looks off.

---

## Starting manually

If you want to bring it up or down yourself:

```bash
docker compose up -d
docker compose down
```

---

## What's in this repo

```
docker-compose.yml      the four services
loki.yml                Loki config
otel-collector.yml      collector pipeline — OTLP in, Loki + Tempo out
tempo.yml               Tempo config
example/
  com.user.lgtm-autostart.plist   launchd agent (copy this to ~/Library/LaunchAgents/)
```

All config files are volume-mounted, so edits take effect after a restart:

```bash
docker compose down && docker compose up -d
```

---

MIT
