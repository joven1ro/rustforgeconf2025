# rustforgeconf 2025 ESP32-RS tutorial

Getting started with Rust development on the ESP-RS development board at RustForgeConf 2025.

## Choosing std vs no_std

**Use std when you need:**
- WiFi/Bluetooth connectivity
- File system access
- Threading and async support
- Rich ecosystem libraries

**Use no_std when you need:**
- Minimal resource usage
- Real-time guarantees
- Direct hardware control
- Faster boot times

## ESP-RS Board Features

### Hardware Specifications
- **ESP32-C3 SoC**: 32-bit RISC-V single-core processor (up to 160MHz)
- **Wireless**: WiFi (802.11 b/g/n) and Bluetooth 5
- **Memory**: 384 KB ROM, 400 KB SRAM, 8 KB RTC SRAM
- **Connectivity**: USB Type-C connector with charging support

### Onboard Peripherals
- **IMU**: ICM-42670-P (I2C)
- **Temperature/Humidity**: SHTC3 sensor (I2C)
- **LEDs**: WS2812 RGB LED (GPIO2), additional LED (GPIO7)
- **Button**: Boot/User button (GPIO9)

### I2C Bus
- SDA: GPIO10
- SCL: GPIO8

### Power
- USB Type-C charging
- Li-Ion battery support (4.2V max)
- No battery voltage reading capability

### no_std (Bare Metal) Development

For resource-constrained applications, real-time systems, or maximum performance.

1. **Setup:**
   ```bash
   rustup toolchain install --component rust-src
   rustup target add riscv32imc-unknown-none-elf
   ```

2. **Optional: try the training examples:**
   ```bash
   git clone https://github.com/esp-rs/no_std-training
   cd no_std-training/intro/hello-world
   cargo run
   ```

3. **Create your own no_std project:**
   ```bash
   esp-generate --chip esp32c3 example-blink
   ```
   When prompted:
   - Under "Flashing, logging and debugging (espflash)": select "use defmt to print messages"
   - Select "esp-backtrace" as the panic handler
   - Optional editor integration: select "zed" or "vscode"


## Basic LED Blink Example

Simple LED control using esp-hal:

**Create the project:**
```bash
esp-generate --chip esp32c3 example-blink
```
When prompted:
- Under "Flashing, logging and debugging (espflash)": select "use defmt to print messages"
- Select "esp-backtrace" as the panic handler
- Optional editor integration: select "zed" or "vscode"

### Key Components

1. **Add the import:**
   ```rust
   use esp_hal::gpio::{Level, Output, OutputConfig};
   ```

2. **Define the peripheral in main():**
   ```rust
   let peripherals = esp_hal::init(config);
   ```

3. **Set the GPIO and LED in main():**
   ```rust
   // Set GPIO7 as an output, and set its state high initially.
   let mut led = Output::new(peripherals.GPIO7, Level::Low, OutputConfig::default());
   led.set_high();
   ```

4. **Toggle in the loop:**
   ```rust
   led.toggle();
   ```

### Complete Example

```rust
#![no_std]
#![no_main]

use defmt::info;
use esp_hal::clock::CpuClock;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use {esp_backtrace as _, esp_println as _};

esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    // Set GPIO7 as an output, and set its state high initially.
    let mut led = Output::new(peripherals.GPIO7, Level::Low, OutputConfig::default());

    led.set_high();

    loop {
        info!("Hello world!");
        let delay_start = Instant::now();
        while delay_start.elapsed() < Duration::from_millis(500) {}

        led.toggle();
    }
}
```

## Button Input Example

Reading button input to control the LED:

**Create the project:**
```bash
esp-generate --chip esp32c3 example-button
```
When prompted:
- Under "Flashing, logging and debugging (espflash)": select "use defmt to print messages"
- Select "esp-backtrace" as the panic handler
- Optional editor integration: select "zed" or "vscode"

### Key Components

1. **Add the input import:**
   ```rust
   use esp_hal::gpio::{Input, InputConfig, Level, Output, OutputConfig};
   ```

2. **Set up the button input:**
   ```rust
   let button = Input::new(peripherals.GPIO9, InputConfig::default());
   ```

3. **Read button state in loop:**
   ```rust
   if button.is_high() {
       led.set_high();
   } else {
       led.set_low();
   }
   ```

### Complete Example

```rust
#![no_std]
#![no_main]

use defmt::info;
use esp_hal::clock::CpuClock;
use esp_hal::gpio::{Input, InputConfig, Level, Output, OutputConfig};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use {esp_backtrace as _, esp_println as _};

esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    // Set GPIO7 as an output, and set its state high initially.
    let mut led = Output::new(peripherals.GPIO7, Level::Low, OutputConfig::default());
    let button = Input::new(peripherals.GPIO9, InputConfig::default());

    info!("Hello world!");

    // Check the button state and set the LED state accordingly.
    loop {
        if button.is_high() {
            led.set_high();
        } else {
            led.set_low();
        }
    }
}
```

## Useful Resources

- [ESP-RS Board Documentation](https://docs.esp-rs.org/esp-rust-board/)
- [ESP-RS Board GitHub](https://github.com/esp-rs/esp-rust-board)
- [Rust on ESP Book](https://docs.espressif.com/projects/rust/book/introduction.html)

## Common Commands

```bash
# Monitor serial output
cargo run --release

# Check for device
espflash board-info

# Erase flash
espflash erase-flash

# Flash without building
espflash write-bin target/xtensa-esp32-espidf/debug/your-app.bin
```

## Troubleshooting

- Ensure USB drivers are installed
- Press and hold BOOT button while connecting USB for manual flash mode
- Check device permissions: `ls -la /dev/tty*`