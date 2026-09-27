# libfprint-gxfp5187

Arch Linux package of [libfprint](https://fprint.freedesktop.org/) 1.94.100
with a driver for the **Goodix GXFP5187** SPI fingerprint sensor, the one in
the power button of the **Huawei MateBook X Pro 2018 (MACH-WX9)**.

It replaces the `libfprint` package (it `provides` and `conflicts` with it),
so `fprintd`, `pam_fprintd`, GNOME and KDE work with the sensor unchanged.

The driver is based on
[libfprint-goodixtls](https://github.com/Sigfrodr/libfprint-goodixtls) by
Benjamin Allègre. The patches in this repository carry its history and
authorship.

## Install

```sh
git clone https://github.com/andmarios/libfprint-gxfp5187.git
cd libfprint-gxfp5187
makepkg -si
```

Then reboot, or bind the sensor and restart fprintd by hand:

```sh
sudo modprobe -r spidev; sudo modprobe spidev
sudo udevadm trigger --action=add --subsystem-match=spi
sudo systemctl restart fprintd
```

Enrol a finger with `fprintd-enroll`. Touch the sensor **lightly** (it is the
power button) and shift your finger a millimetre or two between touches, in the
posture you will use to unlock. A wide enrolment matters more than anything
else for recognition: the sensor only sees about 6×5 mm at a time.

Then enable `pam_fprintd` where you want it. For `sudo`, for example, add these
two lines at the top of the `auth` section of `/etc/pam.d/sudo`:

```
auth    requisite    pam_faillock.so preauth
auth    sufficient   pam_fprintd.so  max-tries=2 timeout=10
```

(The `faillock` line keeps a locked account locked for the fingerprint too.)
Plasma's lock screen uses fingerprints by itself through
`/usr/lib/pam.d/kde-fingerprint`.

## What the package installs

- libfprint with the `goodixtls` driver;
- a udev rule binding the sensor to `spidev` (root-only device node);
- `spidev bufsiz=65536`, needed for the ~22 kB image frame;
- an fprintd drop-in that lets fprintd drive the sensor's reset and interrupt
  lines (`DeviceAllow=/dev/gpiochip0 rw`) and keeps fprintd resident, so the
  fingerprint is offered as soon as the lock screen appears.

## If the sensor stops answering

The sensor's microcontroller can end up in a state where it no longer
responds. A reboot or a suspend does **not** clear it. Power the laptop **off**
with the charger unplugged, wait a couple of minutes, and power it on again.

The driver keeps a single TLS session with the sensor across fprintd's
operations, which is what made this rare. (Opening a new session for every
authentication wedged the sensor after a couple of hundred sessions.)

## Security notes

- **Matching happens on the host**, in the driver, not in the sensor. The
  decision threshold was measured on a single sensor and a small set of
  captures (every capture of four other fingers scored 0), not on a large
  population. Treat it as a convenience login, like any fingerprint reader.
- **The TLS channel protects nothing on the host side.** Its key is read out of
  the sensor's own RAM over the SPI bus. It is a vendor obfuscation layer; the
  device node is kept root-only for that reason.
- Fingerprint templates are stored by fprintd under `/var/lib/fprint`
  (root-only). The driver also keeps up to 20 extra views per finger, learned
  from confident matches, in `/var/lib/fprint/.goodixtls-adapt/`.

## Credits

- The driver: Benjamin Allègre
  ([Sigfrodr/libfprint-goodixtls](https://github.com/Sigfrodr/libfprint-goodixtls)),
  with a contribution by YoranSys.
- Changes in this package (interrupt-gated SPI transport, the idle gap that
  fixed handshake failures, TLS session reuse, a rotation-tolerant matcher,
  review fixes): Marios Andreopoulos.
- Protocol details came from public research by
  [lexakimov/goodix51c0_spi-reversing](https://github.com/lexakimov/goodix51c0_spi-reversing),
  [delitdesnoyers06-del/goodix-gxfp-protocol](https://github.com/delitdesnoyers06-del/goodix-gxfp-protocol),
  [szlukabence/goodix-fingerprint-spi-linux](https://github.com/szlukabence/goodix-fingerprint-spi-linux)
  and [goodix-fp-linux-dev/goodix-fp-dump](https://github.com/goodix-fp-linux-dev/goodix-fp-dump).
  No code was taken from them.
- The packaging is based on Arch Linux's `libfprint` package.

## License

The patches, like libfprint, are **LGPL-2.1-or-later**. The packaging files
(`PKGBUILD`, udev rule, configuration snippets) are **0BSD**, like Arch Linux's
own; see `LICENSE` and `REUSE.toml`.
