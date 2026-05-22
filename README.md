[⟵ English](README-en.md)

# rk-llama.cpp

Fork dari llama.cpp yang telah diintegrasikan dengan NPU Rockchip sebagai backend GGML. Fork ini dioptimalkan untuk berjalan pada perangkat Orange Pi berbasis RK3588, seperti Orange Pi 5, Orange Pi 5B, dan lainnya.

---

## Deskripsi

Proyek ini mengintegrasikan NPU Rockchip sebagai backend komputasi untuk memungkinkan inferensi (inference) model bahasa besar dengan performa tinggi dan konsumsi daya rendah.

Tujuan utamanya adalah memanfaatkan NPU sebagai akselerator yang kuat. Pendekatan ini bertujuan mendapatkan kecepatan lebih tinggi dibandingkan hanya menggunakan CPU, sambil mempertahankan akurasi yang setara dengan konsumsi daya lebih hemat.

Saat ini backend telah dioptimalkan dan diuji untuk SoC **RK3588**. Dukungan untuk SoC Rockchip lainnya dapat ditambahkan melalui pembaruan file konfigurasi.

---

## Memulai dengan Cepat

### 1. Clone repository

```sh
git clone https://github.com/thanhtantran/rk-llama.cpp
cd rk-llama.cpp

# Instal paket yang diperlukan
sudo apt install libcurl4-openssl-dev
sudo apt install cmake make build-essential
```

Periksa apakah RKNPU sudah terpasang:

```bash
admin@orangepi5:~$ sudo cat /sys/kernel/debug/rknpu/version
RKNPU driver: v0.9.8

# Izinkan pembacaan dma_heap
sudo chmod 666 /dev/dma_heap/*
```

---

### 2. Build proyek

```sh
mkdir build && cd build
cmake .. -DLLAMA_RKNPU2=ON
make -j8
```

---

### 3. Menjalankan inference

```sh
# Menjalankan model dari HuggingFace
bin/llama-cli -hf unsloth/gemma-3-1b-it-GGUF

# Menjalankan model Dense
bin/llama-cli -m ~/gguf/gemma-3-1b-it-Q8_0.gguf

# Menjalankan model MoE
bin/llama-cli -m ~/gguf/LFM2-8B-A1B-Q4_0.gguf --cpu-moe
```

---

## Kuantisasi (Quantization)

### Bobot (Weights)

`FP16`  
Bobot **F16** digunakan langsung sebagai **FP16**.

`INT8`  
Bobot **Q8_0** didekuantisasi (dequantize) menjadi FP32, lalu dikuantisasi ulang menjadi **INT8** per-tensor.

`INT4`  
Bobot **Q4_0** didekuantisasi, diputar menggunakan transformasi Hadamard acak (lihat https://arxiv.org/abs/2404.00456), dikalibrasi menggunakan KL-Divergence (lihat https://arxiv.org/abs/2411.02530), lalu dikuantisasi ulang menjadi **INT4**.

---

### Aktivasi (Activations)

`FP16`  
Aktivasi **F32** dikonversi menjadi **FP16**.

`INT8`  
Aktivasi **F32** dikuantisasi menjadi **INT8** menggunakan skala per-channel.

`INT4`  
Aktivasi **F32** diputar menggunakan Hadamard sebelum dikuantisasi menjadi **INT4**.

---

### Hasil (Results)

`FP32`  
Hasil **FP32** dari NPU dipertahankan apa adanya.

`INT32`  
Hasil **INT32** didekuantisasi menjadi **F32** berdasarkan skala bobot dan aktivasi.

`INT16`  
Hasil **INT16** didekuantisasi menjadi **F32** dengan tambahan faktor normalisasi.

---

## Chipset

### RK3588

Backend mendukung tipe komputasi berikut:

- **W16A16**: FP16 bobot & FP16 aktivasi
- **W8A8**: INT8 bobot & INT8 aktivasi
- **W4A4**: INT4 bobot & INT4 aktivasi

---

## Benchmark

| Model | Type | Backend | Perplexity | PP (tok/s) | TG (tok/s) | Power (W) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Granite4.0 350M** | F16 | CPU | 🟢 20.73±0.74 | 🔴 154.1±0.1 | 🟢 25.8±0.1 | 🔴 6.4±0.4 |
| | | NPU | 🟢 20.74±0.74 | 🟢 432.3±0.6 | 🟡 20.2±0.4 | 🟢 3.2±0.2 |
| | Q8_0 | CPU | 🟢 20.71±0.74 | 🔴 163.4±0.2 | 🟢 40.6±0.1 | 🔴 6.4±0.4 |
| | | NPU | 🟡 22.68±0.82 | 🟢 311.8±2.1 | 🔴 25.4±0.4 | 🟢 3.6±0.2 |
| | Q4_0 | CPU | 🟢 24.46±0.88 | 🟢 340.4±0.9 | 🟢 55.2±0.1 | 🔴 6.2±0.4 |
| | | NPU | 🔴 74.09±2.87 | 🔴 163.6±0.2 | 🔴 26.7±0.5 | 🟢 4.0±0.2 |

---

## Kontribusi

Anda dapat membuka issue untuk mendiskusikan bug atau permintaan fitur, atau mengirim pull request untuk kontribusi Anda.
