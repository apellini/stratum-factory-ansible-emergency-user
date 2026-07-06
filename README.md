# stratum_factory.emergency_user

**Ansible factory role** — provisions a last-resort password-based OS user
(`stratum-rescue`) on all STRATUM dev VMs (bastion, k3s, containerlab).

Designed for use via the **GCP serial console** when SSH keys, VPN, and
network access are all broken. The account has sudo (password required,
not passwordless). sshd `PasswordAuthentication` is **not** modified — this
is explicitly a console-only account.

Part of the STRATUM dev environment infrastructure (`apellini/stratum-infra`).
Consumed by the `apellini/stratum-ansible-bastion` wrapper via `emergency-user.yml`.

---

## Decision reference

**D-INFRA-24** — approved in `apellini/stratum-infra`.

---

## Role inputs

| Variable | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `emergency_user_name` | str | no | `stratum-rescue` | OS username |
| `emergency_user_password_hash` | str | **yes** | `""` | Pre-hashed SHA-512 password string (`openssl passwd -6 -stdin`) |
| `emergency_user_shell` | str | no | `/bin/bash` | Login shell |
| `emergency_user_sudo` | bool | no | `true` | Grant sudo with password required (not NOPASSWD) |

`emergency_user_password_hash` must be a pre-hashed string — never plaintext.
The harness computes the hash in memory from the ansible-vault-encrypted password
and exports it as `STRATUM_EMERGENCY_PASSWORD_HASH`. The variable is marked
`no_log: true` in tasks to prevent the hash appearing in Ansible output.

---

## Usage

This role is consumed by the `apellini/stratum-ansible-bastion` wrapper via
the `emergency-user.yml` play (`hosts: all`). It is **not** called directly.

The wrapper's `group_vars/all.yml` sets:

```yaml
emergency_user_name: "{{ lookup('env', 'STRATUM_EMERGENCY_USER') }}"
emergency_user_password_hash: "{{ lookup('env', 'STRATUM_EMERGENCY_PASSWORD_HASH') }}"
```

The harness (`dev/harness.sh`) exports those env vars from the
ansible-vault-encrypted `.harness/keys/emergency-password.vault`.

---

## Emergency access procedure

```bash
# 1. Retrieve the plaintext password
ansible-vault view \
  --vault-password-file .harness/keys/.vault-pass \
  .harness/keys/emergency-password.vault

# 2. Open the serial console for the broken VM
gcloud compute connect-to-serial-port stratum-dev-<vm> --zone europe-west1-b

# 3. Login: stratum-rescue / <password from step 1>

# 4. Escalate (password required)
sudo -i
```

---

## Molecule tests

```bash
cd roles/stratum_factory.emergency_user
molecule test
```

Asserts: user exists, has a password set in `/etc/shadow`, sudoers file present,
`NOPASSWD` absent, shell is `/bin/bash`.

---

## Changelog

### v0.1.0 (2026-07-05)
- Initial release: `stratum-rescue` user creation, SHA-512 password, sudo with password, molecule scenario
