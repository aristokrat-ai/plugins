# Aristokrat local plugin

Marketers install the released plugin from the public marketplace `aristokrat-ai/plugins` with the [install guide](../../tools/aristokrat-release/marketplace/README.md); a released plugin uses https://app.aristokrat.ai unless told otherwise. This guide is for testing a local build against your own stack.

Start the app in one terminal, then install in another (Node 26, Python 3.11+, npm dependencies and the selected host CLIs required). Development is always live and spends money, so start `bin/dev` only with Alex's go; fixtures are for tests only (`bin/aristokrat-marketer-e2e` runs the plugin against them):

```sh
bin/dev
bin/aristokrat-plugin install --host all
```

Keep both terminals open. The second command rebuilds the current platform release in `tmp/aristokrat-plugin/release`, refreshes `aristokrat@aristokrat-local` in Codex and Claude's user configuration, and serves runtime artifacts on loopback port 18765 until Ctrl-C. Use `--host codex` or `--host claude` for one CLI, or `--port PORT` for a different artifact port. The builder currently supports macOS arm64 and Linux arm64.

Open a **fresh Codex, Claude Code, or T3 Code thread** and select the Aristokrat skill. Use an existing local company/page and these prompts in order, replacing the placeholders:

1. “Use Aristokrat at http://localhost:3000 to list my pages.” Approve browser sign-in when requested.
2. “For company COMPANY, page /PATH, replace EXACT OLD TEXT with EXACT NEW TEXT. Preview at wide and narrow widths, check it, and save. Do not publish.” Keep the saved version number N and the prior version number M.
3. “Publish exact saved version N of COMPANY /PATH. Show publication history, then restore version M and show the result.” Both publication and restore target the local Worker, not a public site.

After source changes, Ctrl-C the artifact server and rerun `install`; changed source gets a new plugin cache version and changed runtime bytes get a new checksum identity. Open a new agent session. To restart serving an existing build without reinstalling, use `bin/aristokrat-plugin serve` with the same port. To remove plugin and marketplace entries, stop serving and run `bin/aristokrat-plugin uninstall --host all` (safe to repeat). The command refuses to replace an `aristokrat-local` marketplace owned by another source or checkout. It keeps immutable runtime caches; it never deletes authentication or page work folders.

## T3 Code

`--host all` covers T3's default Codex and Claude providers: new threads use the same user plugin installs. On this Mac, inspection of the running T3 child processes found `HOME=/Users/alexpinsky`, with both `CODEX_HOME` and `CLAUDE_CONFIG_DIR` unset. `~/.t3/userdata/settings.json` contains model favorites/defaults and no home overrides. The installed T3 bundle (`/Applications/T3 Code (Nightly).app/Contents/Resources/app.asar`, `apps/server/dist/binCli-DQcUcrlV.mjs`) resolves provider `homePath` into `CODEX_HOME` or `CLAUDE_CONFIG_DIR` only when configured; otherwise it uses the inherited environment and normal CLI homes. Its Codex `shadowHomePath` is a separate account option. No T3 configuration changes are needed or made.

For a T3 provider configured with a custom home, run the corresponding install with that exact home, for example `CODEX_HOME=/absolute/custom/home bin/aristokrat-plugin install --host codex` or `CLAUDE_CONFIG_DIR=/absolute/custom/home bin/aristokrat-plugin install --host claude`. Use the same environment when uninstalling. Custom provider homes are not automatically discovered.

Local W5 is [closed with accepted limits](../../docs/plans/aristokrat-marketer-plugin/w5.md#accepted-closure). Genuine Claude inference remains unverified, and full CI still fails on the packaged-shell test. Public distribution follows the [production plan (PL1–PL8)](../../docs/plans/going-to-prod/plan.md#marketer-plugin).

See the [release builder](../../tools/aristokrat-release/README.md) for packaging proofs and [developer setup](../../README.md#start-development) for the local stack.
