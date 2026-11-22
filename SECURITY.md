# Security Policy

## Reporting a vulnerability

Please don't report security problems in a public issue, pull request or
discussion. Use
[**Report a vulnerability**](https://github.com/sadayamuthu/airlock/security/advisories/new)
on the repository's Security tab instead. Only the maintainers can see the
report.

Include what you can of:

- the Airlock version (`airlock version`) or commit, your Go version and OS
- how Airlock was used, and its configuration with secrets removed
- steps to reproduce
- what happened and what you expected

You'll get a reply when the report has been read, and again when a fix is
ready. A fix ships as a patch release; the advisory names you unless you'd
rather it didn't.

## Supported versions

Airlock is at the design stage and has no releases yet. Once it does,
fixes land in the latest release only, until 1.0.

## What counts

Airlock intercepts tool calls marked risky by policy, runs each in a fresh sandbox, and copies allowed effects back to your workspace.

Airlock is a security tool, so anything that breaks one of these promises
is a security issue:

- **Isolation per call.** Nothing carries over between sandboxes unless committed. A way for one call to affect another's sandbox is in scope.
- **No network and no secrets by default.** A sandbox that reaches the network or a secret its policy didn't grant is in scope.
- **Effects are checked before they're real.** A change that reaches the real workspace without passing the policy check is in scope.
- **Sandbox escape.** Any way for code in a sandbox to affect the host outside the commit step is in scope.

These are by design, and not vulnerabilities:

- A policy that allows network or mounts secrets allows what it says.
- Effects you approve are committed as they are.

A wrong result that isn't covered above is a bug, not a vulnerability. Report
it in a normal issue; see [CONTRIBUTING.md](CONTRIBUTING.md).
