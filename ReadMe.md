# rp-keypad

A `no_std` async keypad driver for Raspberry Pi Pico using Embassy framework.

## Features

- 🚀 Fully async/await compatible with Embassy
- 🔌 Support for matrix keypads (4x4 by default, configurable)
- ⚡ Efficient row-scanning with debouncing
- 🎯 Zero-allocation design for embedded systems
- 📦 `no_std` compatible

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
rp_keypad = { git="https://github.com/Silen1t/rp-keypad.git" }
```

### Basic Example

```rust
#![no_std]

use embassy_executor::Spawner;
use embassy_rp::{ gpio::AnyPin, Peripherals };
use embassy_time::{ Duration, Timer };
use embassy_keypad::Keypad;

use ::{ defmt_rtt as _, panic_probe as _ };

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p: Peripherals = embassy_rp::init(Default::default());
    
    // Initialize keypad with row and column pins
    let mut keypad = Keypad::new(
        // Row pins (outputs): GPIO 2, 3, 4, 5
        [
            AnyPin::from(p.PIN_2),
            AnyPin::from(p.PIN_3),
            AnyPin::from(p.PIN_4),
            AnyPin::from(p.PIN_5),
        ],
        // Column pins (inputs): GPIO 6, 7, 8, 9
        [
            AnyPin::from(p.PIN_6),
            AnyPin::from(p.PIN_7),
            AnyPin::from(p.PIN_8),
            AnyPin::from(p.PIN_9)
        ]
    );

    loop {
        if let Some(pressed_key) = keypad.read().await {
            // Handle the pressed key
            defmt::info!("Key pressed: {}", pressed_key);
        }

        Timer::after(Duration::from_millis(10)).await;
    }
}
```

## Hardware Setup

This driver is designed for standard 4x4 matrix keypads with the following layout:

```
[1] [2] [3] [A]
[4] [5] [6] [B] 
[7] [8] [9] [C]
[*] [0] [#] [D]
```

### Wiring

- Connect row pins to GPIO outputs
- Connect column pins to GPIO inputs with pull-up resistors
- The driver scans rows sequentially and reads column states

## How It Works

The driver uses a row-scanning technique:

1. Sets each row LOW sequentially (others HIGH)
2. Waits 20ms for debouncing
3. Reads all column states
4. Returns the first detected keypress
5. Resets the row to HIGH before moving to the next