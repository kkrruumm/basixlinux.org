title: Basix Wiki

The seat management stack on Basix consists of two things:
1. [seatd](https://sr.ht/~kennylevinsen/seatd/)
2. [dumb_runtime_dir](https://github.com/ifreund/dumb_runtime_dir)

You may install the `seatd` package from `core` and enable its service for seat management functionality.

If you need an `XDG_RUNTIME_DIR` (you probably do), install the `dumb_runtime_dir` package and enable the `run-user-dir` service, or just create it by hand.
