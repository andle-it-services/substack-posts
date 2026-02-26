# Hardened Docker Host for Free Using Ansible

Practical server automation for small IT shops using Ansible playbooks to provision secure Docker hosts with Fail2Ban and automatic updates.

---

## Why Your Docker Host Is a Liability

### The Problem

Enterprise shops have hardened server images, automated patching pipelines, and security teams reviewing every config. Small IT operations have a fresh Ubuntu VPS, Docker installed from a tutorial, and default settings everywhere.

Attackers know this.

### The Attack Surface

Most compromises on small-org servers follow predictable patterns:

- **Credential brute force:** Bots hammer SSH within minutes of a server going live
- **Unpatched vulnerabilities:** Known CVEs exploited weeks after patches are available
- **Container escape:** Misconfigured Docker giving container processes host-level access
- **Log blindness:** No visibility into what's happening, so you find out when something breaks

### Why Default Configs Fail

A fresh Ubuntu server with Docker installed is not a secure Docker server. Out of the box:

- SSH accepts password authentication
- Root login is enabled
- Docker group membership equals root access
- Security patches apply only when you remember to run `apt upgrade`
- Nothing is watching for brute force attempts

You need a repeatable, automated setup that doesn't rely on you remembering to do the right things every time.

---

## The Ansible Approach

### Flip the Model

Instead of configuring each server by hand (and doing it slightly differently every time), define your desired state once and apply it everywhere.

This is infrastructure-as-code. Your playbook becomes the source of truth. Run it against a fresh VPS and walk away with a hardened server. Run it again six months later and it corrects any drift.

### What We're Automating

Four things, each handled by a dedicated role:

- **common** — SSH hardening, UFW firewall, base packages
- **docker** — Docker CE from the official repo, locked-down daemon config
- **fail2ban** — Intrusion prevention with SSH-aware jails
- **auto-updates** — Automatic security patches, no manual intervention

### The Project Structure

```
ansible-docker-hardened/
├── ansible.cfg
├── inventory.yml
├── playbook.yml
├── group_vars/
│   └── all.yml
└── roles/
    ├── common/
    ├── docker/
    ├── fail2ban/
    └── auto-updates/
```

Each role is self-contained. Pull the Fail2Ban role into another project. Swap the Docker role for Podman. Nothing is coupled.

---

## Building the Playbook

### Inventory and Config

```yaml
# inventory.yml
all:
  children:
    docker_hosts:
      hosts:
        docker-host:
          ansible_host: 192.168.1.50
          ansible_user: deploy
          ansible_ssh_private_key_file: ~/.ssh/ansible_ed25519
          ansible_python_interpreter: /usr/bin/python3
```

```ini
# ansible.cfg
[defaults]
inventory = inventory.yml
retry_files_enabled = False

[ssh_connection]
pipelining = True
```

Verify connectivity before writing any roles:

```bash
ansible all -m ping
# docker-host | SUCCESS => {"ping": "pong"}
```

If that fails, fix SSH access first. Common culprits: key permissions (`chmod 600`), missing `authorized_keys`, firewall blocking port 22.

### SSH Hardening (common role)

The config template does the real work:

```
PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowTcpForwarding no
```

The task validates the config before applying it:

```yaml
- name: Configure SSH
  template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    validate: '/usr/sbin/sshd -t -f %s'
  notify: restart sshd
```

That `validate` line runs `sshd -t` before replacing the file. A bad SSH config can lock you out permanently. Always validate.

### Docker Installation and Daemon Config (docker role)

Install from the official repo, not `apt install docker.io`:

```yaml
- name: Add Docker GPG key
  ansible.builtin.apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Add Docker repository
  ansible.builtin.apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present
```

Lock down the daemon:

```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "no-new-privileges": true,
  "userland-proxy": false
}
```

`no-new-privileges` prevents containers from gaining additional Linux capabilities. `userland-proxy: false` removes a component with a history of security issues. Log rotation keeps your disk from filling up silently.

### Fail2Ban (fail2ban role)

```yaml
# defaults/main.yml
fail2ban_maxretry: 3
fail2ban_bantime: 3600    # 1 hour ban
fail2ban_findtime: 600    # 10 minute window

fail2ban_jails:
  - name: sshd
    enabled: true
    port: ssh
    filter: sshd
    logpath: /var/log/auth.log
```

Three failed SSH attempts in 10 minutes equals a one-hour ban. On a fresh test server exposed to the internet, expect to see dozens of blocked attempts within the first hour.

### Automatic Updates (auto-updates role)

```yaml
- name: Configure unattended-upgrades
  template:
    src: 50unattended-upgrades.j2
    dest: /etc/apt/apt.conf.d/50unattended-upgrades
```

Security patches apply automatically. The Equifax breach was an unpatched Apache Struts vulnerability with a fix available for months. Automate this.

### The Main Playbook

```yaml
# playbook.yml
---
- name: Provision hardened Docker hosts
  hosts: docker_hosts
  become: true

  roles:
    - common
    - docker
    - fail2ban
    - auto-updates
```

Run it:

```bash
ansible-playbook playbook.yml --check  # dry run first
ansible-playbook playbook.yml
```

---

## The Layered Defense

Neither SSH hardening, Fail2Ban, nor automatic updates works perfectly alone. Together they cover each other's gaps:

```
IF SSH accepts passwords:
    → Brute force is viable
ELSE IF Fail2Ban not running:
    → Unlimited key-guessing attempts
ELSE IF patches not automated:
    → Known CVEs stay exploitable
ELSE:
    → You're the boring target. Bots move on.
```

**Why this works:**

- Password auth is disabled — credential stuffing fails immediately
- Fail2Ban bans after 3 attempts — key brute force is impractical
- Automatic updates close known vulnerabilities — no patching debt
- Docker daemon is locked down — container escape is harder

---

## Quick-Start Checklist

### Day 1: Bootstrap

- ☐ Install Ansible (`pipx install ansible-core`)
- ☐ Generate an ed25519 SSH key and copy to target server
- ☐ Create project structure and inventory file
- ☐ Verify `ansible all -m ping` succeeds

### Day 2: Build the Roles

- ☐ Create `common` role with SSH hardening template
- ☐ Create `docker` role with daemon config
- ☐ Create `fail2ban` role with SSH jail
- ☐ Create `auto-updates` role

### Day 3: Test and Validate

- ☐ Run `ansible-playbook playbook.yml --check` first
- ☐ Apply to a test server before production
- ☐ Verify Fail2Ban status: `fail2ban-client status sshd`
- ☐ Confirm password auth is disabled: `ssh -o PreferredAuthentications=password user@host`
- ☐ Check auto-updates config: `apt-config dump | grep Unattended`

### Ongoing Maintenance

- ☐ **Weekly:** Check Fail2Ban logs for patterns
- ☐ **Monthly:** Review Docker daemon config against CIS benchmarks
- ☐ **Quarterly:** Audit who has Docker group membership
- ☐ **Annually:** Full review of SSH config and firewall rules

---

## Cost

$0. Ansible is free. The hardening techniques use tools already on your Ubuntu server. You're just not configuring them.
