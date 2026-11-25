# For agents

What an agent reads to work with the S32, and how to point it at the right page.

## llms.txt and Markdown copies

[llms.txt](/llms.txt) is an index of the AI and s32 CLI pages. Every page it links, except the manual home, is a Markdown copy of a manual page: the same text without the site around it. Add `.md` to the address of a page under `/ai/` or `/developers/` to get its copy, for example `/ai/setup.md`.

## Page buttons

Every page under Use the S32 with AI and under s32 CLI has three buttons at the top:

- **Copy page** copies the Markdown of the page to your clipboard.
- **Open in ChatGPT** opens ChatGPT with a prompt that points at the Markdown copy of the page.
- **Open in Claude** opens Claude with the same prompt.

The prompt names the page and nothing else: no text from the page, and nothing about you.

## Discover the commands

An agent finds commands the way you do. `s32 --help` lists them, `s32 <command> --help` shows the keys of one, `s32 commands --json` lists everything for a program, and `s32 schema --emit-schema-json` lists each device's parameters with their ids and ranges.

<!-- s32-check -->
```bash
s32 --help
s32 device param set --help
s32 commands --json
s32 schema --emit-schema-json
```

## Batches

A script runs one command per line, so an agent can do a whole setup in one go. Save the lines in a file, then run it with `--script`.

<!-- s32-file session.script -->
```text
device add 1 arp
device param set 1 2 rate=6
device list 1
```

<!-- s32-check -->
```bash
s32 --script session.script
```

A script stops at the first line that fails and exits with that line's code. Commands that ask first need `--yes` on their line. For a session you steer by hand, use `repl`: see [Simulator and scripts](../developers/simulator.md).


- [Overview](./index.md) — What an agent can do with your S32, in three steps
- [Set up your agent](./setup.md) — Paste one block; the agent installs, checks and saves the rules
- [Prompts and examples](./prompts.md) — Quick starters and prompts for racks, modulation and knob mappings, with checked commands
- [Change safely](./safe-changes.md) — Read, preview, ask, change, check

