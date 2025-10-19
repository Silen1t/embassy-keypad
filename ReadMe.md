# rp-keypad

A `no_std` async keypad driver for Raspberry Pi Pico using Embassy framework.

## Features

- 🚀 Fully async/await compatible with Embassy
- 🔌 Support for matrix keypads 4x4 
- ⚡ Efficient row-scanning with debouncing
- 🎯 Zero-allocation design for embedded systems
- 📦 `no_std` compatible

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
rp_keypad = { git = "https://github.com/Silen1t/rp-keypad.git" }
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
    
    // Create keypad with row outputs and column inputs
    let mut keypad = Keypad::new(
        // Row pins (outputs): GPIO 2, 3, 4, 5
        [
            Output::new(p.PIN_2, Level::High),
            Output::new(p.PIN_3, Level::High),
            Output::new(p.PIN_4, Level::High),
            Output::new(p.PIN_5, Level::High),
        ],
        // Column pins (inputs): GPIO 6, 7, 8, 9
        [
            Input::new(p.PIN_6, Pull::Up),
            Input::new(p.PIN_7, Pull::Up),
            Input::new(p.PIN_8, Pull::Up),
            Input::new(p.PIN_9, Pull::Up),
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

### Important Notes

⚠️ **Critical:** The `Level` for row outputs and `Pull` for column inputs must match the example above:
- Use `Level::High` for all row `Output` pins
- Use `Pull::Up` for all column `Input` pins

This ensures proper key detection. When a key is pressed, the row output drives the column input to the opposite state (High→Low or Low→High), which is detected as a key press.

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
