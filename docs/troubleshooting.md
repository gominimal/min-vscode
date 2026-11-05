# Troubleshooting

Start with `min-vscode doctor`. It checks `min`, the `code` CLI and its
commit, Microsoft's Remote-SSH extension (`ms-vscode-remote.remote-ssh`),
`jq`/`python3`/`ssh`, the `vscode` loadout, the state dir, the ssh Include and that `ssh -G min-vscode.NAME`
routes through `_proxy` and the settings block, and prints a one-line fix for each problem.
`min-vscode status` lists sessions, whether something is connected, and how
long each has been idle.

- **`openssh is not installed in this session`.** The session was activated
  without a loadout carrying openssh. `min-vscode open` activates with
  `--loadout vscode`; for a session made by `min` directly, re-activate it
  with `--loadout vscode`.
- **Connection problems.** `min-vscode ssh NAME -- -v` runs plain ssh over
  the same transport and prints the negotiation;
  `~/.local/state/min-vscode/proxy.log` has one line per connection attempt:
  `ok`, or `fail: no session named NAME`, `fail: prepare` (the box's facts
  could not be fetched, were invalid or could not be written) or
  `fail: invalid home|sshd in PATH` (a stored fact no longer valid; the next
  `min-vscode attach NAME` rewrites it).
  ssh runs the ProxyCommand under `$SHELL -c`: nushell and xonsh users
  should give VS Code a POSIX `SHELL`.
- **An earlier `Host`/`Match` block shadows the stanza.** doctor says so;
  keep the min-vscode Include at the top of `~/.ssh/config`.
- **A setting of yours moved below `// min-vscode:end`.** VS Code adds new
  settings at the end of the file, which can be inside the marked block; the
  next refresh moves them out, and `--remove` keeps them.
- **Claude sign-in ends on `localhost:PORT/callback` with
  ERR_CONNECTION_REFUSED.** Nothing forwarded PORT to the box, usually
  because the sign-in URL was copied from VS Code's "Do you want Code to open
  the external website?" dialog rather than opened with **Open**. min-vscode has the box forward
  listening ports ([Ports](vscode.md#vs-codes-remote-ssh-and-a-box)), so
  PORT should appear in the Ports view while the sign-in waits. If it does
  not:
  - A window connected before this fix picks the setting up on its next
    connect: run **Developer: Reload Window**, then sign in again.
  - Check **Preferences: Open Remote Settings** for a
    `remote.autoForwardPortsSource` other than `process`.
  - Forward PORT by hand in the Ports view, then retry the callback URL in
    the browser while the sign-in still waits.
  - Sign in without a callback: use the URL Claude prints in its terminal or
    shows in its panel, the one that ends on a claude.com page with a code,
    and paste that code back into Claude.
  - Choose **Configure Trusted Domains** in the dialog and trust `claude.com`
    and its subdomains: VS Code then opens the URL itself, and that opens the
    tunnel.
- **`remote.SSH.remotePlatform in the // min-vscode:begin block is not an
  object; fix it by hand`.** Something rewrote it inside the block; make it
  an object (or delete it), then run `min-vscode config` again.

VS Code specifics (platform prompt):
[vscode.md](vscode.md#troubleshooting).
