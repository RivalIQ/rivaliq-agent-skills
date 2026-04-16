# Rival IQ Agent Skills

Public distribution repo for Rival IQ agent skills.

This repo contains installable skill directories in a standard Agent Skills
layout. The `main` branch represents the latest public skills compatible with
the latest released `rivaliq-server`.

## Skills

- `rivaliq-api` — Rival IQ customer API skill for competitive owned-social
  analytics

## Install

Download the current repo tarball:

```text
https://github.com/RivalIQ/rivaliq-agent-skills/archive/refs/heads/main.tar.gz
```

Then copy `skills/rivaliq-api/` into your agent's local skills directory.

For example:

```bash
tmp_dir="$(mktemp -d)"
/usr/bin/curl -sL \
  https://github.com/RivalIQ/rivaliq-agent-skills/archive/refs/heads/main.tar.gz \
  | tar xz -C "$tmp_dir"
cp -R "$tmp_dir"/rivaliq-agent-skills-main/skills/rivaliq-api /path/to/skills/
rm -rf "$tmp_dir"
```

## Repo Contract

- `main` is the default install target
- `skills/<skill-name>/` is the canonical layout
- skill folders are intended to be copied directly into a local skills
  directory

## Source Of Truth

This repo is the public distribution surface. Skill content is synced from
`rivaliq-server` release automation.
