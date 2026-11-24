# AgentBox: thesis

**In one line:** the right unit of isolation for an agent is the single tool
call, with a review step between "it ran" and "its effects are real".

## The problem

Coding and ops agents run commands with the developer's credentials, network
and files. Container sandboxes help, but they usually last a whole session,
are shared across calls, and can reach the network, so one bad command can
still read secrets or send them out.

## The bet

1. **Per-call sandboxes are now cheap enough.** Firecracker microVMs and WASM
   runtimes with copy-on-write workspace snapshots can start fast enough to
   leave on for every risky call.
2. **Effects, not commands, are what to review.** Running the call first and
   checking captured effects (files changed, connections attempted) against a
   policy catches what reading the command string can't, such as a malicious
   postinstall script.
3. **No network and no secrets by default** removes the main exfiltration
   path without needing to detect the attack.

## Why now

Coding agents run shell commands constantly, supply-chain attacks through
package installs are routine, and microVM tooling has matured to the point
where per-call isolation is an engineering problem, not a research one.

## What success looks like

- Sandbox start is sub-second for the common case, so developers leave it on.
- It blocks the effects of known malicious-package and exfiltration patterns
  in a public test set.
- At least one popular coding agent is usable with AgentBox through the shell
  shim or MCP proxy with no workflow complaints.

## What would prove us wrong

- Latency or workspace-snapshot cost makes developers turn it off.
- Too many legitimate calls need network or secrets, so policies become
  allow-all.
- Agent vendors ship equivalent per-call isolation built in.
- macOS support (where many developers work) proves impractical.

## Not a bet on

Detecting malicious intent in commands, or replacing container platforms.

## Where it fits

Area VIII, AI Runtime Security. Effect reports feed
[AgentRewind](https://github.com/sadayamuthu/agentrewind); hosts risky
[AgentChaos](https://github.com/sadayamuthu/agentchaos) experiments.
