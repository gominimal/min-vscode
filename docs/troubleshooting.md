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
  `ok`, or `fail: no session named NAME`, `fail: no id` (min-vscode cannot
  tell which box is NAME's: the command says why, see below), `fail: min
  ls`, `fail: prepare` (the box's facts
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
- **"several boxes are named NAME, and min-vscode cannot tell which is its
  own".** min 0.8 lets boxes share a name, and NAME's key dir is from
  before min-vscode recorded box ids. min-vscode leaves all of them alone
  (no list row, no stanza, no settings entry, no connect) rather than let
  min pick the running one. `min ls --json` shows their ids: give the
  others new names (`min session rename ID NEW`) and min-vscode takes the
  one left, or remove `~/.local/state/min-vscode/sessions/NAME` to let go
  of it.
- **"Several boxes are named NAME, and none is min-vscode's."** You named a
  box min-vscode never connected to, and several have that name. Give the
  others new names with `min session rename ID NEW` (`min ls --json` lists
  their ids), then try again.
- **"…/sessions/NAME/id is not a session id".** The file was edited or
  damaged; min-vscode never passes it to min. Write the box's id (from `min
  ls --json`) into it, or, if the box is gone, remove
  `~/.local/state/min-vscode/sessions/NAME`. Removing only the file makes
  min-vscode take whatever one box has the name now, which may not be
  yours.
- **"your box NAME is gone; another box now has the name".** min-vscode's
  box NAME was destroyed outside min-vscode and another tool (or `min`)
  made a box with the same name. min-vscode never reaches that box under
  your record: `min-vscode destroy NAME` drops what is left of yours; then
  `attach NAME` reaches the other box if you mean to.
- **"NAME was renamed outside min-vscode, to NEW. To use it here again: min
  session rename ID NAME."** (or "min lists box NAME with no name now").
  Run the command it gives (it names your box's id). Or remove
  `~/.local/state/min-vscode/sessions/NAME` to let go of it.
- **The VM will not start after going from `unstable` to `rc`.** Don't
  downgrade `min` that way: the older daemon cannot read the box index the
  newer one wrote ([minimal#3060](https://github.com/gominimal/minimal/issues/3060)).
  If it has already happened, install the build that last wrote the volume
  (`unstable`) again and run `min ls`.

VS Code specifics (platform prompt):
[vscode.md](vscode.md#troubleshooting).
