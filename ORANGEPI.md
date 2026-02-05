# Orange Pi Zero 2W Support

This library now supports the **Orange Pi Zero 2W** with the Allwinner H616 SoC!

## Overview

The Orange Pi Zero 2W is a low-cost single-board computer with a 40-pin GPIO header compatible with RGB LED matrix panels. This port adapts the original Raspberry Pi RGB LED matrix library to work with the Allwinner H616 SoC.

## Hardware Requirements

- Orange Pi Zero 2W board
- RGB LED matrix panel (HUB75 interface)
- 5V power supply (adequate for your LED matrix - typically 2-4A per 32x32 panel)
- Wiring connections as described in [wiring.md](wiring.md)

## Quick Start

### 1. Installation

Clone and build the library:

```bash
git clone https://github.com/sz1jrh/rpi-rgb-led-matrix.git
cd rpi-rgb-led-matrix
make -C lib
make -C examples-api-use
```

### 2. Running Examples

To run the demo on Orange Pi Zero 2W, you **must** specify the Orange Pi GPIO mapping:

```bash
sudo ./examples-api-use/demo --led-gpio-mapping=orangepi-zero2w --led-rows=32 --led-cols=64
```

### 3. Common Command Line Options

```bash
# Basic 32x64 panel
sudo ./examples-api-use/demo \
  --led-gpio-mapping=orangepi-zero2w \
  --led-rows=32 \
  --led-cols=64

# With brightness control
sudo ./examples-api-use/demo \
  --led-gpio-mapping=orangepi-zero2w \
  --led-rows=32 \
  --led-cols=64 \
  --led-brightness=50

# Multiple panels chained
sudo ./examples-api-use/demo \
  --led-gpio-mapping=orangepi-zero2w \
  --led-rows=32 \
  --led-cols=64 \
  --led-chain=2
```

## GPIO Mapping

The Orange Pi Zero 2W uses the same logical GPIO pin numbers as Raspberry Pi for the 40-pin header. The hardware mapping `orangepi-zero2w` is configured to work with standard HUB75 RGB panels using the same wiring as described in [wiring.md](wiring.md).

### Pin Mapping (40-pin header)

The mapping follows the standard RGB matrix wiring:
- OE (Output Enable): GPIO 18
- CLK (Clock): GPIO 17  
- LAT (Latch/Strobe): GPIO 4
- Address lines: GPIO 22, 23, 24, 25, 15 (A, B, C, D, E)
- RGB data: GPIOs 11, 27, 7, 8, 9, 10 (chain 0)

See [wiring.md](wiring.md) for detailed wiring instructions.

## Supported Features

✅ **Supported:**
- Up to 3 parallel chains
- PWM brightness control
- Multiple panel chaining
- Standard HUB75 panels
- 32x32, 32x64, 64x32, 64x64 panels
- Various multiplexing modes

⚠️ **Notes:**
- The library auto-detects Orange Pi Zero 2W by checking for "Allwinner" or "sun50iw9" in `/proc/cpuinfo`
- Hardware PWM timing uses the same characteristics as Raspberry Pi 3
- Memory mapping uses peripheral base address 0x03000000 (H616 SoC specific)

## Permissions

You need root access to access `/dev/mem` for GPIO control:

```bash
sudo ./your-program --led-gpio-mapping=orangepi-zero2w ...
```

## Performance

The Orange Pi Zero 2W has a quad-core ARM Cortex-A53 CPU running at ~1.5GHz, providing performance similar to Raspberry Pi 3. You should be able to:

- Drive panels at high refresh rates (>100Hz)
- Chain multiple panels (tested up to 3 chains)
- Run complex animations smoothly

## Troubleshooting

### "Could not initialize GPIO"
- Make sure you're running with `sudo`
- Verify the board is actually an Orange Pi Zero 2W
- Check that `/dev/mem` is accessible

### Display shows garbage or random patterns
- Check your wiring connections
- Verify power supply is adequate
- Try adding `--led-slowdown-gpio=2` to slow down the signal

### Display is dim or flickering
- Increase brightness: `--led-brightness=100`
- Check power supply voltage and current capacity
- Try `--led-pwm-lsb-nanoseconds=130` to adjust PWM timing

### Wrong GPIO mapping detected
- Explicitly specify: `--led-gpio-mapping=orangepi-zero2w`
- Check that detection message shows "Detected Orange Pi Zero 2W"

## Programming Your Own Applications

When programming with the library in C++:

```cpp
#include <led-matrix.h>

using namespace rgb_matrix;

int main(int argc, char *argv[]) {
    RGBMatrix::Options options;
    options.hardware_mapping = "orangepi-zero2w";  // Important!
    options.rows = 32;
    options.cols = 64;
    options.chain_length = 1;
    
    RuntimeOptions runtime;
    runtime.gpio_slowdown = 1;
    
    RGBMatrix *matrix = RGBMatrix::CreateFromOptions(options, runtime);
    if (matrix == NULL) {
        return 1;
    }
    
    // Your code here...
    
    delete matrix;
    return 0;
}
```

## Additional Resources

- [Original README](README.md) - Main documentation
- [Wiring Guide](wiring.md) - How to connect your panel
- [API Documentation](include/led-matrix.h) - C++ API reference

## Known Limitations

- Orange Pi 5 and other Allwinner SoC boards are not yet supported (only Zero 2W)
- Hardware timing differences may require tuning for some panel types
- Some advanced features may need testing on actual hardware

## Contributing

If you test this on Orange Pi Zero 2W hardware, please report:
- What works
- What doesn't work  
- Any timing adjustments needed
- Panel types tested

## License

Same as the original library: GNU General Public License Version 2.0 (or any later version).
