# Aristokrat plugins

The Aristokrat plugin lets Claude Code, Codex and T3 Code work on your Aristokrat landing pages and brand kit at [app.aristokrat.ai](https://app.aristokrat.ai). Ask your agent to change a page, preview it, check it, save it, publish an exact version or restore an earlier one. Give it your brand guidelines PDF and it builds a brand kit and uploads it to your company.

## What you need

- A Mac with Apple silicon (M1 or later). Intel Macs and Windows are not supported yet.
- Claude Code, Codex or T3 Code, already signed in.
- An Aristokrat account on [app.aristokrat.ai](https://app.aristokrat.ai).
- Git. If your Mac does not have it, the first command below asks to install Apple's Command Line Tools. Accept, then run the command again.

You do not need Node, npm or anything else. The plugin downloads its own pinned runtime from this repository's releases on first use and checks its checksum before running it.

## Install

Open Terminal and run the lines for the app you use.

Claude Code:

```sh
claude plugin marketplace add aristokrat-ai/plugins
claude plugin install aristokrat@aristokrat
```

Codex:

```sh
codex plugin marketplace add aristokrat-ai/plugins
codex plugin add aristokrat@aristokrat
```

T3 Code uses the Claude Code and Codex installs above. Install for the provider you pick in T3 (or both), then start a new thread.

## First use

Start a new session or thread and ask, for example, "Use Aristokrat to list my pages." The first request takes a minute while the plugin downloads its runtime.

The agent shows a sign-in link and a short code. Open the link, sign in to Aristokrat, check that the code matches and approve. The agent continues by itself. You sign in once per computer.

To build a brand kit, attach or point to your brand guidelines PDF and ask, for example, "Build an Aristokrat brand kit from this PDF for company chamelio and show it to me before uploading."

## Update

Claude Code:

```sh
claude plugin marketplace update aristokrat
claude plugin update aristokrat@aristokrat
```

Codex:

```sh
codex plugin marketplace upgrade aristokrat
codex plugin add aristokrat@aristokrat
```

Start a new session afterwards.

## Remove

```sh
claude plugin uninstall aristokrat@aristokrat
claude plugin marketplace remove aristokrat
codex plugin remove aristokrat@aristokrat
codex plugin marketplace remove aristokrat
```

Removing the plugin keeps your sign-in (`~/.config/aristokrat`) and the downloaded runtime (`~/.local/share/aristokrat`). Delete those folders to remove them too.

## If something goes wrong

- "download failed; retry the same command": the runtime download did not finish. Check your connection and ask the agent to try again.
- "unsupported platform": this Mac is not Apple silicon.
- The agent cannot find the plugin: start a new session after installing.
