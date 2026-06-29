# Running the firmware in the simulator

The repository bundles a fork of QEMU (called [mini404](https://github.com/vintagepc/MINI404),
shipped as `qemu-system-buddy`) that boots the real firmware and renders the
printer's LCD/UI in a window. It is handy for previewing the UI, clicking
through menus, or reproducing a bug without flashing real hardware.

*Note: Only **MINI** and **MK4** are currently usable. `XL` is recognized but not
yet implemented; MK3.5/MK3.9/CORE One are not supported by the simulator.*

## Get the dependencies

`bootstrap.py` downloads the toolchain and the `qemu-system-buddy` binary, creates
the `.venv` virtual environment, and installs the Python packages the simulator
needs (`click`, `pillow`, ...).

```
python utils/bootstrap.py
```

*Note: `utils/build.py`, `utils/bootstrap.py` and `utils/simulator` all
re-execute themselves inside `.venv` automatically, so you don't strictly have to
activate it. If you want to run them as a plain `python ...` from an activated
shell instead, do:*

```
source .venv/bin/activate
```

## Build the firmware

The simulator runs a firmware you build yourself. Build for one of the supported
printers (see [the README](../README.md) for all build options):

```
python utils/build.py --preset mini
```

The binaries land in `./build/mini_release_boot/` (and a copy in
`./build/products`). Use `--preset mk4` for the MK4.

## Install system libraries (Linux / WSL)

The `qemu-system-buddy` binary is linked against a few OpenGL runtime libraries
that are not always installed. If they are missing you will see an error like
`error while loading shared libraries: libGLU.so.1: cannot open shared object
file`. On Debian/Ubuntu install them with:

```
sudo apt install libglu1-mesa freeglut3 libglew2.2
```

The three libraries are `libGLU`, `libglut` and `libGLEW`; on other distributions
install the equivalent packages.

*Note: On WSL2 the simulator window is shown through WSLg, so make sure you are on
a WSL build that includes it (Windows 11, or an up-to-date Windows 10).*

## Run the simulator

Point the simulator at the build directory. It reads the printer model and the
bootloader setting from that directory's `CMakeCache.txt`, copies in the matching
bootloader from `.dependencies`, and opens the window:

```
python utils/simulator build/mini_release_boot
```

That's it ‒ the printer's UI should boot in a few seconds.

*Note: With no argument the simulator looks for a build in `./build-vscode-buddy`
(the directory the VSCode integration creates). When you build with `build.py`,
pass the build directory explicitly as shown above.*

## Tips

- The simulated USB flash drive is the `usbdir/` folder inside the simulator's
  state directory (`build/<preset>/simulator-<printer>/usbdir/`). Drop a `.bbf`
  there to test a firmware update, or a `.gcode` to test printing.
- See all options with `python utils/simulator --help` (e.g. `--printer`,
  `--firmware`, `--simulator-path`).
- To run the automated, OCR-based UI tests against the firmware in the same
  simulator, see [tests/integration/README.md](../tests/integration/README.md).
