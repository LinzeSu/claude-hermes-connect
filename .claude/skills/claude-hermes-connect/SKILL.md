---
name: claude-hermes-connect
description: Configure and operate a remote Hermes instance from Claude Code over SSH as a real MCP server, while maintaining a durable Hermes role profile that tells Claude when to use it. Use this skill whenever the user wants to set up, add, update, troubleshoot, or actively use a remote Hermes MCP connection in Claude Code, especially when they mention remote Hermes, SSH-based MCP, recurring notifications, personal knowledge base, AI friend, tech-news summaries, long-term memory, or want Claude to decide when `remote-hermes` should be used with minimal extra confirmation.
---

# Claude Hermes Connect

Use this skill to wire a remote Hermes instance into local Claude Code as a real MCP server entry and to maintain a long-lived profile of what that Hermes is for.

The important distinction is transport versus workflow:

- A normal skill can guide setup, edit config, and help Claude decide when to call Hermes.
- It does not replace Claude Code's MCP transport layer.
- If the user's goal is for local Claude Code to actually call remote Hermes as a tool provider, prefer adding or updating a real `mcpServers.remote-hermes` entry in Claude settings.

## The intended interaction model

This skill should behave like a setup-plus-operations assistant, not just a config snippet generator.

### Phase 1: learn what Hermes is for

Ask the user what roles their Hermes plays. Encourage concrete roles, not just a generic label. Examples:

- regular tech news summarizer
- personal knowledge base
- AI friend or companion
- reminder and notification assistant
- long-term project memory
- doc drafting helper
- social or messaging helper through Hermes-side integrations

Capture the roles in durable project state so they can grow over time.

### Phase 2: configure the MCP connection

After roles are known, add or update the real `remote-hermes` MCP entry and encode those roles into the MCP description so Claude can better infer when to use it.

### Phase 3: operate Hermes later with low friction

When the user later asks for something Hermes should handle, such as:

- "let my Hermes remind me every day at 8:00 pm to check Hacker News"
- "have Hermes keep track of tech news for me"
- "ask Hermes to remember this as part of my personal knowledge base"

then use the stored Hermes profile to decide quickly whether `remote-hermes` is the right tool.

The goal is minimal repeated confirmation. Once the Hermes roles and connection info are known, do not keep re-interviewing the user for routine tasks that clearly match those roles.

## Durable project state

Maintain a small project-local profile file at:

`/.claude/remote-hermes-profile.json`

Use it to remember:

- SSH target
- chosen settings scope
- whether passwordless SSH is confirmed
- whether the remote login shell can find `hermes`
- declared Hermes roles
- examples of tasks Hermes should handle
- any caution notes, such as personal-context sensitivity

Suggested structure:

```json
{
  "name": "remote-hermes",
  "ssh_target": "username@your-remote-ip",
  "settings_scope": "project",
  "passwordless_ssh": true,
  "hermes_in_login_shell": true,
  "roles": [
    "regular tech news summarizer",
    "personal knowledge base",
    "AI friend"
  ],
  "task_examples": [
    "send me a nightly Hacker News reminder",
    "store personal notes for later recall",
    "help draft docs using long-term memory"
  ],
  "use_remote_hermes_when": [
    "the request clearly matches one of the declared Hermes roles",
    "the task benefits from Hermes-side memory, logs, integrations, or ongoing reminders"
  ],
  "ask_before_using_personal_context_when": [
    "the request is ambiguous",
    "the task may use private personal integrations in a way the user did not imply"
  ]
}
```

Keep this file current. When the user adds a new Hermes role later, update this profile and refresh the MCP description accordingly.

## What this skill should do

1. Confirm whether the user is trying to set up Hermes, update the Hermes profile, troubleshoot the connection, or ask Hermes to do work.
2. If roles are not yet known, ask what Hermes is for and gather concrete examples.
3. Read or create `/.claude/remote-hermes-profile.json`.
4. Check whether SSH target, passwordless SSH status, and remote `hermes` availability are known.
5. If setup information is missing, ask only for the missing pieces.
6. Add or update the real MCP entry in Claude settings.
7. Write the Hermes roles into the MCP description so Claude gets better routing signals later.
8. For later operational prompts, prefer using `remote-hermes` automatically when the task clearly fits the stored Hermes roles.
9. If setup cannot proceed, provide clear correction steps instead of guessing.

## Questions to ask early

Ask only the questions that are still unanswered.

- What roles does your Hermes play?
  - Ask for natural descriptions, not a forced taxonomy.
  - Pull out 2-6 short role phrases from the answer.
- What SSH target should be used, for example `username@host`?
- Do you want this MCP only for the current project, or across all Claude Code projects?
- Have you already configured passwordless SSH from this machine to the remote Hermes host?
- Does the remote machine already have the `hermes` CLI available in the login shell?

If the user has already provided some of this earlier, do not ask again.

If the user does not specify a settings scope, recommend project-local configuration first when the remote Hermes has personal context or personal integrations.

## When to add a real MCP entry

Add or update the real MCP server entry when the user wants Claude Code itself to be able to call the remote Hermes as a tool provider.

Do not present a one-off shell command as an equivalent replacement for MCP configuration. A shell command can test connectivity, but it does not make the remote Hermes available to Claude Code as a configured MCP server.

## Recommended MCP entry

Use this structure, filling in the real SSH target and tailoring the description to the Hermes profile:

```json
{
  "mcpServers": {
    "remote-hermes": {
      "command": "ssh",
      "args": [
        "-q",
        "username@your-remote-ip",
        "bash --login -c 'hermes mcp serve'"
      ],
      "description": "Remote Hermes with long-term memory and personal capabilities. Roles: regular tech news summarizer, personal knowledge base, AI friend, reminder assistant. Prefer it when a request matches these roles or needs Hermes-side memory, reminders, or integrations. Ask before using personal context if the request is ambiguous."
    }
  }
}
```

The description should be compact but informative. It should mention:

- the major Hermes roles
- the kinds of tasks that should route to Hermes
- whether extra care is needed around personal context

Do not leave the description generic if the roles are known.

## Settings edit policy

Before making changes:

1. Read the target settings file.
2. Read `/.claude/remote-hermes-profile.json` if it exists.
3. Show the exact JSON fragment to be added or updated.
4. Confirm the target scope if it is still unclear.
5. Edit only the minimal necessary part of the file.
6. Update the profile file after a successful config change.

Preferred default target:

- Project-only use: `.claude/settings.local.json`
- Cross-project use: the user's broader Claude settings file, if the user explicitly asks for that scope

If the target file does not yet contain `mcpServers`, add it without disturbing unrelated settings.
If `remote-hermes` already exists, update only the fields needed for the SSH target or description.

## Passwordless SSH prerequisite

This skill should not pretend it can complete the MCP connection if passwordless SSH is missing.
Because the MCP server is launched over stdio through `ssh`, the local Claude process needs non-interactive login to succeed.

If the user or host is missing, ask for it.
If passwordless SSH is not set up yet, stop and provide correction steps first.

## Windows SSH setup guidance

Use these commands on Windows PowerShell.

1. Create an SSH key if one does not already exist:

```powershell
ssh-keygen -t ed25519
```

Check whether the public key already exists:

```powershell
ls $env:USERPROFILE\.ssh\id_ed25519.pub
```

2. Print the public key and copy it:

```powershell
cat $env:USERPROFILE\.ssh\id_ed25519.pub
```

3. SSH into the Hermes remote server:

```powershell
ssh username@your-remote-ip
```

4. On the remote server, append the copied public key to `authorized_keys`:

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo 'your-public-key' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

5. Verify that login is now passwordless:

```powershell
ssh username@your-remote-ip
```

If this no longer prompts for a password, SSH is ready for the MCP connection.

## Linux SSH setup guidance

Use these commands on Linux.

1. Create an SSH key if one does not already exist:

```bash
ssh-keygen -t ed25519
```

Check whether the public key already exists:

```bash
ls ~/.ssh/id_ed25519.pub
```

2. Print the public key and copy it:

```bash
cat ~/.ssh/id_ed25519.pub
```

3. SSH into the Hermes remote server:

```bash
ssh username@your-remote-ip
```

4. On the remote server, append the copied public key to `authorized_keys`:

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo 'your-public-key' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

5. Verify that login is now passwordless:

```bash
ssh username@your-remote-ip
```

If this no longer prompts for a password, SSH is ready for the MCP connection.

## Connectivity checks

If the user wants troubleshooting help, guide them through these checks:

1. Verify passwordless SSH works.
2. Verify the `hermes` command exists on the remote login shell.
3. Test the remote command manually:

```bash
ssh -q username@your-remote-ip "bash --login -c 'hermes mcp serve'"
```

Do not leave that command running in the foreground as a final answer unless the user explicitly asked for a manual connectivity test.
Use it as a diagnostic step.

If the command fails, explain the likely cause plainly:

- bad SSH target
- passwordless SSH not finished
- remote `hermes` missing from the login shell
- remote shell init problem

## When the MCP should be used

After configuration, use the stored Hermes profile as the routing hint.

Prefer `remote-hermes` automatically when:

- the request clearly matches one of the declared Hermes roles
- the task needs Hermes-side memory, reminders, notifications, or integrations
- the user asks Hermes to do something ongoing, such as periodic reminders or recurring summaries

Ask for confirmation only when:

- required setup data is missing
- the task could use personal context in a way the user did not imply
- the task has side effects that are broader than the user asked for
- the request could plausibly be handled either locally or by Hermes and the tradeoff matters

Do not keep asking for confirmation on routine role-matching tasks once the user has already declared that Hermes is for those tasks.

## Role updates over time

If the user later says Hermes now also does something else, for example:

- "also use Hermes as my nightly market-news summarizer"
- "add AI friend to the Hermes role list"
- "Hermes should also handle reminders for me"

then:

1. update `/.claude/remote-hermes-profile.json`
2. refresh the `mcpServers.remote-hermes.description`
3. briefly summarize the new routing behavior

Treat this as normal maintenance, not as a full re-setup flow.

## Output style

When using this skill, produce:

1. A short summary of what you understand Hermes to be for.
2. The missing setup facts, if any.
3. The target settings scope.
4. The exact MCP JSON fragment to add or update.
5. Any prerequisite SSH correction steps still needed.
6. A concise explanation of when Claude should now use `remote-hermes`.

For later operational requests, keep the response lean. If the task clearly matches stored Hermes roles and setup is already valid, proceed with minimal extra confirmation.

## Example flows

**Example 1:**
User: Set up Claude Code to use my remote Hermes server at `alice@10.0.0.8`. Hermes is my personal knowledge base, AI friend, and nightly tech-news summarizer.

Good response pattern:
- Extract those roles into short phrases.
- Ask only whether passwordless SSH already works and whether setup should be project-only or broader.
- Create or update `/.claude/remote-hermes-profile.json`.
- Show the `remote-hermes` MCP entry with a role-aware description.
- Ask for confirmation before editing settings.

**Example 2:**
User: Let my Hermes remind me every night at 8:00 pm to check Hacker News.

Good response pattern:
- Read the stored Hermes profile.
- If reminder assistant or similar role is already present and setup is valid, prefer using `remote-hermes` with minimal extra confirmation.
- If SSH target or connection readiness is missing, ask only for the missing information.
- If passwordless SSH is known to be broken or unconfirmed, stop and tell the user what to fix first.

**Example 3:**
User: Hermes also acts as my reminder assistant now.

Good response pattern:
- Update the Hermes profile file.
- Refresh the MCP description to include reminder handling.
- Explain that future reminder-like tasks can now route to `remote-hermes` more directly.
