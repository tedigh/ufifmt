# ufifmt

Format floppy disks in USB UFI floppy drives using PyUSB / libusb.

This is a Python rewrite of [ufiformat](https://github.com/tedigh/ufiformat),
originally written in C.

## Requirements

- Python 3.10+
- [pyusb](https://github.com/pyusb/pyusb)
- libusb 1.0
- [pyudev](https://pyudev.readthedocs.io/) *(optional, Linux only)*
  — enables device path arguments such as `/dev/sdb` or `/dev/sg1`,
  and shows device names in `--inquire` list output

## Installation

### libusb

```sh
# macOS
brew install libusb

# Debian / Ubuntu
sudo apt install libusb-1.0-0
```

### Python dependencies

```sh
pip install pyusb

# Optional (Linux only)
pip install pyudev
```

## Usage

```
usage: ufiformat [-h] [-i] [-f KB] [-V] [-F] [-v] [-q] [--version] [LOCATION]

Format a floppy disk in a USB UFI floppy drive.

positional arguments:
  LOCATION              device location (e.g. 1-2.3, /dev/sdb, /dev/sg1);
                        omit with -i to list all devices

options:
  -h, --help            show this help message and exit
  -i, --inquire         show device information instead of formatting;
                        omit LOCATION to list all devices
  -f KB, --format KB    format size in KB (default: use current media size)
  -V, --verify          verify after formatting
  -F, --force           skip safety checks
  -v, --verbose         increase verbosity (-v or -vv)
  -q, --quiet           suppress all non-error output
  --version             show program's version number and exit

macOS: eject the disk in Finder (or: diskutil eject /dev/diskN) before running.
Linux: run as root, or add a udev rule for the device.
```

### Examples

```sh
# List all connected UFI floppy drives
ufifmt -i

# Show device and media information
ufifmt -i 1-2.3
ufifmt -i /dev/sdb        # requires pyudev

# Format using the current media size
ufifmt 1-2.3
ufifmt /dev/sdb           # requires pyudev

# Format to a specific size
ufifmt -f 720 1-2.3

# Format and verify
ufifmt -V 1-2.3
```

Supported format sizes: 640, 720, 1200, 1232, 1440 KB — available sizes depend on what your drive reports as formattable.

## Linux Notes

The kernel USB mass-storage driver is detached automatically before formatting
and re-attached on exit.

You may need to run as root, or add a udev rule to grant access without root:

```
# /etc/udev/rules.d/99-ufifmt.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="YOUR_VID", ATTR{idProduct}=="YOUR_PID", MODE="0666"
```

Run `ufifmt -i` to find the VID/PID of your drive, then reload rules:

```sh
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Installing pyudev and libudev enables device path arguments such as `/dev/sdb`
or `/dev/sg1`, and adds a `devices` column to the `--inquire` list output.

## macOS Notes

Before running, eject the floppy disk in Finder, or from the command line:

```sh
diskutil eject /dev/diskN
```

This releases the IOKit driver so libusb can claim the USB interface.

**macOS support has not been tested.** IOKit security restrictions may prevent
libusb from claiming the interface even after ejecting. Reports from macOS users
are very welcome — please open an issue with your macOS version, drive model,
and any error messages.

## Known Limitations

- macOS: untested; may not work due to IOKit driver restrictions.
- Device path arguments (`/dev/sdb`, `/dev/sg1`) require pyudev and are Linux only.

## License

GNU General Public License v2.0 or later — see [COPYING](COPYING) or
<https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>.
