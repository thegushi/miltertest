# miltertest

A test harness for milter-based mail filters. Drives a filter through the milter protocol using Lua scripts to simulate MTA behavior and assert expected outcomes.

Originally developed as part of [OpenDKIM](https://github.com/trusteddomainproject/OpenDKIM).

## Dependencies

- libmilter (sendmail-devel or libmilter-dev)
- Lua 5.1 or later (liblua-dev / liblua5.4-dev)

## Building

```sh
autoreconf -fvi
./configure
make
sudo make install
```

## Usage

See `miltertest(8)` for full documentation.

```sh
miltertest -s test.lua
```

## License

Copyright (c) 2009-2014, The Trusted Domain Project. All rights reserved.
See source for license terms.
