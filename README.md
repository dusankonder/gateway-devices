# gateway-devices

Device profiles for the gateway: register maps of Modbus devices, and integrations, as JSON. Data only, no code.

> **None of these profiles has been tested on hardware by this project.** A wrong register can switch the wrong
> output. Check a profile against the manual of your device before you use it.

## Devices

| Profile | Device | Content | Taken from |
| --- | --- | --- | --- |
| `waveshare-rtu-io-8ch` | Waveshare Modbus RTU IO 8CH | 8 inputs, 8 switched outputs | manufacturer's documentation |
| `waveshare-rtu-relay-8ch` | Waveshare Modbus RTU Relay (8 relays) | 8 switched outputs | manufacturer's documentation |
| `waveshare-rtu-relay-16ch` | Waveshare Modbus RTU Relay 16CH | 16 switched outputs | manufacturer's documentation |
| `sofar-hyd-ktl-3ph` | Sofar HYD 5-20KTL-3PH | 212 values, read only | homeassistant-solax-modbus |
| `velux-klf200` | Velux KLF 200 (io-homecontrol) | integration: blinds, awnings, windows of the KLF | Velux KLF 200 API specification |

The exact source of each profile is in its `source` field.

## Files

- `index.json`: the list that is downloaded first (id, version, kind, name, counts, path).
- `devices/io/<id>.json`: actuators and inputs.
- `devices/<kind>/<id>.json`: devices with their own logic (`inverter`, `wallbox`, `battery`, `heatpump`, `other`).
- `devices/integration/<id>.json`: another system the gateway talks to with a driver of its own firmware.

For a gateway without internet download the files and load them as files, or copy the folder `devices` to
`/library` on the SD card.

## Format 1

Every profile has `format` (1), `id`, `version`, `class`, `vendor`, `model`, `source` (https link) and, except an integration, `serial`
(factory settings: `baud`, `parity` = none / even / odd, `stopBits` = 1 / 2).

- `class: "io"` has `channels`, groups of channels that follow each other: `table`, `start`, `count`, `label`,
  optionally `signal` (output / input / value) and `data`. A coil is an output, a discrete input is an input,
  a holding register is a value unless `signal` says output (writes 1 / 0). At most 128 channels.
- `class: "device"` has `kind`, `license`, `attribution` and `points`: `label`, `group`, `default` (selected
  when the profile is picked) and `register` (`table`, `address`, `data`). All of them are read only.
  Optional `detect`: registers with a serial number whose prefix identifies the family.
- `class: "integration"` has `driver` (the part of the gateway firmware that talks to it, e.g. `velux-klf200`) and
  `port`; no `serial`, no registers. The gateway needs a firmware with that driver.
- `data`: `type` (U16, S16, U32, S32, F32, STR), `order` (big = high word first, little), `scale`, `unit`,
  for STR also `words`, optionally `options` (the meaning of the numbers).
- Tables are `coil`, `discrete`, `holding`, `input`. Addresses are counted from 0, as they travel on the wire.

## Rules

- An `id` never changes. Any change of a profile raises its `version`, and `index.json` follows.
- Every profile names a source that can be checked.
- Only plain channels and values. No all-channels, flash or configuration commands.
- No data of an installation: no IP addresses, serial numbers, names or passwords.

## License

[Apache-2.0](LICENSE).

`devices/inverter/sofar-hyd-ktl-3ph.json` is derived from
[homeassistant-solax-modbus](https://github.com/wills106/homeassistant-solax-modbus) (Copyright 2025 William Swann
and contributors, Apache-2.0, see [LICENSE-solax-modbus](LICENSE-solax-modbus)), file
`custom_components/solax_modbus/plugin_sofar.py` at commit `4acd644e82c1405ea4aa99b678de239d55f2aaae`. It was changed:

- converted to JSON
- read only, nothing that writes was taken over
- values of battery packs (a pack is selected by a write) and computed values left out
- units of setting read-backs taken from the number entities

This project is not affiliated with or endorsed by the manufacturers or by that integration. Product names
belong to their owners.
