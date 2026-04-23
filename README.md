# Remote Linux Manager

Self-hosted control plane for Linux servers — Telegram bot, web UI, and
AI-driven command runner — over reverse SSH tunnels.  Single native binary;
no Docker required.

> Download page: https://github.com/jayeshbarman/remote-ssh-releases/releases/latest

## Install

### Ubuntu 22.04 / 24.04

```bash
curl -LO https://github.com/jayeshbarman/remote-ssh-releases/releases/latest/download/remote-linux-manager_*_amd64_ubuntu22.04.deb
sudo apt-get install ./remote-linux-manager_*_amd64_ubuntu22.04.deb
sudo rlm config edit      # fill in bot token, allowed Telegram IDs, public IP
sudo rlm start
# Open the UI at http://<this-server>:8080/ui/
```

### Amazon Linux 2023 / RHEL 9

```bash
curl -LO https://github.com/jayeshbarman/remote-ssh-releases/releases/latest/download/remote-linux-manager-*.x86_64.rpm
sudo dnf install ./remote-linux-manager-*.x86_64.rpm
sudo rlm config edit
sudo rlm start
```

## Managing the service

```bash
sudo rlm start            # start
sudo rlm stop             # stop
sudo rlm restart          # restart after config changes
sudo rlm status           # one-shot systemd status
sudo rlm logs -f          # tail the journal
sudo rlm config edit      # opens /etc/remote-linux-manager/config.yml in $EDITOR
sudo rlm config validate  # lint the config file
sudo rlm version          # print installed version
sudo rlm uninstall        # stop + remove package (keeps data)
sudo rlm uninstall --purge  # remove everything, including data and config
```

## First boot

After installation, set the Telegram bot token and your allowed user IDs:

```bash
sudo rlm config edit
```

Key fields to fill in:

```yaml
telegram:
  bot_token: "<get from @BotFather>"
  allowed_user_ids:
    - <your Telegram user id>     # get it from @userinfobot
hub:
  host: "<this server's public IP>"
```

Then:

```bash
sudo rlm config validate
sudo rlm start
sudo rlm logs -f
```

Open the web UI at `http://<your-server>:8080/ui/`.  On first visit you'll
be asked to set an admin password.  You can also pre-set it:

```bash
sudo RLM_ADMIN_PASSWORD='…' systemctl start rlm-hub
```

## Adding a managed server

On the UI, click **Add server**.  You can either:

1. Get an install command and paste it on the remote Linux host, or
2. Auto-install over SSH — enter the target's IP + credentials and RLM does
   the rest.

The one-liner run on the remote host looks like:

```bash
curl -sSL http://<hub-ip>:8080/install.sh | sudo bash -s -- \
    --name "my-server" --hub <hub-ip> --token "<token-from-ui>"
```

The installer downloads a single compiled binary, verifies its SHA-256,
and installs two small systemd units.  No Python or other runtime is
required on the remote.

## Verifying downloads

Every release publishes `SHA256SUMS` for the binaries.  If signing is
enabled, `SHA256SUMS.asc` is also provided.

```bash
# 1. Verify checksums
sha256sum -c SHA256SUMS --ignore-missing

# 2. (Optional) verify the signature
#    One-time: import the project public key
gpg --import rlm-public.asc
gpg --verify SHA256SUMS.asc SHA256SUMS
```

## Requirements

- An EC2 `t3.micro` or equivalent (1 vCPU, 1 GB RAM).
- Ports 8080 (UI) and 2222 (agent tunnels) reachable from where you need
  them.  8080 can be restricted to your admin IPs; 2222 needs to be
  reachable from every managed agent.
- An elastic IP / DNS name that the agents can connect back to.

## Features

- 📟 **Telegram bot** — `/list`, `/exec`, `/ssh`, `/logs`, `/reboot`,
  `/ai` (natural-language command runner).
- 🖥 **Web UI** — dashboard with live heartbeat, resource stats (CPU /
  RAM / disk / GPU), and per-server detail pages.
- 🔔 **Alerts** — Telegram, email (SMTP), Microsoft Teams (webhook or
  Graph API bot).
- 🤖 **Root Cause Analysis** — when a server recovers from downtime, the
  hub SSHes in, collects diagnostic data, asks Claude to summarise, and
  emails the report.
- 🌐 **Domain uptime monitor** — HTTP checks with alert routing.
- 🔐 **Single native binary** — no Docker, no Python to manage, runs as
  an unprivileged `rlm` user with systemd hardening.

## Public signing key

<!-- after enabling GPG signing, paste `rlm-public.asc` contents here -->

```
(not yet signing releases)
```

## Licence & support

This is a proprietary product.  Installation and use is governed by the
end user licence agreement in each package's `/usr/share/doc/remote-linux-manager/`
directory.  For issues, open a ticket in this repository or email the
maintainer.
