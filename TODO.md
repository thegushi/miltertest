# TODO

## Tools to add to this repo

### file-driven test milter (from OpenDMARC test.c)

OpenDMARC has `opendmarc/test.c` (~722 lines) - a command-line tool that drives a
milter through its callbacks by reading message files from disk, with stub
implementations of all `smfi_*` functions. No socket, no Lua, no MTA needed.

Currently it calls OpenDMARC's `mlfi_*` callback functions by name directly, which
is the only thing coupling it to OpenDMARC. The generalization is:

- Accept a milter shared library (or linked-in descriptor) as input
- Drive callbacks through the `smfiDesc_str` function pointer table that every
  milter registers via `smfi_register()`, instead of hardcoded `mlfi_*` symbols
- Message file format is simple: headers as `Name: Value` lines, blank line, body

This would give anyone writing a milter a file-driven test harness with no
dependencies beyond libmilter. OpenDMARC (and OpenDKIM, OpenARC) could then
become consumers of it rather than maintaining bespoke copies.

Reference: trusteddomainproject/OpenDMARC opendmarc/test.c and opendmarc/test.h

### CGI-style exec milter

A milter that implements all callbacks and acts as a gateway to an external
program - the milter equivalent of CGI. For each phase, stuff the relevant
data into environment variables and exec a user-specified command (Perl script,
shell script, Python, anything). The command's exit code determines the milter
response (continue, reject, tempfail, etc.).

This makes milter logic accessible to anyone who can write a shell script,
without needing to know the milter C API at all.

Key design questions:
- **Header accumulation**: call the script once per header with
  `MILTER_HEADER_NAME` / `MILTER_HEADER_VALUE`, or buffer all headers and call
  once at EOH with the full set? Once at EOH is more useful for real filtering
  logic (you can see all headers before deciding), but loses the streaming model.
- **Body handling**: pass as stdin to the script, or as a temp file path in an
  env var?
- **Response mapping**: simple exit codes (0=continue, 1=reject, 2=tempfail),
  or allow the script to write a response to stdout for more control (e.g.
  custom SMTP reply text)?

Inspired by the CGI pattern: same "turn protocol data into environment variables
and exec" approach that made CGI so accessible in the early web.
