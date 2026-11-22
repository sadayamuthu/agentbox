# airlock

A disposable sandbox for every risky tool call. `airlock` runs each tool call
an AI agent makes (shell commands, code execution, file edits, package
installs) in a short-lived microVM or WebAssembly sandbox with no network by
default and a scratch filesystem. The results are checked against a policy
before anything is copied back to the real system.

> **Status: design stage.** There is no code yet. The commands, policy format
> and output below are the proposal, and they will change. Part of the
> [OpenAstra](https://openastra.org) research agenda, area VIII: AI Runtime
> Security.

## The problem

Coding and ops agents run commands on real machines with the developer's
credentials, network and files. Container sandboxes help but are usually
long-lived, shared across a whole session, and allowed to reach the network,
so one bad command can still read secrets or send them out. What's missing is
isolation at the level of the individual call, with a review step between
"the command ran" and "its effects are real".

## How it works

1. **Intercept.** `airlock` wraps the agent's tool layer (an MCP proxy or a
   shell shim) and catches calls marked risky by policy.
2. **Boot.** Start a fresh sandbox per call: Firecracker microVM for full
   Linux workloads, or a WASM runtime for lighter ones. It starts from a
   copy-on-write snapshot of the workspace, with no secrets and no network
   unless the policy grants them.
3. **Run and capture.** Record stdout, exit code, file changes and any
   attempted network connections.
4. **Check and commit.** Compare the captured effects with the policy. Allowed
   changes are copied back; everything else is shown to the user. The sandbox
   is destroyed either way.

## Example (proposed)

```console
$ airlock run -- npm install left-pad-pro
sandbox   vm-3f1a  (no network, workspace snapshot, 2 vCPU, 30s)
attempted network: registry.npmjs.org:443   allowed by policy: npm-registry
attempted network: 185.199.x.x:4444        BLOCKED (not in policy)
file changes: package.json, package-lock.json, node_modules/ (1,204 files)
postinstall script wrote ~/.ssh/authorized_keys  BLOCKED (outside workspace)

Result: not committed. 2 policy violations. Run `airlock show vm-3f1a` for details.
```

## Planned surfaces

- CLI (`airlock run`) and a shell shim for agents that call `bash -c`
- MCP proxy mode for tool servers
- Policy file: network allowlist, writable paths, secrets to mount, CPU/time limits
- Effect reports in JSON for audit and for [Rewind](https://github.com/sadayamuthu/rewind)

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP Agentic Top 10 | Unexpected code execution, tool misuse |
| OWASP LLM Top 10 (2025) | LLM05 Improper Output Handling, LLM06 Excessive Agency |
| NIST SP 800-53 Rev. 5 | SC-39 Process Isolation, SC-44 Detonation Chambers, SC-7 Boundary Protection |

## Design principles

- **One call, one sandbox.** Nothing carries over between calls unless committed.
- **No network and no secrets by default.**
- **Effects are reviewed before they're real.**
- **Fast enough to leave on.** Target sub-second sandbox start for the common case.

## Roadmap

1. Policy format and effect report schema
2. Linux microVM backend with workspace snapshots
3. Shell shim for coding agents
4. MCP proxy mode
5. WASM backend and macOS support

## Built on switchyard

`airlock` will be a Go binary built on [switchyard](https://github.com/sadayamuthu/switchyard), OpenAstra's shared MCP proxy core: MCP proxy mode is a switchyard hook that routes risky calls into a sandbox. It stacks with the other switchyard tools by chaining (`agent → fuse → intentlock → taintline → rewind → MCP server`).

## Related OpenAstra projects

- [ZeroEgress](https://github.com/sadayamuthu/zero-egress): the same "nothing leaves" stance, for the editor
- [Rewind](https://github.com/sadayamuthu/rewind): undo for effects that were committed
- OpenEQ (in the lab): egress records across tools

## License

To be decided before the first release.
