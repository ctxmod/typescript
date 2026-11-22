# Local Server Lifecycle

Start, probe, reset, and stop local ECMAScript/TypeScript test servers and mocks without port collisions or self-kills.

Dev servers and mocks started by an agent for a manual check or a test run
outlive the step that launched them unless their lifecycle is explicit.
Several agents share one host, so a port, a process name, or a mock's seed
data is shared state. Linux and macOS only.

## Symptoms this prevents

- `EADDRINUSE` when two agents start a server on the same default port.
- Stale-server state contamination: an earlier run's server keeps answering
  on the port, or a mock keeps mutations from a previous run, and a diff is
  judged against the wrong state.
- A kill command that terminates the agent's own shell mid-step.

## Ports

- Allocate every port through one shared ephemeral-port helper in the
  repository, never uniform-random draws. Prefer letting the server bind
  port `0` and report the assigned port; otherwise the helper asks the OS
  for an ephemeral port and hands it straight to the server.
- The WHATWG bad-port blocklist applies to `fetch` but not to raw sockets:
  a server binds a blocked port without complaint, then every `fetch`
  against it fails. The helper harvests the live list from the runtime
  (a `fetch` to a blocked port rejects with a bad-port error before any
  connection) instead of pasting a copy, and skips those ports.
- Use one port per concurrent agent, never a shared default such as the
  framework's documented dev port. Pass the port explicitly on the command
  line and record it next to the PID.

## Start and readiness

- Launch the server executable directly rather than through a package
  script wrapper, so the recorded PID is the server itself, and record it:
  `node server.js --port "$PORT" > "$LOG" 2>&1 & echo $! > "$PIDFILE"`.
- Wait for readiness with a bounded curl poll, never `sleep`:
  `curl -fsS -o /dev/null --retry 30 --retry-connrefused --retry-max-time 30 "http://127.0.0.1:$PORT/"`.
  A non-zero exit means the server never came up; read `$LOG`, do not retry blindly.

## Stop

- Always kill by recorded PID first: `kill -TERM "$(cat "$PIDFILE")"`, a bounded wait, then `kill -KILL` on the same PID.
- A `pkill -f` pattern also matches the caller's own eval/command line, because the shell running the kill carries the pattern text in its arguments.
- The kill then lands on the agent's own shell, and the step ends at exit 144 with partial output.
- Exit 144 means a SIGTERM from outside the harness reached the agent's own Bash wrapper shell — harness-initiated kills report 137 or 143 instead — and the A3-INV investigation attributed every baseline exit 144 to a `pkill -f` self-match.
- Use a bracket pattern such as `pkill -f '[v]ite'` only when no PID was recorded and you are sure no peer runs a matching process; the bracket keeps the pattern from matching its own command text, but it still kills a concurrent peer's shell or server whose command text matches the pattern.

## Mock state

- Restart the mock to reset seed state before judging diffs. A mock that
  served earlier writes (for example a PATCH from a previous run) returns
  mutated records, and the diff reports a regression that is only residue.
- Treat an unexpected `EADDRINUSE` on your own port as a stale server from
  an earlier step: stop it by its recorded PID, then restart cleanly.

## Related

- [Application Development](../../modules/reference/application_development.md)
- [Project Setup](../../toolchain/reference/project_setup.md)
