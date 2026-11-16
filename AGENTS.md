# Halo (npm `fleet-halo`, formerly `screenshare-tunnel`)

**This repo is public and published to npm as `fleet-halo`.** Everything under `bin/`
and `skills/` is downloaded by anyone who runs `npm i -g fleet-halo`.

Never commit hostnames, tailnet IPs, ssh aliases, passwords or usernames from a real machine.
The script reads all of that at runtime (`SCREENSHARE_HOST`, `SCREENSHARE_IP`,
`~/.screenshare/password` on the worker). Git history counts: check `git diff --cached` before
every commit, not after.

- `bin/halo` is the whole tool (also installed as `screenshare`): plain bash, controller side, with the worker half sent
  over ssh as a heredoc. `bash -n` and `shellcheck -S warning` must pass.
- `skills/halo/SKILL.md` is the agent skill. Keep its command table in sync with
  the script's usage comment.
- Release: bump `version` in `package.json` in the same PR, then follow the npm-publish steps
  (`npm pack --dry-run`, `npm publish`, tag `v<version>`).
