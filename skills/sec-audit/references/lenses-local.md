# Specialist lens: CLI, local processes, and native code

Use for code that runs on a user's or operator's machine: CLIs, local daemons, helpers, webview bridges, deep links, and any native or `unsafe` code.

## Discipline

- State the realistic attacker: another local app, another OS user, an untrusted document, a remote web page, or a malicious repo the tool runs in. Harm within your own account and authority isn't a boundary.
- Paths, process names, and claimed sender fields aren't authentication. Use OS peer credentials, code identity, or protected channels.
- A helper must authorize each operation after parsing. A trusted UI doesn't make its arguments trusted.

## CLI and config writers

- **Untrusted project files:** a tool reading config from the current directory treats a cloned repo as input. Can a malicious repo change what the tool runs, where it sends a key, or which endpoint it trusts?
- **Key handling:** API keys written to other tools' config files. Check file permissions, whether keys land in shell history, logs, process arguments, or error output, and whether a key can be sent to a host taken from untrusted config.
- **Config edits:** writing to another agent's config by path. Check symlinks, TOCTOU, merging of untrusted content, and whether an edit can add a command that runs later.
- **Command building:** shell strings with user or project values. Prefer argument arrays.
- **Update and download:** anything fetched and run, and how it is verified.
- **Temp files:** predictable names, world-readable secrets, and cleanup.

## Local channels

- **Unauthenticated IPC:** Unix sockets, named pipes, loopback listeners, or native messaging accepting a lower-trust peer. A localhost HTTP server is reachable from web pages too: check `Origin`, Host, and DNS rebinding.
- **Claimed principal:** the channel belongs to one app or user, but a request field chooses another.
- **Lifecycle:** predictable request IDs, reused handles, stale channels, world-writable socket paths.

## Handlers and bridges

- **Deep links and custom schemes:** routes that change state, import data, or finish auth without a session binding.
- **Handoff:** OAuth, magic-link, and device-pairing callbacks returning to the wrong app or pending transaction.
- **File associations:** opening a document triggering a privileged action.
- **Webview bridges:** remote content reaching a native bridge. Check origin at call time and after navigation. Generic bridges exposing files or commands are the main risk.
- **Privileged helpers and installers:** a low-privilege caller choosing a command or file; scripts or paths writable by a lower-trust user after authorization.
- **Local secrets:** tokens readable by another app or user, in backups, clipboard, or notifications. Plaintext readable only by the same intended OS account isn't automatically a finding.

## Native and `unsafe` code

Applies only to C, C++, Rust `unsafe`, FFI, kernels, and binary parsers. Memory-safety findings don't apply to TypeScript or Python themselves.

Check bounds and offsets, integer overflow and truncation, unit confusion (bytes versus elements), uninitialized data, use-after-free and double free on error paths, invalid downcasts, FFI ownership and layout mismatches, library search paths writable by a lower-trust user, and repeated reads of user memory in a kernel. Claim only the effect that's established: a crash is not code execution. These almost always need a sanitizer run, so expect `needs_validation`.
