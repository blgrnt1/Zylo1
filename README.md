# Zylo - Construction Marketplace App

A modern Android application built with Kotlin and Jetpack Compose for construction project management and marketplace functionality.

## 🏗️ Features

- **Firebase Authentication** - Secure user login and registration
- **Firestore Database** - Real-time project data management  
- **Firebase Storage** - Image and file storage for construction photos
- **Jetpack Compose UI** - Modern, responsive user interface
- **Construction Project Management** - Track and manage construction projects

## 🛠️ Tech Stack

- **Language:** Kotlin
- **UI Framework:** Jetpack Compose
- **Backend:** Firebase (Auth, Firestore, Storage)
- **Architecture:** MVVM with Repository pattern
- **Build System:** Gradle with Kotlin DSL
- **Minimum SDK:** 23 (Android 6.0)
- **Target SDK:** 36 (Android 14)

## 📱 Getting Started

### Prerequisites
- Android Studio Hedgehog or newer
- JDK 11 or higher
- Android SDK 36
- Firebase project setup

### Setup
1. Clone the repository
2. Open in Android Studio
3. Add your `google-services.json` file to the `app/` directory
4. Sync the project with Gradle files
5. Build and run

### Build Commands
```bash
# Debug build
./gradlew assembleDebug

# Release build  
./gradlew assembleRelease

# Install on device
./gradlew installDebug

# Run tests
./gradlew test
```

## 🔧 Configuration

### Firebase Setup
1. Create a Firebase project
2. Add Android app with package name: `com.zylo.marketplace`
3. Download `google-services.json` and place in `app/` directory
4. Enable Authentication, Firestore, and Storage in Firebase Console

## 📂 Project Structure

```
app/
├── src/main/java/com/zylo/marketplace/
│   ├── ui/                 # Compose UI components
│   ├── data/              # Repository and data sources
│   ├── domain/            # Business logic and models
│   └── MainActivity.kt    # Main entry point
├── src/main/res/          # Resources (layouts, strings, etc.)
└── build.gradle.kts       # App-level build configuration
```

## 🚀 Development

This project follows Android development best practices:
- MVVM architecture pattern
- Repository pattern for data management
- Dependency injection ready
- Material 3 design system
- Reactive programming with Compose

## 📄 License

[Add your license here]

## 🤝 Contributing

[Add contributing guidelines here]
