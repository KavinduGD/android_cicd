Firebase App Distribution **does not create the version number itself**. It simply **reads the version information that is already inside your APK or AAB**.

For an Android app, the version comes from two values:

- **`versionName`** – the human-readable version (e.g., `1.0.8`)
- **`versionCode`** – an integer that increases with every release (e.g., `42`)

When you upload an APK, Firebase displays:

```text
Version 1.0.8 (Build 42)
```

Where:

- `1.0.8` → `versionName`
- `42` → `versionCode`

---

## If you're using a recent Android project (Gradle Kotlin DSL)

In `app/build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        versionCode = 42
        versionName = "1.0.8"
    }
}
```

---

## If you're using the Groovy DSL

In `app/build.gradle`:

```groovy
android {
    defaultConfig {
        versionCode 42
        versionName "1.0.8"
    }
}
```

---

## Manual versioning

Before each release, update the values:

```groovy
versionCode 43
versionName "1.0.9"
```

Firebase will then show:

```text
Version 1.0.9 (Build 43)
```

---

## Automatic versioning in Azure DevOps

Most teams don't edit these values manually. Instead, the CI/CD pipeline updates them.

For example:

- Build #101

```text
versionName = "1.0.8"
versionCode = 101
```

- Build #102

```text
versionName = "1.0.8"
versionCode = 102
```

Or, if releasing a new feature:

```text
versionName = "1.1.0"
versionCode = 150
```

The pipeline can use Azure DevOps variables such as:

```text
$(Build.BuildId)
```

or

```text
$(Build.BuildNumber)
```

to generate a unique `versionCode`.

---

## Using Git commits

Some teams generate versions like:

```text
Version: 1.0.8
Build: 573
Commit: a4f91c2
```

and include the commit hash in the Firebase release notes while keeping:

- `versionName = 1.0.8`
- `versionCode = 573`

---

## Best practice

A common strategy is:

- **`versionName`**: Only change when you intentionally release a new app version (e.g., `1.2.0` → `1.3.0`).
- **`versionCode`**: Increase for **every single build**. It must always be higher than the previous one, especially if you later publish to Google Play.

This gives Firebase entries such as:

| `versionName` | `versionCode` | Firebase displays        |
| ------------- | ------------- | ------------------------ |
| 1.0.0         | 1             | Version 1.0.0 (Build 1)  |
| 1.0.0         | 2             | Version 1.0.0 (Build 2)  |
| 1.0.1         | 3             | Version 1.0.1 (Build 3)  |
| 1.1.0         | 10            | Version 1.1.0 (Build 10) |

If you're using **Azure DevOps**, I can also show you how to **automatically update `versionCode` and `versionName` during the pipeline** before uploading to Firebase App Distribution.
