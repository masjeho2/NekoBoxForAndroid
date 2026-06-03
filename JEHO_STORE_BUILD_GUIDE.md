Saya sudah mengumpulkan semua informasi tentang struktur NekoBoxForAndroid. Sekarang saya akan buat panduan lengkap modifikasi untuk Anda.
Action: file_editor create /app/JEHO_STORE_BUILD_GUIDE.md --file-text "# 🎮 PANDUAN LENGKAP: Build \"Jeho-Store\" dari NekoBoxForAndroid (Tema Retro)

> Panduan modifikasi resmi **NekoBoxForAndroid** (sing-box / universal proxy toolchain) menjadi aplikasi VPN **Jeho-Store** dengan tema **Retro 80an** (CRT / amber phosphor / neon).
>
> Repo sumber: https://github.com/MatsuriDayo/NekoBoxForAndroid
> Versi acuan: **1.4.2** (rilis Feb 2026, commit `5768494`)
> Lisensi: **GPL-3.0** — fork Anda WAJIB tetap open-source di bawah GPL-3.0.

---

## 📋 DAFTAR ISI

1. [Persyaratan Sistem](#1-persyaratan-sistem)
2. [Clone & Setup Awal](#2-clone--setup-awal)
3. [Build Native Core (sing-box / libcore)](#3-build-native-core-sing-box--libcore)
4. [Ubah Identitas Aplikasi (Nama, Package, Ikon)](#4-ubah-identitas-aplikasi)
5. [Pasang Tema Retro 80an](#5-pasang-tema-retro-80an)
6. [Build APK](#6-build-apk)
7. [Signing & Release](#7-signing--release)
8. [Troubleshooting](#8-troubleshooting)
9. [Checklist Final](#9-checklist-final)

---

## 1. PERSYARATAN SISTEM

### Komputer
| Item | Minimal | Disarankan |
|------|---------|------------|
| OS | Windows 10 / macOS 12 / Ubuntu 20.04 | Linux/macOS |
| RAM | 8 GB | 16 GB |
| Storage | 20 GB free | 40 GB SSD |
| CPU | 4 core | 8 core |

### Software (wajib install dulu)

| Software | Versi | Link Download |
|----------|-------|---------------|
| **Android Studio** | Hedgehog 2023.1.1+ (atau lebih baru) | https://developer.android.com/studio |
| **JDK** | 17 (bundled dengan Android Studio) | otomatis |
| **Android SDK** | API 35 (compile) + API 21 (min) | via Android Studio SDK Manager |
| **Android NDK** | 26.1.x atau 27.x | via SDK Manager → SDK Tools |
| **Go** | 1.22+ (untuk build sing-box core) | https://go.dev/dl/ |
| **Git** | terbaru | https://git-scm.com/ |
| **Build Tools** | 35.0.1 | via SDK Manager |

**Verifikasi instalasi (terminal):**
```bash
java -version       # harus 17.x
go version          # harus 1.22+
git --version
echo $ANDROID_HOME  # harus terisi (Linux/macOS)
```

Windows: cek `ANDROID_HOME` di Environment Variables (biasanya `C:\Users\<USER>\AppData\Local\Android\Sdk`).

---

## 2. CLONE & SETUP AWAL

### 2.1 Clone repo
```bash
mkdir -p ~/AndroidProjects && cd ~/AndroidProjects
git clone https://github.com/MatsuriDayo/NekoBoxForAndroid.git jeho-store
cd jeho-store
git checkout 1.4.2     # pin ke versi stabil terbaru
```

### 2.2 Init submodule (penting!)
NekoBox punya submodule untuk sing-box core. Inisialisasi:
```bash
git submodule update --init --recursive
```

### 2.3 Buat branch sendiri
```bash
git checkout -b jeho-store-retro
```

---

## 3. BUILD NATIVE CORE (sing-box / libcore)

NekoBox memakai **Go binary (sing-box)** yang dikompilasi jadi `.so` (native library) lewat module `libcore`. Tanpa langkah ini, app **tidak bisa tunneling**.

### 3.1 Set environment variables
**Linux/macOS** (`~/.bashrc` atau `~/.zshrc`):
```bash
export ANDROID_HOME=$HOME/Android/Sdk           # sesuaikan
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/27.0.12077973   # sesuaikan versi NDK Anda
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:/usr/local/go/bin
```

**Windows (PowerShell, sebagai Admin):**
```powershell
[Environment]::SetEnvironmentVariable(\"ANDROID_HOME\", \"$env:LOCALAPPDATA\Android\Sdk\", \"User\")
[Environment]::SetEnvironmentVariable(\"ANDROID_NDK_HOME\", \"$env:LOCALAPPDATA\Android\Sdk\ndk\27.0.12077973\", \"User\")
```
Restart terminal.

### 3.2 Install gomobile
```bash
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init
```

### 3.3 Build libcore
Dari root project:
```bash
cd libcore
./gradlew :libcore:assembleRelease
# atau jalankan script otomatis:
cd ..
bash buildScript/lib/core.sh
```

> Jika ada error \"go: command not found\" → pastikan Go ada di PATH.
> Jika \"NDK not found\" → buka Android Studio → File → Project Structure → SDK Location → NDK location, copy path-nya ke `ANDROID_NDK_HOME`.

Hasilnya berupa file `.aar` di `app/libs/libcore.aar`. **Jangan dihapus.**

---

## 4. UBAH IDENTITAS APLIKASI

### 4.1 Ubah `applicationId` (package name)

📁 **File: `nb4a.properties`** (di root project)

Buka dan ubah baris `PACKAGE_NAME`:
```properties
# SEBELUM
PACKAGE_NAME=moe.matsuri.nb4a
VERSION_NAME=1.4.2
VERSION_CODE=10402

# SESUDAH
PACKAGE_NAME=com.jehostore.vpn
VERSION_NAME=1.0.0
VERSION_CODE=10000
```

> ⚠️ Gunakan reverse-domain unik (`com.jehostore.vpn` atau `com.namaanda.jehostore`). Setelah dipublish, **TIDAK BISA diubah**.

### 4.2 Ubah `namespace` (opsional tapi disarankan)

📁 **File: `app/build.gradle.kts`** (baris ~32)

Biarkan namespace `io.nekohasekai.sagernet` apa adanya — ini hanya namespace internal Kotlin, bukan applicationId yang dilihat user. Mengubahnya butuh refactor besar-besaran (ribuan import). **Skip.**

### 4.3 Ubah nama aplikasi (label di launcher)

📁 **File: `app/src/main/res/values/strings.xml`** (baris 3-4)

```xml
<!-- SEBELUM -->
<string name=\"app_name\" translatable=\"false\">NekoBox</string>
<string name=\"app_name_long\" translatable=\"false\">NekoBox for Android</string>
<string name=\"app_desc\">The universal proxy toolchain for Android, written in Kotlin.</string>

<!-- SESUDAH -->
<string name=\"app_name\" translatable=\"false\">Jeho-Store</string>
<string name=\"app_name_long\" translatable=\"false\">Jeho-Store VPN</string>
<string name=\"app_desc\">Retro universal proxy toolchain — Jeho-Store edition.</string>
```

### 4.4 Ubah nama file APK output

📁 **File: `buildSrc/src/main/kotlin/Helpers.kt`** (baris ~190-200)

Cari blok:
```kotlin
outputFileName = if (isPreview) {
    outputFileName.replace(
        project.name,
        \"NekoBox-\" + requireMetadata().getProperty(\"PRE_VERSION_NAME\")
    ).replace(\"-preview\", \"\")
} else {
    outputFileName.replace(project.name, \"NekoBox-$versionName\")
        .replace(\"-release\", \"\")
        .replace(\"-oss\", \"\")
}
```

Ganti **kedua** `\"NekoBox-\"` menjadi `\"JehoStore-\"`:
```kotlin
outputFileName = if (isPreview) {
    outputFileName.replace(
        project.name,
        \"JehoStore-\" + requireMetadata().getProperty(\"PRE_VERSION_NAME\")
    ).replace(\"-preview\", \"\")
} else {
    outputFileName.replace(project.name, \"JehoStore-$versionName\")
        .replace(\"-release\", \"\")
        .replace(\"-oss\", \"\")
}
```

### 4.5 Ganti Ikon Aplikasi

**Cara mudah (lewat Android Studio):**
1. Buka project di Android Studio
2. Klik kanan folder `app/src/main/res` → **New → Image Asset**
3. **Icon Type**: Launcher Icons (Adaptive and Legacy)
4. **Name**: `ic_launcher`
5. Pilih asset:
   - **Foreground**: Image → pilih PNG transparan logo Jeho-Store (1024×1024 disarankan)
   - **Background**: Color → pilih `#0D0D0D` (hitam retro) atau `#FFB000` (amber)
6. Klik **Next → Finish** → overwrite semua file yang ada.

File yang otomatis diganti:
- `app/src/main/res/mipmap-*/ic_launcher.png` (semua density)
- `app/src/main/res/mipmap-*/ic_launcher_round.png`
- `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`

**Tips desain ikon retro:**
- Background gelap (`#0D0D0D`)
- Logo amber/orange (`#FFB000`) atau cyan (`#00FFFF`)
- Font pixel/8-bit untuk huruf \"J\" atau \"JS\"
- Bisa pakai: https://fontmeme.com/8bit-font/ untuk generate logo cepat

### 4.6 Hapus konten \"donate / matsuri\" (opsional)

📁 **File: `app/src/main/res/values/strings.xml`**

Hapus atau kosongkan string ini agar Jeho-Store tidak menampilkan donasi NekoBox author:
```xml
<string name=\"donate\"></string>
<string name=\"donate_info\"></string>
```

📁 **File: `app/src/main/res/menu/*.xml`** — hapus menu item donate jika ada.

📁 **File: `app/src/main/java/io/nekohasekai/sagernet/ui/AboutFragment.kt`** (atau serupa) — hapus link Telegram/donasi original.

---

## 5. PASANG TEMA RETRO 80AN

### 5.1 Palet warna retro

📁 **File: `app/src/main/res/values/colors.xml`** (buat baru jika tidak ada, atau edit)

**SEBELUM** (warna default Material):
```xml
<resources>
    <color name=\"colorPrimary\">#3F51B5</color>
    <color name=\"colorPrimaryDark\">#303F9F</color>
    <color name=\"colorAccent\">#FF4081</color>
</resources>
```

**SESUDAH** (palet retro CRT/amber phosphor):
```xml
<?xml version=\"1.0\" encoding=\"utf-8\"?>
<resources>
    <!-- RETRO 80s PALETTE - Jeho-Store -->

    <!-- Primary: Amber CRT phosphor -->
    <color name=\"jeho_amber\">#FFB000</color>
    <color name=\"jeho_amber_dark\">#CC8800</color>
    <color name=\"jeho_amber_light\">#FFD24D</color>

    <!-- Accent: Neon cyan -->
    <color name=\"jeho_cyan\">#00FFFF</color>
    <color name=\"jeho_cyan_dark\">#00B8B8</color>

    <!-- Accent: Neon magenta/pink -->
    <color name=\"jeho_magenta\">#FF006E</color>
    <color name=\"jeho_magenta_dark\">#B8004F</color>

    <!-- Background: CRT black -->
    <color name=\"jeho_black\">#0D0D0D</color>
    <color name=\"jeho_black_soft\">#1A1A1A</color>
    <color name=\"jeho_panel\">#1F1F1F</color>

    <!-- Status -->
    <color name=\"jeho_green\">#00FF41</color>   <!-- terminal green -->
    <color name=\"jeho_red\">#FF3131</color>     <!-- error red -->
    <color name=\"jeho_yellow\">#FFEE00</color>  <!-- warning -->

    <!-- Text -->
    <color name=\"jeho_text_primary\">#FFB000</color>     <!-- amber -->
    <color name=\"jeho_text_secondary\">#FFD24D</color>
    <color name=\"jeho_text_dim\">#806000</color>
    <color name=\"jeho_divider\">#332600</color>

    <!-- Material overrides (digunakan tema) -->
    <color name=\"colorPrimary\">@color/jeho_amber</color>
    <color name=\"colorPrimaryDark\">@color/jeho_black</color>
    <color name=\"colorAccent\">@color/jeho_cyan</color>
    <color name=\"colorOnPrimary\">@color/jeho_black</color>
    <color name=\"colorBackground\">@color/jeho_black</color>
    <color name=\"colorSurface\">@color/jeho_panel</color>
    <color name=\"colorOnSurface\">@color/jeho_amber</color>
</resources>
```

### 5.2 Override tema Material

📁 **File: `app/src/main/res/values/themes.xml`** (atau `styles.xml`)

Cari `Theme.SagerNet` atau `AppTheme`. Ganti parent dan warna:

```xml
<resources xmlns:tools=\"http://schemas.android.com/tools\">

    <!-- BASE THEME - Jeho-Store Retro -->
    <style name=\"Theme.SagerNet\" parent=\"Theme.Material3.Dark.NoActionBar\">
        <!-- Primary -->
        <item name=\"colorPrimary\">@color/jeho_amber</item>
        <item name=\"colorPrimaryVariant\">@color/jeho_amber_dark</item>
        <item name=\"colorOnPrimary\">@color/jeho_black</item>

        <!-- Secondary -->
        <item name=\"colorSecondary\">@color/jeho_cyan</item>
        <item name=\"colorSecondaryVariant\">@color/jeho_cyan_dark</item>
        <item name=\"colorOnSecondary\">@color/jeho_black</item>

        <!-- Background -->
        <item name=\"android:colorBackground\">@color/jeho_black</item>
        <item name=\"colorSurface\">@color/jeho_panel</item>
        <item name=\"colorOnBackground\">@color/jeho_amber</item>
        <item name=\"colorOnSurface\">@color/jeho_amber</item>

        <!-- Status / Nav bar -->
        <item name=\"android:statusBarColor\">@color/jeho_black</item>
        <item name=\"android:navigationBarColor\">@color/jeho_black</item>
        <item name=\"android:windowLightStatusBar\">false</item>
        <item name=\"android:windowLightNavigationBar\">false</item>

        <!-- Text -->
        <item name=\"android:textColor\">@color/jeho_amber</item>
        <item name=\"android:textColorPrimary\">@color/jeho_text_primary</item>
        <item name=\"android:textColorSecondary\">@color/jeho_text_secondary</item>
        <item name=\"android:textColorHint\">@color/jeho_text_dim</item>

        <!-- Font (lihat 5.3) -->
        <item name=\"android:fontFamily\">@font/vt323</item>

        <!-- Ripple -->
        <item name=\"colorControlHighlight\">#33FFB000</item>
    </style>

    <!-- App theme aliases (cek nama yang dipakai di AndroidManifest) -->
    <style name=\"AppTheme\" parent=\"Theme.SagerNet\" />
    <style name=\"AppTheme.NoActionBar\" parent=\"Theme.SagerNet\" />

</resources>
```

> 🔎 **Cara cek tema mana yang aktif**: buka `app/src/main/AndroidManifest.xml`, lihat atribut `android:theme` pada `<application>`. Itulah tema yang harus Anda override.

### 5.3 Font retro pixel

1. Download font **VT323** (free, OFL) dari https://fonts.google.com/specimen/VT323
   - Alternatif: **Press Start 2P**, **Major Mono Display**, **Share Tech Mono**
2. Copy file `VT323-Regular.ttf` ke folder `app/src/main/res/font/` (buat folder jika belum ada)
3. Rename jadi `vt323.ttf` (huruf kecil semua, tanpa strip)
4. Tema sudah otomatis pakai (`android:fontFamily=\"@font/vt323\"` di langkah 5.2)

### 5.4 Tombol \"Connect\" gaya retro (opsional, ubah drawable)

📁 **File baru: `app/src/main/res/drawable/btn_jeho_connect.xml`**

```xml
<?xml version=\"1.0\" encoding=\"utf-8\"?>
<selector xmlns:android=\"http://schemas.android.com/apk/res/android\">
    <item android:state_pressed=\"true\">
        <shape android:shape=\"rectangle\">
            <solid android:color=\"@color/jeho_amber_dark\" />
            <stroke android:width=\"2dp\" android:color=\"@color/jeho_amber\" />
            <corners android:radius=\"0dp\" />
        </shape>
    </item>
    <item>
        <shape android:shape=\"rectangle\">
            <solid android:color=\"@color/jeho_black\" />
            <stroke android:width=\"2dp\" android:color=\"@color/jeho_amber\" />
            <corners android:radius=\"0dp\" />
        </shape>
    </item>
</selector>
```

> Corners radius **0dp** = sudut tajam khas tampilan terminal/CRT (bukan rounded modern).

### 5.5 Efek scanline CRT (opsional, advanced)

Buat overlay scanline di atas activity utama.

📁 **File baru: `app/src/main/res/drawable/scanlines.xml`**
```xml
<?xml version=\"1.0\" encoding=\"utf-8\"?>
<layer-list xmlns:android=\"http://schemas.android.com/apk/res/android\">
    <item>
        <bitmap
            android:src=\"@drawable/scanline_tile\"
            android:tileMode=\"repeat\" />
    </item>
</layer-list>
```

Tambahkan `scanline_tile.png` (gambar 1×4 px: 2 px transparan + 2 px hitam 50%) ke `res/drawable/`. Tempel sebagai `foreground` di root layout MainActivity.

### 5.6 Splash screen retro

📁 **File: `app/src/main/res/values/themes.xml`** — tambahkan:
```xml
<style name=\"Theme.JehoStore.Splash\" parent=\"Theme.SplashScreen\">
    <item name=\"windowSplashScreenBackground\">@color/jeho_black</item>
    <item name=\"windowSplashScreenAnimatedIcon\">@mipmap/ic_launcher_foreground</item>
    <item name=\"postSplashScreenTheme\">@style/Theme.SagerNet</item>
</style>
```

📁 **File: `app/src/main/AndroidManifest.xml`** — set theme di activity launcher:
```xml
<activity android:name=\".ui.MainActivity\"
    android:theme=\"@style/Theme.JehoStore.Splash\">
```

---

## 6. BUILD APK

### 6.1 Sync project
1. Buka Android Studio → **File → Open** → pilih folder `jeho-store`
2. Tunggu Gradle sync selesai (3–10 menit pertama kali)
3. Jika ada error library missing → **File → Invalidate Caches → Restart**

### 6.2 Pilih build variant
Bawah kiri Android Studio → **Build Variants** tab.
Pilih: **`ossDebug`** (untuk testing) atau **`ossRelease`** (untuk distribusi).

> Hindari `play` flavor (memerlukan Google billing) dan `fdroid` (untuk F-Droid only).

### 6.3 Build debug APK (testing cepat)
```bash
./gradlew assembleOssDebug
```
Output: `app/build/outputs/apk/oss/debug/JehoStore-<arch>-<version>-oss-debug.apk`

### 6.4 Build release APK (perlu signing — lihat section 7)
```bash
./gradlew assembleOssRelease
```

### 6.5 Install ke HP via ADB
1. Aktifkan **USB Debugging** di HP (Settings → About → tap Build Number 7×, lalu Developer Options → USB Debugging)
2. Sambungkan HP via USB
3. ```bash
   adb devices                                              # cek HP terdeteksi
   adb install -r app/build/outputs/apk/oss/debug/JehoStore-arm64-v8a-1.0.0-oss-debug.apk
   ```

---

## 7. SIGNING & RELEASE

> WAJIB untuk distribusi. APK debug **tidak bisa** diupload ke Play Store atau sideload jangka panjang.

### 7.1 Generate keystore baru
```bash
keytool -genkey -v \
  -keystore jeho-store.keystore \
  -alias jeho-store \
  -keyalg RSA \
  -keysize 2048 \
  -validity 36500 \
  -storepass GANTI_PASSWORD_KUAT \
  -keypass GANTI_PASSWORD_KUAT \
  -dname \"CN=JehoStore, OU=Dev, O=JehoStore, L=Jakarta, ST=DKI, C=ID\"
```

Simpan file `jeho-store.keystore` di **tempat sangat aman** + backup. Kehilangannya = tidak bisa update aplikasi selamanya.

### 7.2 Konfigurasi signing

Pilihan A — letakkan keystore di root project:
```bash
cp jeho-store.keystore /path/to/jeho-store/release.keystore
```

Buat file `local.properties` (di root, **JANGAN commit ke git**):
```properties
sdk.dir=/path/to/your/Android/Sdk
KEYSTORE_PASS=GANTI_PASSWORD_KUAT
ALIAS_NAME=jeho-store
ALIAS_PASS=GANTI_PASSWORD_KUAT
```

> Sistem signing NekoBox membaca otomatis dari `local.properties` (lihat `Helpers.kt` line 95-110).

### 7.3 Build signed release
```bash
./gradlew assembleOssRelease
```
Output APK ter-sign ada di:
`app/build/outputs/apk/oss/release/JehoStore-arm64-v8a-1.0.0.apk`

### 7.4 Verifikasi signature
```bash
$ANDROID_HOME/build-tools/35.0.1/apksigner verify --print-certs \
  app/build/outputs/apk/oss/release/JehoStore-arm64-v8a-1.0.0.apk
```

---

## 8. TROUBLESHOOTING

| Error | Solusi |
|-------|--------|
| `NDK not configured` | Buka SDK Manager → SDK Tools → centang NDK (Side by side) → Apply |
| `Could not find libcore.aar` | Jalankan ulang langkah 3.3 (build libcore) |
| `Go: command not found` | Install Go 1.22+, tambahkan ke PATH, restart terminal & Android Studio |
| `Task :app:lintVitalAnalyzeOssRelease FAILED` | Edit `Helpers.kt` → set `warningsAsErrors = false` sementara |
| `Duplicate resources` | Hapus drawable lama (logo NekoBox) yang masih ada |
| `App crash di Android 14+` | Pastikan `targetSdk = 35` dan tambahkan `android:foregroundServiceType=\"dataSync\|systemExempted\"` di `AndroidManifest.xml` untuk VpnService |
| Tema tidak berubah | Periksa nama style di `AndroidManifest.xml` `android:theme=...` cocok dengan yang Anda override |
| Font tidak muncul | Pastikan nama file font **huruf kecil, tanpa strip/spasi** (`vt323.ttf`) |
| Keystore lost | **TIDAK ADA SOLUSI** — backup baik-baik |

### Jika app tidak bisa connect ke VPN
1. Pastikan permission VPN diberikan saat first launch (popup Android)
2. Cek `Settings → VPN` di Android — pastikan Jeho-Store ada di daftar
3. Cek logcat: `adb logcat | grep -i \"jeho\|sagernet\|sing-box\"`
4. Test dengan profile sederhana dulu (Shadowsocks public server)

---

## 9. CHECKLIST FINAL

Sebelum distribusi:

- [ ] `nb4a.properties` → `PACKAGE_NAME=com.jehostore.vpn`, `VERSION_NAME=1.0.0`
- [ ] `strings.xml` → `app_name=Jeho-Store`
- [ ] Ikon launcher diganti (semua 6 density)
- [ ] `colors.xml` & `themes.xml` palet retro
- [ ] Font VT323 di `res/font/`
- [ ] Output APK terbaca **JehoStore-x.y.z.apk** (cek `Helpers.kt`)
- [ ] Donate / Telegram link NekoBox dihapus
- [ ] Keystore signed release
- [ ] Test install di HP fisik (minimal Android 7.0)
- [ ] Test connect VPN dengan profile Shadowsocks/VMess
- [ ] README.md fork sebutkan kredit ke MatsuriDayo (wajib GPL-3.0)

### Wajib hukum (GPL-3.0)
Karena NekoBox lisensi **GPL-3.0**, fork \"Jeho-Store\" Anda HARUS:
1. Tetap open-source
2. Publish source code di GitHub publik
3. Sebutkan kredit `Matsuri Dayo / NekoBoxForAndroid` di About / README
4. Tetap pakai lisensi GPL-3.0

Tambahkan ke `README.md` fork Anda:
```markdown
# Jeho-Store VPN

A retro-themed fork of [NekoBoxForAndroid](https://github.com/MatsuriDayo/NekoBoxForAndroid)
by Matsuri Dayo. Licensed under GPL-3.0.

## Credits
- Original: [MatsuriDayo/NekoBoxForAndroid](https://github.com/MatsuriDayo/NekoBoxForAndroid)
- Core: [SagerNet/sing-box](https://github.com/SagerNet/sing-box)
```

---

## 📞 BANTUAN LANJUTAN

- **Forum NekoBox**: https://t.me/Matsuridayo
- **sing-box docs**: https://sing-box.sagernet.org/
- **Plugin tambahan**: https://matsuridayo.github.io/nb4a-plugin/
- **Android dev docs**: https://developer.android.com/studio/build

---

**Selamat membangun Jeho-Store!** 🕹️🎮
Estimasi waktu setup awal (clone + build pertama): **45–90 menit**.
Setelah setup awal beres, rebuild cuma butuh **2–5 menit**.
"
Observation: Create successful: /app/JEHO_STORE_BUILD_GUIDE.md

Fetching debug logs for expo

==> /var/log/supervisor/expo.err.log <==
CommandError: failed to start tunnel

remote gone away

Check the Ngrok status page for outages: https://status.ngrok.com/
error Command failed with exit code 1.

==> /var/log/supervisor/expo.out.log <==
yarn run v1.22.22
$ /app/frontend/node_modules/.bin/expo start --tunnel --port 3000
env: load .env
env: export EXPO_TUNNEL_SUBDOMAIN EXPO_PACKAGER_HOSTNAME EXPO_PUBLIC_BACKEND_URL EXPO_USE_FAST_RESOLVER METRO_CACHE_ROOT EXPO_PACKAGER_PROXY_URL
Starting project at /app/frontend
Fast resolver is enabled.
Metro is running in CI mode, reloads are disabled. Remove CI=true to enable watch mode.
Starting Metro Bundler
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
yarn run v1.22.22
$ /app/frontend/node_modules/.bin/expo start --tunnel --port 3000
env: load .env
env: export EXPO_TUNNEL_SUBDOMAIN EXPO_PACKAGER_HOSTNAME EXPO_PUBLIC_BACKEND_URL EXPO_USE_FAST_RESOLVER METRO_CACHE_ROOT EXPO_PACKAGER_PROXY_URL
Starting project at /app/frontend


Fetching debug logs for backend

==> /var/log/supervisor/backend.err.log <==
INFO:     Will watch for changes in these directories: ['/app/backend']
INFO:     Uvicorn running on http://0.0.0.0:8001 (Press CTRL+C to quit)
INFO:     Started reloader process [236] using WatchFiles
INFO:     Started server process [264]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Shutting down
INFO:     Waiting for application shutdown.
INFO:     Application shutdown complete.
INFO:     Finished server process [264]
INFO:     Stopping reloader process [236]
INFO:     Will watch for changes in these directories: ['/app/backend']
INFO:     Uvicorn running on http://0.0.0.0:8001 (Press CTRL+C to quit)
INFO:     Started reloader process [369] using WatchFiles

==> /var/log/supervisor/backend.out.log <==


Fetching debug logs for mongodb

tail: cannot open '/var/log/supervisor/mongodb*.log' for reading: No such file or directory
