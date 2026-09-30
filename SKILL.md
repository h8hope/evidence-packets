---
name: evidence-packets
description: Prove host/infra changes with numbered before/after artifacts.
version: 1.0
license: MIT
metadata:
  tags: [devops, audit-trail, infrastructure]
---

# Evidence Packets for Host Actions

## When to Use
Load when an agent (or human) performs infrastructure or host-state changes that
must be defensible later: firewall rule edits, new VM/container, migrations,
permission changes, cron/service installs. The output is an evidence packet: an
ordered, hashed set of artifacts proving what the state was, what was done, and
what it became.

## Packet layout
Create `evidence/<TASK-ID>-<slug>/` containing files named `NN-topic.ext`
in strict order:

```
00-node-identity.ext        # who/where/when the action ran
01-<resource>-capacity.ext  # preconditions verified
02-<free-address-slot>.ext  # allocation checks (no collision)
03-ruleset-before.raw.ext   # STATE BEFORE, raw tool output
04-<related>-before.raw.ext # every object the change touches, pre-state
05-<action>-create.ext      # the change itself (command + full output)
06-containment.ext          # blast-radius check after
07-first-boot.ext           # immediate post-change verification
08-steady-state.ext         # verification after settling time
09-mechanism-install.ext    # any helper/monitor installed
10-window-open.ext          # transient risk window, its start
11-independent-readback.ext # read via a DIFFERENT tool/path than the change
12-gate-evidence.ext        # done-criteria checklist, each item proven
MANIFEST.sha256             # sha256 + byte size of every file above
```

## Procedure
1. **Pre-state first.** Before any change, capture raw outputs of the tools that
   report current state. An evidence packet without a "before" section is void.
2. **Execute once, log fully.** Append the exact command and its complete stdout/
   stderr. No paraphrasing, no summarizing at capture time.
3. **Independent readback.** Verify the resulting state through a different channel
   than the change used (different CLI, different host view, user-space vs kernel).
   A change confirmed only by the tool that made it is a claim, not evidence.
4. **Gate checklist.** Define done-criteria BEFORE starting; prove each in file 12;
   any unmet criterion stops the task, it does not "proceed with caveat".
5. **Hash last.** `sha256sum` every file into `MANIFEST.sha256`; re-verify the
   manifest reads back clean.

## Rules
- Artifacts are raw, timestamped, append-only; never rewrite history, add `NN+`
  files for corrections and reference the superseded file inside the new one.
- Secrets in captured output: replace values with `REDACTED`, never store them;
  note the redaction in the file itself.
- Packet directory goes to the project repo under git; one commit per completed
  packet.
- Reuse the template for every infra task; a shorter packet is fine when the
  action is smaller, but items 03/04 (before), 11 (independent readback) and
  MANIFEST are never optional.
