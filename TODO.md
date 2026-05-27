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
