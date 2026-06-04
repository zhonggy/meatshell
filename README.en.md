# meatshell

[简体中文](./README.md) | **English**

A lightweight, low-memory SSH / terminal client inspired by FinalShell, but
written entirely in **Rust + [Slint](https://slint.dev)**. The goal is to keep
FinalShell's core experience (resource-monitor sidebar, session management,
tabbed terminals) while cutting memory use from the 400 MB+ of a JVM app down to
the tens-of-MB range of a native binary.

## Roadmap

### v0.1 (current)

- [x] FinalShell-style dark theme UI
- [x] Local system monitor sidebar (CPU / memory / swap / network throughput, 1 Hz)
- [x] Tabs (welcome page + multiple terminal sessions)
- [x] Session management: create / edit / delete, persisted to local JSON
  - Config location: `%APPDATA%/meatshell/sessions.json` (Windows)
    / `~/.config/meatshell/sessions.json` (Linux)
    / `~/Library/Application Support/meatshell/sessions.json` (macOS)
- [x] SSH connection scaffold (`russh`, pure Rust, password + private key)
- [x] Line-buffered terminal view (type a line → Enter to send)

### v0.2

- [ ] Full VT/ANSI terminal emulation (integrate [`alacritty_terminal`](https://crates.io/crates/alacritty_terminal))
- [ ] Remote host resource monitoring (run a remote collector script, like FinalShell)
- [x] SFTP file browser + drag-and-drop upload/download
- [ ] Known-hosts (`known_hosts`) verification
- [ ] Store session passwords in the OS keychain

### v0.3+

- [ ] Split panes for tabbed terminals
- [ ] Session groups / folders
- [ ] Theme switching (light / follow system)
- [ ] Command history & snippet management

## Tech stack

| Module        | Choice                                                            |
| ------------- | ----------------------------------------------------------------- |
| UI            | [Slint](https://slint.dev) (compiled pure Rust, no GC)            |
| Async runtime | [`tokio`](https://tokio.rs)                                       |
| SSH protocol  | [`russh`](https://crates.io/crates/russh) (no libssh dependency)  |
| System metrics| [`sysinfo`](https://crates.io/crates/sysinfo)                     |
| Serialization | `serde` + `serde_json`                                            |
| Logging       | `tracing` + `tracing-subscriber`                                  |

## Running

```bash
cargo run --release
```

On first launch an empty session store is created at
`%APPDATA%/meatshell/sessions.json`. Click **"＋ New Session"** in the top-right
to add your first server.

## Project layout

```
meatshell/
├── Cargo.toml
├── build.rs                 # Slint compiler entry point
├── ui/
│   ├── app.slint            # top-level window
│   ├── theme.slint          # design tokens
│   ├── widgets.slint        # reusable buttons / inputs / sparkline
│   ├── sidebar.slint        # left-hand system monitor panel
│   ├── tabs.slint           # top tab bar
│   ├── welcome.slint        # welcome page / quick connect
│   ├── session_dialog.slint # new / edit session dialog
│   └── terminal_view.slint  # terminal view (v0.1 line-buffered)
└── src/
    ├── main.rs
    ├── app.rs               # UI ↔ backend bridge
    ├── config.rs            # session JSON persistence
    ├── system.rs            # CPU / memory / network sampling
    └── ssh.rs               # SSH session worker
```

## Development notes

- Slint widgets use a strict layout DSL; after editing a `.slint` file,
  `cargo check` is the fastest feedback loop.
- The application event loop is single-threaded (required by Slint); all
  cross-thread UI updates go through `slint::invoke_from_event_loop` callbacks.
- `check_server_key` currently accepts any server key (like
  `StrictHostKeyChecking=no`); wire up known-hosts verification before
  production use.

## License

Dual-licensed under MIT OR Apache-2.0.
