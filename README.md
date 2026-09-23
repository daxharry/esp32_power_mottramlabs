# ESP32 MottramLabs — 4 current-transformer channels

[ESPHome](https://esphome.io/) firmware for the **MottramLabs 4-channel CT** board (ESP32): four RMS currents from **SCT-013** clamps, exposed to Home Assistant.

<p align="center">
  <img src="images/hardware.jpg" alt="4-channel CT board and SCT-013 clamps" width="720">
</p>

<p align="center">
  <img src="images/MLP201185.jpg" alt="MottramLabs MLP201185 PCB" width="480">
</p>

*Vendor render of MLP201185 (Wemos ESP32) — [MottramLabs](https://www.mottramlabs.com/ct_products.html).*

---

## What it is for

Measure **current** on 4 circuits (consumer unit, heat pump, water heater, sockets, PV…) without cutting wires: the clamp goes around **one** live conductor.

This config reports **RMS amperes** (not watts). For power, multiply by your voltage (e.g. 230 V) in HA; without a voltage sensor that assumes cos φ ≈ 1.

The 4 channels have **different calibration curves** (real loads). Recalibrate if you change clamps or boards.

---

## Hardware

| Part | Role | Link |
| --- | --- | --- |
| **MLP201185** (Wemos ESP32) | 4× 3.5 mm jacks + burden | [MottramLabs CT products](https://www.mottramlabs.com/ct_products.html) · [GitHub + schematics](https://github.com/Mottramlabs/ESP32-4-Channel-Mains-Current-Sensor) |
| Variants | MLP201188 NodeMCU 30-pin, MLP201191 ESP32-S2, MLP201193 38-pin | same page |
| **ESP32** (Wemos / NodeMCU) | plugs onto the board | — |
| **SCT-013-000** (100 A / 50 mA) or SCT-013-030 | clamps | [YHDC SCT-013](https://www.yhdc.com/) |
| USB supply | board | — |

<p align="center">
  <img src="images/MLP201185-dims.png" alt="MLP201185 dimensions" width="420">
</p>

ADC pins in **this** YAML (classic ESP32, ADC1 — Wi‑Fi safe):

| Channel | GPIO | Sensor |
| --- | --- | --- |
| CT1 | **GPIO34** | large feeder (calibrated up to ~32 A) |
| CT2 | **GPIO35** | small load (capped at 1 A in the YAML) |
| CT3 | **GPIO36** | ~13 A |
| CT4 | **GPIO39** | ~13 A |

---

## Wiring / safety

1. **Power off** before opening the consumer unit.
2. One clamp = **one** live wire (never live+neutral together, or the current cancels).
3. 3.5 mm jack into CT1…CT4.
4. **mA / 1 V** jumpers must match the clamp type (SCT-013-000 = current, 22 Ω burden on the board).

Mains current is dangerous. If you are not qualified, have an electrician fit the clamps.

---

## ESPHome setup

1. Copy `esp32-power.yaml` + `secrets.yaml.example` → `secrets.yaml`.
2. API key: `openssl rand -base64 32`
3. Flash over USB (ESP32 Dev Module), then OTA:

```bash
esphome run esp32-power.yaml
```

4. Home Assistant: 4 sensors `CT1 Current` … `CT4 Current` (A).

---

## Calibration

The `calibrate_linear` maps come from measurements on **this** board. To recalibrate:

1. Known load (2000 W heater ≈ 8.7 A at 230 V) or a reference clamp.
2. Note the raw ADC (`ctN_adc` is `internal`: set `internal: false` while calibrating).
3. Add `adc -> amps` points in `calibrate_linear`.
4. Set `internal: true` again.

Approximate power in HA:

```yaml
{{ states('sensor.ct1_current') | float(0) * 230 }}
```

---

## License

MIT — see `LICENSE`.
PCB / photos MottramLabs: [ESP32-4-Channel-Mains-Current-Sensor](https://github.com/Mottramlabs/ESP32-4-Channel-Mains-Current-Sensor) (© MottramLabs).
SCT-013 is a YHDC designation.
