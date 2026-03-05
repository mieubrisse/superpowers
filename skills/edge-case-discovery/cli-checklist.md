# CLI Edge Case Checklist

Additional edge cases specific to command-line tools and scripts.

## Arguments and Flags

- Missing required arguments
- Unknown flags or arguments (typos)
- Conflicting flags (e.g., `--verbose` and `--quiet`)
- Arguments with spaces, quotes, or special shell characters
- Very long argument values (exceeding ARG_MAX)
- Arguments passed via environment variables vs. flags (precedence)

## Standard I/O

- stdin is a pipe vs. a terminal (interactive vs. non-interactive)
- stdin is empty or closed immediately
- stdout is redirected to a file vs. a pipe vs. a terminal
- stderr and stdout interleaving when both are used
- Binary data on stdin when text is expected
- Encoding mismatches (UTF-8 input on a non-UTF-8 locale)

## Filesystem

- Target file does not exist
- Target file exists and is read-only
- Target path is a directory when a file is expected (and vice versa)
- Symlinks (circular, broken, pointing outside expected directory)
- Path contains spaces, Unicode, or special characters
- Insufficient disk space during write
- File is locked by another process
- Permissions: running as root vs. unprivileged user
- Relative paths vs. absolute paths (especially when the tool changes directories)

## Signals and Process Lifecycle

- SIGINT (Ctrl+C) during long-running operation — cleanup needed?
- SIGTERM during operation — graceful shutdown?
- SIGHUP when terminal disconnects
- SIGPIPE when downstream consumer stops reading (e.g., `your-tool | head`)
- Child processes left running after parent exits
- Temporary files left behind after abnormal exit

## Exit Codes

- Distinct exit codes for different failure modes (not just 0 and 1)
- Exit code when killed by signal
- Exit code propagation from child processes

## Environment

- Missing expected environment variables
- HOME, TMPDIR, or PATH not set or set to unexpected values
- Running inside a container (minimal filesystem, no HOME)
- Running in CI (no TTY, limited environment)
- Locale settings affecting string sorting, date formatting, number parsing
