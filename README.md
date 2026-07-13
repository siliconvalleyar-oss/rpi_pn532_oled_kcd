# NFC RPi Module

KiCad schematic for an NFC module combining a Raspberry Pi Zero 2W, PN532 NFC reader, and OLED display.

## Project Structure

```
nfc_rpi.kicad_sch          Root schematic
├── nfc.kicad_sch          NFC module (page 2)
│   ├── raspberry_pn532.kicad_sch   Raspberry Pi connector (page 4)
│   ├── pn532.kicad_sch             PN532 NFC controller
│   └── ...
└── ...
```

## Components

- **Raspberry Pi Zero 2W** - Main controller
- **PN532** - NFC/RFID reader/writer (SPI interface)
- **OLED Display** - Status display (I2C interface)

## Hierarchical Pin Mapping

### Raspberry Pi Sheet (J1)

| Pin | Function | Interface |
|-----|----------|-----------|
| SDA | I2C Data | OLED |
| SCL | I2C Clock | OLED |
| GPIO5 | General Purpose | - |
| GPIO6 | General Purpose | - |
| MISO | SPI Data In | PN532 |
| SCK | SPI Clock | PN532 |
| 5V | Power Supply | - |
| GND | Ground | - |

## Notes

- Hierarchical pins connect J1's GPIO pins to the parent schematic
- Global labels (SCL_OLED_RPI, SDA_OLED_RPI) handle I2C routing
- Power distribution uses hierarchical pins for 5V and GND
