# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simple Live is a cross-platform live streaming aggregation application built with Flutter/Dart. It aggregates multiple Chinese live streaming platforms (Huya, Douyu, Bilibili, Douyin) into a single unified interface.

## Project Structure (Monorepo)

This repository contains 4 distinct Flutter/Dart projects:

- **simple_live_core**: Core library that implements live stream data fetching and danmaku (bullet comments) parsing for all supported platforms. Defines platform-agnostic interfaces (`LiveSite`, `LiveDanmaku`) with platform-specific implementations.

- **simple_live_app**: Main Flutter application for mobile and desktop (Android/iOS/Windows/macOS/Linux). Uses GetX for state management, Hive for persistence, and media_kit for video playback.

- **simple_live_tv_app**: Android TV optimized version with remote control navigation.

- **simple_live_console**: Command-line testing tool based on simple_live_core.

## Development Commands

### Building and Running

```bash
# Run mobile/desktop app
cd simple_live_app
flutter pub get
flutter run

# Run TV app
cd simple_live_tv_app
flutter pub get
flutter run

# Run console app
cd simple_live_console
dart pub get
dart run bin/simple_live_console.dart

# Run core library tests
cd simple_live_core
dart pub get
dart test
```

### Code Generation

The app uses Hive for database models which requires code generation:

```bash
cd simple_live_app
flutter pub run build_runner build --delete-conflicting-outputs
```

### App Icon Generation

```bash
cd simple_live_app
flutter pub run flutter_launcher_icons
```

## Architecture Details

### Core Library (`simple_live_core`)

The core library uses a **plugin pattern** where each platform implements two main interfaces:

1. **LiveSite**: Fetches room details, categories, search results, and playback URLs
   - `BilibiliSite`, `DouyuSite`, `HuyaSite`, `DouyinSite`

2. **LiveDanmaku**: WebSocket-based real-time danmaku/chat parsing
   - `BilibiliDanmaku`, `DouyuDanmaku`, `HuyaDanmaku`, `DouyinDanmaku`

Platform implementations are in `lib/src/*_site.dart` and `lib/src/danmaku/*_danmaku.dart`.

Each platform has unique protocols:
- **Bilibili**: Uses protobuf for danmaku
- **Douyu**: Custom STT protocol
- **Huya**: TARS protocol (uses custom `tars_dart` package)
- **Douyin**: Protobuf with signature verification via QuickJS

### App Architecture (`simple_live_app`)

Uses **GetX MVC pattern**:

- **Models** (`lib/models/`): Data models and Hive database adapters
  - `db/`: Hive models for follow_user, history, tags
  - `account/`: Platform account models

- **Services** (`lib/services/`): Global singleton services initialized at startup
  - `DBService`: Hive database management
  - `LocalStorageService`: Key-value storage
  - `FollowService`: Follow/subscription management
  - `SyncService`: WebDAV cloud sync
  - `BiliBiliAccountService`, `DouyinAccountService`: Platform login

- **Modules** (`lib/modules/`): Feature modules, each with:
  - `*_controller.dart`: GetX controller (business logic)
  - `*_page.dart`: UI view
  - Sub-modules for different platforms/features

- **Routes** (`lib/routes/`): GetX navigation configuration

### Video Playback

Uses `media_kit` with native libmpv backend. Platform-specific implementations handle:
- Fullscreen management
- Picture-in-Picture (PIP)
- Brightness/volume gestures
- Screen orientation

### Danmaku Rendering

Uses `canvas_danmaku` package for efficient Canvas-based rendering with multiple lanes to prevent overlapping.

## Key Dependencies

- **GetX**: State management, routing, dependency injection
- **Hive**: NoSQL database for offline storage
- **media_kit**: Video player with libmpv backend
- **Dio**: HTTP client with interceptors
- **canvas_danmaku**: Danmaku/bullet comment rendering

## Testing Notes

- Core library has unit tests in `simple_live_core/test/`
- Run tests with `dart test` in the core directory
- Apps are primarily tested manually; no UI test framework configured

## Platform-Specific Notes

### Desktop (Windows/macOS/Linux)
- Uses `window_manager` for window control
- Data migrates from Documents to Application Support directory on first run
- Minimum window size: 280x280

### Android TV
- Uses `flutter_screenutil` for TV screen adaptation
- Optimized for D-pad navigation
- No touch gestures

### Mobile
- Supports PIP mode via `floating` package
- Handles system UI (status bar, navigation bar) with edge-to-edge display
- Permission handling for storage, notifications

## Important Files

- `assets/play_config.json`: Video player configuration
- `assets/tv_app_version.json`: TV app version info
- `.github/workflows/`: CI/CD for building releases across platforms

## Development Workflow

1. Core library changes should be tested with `dart test` before use in apps
2. When modifying Hive models, regenerate code with build_runner
3. Apps depend on core via path dependency (`path: ../simple_live_core`)
4. Desktop builds use flutter_distributor for packaging (see `distribute_options.yaml`)
