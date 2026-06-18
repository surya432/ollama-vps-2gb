# ollama-vps-2gb

Panduan ini menjelaskan cara memasang dan menjalankan layanan **Ollama + LiteLLM** pada VPS atau mesin lokal menggunakan Docker Compose.

## Prasyarat

Pastikan sistem Anda memiliki:

- Docker
- Docker Compose
- Akses internet untuk mendownload image dan model

## Struktur Proyek

- `docker-compose.yml`: Definisi layanan Ollama dan LiteLLM
- `litellm_config.yaml`: Konfigurasi model dan API key LiteLLM

## Konfigurasi Awal

Sebelum menjalankan layanan, perbarui `litellm_config.yaml` dengan nilai yang sesuai:

- `master_key`: kunci utama admin untuk LiteLLM
- `api_key` pada `saved_keys`: API key yang akan dipakai oleh aplikasi Anda
- `allowed_models`: model yang diizinkan untuk kunci tersebut

Contoh:

```yaml
model_list:
  - model_name: qwen2.5-coder:1.5b
    litellm_params:
      model: ollama/qwen2.5-coder:1.5b
      api_base: "http://ollama:11434"

general_settings:
  master_key: sk-master-key-admin

litellm_settings:
  saved_keys:
    - api_key: "sk-rahasia-vps-kamu"
      allowed_models: ["qwen2.5-coder:1.5b"]
```

> Ganti `sk-master-key-admin` dan `sk-rahasia-vps-kamu` dengan nilai yang Anda inginkan.

## Menambah Token Baru

Untuk menambahkan API key baru, tambahkan item baru di bawah `saved_keys` pada `litellm_config.yaml`:

```yaml
litellm_settings:
  saved_keys:
    - api_key: "sk-rahasia-vps-kamu"
      allowed_models: ["qwen2.5-coder:1.5b"]
    - api_key: "sk-token-baru-anda"
      allowed_models: ["qwen2.5-coder:1.5b"]
```

Jika Anda ingin token baru hanya mengizinkan model tertentu, sesuaikan nilai `allowed_models` untuk setiap token.

Setelah menambahkan token, restart container LiteLLM dengan perintah:

```bash
docker compose restart litellm
```

## Menjalankan Layanan

Jalankan perintah berikut di direktori proyek:

```bash
docker compose up -d
```

Perintah ini akan:

1. Menjalankan container `ollama` dengan penyimpanan model di volume `ollama_storage`
2. Menjalankan container sementara `ollama-pull` untuk mengunduh model `qwen2.5-coder:1.5b`
3. Menjalankan container `litellm` pada port `4000`

## Memeriksa Status

Gunakan perintah berikut untuk melihat container yang berjalan:

```bash
docker compose ps
```

## Menggunakan API LiteLLM

Setelah layanan berjalan, LiteLLM akan tersedia di `http://localhost:4000`.

Contoh permintaan API menggunakan `curl` (ganti `sk-rahasia-vps-kamu` dengan API key Anda):

```bash
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-rahasia-vps-kamu" \
  -d '{
    "model": "qwen2.5-coder:1.5b",
    "messages": [
      {"role": "user", "content": "Halo, bantu saya membuat kode Python."}
    ]
  }'
```

Jika endpoint LiteLLM berbeda di versi Anda, sesuaikan URL dan format body sesuai dokumentasi LiteLLM yang digunakan.

## Hentikan Layanan

Untuk menghentikan layanan:

```bash
docker compose down
```

## Catatan Tambahan

- Folder `./litellm` di mount ke container `litellm` untuk konfigurasi.
- Model `qwen2.5-coder:1.5b` akan didownload otomatis oleh container `ollama-pull`.
- Jika Anda ingin menambahkan model lain, tambahkan entry baru di `litellm_config.yaml` dan update konfigurasi Ollama jika diperlukan.
