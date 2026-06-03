"# 🪟 PANDUAN BUILD JEHO-STORE.APK DI WINDOWS (Step-by-Step)

> Estimasi total waktu: **2-4 jam** (sekali setup), build berikutnya cuma **2-5 menit**.
> Tested di: **Windows 10 / Windows 11** (64-bit).

---

## 📦 BAGIAN A — INSTALL SEMUA SOFTWARE (sekali saja)

### A1. Install Git for Windows
1. Download: https://git-scm.com/download/win
2. Jalankan installer → **Next semua** (default sudah benar)
3. Verifikasi (buka PowerShell baru):
   ```powershell
   git --version
   ```
   Harus muncul versi (contoh: `git version 2.45.0`).

---

### A2. Install Java JDK 17
1. Download **JDK 17 LTS** (Eclipse Temurin / Adoptium):
   https://adoptium.net/temurin/releases/?version=17
2. Pilih: **Windows x64 → .msi installer**
3. Saat install, **CENTANG** opsi:
   - ✅ Set JAVA_HOME variable
   - ✅ Add to PATH
4. Restart PowerShell, verifikasi:
   ```powershell
   java -version       # harus muncul \"openjdk version 17...\"
   echo $env:JAVA_HOME # harus terisi
   ```

---

### A3. Install Go (untuk build sing-box core)
1. Download: https://go.dev/dl/ → pilih `go1.22.x.windows-amd64.msi` (atau versi terbaru)
2. Install → Next default
3. Verifikasi (buka PowerShell baru):
   ```powershell
   go version    # harus muncul \"go version go1.22...\"
   ```

---

### A4. Install Android Studio
1. Download: https://developer.android.com/studio (~1 GB)
2. Jalankan installer → **Next** sampai Finish (5-15 menit)
3. Buka Android Studio pertama kali:
   - Pilih **Standard** setup
   - Setuju semua license
   - Tunggu download Android SDK (~3-5 GB, 15-30 menit tergantung internet)
4. Setelah selesai, di layar **Welcome**, klik kanan bawah **More Actions → SDK Manager**

---

### A5. Install Android SDK & NDK Components
Di **SDK Manager** (Settings → Languages & Frameworks → Android SDK):

#### Tab \"SDK Platforms\" — centang:
- ✅ **Android 15.0 (API 35)** (wajib, untuk compileSdk)
- ✅ **Android 7.0 (API 24)** (untuk testing minimum)

#### Tab \"SDK Tools\" — centang:
- ✅ **Android SDK Build-Tools 35.0.1**
- ✅ **Android SDK Command-line Tools (latest)**
- ✅ **NDK (Side by side)** → versi **27.0.x** (atau yang terbaru)
- ✅ **Android SDK Platform-Tools** (biasanya sudah tercentang)
- ✅ **CMake** (opsional, kadang dibutuhkan)

Klik **Apply** → tunggu download (~2-3 GB, 10-20 menit).

📝 **CATAT 2 PATH INI** (akan dipakai nanti):
- SDK Location: biasanya `C:\Users\<USERNAME>\AppData\Local\Android\Sdk`
- NDK Location: `C:\Users\<USERNAME>\AppData\Local\Android\Sdk
dk\27.0.xxx`

---

### A6. Set Environment Variables

1. Tekan **Win + R**, ketik `sysdm.cpl`, Enter
2. Tab **Advanced** → tombol **Environment Variables**
3. Di **User variables**, klik **New** untuk masing-masing berikut:

| Variable Name | Value (sesuaikan path Anda) |
|---|---|
| `ANDROID_HOME` | `C:\Users\<USERNAME>\AppData\Local\Android\Sdk` |
| `ANDROID_NDK_HOME` | `C:\Users\<USERNAME>\AppData\Local\Android\Sdkdk\27.0.12077973` |

4. Edit variable **Path** (klik Edit → New) → tambahkan **3 baris** ini:
   ```
   %ANDROID_HOME%\platform-tools
   %ANDROID_HOME%\build-tools\35.0.1
   %JAVA_HOME%\bin
   ```

5. **Tutup semua PowerShell**, buka baru, verifikasi:
   ```powershell
   echo $env:ANDROID_HOME
   echo $env:ANDROID_NDK_HOME
   adb --version            # harus muncul
   ```

---

## 📥 BAGIAN B — DOWNLOAD & SETUP PROJECT

### B1. Clone NekoBoxForAndroid
Buka **PowerShell**:
```powershell
# Buat folder kerja
mkdir C:\AndroidProjects
cd C:\AndroidProjects

# Clone repo
git clone https://github.com/MatsuriDayo/NekoBoxForAndroid.git jeho-store
cd jeho-store

# Pin ke versi stabil
git checkout 1.4.2

# Inisialisasi submodule (PENTING!)
git submodule update --init --recursive

# Buat branch sendiri
git checkout -b jeho-store-retro
```

⏱️ Estimasi: 5-15 menit (tergantung internet, total ~500 MB).

---

### B2. Install gomobile
```powershell
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init
```
> Jika error \"gomobile not found\" setelah install, restart PowerShell.

---

## ✏️ BAGIAN C — UBAH IDENTITAS APP JADI \"JEHO-STORE\"

### C1. Edit `nb4a.properties`
Buka file `C:\AndroidProjects\jeho-store
b4a.properties` pakai **Notepad** atau **VS Code**:

**SEBELUM:**
```properties
PACKAGE_NAME=moe.matsuri.nb4a
VERSION_NAME=1.4.2
VERSION_CODE=10402
```

**SESUDAH:**
```properties
PACKAGE_NAME=com.jehostore.vpn
VERSION_NAME=1.0.0
VERSION_CODE=10000
```

💾 Save.

---

### C2. Edit nama app di `strings.xml`
Buka: `C:\AndroidProjects\jeho-store\app\src\main
es\values\strings.xml`

Cari **3 baris paling atas**, ganti:
```xml
<string name=\"app_name\" translatable=\"false\">Jeho-Store</string>
<string name=\"app_name_long\" translatable=\"false\">Jeho-Store VPN</string>
<string name=\"app_desc\">Retro universal proxy toolchain — Jeho-Store edition.</string>
```

💾 Save.

---

### C3. Edit nama output APK
Buka: `C:\AndroidProjects\jeho-store\buildSrc\src\main\kotlin\Helpers.kt`

Pakai **Ctrl+F**, cari `\"NekoBox-\"` (ada 2 tempat), ganti kedua-duanya jadi `\"JehoStore-\"`.

💾 Save.

---

## 🎨 BAGIAN D — PASANG TEMA RETRO

### D1. Buat file warna retro
Buka: `C:\AndroidProjects\jeho-store\app\src\main
es\values\colors.xml`
**Hapus semua isinya**, ganti dengan:

```xml
<?xml version=\"1.0\" encoding=\"utf-8\"?>
<resources>
    <!-- Jeho-Store Retro Palette -->
    <color name=\"jeho_amber\">#FFB000</color>
    <color name=\"jeho_amber_dark\">#CC8800</color>
    <color name=\"jeho_amber_light\">#FFD24D</color>
    <color name=\"jeho_cyan\">#00FFFF</color>
    <color name=\"jeho_magenta\">#FF006E</color>
    <color name=\"jeho_black\">#0D0D0D</color>
    <color name=\"jeho_panel\">#1F1F1F</color>
    <color name=\"jeho_green\">#00FF41</color>
    <color name=\"jeho_red\">#FF3131</color>

    <color name=\"colorPrimary\">@color/jeho_amber</color>
    <color name=\"colorPrimaryDark\">@color/jeho_black</color>
    <color name=\"colorAccent\">@color/jeho_cyan</color>
    <color name=\"colorOnPrimary\">@color/jeho_black</color>
    <color name=\"colorBackground\">@color/jeho_black</color>
    <color name=\"colorSurface\">@color/jeho_panel</color>
</resources>
```

💾 Save.

---

### D2. Download font VT323
1. Buka: https://fonts.google.com/specimen/VT323
2. Klik **Download family** (file `.zip`)
3. Extract, ambil file `VT323-Regular.ttf`
4. **Rename** jadi `vt323.ttf` (huruf kecil semua)
5. Copy ke folder: `C:\AndroidProjects\jeho-store\app\src\main
es\font\`
   - Jika folder `font` belum ada, buat manual.

---

### D3. Edit tema
Cari file `themes.xml` di:
`C:\AndroidProjects\jeho-store\app\src\main
es\values	hemes.xml`
(atau `styles.xml` jika themes.xml tidak ada)

Cari style yang namanya `Theme.SagerNet` atau `AppTheme`. Tambahkan/ganti item-itemnya:

```xml
<style name=\"Theme.SagerNet\" parent=\"Theme.Material3.Dark.NoActionBar\">
    <item name=\"colorPrimary\">@color/jeho_amber</item>
    <item name=\"colorOnPrimary\">@color/jeho_black</item>
    <item name=\"colorSecondary\">@color/jeho_cyan</item>
    <item name=\"android:colorBackground\">@color/jeho_black</item>
    <item name=\"colorSurface\">@color/jeho_panel</item>
    <item name=\"android:statusBarColor\">@color/jeho_black</item>
    <item name=\"android:navigationBarColor\">@color/jeho_black</item>
    <item name=\"android:textColor\">@color/jeho_amber</item>
    <item name=\"android:fontFamily\">@font/vt323</item>
</style>
```

💾 Save.

---

### D4. Ganti ikon launcher
1. Buka project di **Android Studio**: `File → Open → C:\AndroidProjects\jeho-store`
2. Tunggu Gradle sync (5-10 menit pertama kali, **WAJIB SABAR**)
3. Setelah sync selesai:
   - Klik kanan folder `app/src/main/res`
   - Pilih **New → Image Asset**
4. Di dialog:
   - **Icon Type**: Launcher Icons (Adaptive and Legacy)
   - **Name**: `ic_launcher`
   - **Foreground Layer → Asset Type**: Image → pilih PNG logo Anda (1024×1024)
     - Tidak punya logo? Pakai text: pilih **Text** → ketik \"J\" atau \"JS\", font: VT323 → color: `#FFB000`
   - **Background Layer → Color**: `#0D0D0D`
5. Klik **Next → Finish** → **Overwrite** semua file lama.

---

## 🔨 BAGIAN E — BUILD APK

### E1. Sync Gradle di Android Studio
1. Di Android Studio, lihat bawah → **Build** tab
2. Jika ada notif kuning \"Gradle files have changed\", klik **Sync Now**
3. Tunggu sampai bawah muncul \"**BUILD SUCCESSFUL**\" atau \"**Gradle sync finished**\"

⚠️ Jika error sync:
- **File → Invalidate Caches → Invalidate and Restart**
- Atau tutup Android Studio, hapus folder `C:\Users\<USER>\.gradle\caches`, buka lagi

---

### E2. Build native core (sing-box)
Buka **PowerShell di folder project**:
```powershell
cd C:\AndroidProjects\jeho-store
.\gradlew :libcore:assembleRelease
```

⏱️ Estimasi: 5-15 menit (download Go modules + compile).

✅ Sukses jika muncul: `BUILD SUCCESSFUL in Xm Ys`
✅ Cek ada file: `app\libs\libcore.aar`

❌ Jika error \"go not found\": tutup PowerShell, buka baru, ulangi.
❌ Jika error \"NDK not found\": cek `$env:ANDROID_NDK_HOME` benar.

---

### E3. Pilih Build Variant
1. Di Android Studio, klik tab **Build Variants** (biasanya di sidebar kiri bawah)
2. Untuk `:app` module, pilih variant: **`ossDebug`**

> `oss` = open source (paling stabil), `Debug` = bisa install langsung tanpa signing.

---

### E4. Build Debug APK
**Cara 1 — Lewat Android Studio (gampang):**
- Menu: **Build → Build Bundle(s) / APK(s) → Build APK(s)**
- Tunggu (5-15 menit)
- Notif muncul: **\"APK(s) generated successfully\"** → klik **locate**

**Cara 2 — Lewat PowerShell (lebih cepat untuk rebuild):**
```powershell
cd C:\AndroidProjects\jeho-store
.\gradlew assembleOssDebug
```

✅ APK output ada di:
```
C:\AndroidProjects\jeho-store\app\build\outputs\apk\oss\debug\
```

File-nya akan ada beberapa (untuk berbagai arsitektur CPU):
- `JehoStore-arm64-v8a-1.0.0-oss-debug.apk` ← **HP modern (2018+), pakai ini**
- `JehoStore-armeabi-v7a-1.0.0-oss-debug.apk` ← HP lama 32-bit
- `JehoStore-x86_64-1.0.0-oss-debug.apk` ← emulator
- `JehoStore-x86-1.0.0-oss-debug.apk` ← emulator lama

---

## 📲 BAGIAN F — INSTALL KE HP ANDROID

### F1. Aktifkan Developer Mode di HP
1. Buka **Settings → About Phone**
2. Cari **Build Number**, **tap 7 kali** sampai muncul \"You are now a developer\"
3. Kembali ke Settings → **System → Developer Options**
4. Aktifkan: **USB Debugging** ✅
5. Aktifkan: **Install via USB** ✅ (kalau ada)

### F2. Sambungkan HP via USB
1. Colok kabel USB
2. Di HP muncul popup \"**Allow USB debugging?**\" → centang **Always allow** → **OK**
3. Verifikasi di PowerShell:
   ```powershell
   adb devices
   ```
   Harus muncul:
   ```
   List of devices attached
   ABCD1234567       device
   ```

### F3. Install APK
```powershell
cd C:\AndroidProjects\jeho-store\app\build\outputs\apk\oss\debug
adb install -r JehoStore-arm64-v8a-1.0.0-oss-debug.apk
```

✅ Berhasil: `Success`
✅ Buka HP → cari icon **Jeho-Store** di launcher!

---

### F4. Alternatif: Install manual (tanpa kabel)
1. Copy file `.apk` ke HP via **WhatsApp/Telegram ke diri sendiri**, atau Google Drive
2. Buka file di HP → pasti muncul \"**Install blocked**\"
3. Settings → **Apps → Special Access → Install unknown apps** → izinkan untuk browser/file manager
4. Install ulang.

---

## 🔐 BAGIAN G — BUILD RELEASE (untuk Distribusi)

> Hanya perlu jika mau share APK ke orang lain / upload ke Play Store.

### G1. Generate Keystore
```powershell
cd C:\AndroidProjects\jeho-store
keytool -genkey -v -keystore release.keystore -alias jeho-store -keyalg RSA -keysize 2048 -validity 36500
```

Isi pertanyaannya:
- **Password keystore**: buat password kuat (CATAT! jangan lupa!)
- **First/last name**: nama Anda
- **Organizational unit**: Dev
- **Organization**: JehoStore
- **City**: Jakarta (atau kota Anda)
- **State**: DKI
- **Country code**: ID
- **Password key (alias)**: sama dengan password keystore (Enter saja)

✅ File `release.keystore` tergenerate di root project.

⚠️ **BACKUP file ini ke 3 tempat berbeda!** (Drive, USB, email diri sendiri).
Kalau hilang → tidak bisa update aplikasi selamanya.

---

### G2. Buat `local.properties`
Buat file baru: `C:\AndroidProjects\jeho-store\local.properties`

Isinya:
```properties
sdk.dir=C\:\\Users\\YOUR_USERNAME\\AppData\\Local\\Android\\Sdk
KEYSTORE_PASS=password_keystore_anda
ALIAS_NAME=jeho-store
ALIAS_PASS=password_keystore_anda
```

> Ganti `YOUR_USERNAME` dengan username Windows Anda. Perhatikan `\\` (double backslash).

---

### G3. Build Release APK
```powershell
cd C:\AndroidProjects\jeho-store
.\gradlew assembleOssRelease
```

⏱️ 10-20 menit.

✅ Output: `app\build\outputs\apk\oss
elease\JehoStore-arm64-v8a-1.0.0.apk`

APK ini sudah ter-sign dan siap dibagikan!

---

## 🚨 TROUBLESHOOTING WINDOWS

| Error | Solusi |
|-------|--------|
| `'gradlew' is not recognized` | Pakai `.\gradlew` (titik backslash) di PowerShell, atau `gradlew.bat` di CMD |
| `JAVA_HOME is not set` | Reinstall JDK, centang \"Set JAVA_HOME\". Restart PowerShell. |
| `SDK location not found` | Edit `local.properties`, set `sdk.dir=` ke path Android SDK |
| `NDK not configured` | SDK Manager → SDK Tools → centang NDK → Apply |
| Gradle sync stuck di \"Downloading...\" | Internet lambat, sabar. Atau pakai VPN/proxy. |
| `Could not resolve dependencies` | Tutup Android Studio, hapus `C:\Users\<USER>\.gradle\caches`, buka lagi. |
| `Execution failed for task ':app:lintVitalAnalyzeOssRelease'` | Edit `buildSrc\src\main\kotlin\Helpers.kt` → cari `warningsAsErrors = true` ganti ke `false` |
| APK install gagal \"App not installed\" | Uninstall app NekoBox/Jeho-Store lama dulu di HP, lalu install ulang. |
| Build sukses tapi app crash di HP | Pastikan APK yang diinstall sesuai arsitektur CPU HP (arm64-v8a untuk HP modern) |
| Port 8080 atau adb tidak konek | Buka Task Manager → kill process `adb.exe` → jalankan `adb start-server` |
| `path too long` error | Pindahkan project ke folder lebih pendek: `C:\jeho` |

---

## ✅ CHECKLIST URUT (cetak untuk panduan)

### Setup (sekali saja, ~1-2 jam)
- [ ] Install Git
- [ ] Install JDK 17 (centang JAVA_HOME)
- [ ] Install Go 1.22+
- [ ] Install Android Studio
- [ ] Install Android SDK 35 + Build Tools 35.0.1
- [ ] Install NDK 27.0.x
- [ ] Set `ANDROID_HOME` & `ANDROID_NDK_HOME` env var
- [ ] Verifikasi: `adb --version`, `go version`, `java -version` di PowerShell baru

### Project (sekali saja, ~30 menit)
- [ ] `git clone` repo NekoBox → folder `jeho-store`
- [ ] `git checkout 1.4.2`
- [ ] `git submodule update --init --recursive`
- [ ] `go install golang.org/x/mobile/cmd/gomobile@latest && gomobile init`

### Customization (~20 menit)
- [ ] Edit `nb4a.properties` (PACKAGE_NAME, VERSION_NAME)
- [ ] Edit `strings.xml` (app_name = Jeho-Store)
- [ ] Edit `Helpers.kt` (NekoBox → JehoStore)
- [ ] Edit `colors.xml` (palet retro)
- [ ] Edit `themes.xml` (apply colors)
- [ ] Copy font `vt323.ttf` ke `res/font/`
- [ ] Ganti ikon launcher (Image Asset Studio)

### Build (~15 menit pertama, ~3 menit rebuild)
- [ ] Buka project di Android Studio → Sync Gradle
- [ ] `.\gradlew :libcore:assembleRelease`
- [ ] Pilih variant `ossDebug` di Build Variants
- [ ] `.\gradlew assembleOssDebug`
- [ ] Cek file APK ada di `app\build\outputs\apk\oss\debug\`

### Install & Test (~5 menit)
- [ ] Aktifkan USB Debugging di HP
- [ ] Colok HP, `adb devices` muncul
- [ ] `adb install -r JehoStore-arm64-v8a-1.0.0-oss-debug.apk`
- [ ] Buka app di HP, cek tampilan retro
- [ ] Test connect VPN dengan profile Shadowsocks/VMess

### Release (opsional, untuk distribusi)
- [ ] Generate `release.keystore` dengan keytool
- [ ] BACKUP keystore ke 3 tempat aman
- [ ] Buat `local.properties` dengan password
- [ ] `.\gradlew assembleOssRelease`
- [ ] Share APK di `app\build\outputs\apk\oss
elease\`

---

🎮 **Selamat membangun Jeho-Store!**

Jika stuck di langkah manapun, screenshot error → kirim ke saya, saya bantu debug.
"
