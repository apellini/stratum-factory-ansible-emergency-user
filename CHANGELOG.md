# Changelog — stratum_factory.emergency_user

## v0.1.0 (2026-07-05)

Initial release.

- Creates `stratum-rescue` last-resort OS user with SHA-512 password hash
- Grants sudo with password required (no NOPASSWD)
- `no_log: true` on password task — hash never appears in Ansible output
- `meta/argument_specs.yml` typed interface with `emergency_user_password_hash` as no_log required
- Molecule scenario: asserts user exists, password set, sudoers present, NOPASSWD absent
