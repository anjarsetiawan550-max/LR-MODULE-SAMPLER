# LR Module Sampler — Dual Channel Sampler (VST3 + Standalone)

Modul trigger drum pad 2 kanal berbasis **JUCE 8**. Input 1/2 (L/R) mendeteksi pukulan,
lalu memainkan sampling one-shot ke Output 1/2 (L/R).

## Build

Butuh CMake ≥ 3.22 dan compiler C++17. JUCE diunduh otomatis (FetchContent).

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release --parallel
```
Hasil: `build/SampleCore_artefacts/Release/` — `VST3/LR Module Sampler.vst3` dan `Standalone/LR Module Sampler.exe` (di macOS/Linux sesuai platform). Ikon aplikasi dari logo LR (`assets/icon/`).

Linux butuh: `libasound2-dev libfreetype-dev libfontconfig1-dev libx11-dev libxcomposite-dev libxcursor-dev libxext-dev libxinerama-dev libxrandr-dev libxrender-dev libgl1-mesa-dev`.

**GitHub Actions** (`.github/workflows/build.yml`) otomatis build Windows/macOS/Linux setiap push;
file plugin ada di tab *Actions → Artifacts*.

> Versi Standalone: aktifkan input audio di *Options → Audio/MIDI Settings* dan matikan *Mute audio input*.

## Struktur
```
CMakeLists.txt
assets/            semua PNG GUI (masuk BinaryData otomatis)
Source/
  PluginProcessor  trigger, filter, sampler, preset, state
  PluginEditor     layout GUI sesuai mockup
  LcdDisplay.h     LCD: preset, meter, FFT + node HPF/LPF
  UiKit.h          LookAndFeel filmstrip knob/fader, tombol 3-state, knop preset
  Dsp.h            biquad HPF/LPF (24 dB/oct)
```

## Cara pakai
| Kontrol | Fungsi |
|---|---|
| **LCD** | Nama preset, lalu baris sampling (nama file sampling menggantikan "Sampling 1/2") dengan meter output + nilai dB |
| **SELECT (CH1/CH2)** | Pilih channel. Knob parameter utama bekerja untuk channel ini. LCD berubah ke FFT input + node HPF/LPF (geser node) + nama sampling |
| **WAVE** | Masukkan sampling ke channel yang di-SELECT. Tanpa SELECT tidak bekerja |
| **Knop PRESET** | Putar untuk memilih preset. **Klik kiri** pada knop untuk me-restore preset yang dipilih |
| **SAVE** | Simpan semua parameter, sampling, dan level fader; muncul popup Rename |
| **RESET** | Hapus semua preset dan sampling (dengan konfirmasi) + parameter kembali default |
| **LINK** | Gabungkan output 1 dan 2 (kedua output membawa jumlah kedua channel) |
| **CANCEL** | Kembali ke tampilan awal LCD |
| **MUTE / FEDER** | Mute dan level output tiap channel |
| **Knop SENSITIV (strip)** | Sama dengan parameter Sensitivity channel tersebut |

### Parameter (per channel)
- **Sensitivity** — penguatan input sebelum deteksi (−12 … +36 dB). Tinggi = pukulan lembut pun berbunyi.
- **HPF / LPF Input** — 20 Hz – 20 kHz, 24 dB/oct, mencegah bocor antar pad.
- **Threshold** — batas minimum agar trigger aktif.
- **Retrigger** — jeda 5–500 ms setelah satu pukulan (cegah bunyi ganda).
- **Velocity** — hubungan kekuatan pukulan ↔ volume + kecerahan suara. Knob di minimum = **OFF** (volume tetap).

### Lokasi data
Preset (`.scpreset`) dan salinan sampling disimpan di folder `SampleCore/` pada direktori application-data pengguna
(Windows `%APPDATA%`, macOS `~/Library`, Linux `~/.config`).

## Catatan teknis
- Jendela GUI 984×404 px (menyesuaikan susunan aset yang ada).
- Deteksi membaca puncak hanya ~0,6 ms setelah threshold terlewati (untuk velocity), jadi tambahan latensi trigger < 1 ms. Latensi total terutama ditentukan buffer audio: pakai driver ASIO dan buffer 64–128 sampel.
- Polifoni 8 voice per channel, sampling dimix ke mono dan dibatasi 30 detik.
