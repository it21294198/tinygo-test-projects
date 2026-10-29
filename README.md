# TinyGo Blink program on an Arduino Uno

### Development workflow

```
┌───────────────────────────────┐
│       Your MacBook            │
│                               │
│  main.go                      │
│     │                         │
│     ▼                         │
│  Docker: TinyGo compiler      │
│     │                         │
│     ▼                         │
│  blink.hex                    │
│     │                         │
│     ▼                         │
│  macOS: AVRDude                │
└──────────────┬────────────────┘
               │ USB
               ▼
┌───────────────────────────────┐
│        Arduino Uno            │
│                               │
│  ATmega328P                   │
│     │                         │
│     ▼                         │
│  Built-in LED: ON / OFF       │
└───────────────────────────────┘
```

#### Project structure

```bash
go mod init arduino-blink
touch main.go
```

```bash
arduino-blink/
├── go.mod
└── main.go
```

##### Connect the Arduino, Now identify the serial port.
```bash
ls /dev/cu.*
```
find one like
```
/dev/cu.usbmodem1101
```

### Start docker on desktop.

### Run following to build to `.hex` file.
```bash
docker run --rm \
  -v "$PWD:/src" \
  -w /src \
  tinygo/tinygo:0.44.0 \
  tinygo build \
    -target=arduino-uno \
    -o=/src/blink.hex \
    .
```
| Argument | Purpose |
|---|---|
| `docker run` | Creates and starts a new Docker container. |
| `--rm` | Automatically removes the container after the command finishes. |
| `-v "$PWD:/src"` | Mounts the current directory on your Mac into `/src` inside the container. |
| `-w /src` | Sets `/src` as the container's working directory. |
| `tinygo/tinygo:0.43.0` | Specifies the TinyGo Docker image and version to use. |
| `tinygo build` | Compiles the Go program using TinyGo. |
| `-target=arduino-uno` | Specifies the Arduino Uno compilation target. |
| `-o=/src/blink.hex` | Specifies the output firmware file path. |
| `.` | Tells TinyGo to compile the Go package in the current directory. |

### Upload the build `.hex` firmware
```bash
./avrdude \
  -p atmega328p \
  -c arduino \
  -P /dev/cu.usbmodem1101 \
  -b 115200 \
  -D \
  -U flash:w:blink.hex:i
```

| Argument | Purpose |
|---|---|
| `-p atmega328p` | Selects the microcontroller on the classic Uno R3. |
| `-c arduino` | Uses the Arduino serial bootloader protocol. |
| `-P /dev/cu.usbmodem1101` | Selects the serial port. |
| `-b 115200` | Sets the upload baud rate used by the standard Uno bootloader. |
| `-D` | Prevents AVRDude from issuing its usual chip-erase operation before writing. |
| `-U flash:w:blink.hex:i` | Writes the Intel HEX file to flash memory. |

[Click here For More](https://github.com/avrdudes/avrdude/)