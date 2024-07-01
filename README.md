
markdown
Copy code
# Flutter Riverpod MVVM Template

A comprehensive Flutter application template using Riverpod for state management and MVVM architecture. This template includes localization, flavor separation, initialization with dart define JSON, helpful extensions, application preferences, and more. It also demonstrates best practices for using GoRouter for navigation, logging, and interceptors with Dio.

## Features

- State Management: Riverpod
- Architecture: MVVM (Model-View-ViewModel)
- Localization
- Flavor Separation (Environments)
- Initialization with dart define JSON
- Helpful Extensions (e.g., date-time formatting)
- Application Preferences
- Theming: Text themes and component colors generated from XML using flutter_gen
- Asset Generation: Using flutter_gen
- App Preparation: Using Melos
- Native Splash Screen Generator: Using flutter_native_splash
- Navigation: GoRouter
- Logging: Logger and observer patterns
- Network Requests: Dio with interceptors

## Getting Started

### Prerequisites

- Flutter SDK
- Dart SDK

### Installation

Clone the repository:

```bash
git clone https://github.com/yourusername/flutter_riverpod_mvvm_template.git
cd flutter_riverpod_mvvm_template
```

```bash
flutter pub get
```

```bash
flutter pub run build_runner build
```

## Project Structure

```plaintext
lib/
├── src/
│   ├── auth/
│   ├── router/
│   ├── screen/
│   ├── utils/
│   └── ...
├── main.dart
└── ...

```



