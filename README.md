# server-bootstrap

A GitHub Actions workflow that turns a fresh Ubuntu VPS into a Docker-ready server — without you having to SSH in and run commands manually.

Works with VPS running Ubuntu 20.04+.

---

## The problem

Every time you spin up a new server, you end up doing the same thing. Update packages. Install Docker. Set up a firewall. Create a user. Harden SSH. It takes 20–30 minutes and if you don't write it down somewhere, you're Googling half of it again next time.

This workflow does all of that in one click from GitHub.

---

## What it does

Runs the following on your server, in order:

- Updates and upgrades all system packages
- Installs Docker Engine and Docker Compose v2 (from Docker's official repo, not the outdated `apt` version)
- Creates a non-root `deploy` user and adds it to the docker group
- Configures UFW firewall — you choose which ports to open
- Sets up Fail2Ban to block SSH brute-force attempts
- Creates a 2GB swap file (toggle off if you don't need it)
- Sets up a standard `/srv/app/` directory for your projects
- Hardens SSH to disallow password login

---

## Setup

**1. Add your SSH private key as a GitHub secret**

If you don't have a key pair yet:

```bash
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/deploy_key
```

Copy the public key to your server:

```bash
ssh-copy-id -i ~/.ssh/deploy_key.pub root@YOUR_SERVER_IP
```

Then go to your repo → **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret | Value |
|--------|-------|
| `SSH_PRIVATE_KEY` | Contents of `~/.ssh/deploy_key` (the private key) |

**2. Add the workflow to your repo**

Copy `server-setup.yml` into `.github/workflows/` in your repository.

**3. Run it**

Go to **Actions → 🛠️ Server Bootstrap → Run workflow** and fill in your server details.

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `ssh_host` | ✅ | — | Server IP or hostname |
| `ssh_user` | ✅ | `root` | SSH user (usually `root` on a fresh server) |
| `ssh_port` | — | `22` | SSH port |
| `deploy_user` | — | `deploy` | Non-root user to create. Leave empty to skip |
| `open_ports` | — | `22,80,443` | Comma-separated ports to open in UFW |
| `install_extras` | — | `true` | Installs htop, ncdu, vim, make, and a few other useful tools |
| `setup_swap` | — | `true` | Creates a 2GB swap file |

---

## After it runs

Your server will have:

- Docker and Docker Compose ready to use
- A `deploy` user you can SSH into for day-to-day work (no need to use `root`)
- UFW active with only the ports you specified open
- SSH locked to key-based auth only

From here you can start deploying your containers. If you're using Docker Compose, copy your `docker-compose.yml` to `/srv/app/` and run `docker compose up -d`.

---

## Tested on

- Ubuntu 22.04 LTS
- Ubuntu 24.04 LTS
- DigitalOcean Droplets
  
---

## Contributing

I built it for my own use and put it up in case it helps someone.
If something's broken, feel free to fork it and adapt it to your setup.

---

## License

MIT
