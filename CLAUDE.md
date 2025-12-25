# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

chiaki-ng is a cross-platform PlayStation Remote Play client that implements the proprietary PlayStation protocol. It's written in C/C++ with Qt for the GUI and supports Linux, Windows, macOS, Android, and Nintendo Switch.

## Build Commands

### Standard Linux Build
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCHIAKI_ENABLE_TESTS=ON -DCHIAKI_ENABLE_CLI=OFF ..
make -j4
```

### Run Tests
```bash
./test/chiaki-unit
# or
ctest
```

### Platform-Specific Builds
- **AppImage**: `scripts/build-appimage.sh`
- **Flatpak**: `flatpak-builder --repo=chiaki-ng-diy --force-clean build chiaki-ng.yaml`
- **Windows**: Use MSYS2 or Visual Studio with vcpkg
- **macOS**: Standard CMake build with Xcode tools

### Key CMake Options
- `CHIAKI_ENABLE_TESTS=ON` - Enable unit tests
- `CHIAKI_ENABLE_GUI=ON` - Build Qt GUI (default)
- `CHIAKI_ENABLE_CLI=ON` - Build command-line tools
- `CHIAKI_ENABLE_FFMPEG_DECODER=AUTO` - FFmpeg video decoder
- `CHIAKI_ENABLE_STEAMDECK_NATIVE=ON` - Steam Deck features

## Architecture Overview

### Core Library (`lib/`)
The C library implements the PlayStation Remote Play protocol:
- **Session Management** (`include/chiaki/session.h`) - Main API entry point
- **Takion Protocol** (`src/takion.c`) - Core protocol implementation
- **RP Crypt** (`src/rpcrypt.c`) - PlayStation encryption/authentication
- **Stream Handling** (`src/streamconnection.c`) - Media stream management
- **Network** (`src/remote/`) - RUDP and holepunch for NAT traversal

### GUI Application (`gui/`)
Qt-based interface that wraps the core library:
- **QML Backend** (`src/qmlbackend.cpp`) - Coordinates GUI and core
- **Stream Session** (`src/streamsession.cpp`) - Qt wrapper for sessions
- **Controller Manager** (`src/controllermanager.cpp`) - SDL2 controller support

### Protocol Flow
1. **Discovery** - Find consoles on network (`lib/src/discovery.c`)
2. **Registration** - Exchange keys with console (`lib/src/regist.c`)
3. **Connection** - Establish Takion protocol session (`lib/src/takion.c`)
4. **Streaming** - Handle video/audio/data channels (`lib/src/streamconnection.c`)

### Key Design Patterns
- **Event-driven callbacks** - All async operations use callback system
- **Platform abstraction** - Platform code isolated in separate directories
- **Layered protocol** - Transport → Protocol → Application layers

### Important Files
- **Main GUI entry**: `gui/src/main.cpp`
- **Core API**: `lib/include/chiaki/chiaki.h`
- **Protocol defs**: `lib/protobuf/takion.proto`
- **Session events**: `lib/include/chiaki/session.h`

## Development Notes

### Dependencies
- CMake 3.10+
- Qt6 (for GUI)
- SDL2 (controller support)
- FFmpeg (video decoding)
- Protobuf (protocol messages)
- OpenSSL/MbedTLS (cryptography)

### Testing
- Unit tests use munit framework
- Test files in `test/` directory
- Run with `./test/chiaki-unit` after building

### Platform Considerations
- **Steam Deck**: Native gyro/haptics in `steamdeck_native/`
- **Android**: JNI bridge in `android/`
- **Switch**: Borealis UI in `switch/`
- **Input**: Setsu library for touchpad (`setsu/`)

### Common Tasks
- **Add new protocol message**: Update `lib/protobuf/takion.proto`
- **Modify GUI**: Edit QML files in `gui/res/qml/`
- **Add controller mapping**: Update SDL2 gamecontrollerdb
- **Debug protocol**: Check `CHIAKI_LOG_VERBOSE` environment variable