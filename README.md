# Android Build Variants Issue on AGP 7.2.1
Reproduces an issue encountered with Android build variants for the [AGP 7.2.1](https://developer.android.com/studio/releases/gradle-plugin#7-2-0) update.

## Android Studio Version
```
Android Studio Chipmunk | 2021.2.1 Patch 1
Build #AI-212.5712.43.2112.8609683, built on May 18, 2022
Runtime version: 11.0.12+0-b1504.28-7817840 aarch64
VM: OpenJDK 64-Bit Server VM by JetBrains s.r.o.
macOS 12.4
GC: G1 Young Generation, G1 Old Generation
Memory: 10240M
Cores: 10
Registry: external.system.auto.import.disabled=true, ide.instant.shutdown=false
Non-Bundled Plugins: org.toml.lang (0.2.155.4114-212), Docker (212.5712.51), org.intellij.plugins.markdown (212.5457.16), org.jetbrains.kotlin (212-1.6.21-release-334-AS5457.46)
```

## Steps to Reproduce
1. Simply clone this repo and try to run a gradle sync on `main` or branch [`agp-7.2.1`](https://github.com/ivanalvarado/Android-Variants-Issue-For-New-AGP/tree/agp-7.2.1)
2. Observe failure
3. Switch to branch [`agp-7.1.3`](https://github.com/ivanalvarado/Android-Variants-Issue-For-New-AGP/tree/agp-7.1.3) and run a gradle sync
4. Observe success
5. Switch to branch [`agp-7.2.1-fix`](https://github.com/ivanalvarado/Android-Variants-Issue-For-New-AGP/tree/agp-7.2.1-fix) and run a gradle sync
6. Observe success

## Build Variant Configuration
- Build types: `debug` and `release`
- Flavor Dimensions: `service` and `distribution`

#### Build Variant List
```
gmsGoogleDebug
gmsSamsungDebug
gmsVivoDebug
gmsOppoDebug
gmsXiaomiDebug
gmsHuaweiDebug
hmsGoogleDebug
hmsSamsungDebug
hmsVivoDebug
hmsOppoDebug
hmsXiaomiDebug
hmsHuaweiDebug
gmsGoogleRelease
gmsSamsungRelease
gmsVivoRelease
gmsOppoRelease
gmsXiaomiRelease
gmsHuaweiRelease
hmsGoogleRelease
hmsSamsungRelease
hmsVivoRelease
hmsOppoRelease
hmsXiaomiRelease
hmsHuaweiRelease
```

## Issue
The project has 24 different variants, which all are not needed for local development. Thus, we're [filtering](https://github.com/ivanalvarado/Android-Variants-Issue-For-New-AGP/blob/main/app/build.gradle#L92) the ones not needed in order to keep the number of tasks generated low. The only ones needed for local development are `gmsGoogleDebug` and `gmsGoogleRelease`. However, on AGP 7.2.1, if we filter all but those variants, we get the following error during a gradle sync:
```
Cannot find a variant matching build type 'debug' and product flavors '[gms]' in :app
```
which is confusing because `gmsGoogleDebug` matches that configuration.

To bypass this error, we simply [**unfilter at least one of the filtered variants**](https://github.com/ivanalvarado/Android-Variants-Issue-For-New-AGP/blob/agp-7.2.1-fix/app/build.gradle#L7) (doesn't matter which) and the gradle sync completes successfully.

Another thing to note, running a simple build command from the terminal like `./gradlew assembleDebug` completes successfully which inclines me to think this issue is at the IDE level.

#### Issue tracker
https://issuetracker.google.com/issues/234353685
