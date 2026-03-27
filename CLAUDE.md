# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MacroTrack is a personal-use Flutter iOS app for tracking daily macronutrient intake (protein, carbs, fat, calories). Built for a single user — no auth, no cloud sync. Targets iOS only; builds are handled via Codemagic CI from a public GitHub repo (Windows development machine).

## Tech Stack

- **State management:** Riverpod with code generation (`@riverpod`)
- **Local database:** Drift (SQLite) — offline-first, no remote sync
- **Nutrition API:** Nutritionix (barcode + search lookups)
- **Barcode scanning:** `mobile_scanner`
- **Charts:** `fl_chart`

## Common Commands

```bash
flutter pub get                      # Install dependencies
flutter pub run build_runner build   # Generate Riverpod/Drift code (run after model changes)
flutter pub run build_runner watch   # Watch mode for code generation during development
flutter run                          # Run on connected device/simulator
flutter test                         # Run all tests
flutter test test/foo_test.dart      # Run a single test file
flutter analyze                      # Static analysis
dart format lib/                     # Format Dart code
```

## Architecture

Feature-first structure under `lib/`:

```
lib/
├── main.dart
├── app/               # App-level config, routing (go_router), theme
├── features/
│   ├── dashboard/     # Today's macro rings + meal log
│   ├── library/       # SavedFood and Recipe management
│   ├── analytics/     # Weekly/monthly charts
│   └── profile/       # Goals, unit preferences
├── shared/
│   ├── models/        # Drift table definitions and data classes
│   ├── widgets/       # Reusable UI components
│   └── utils/
└── services/
    ├── database/      # Drift DB setup and DAOs
    ├── nutrition_api/ # Nutritionix client
    └── storage/       # Shared preferences (user goals, settings)
```

Each feature folder follows the pattern: `screen.dart`, `provider.dart`, `widgets/`.

## Data Model

- **UserGoals** — daily calorie/macro targets, unit preference (single row, no user ID)
- **SavedFood** — id, name, macros per serving, serving size/unit — atomic unit for both direct logging and recipe ingredients
- **Recipe** — id, name; macros are derived by summing `RecipeIngredient` children
- **RecipeIngredient** — links a `SavedFood` to a parent `Recipe` with a quantity multiplier
- **FoodEntry** — id, date, meal slot (breakfast/lunch/dinner/snacks), servings, source (manual/barcode/library), nullable FK to either `SavedFood` or `Recipe`
- **WaterLog** — date, total ml consumed

## Key Behaviors

- **Barcode flow:** Scan → Nutritionix lookup → pre-fill form defaulting to 1 serving (user adjusts multiplier) → confirm to log
- **Barcode fallback:** Product not found or no network → manual entry form with toast explaining why
- **Navigation:** 4-tab bottom nav — Dashboard, Library, Analytics, Profile
- **Units:** Metric/imperial toggle in Profile; stored preference applied app-wide
- **Dark mode:** Supported from day one

## Code Generation

Drift and Riverpod both require `build_runner`. Always run `flutter pub run build_runner build --delete-conflicting-outputs` after:
- Adding or modifying Drift table definitions
- Adding or modifying `@riverpod` annotated providers
