# Cara Menjalankan CI/CD GitHub Actions & GitHub Pages Menggunakan Branch Selain `main`

Secara default, tutorial dan konfigurasi standar GitHub Actions sering kali mengasumsikan branch yang digunakan adalah branch utama (`main`). 

Jika Anda mengerjakan tugas di branch lain (misalnya `belajar_1`, `belajar_10`, `feature/...`, dsb.), berikut adalah hal-hal yang **wajib disesuaikan**:

---

## 1. Menyesuaikan Trigger Workflow di File YAML (`.github/workflows/*.yaml`)

Secara default, bagian trigger sering kali hanya menyasar branch `main`:
```yaml
# SEBELUM (Hanya trigger jika push ke main):
on:
  push:
    branches: [main]
```

### Solusi:
Ubah bagian `on:` agar mengenali branch Anda atau seluruh branch:

**Opsi A (Mengenali seluruh branch - Paling Direkomendasikan saat praktikum):**
```yaml
on: [push]
```

**Opsi B (Mendaftarkan branch tertentu):**
```yaml
on:
  push:
    branches: [main, belajar_10]
```

---

## 2. Mengaktifkan GitHub Pages via GitHub Actions

Jika workflow menyertakan deployment ke GitHub Pages (seperti modul 10 / Selenium Behave report):
1. Buka repositori Anda di browser:  
   `https://github.com/[USERNAME]/[REPO_NAME]/settings/pages`
2. Di bagian **Build and deployment** ➔ **Source**:
   - Ubah dari *Deploy from a branch* menjadi **GitHub Actions**.

---

## 3. Mengatasi Error Proteksi Environment: *"Branch is not allowed to deploy to github-pages due to environment protection rules"*

### Penyebab:
Secara default, environment **`github-pages`** di GitHub hanya mengizinkan deployment dari branch `main` (atau default branch). Push dari branch lain (misal `belajar_10`) akan langsung ditolak dengan pesan:
> `Branch "belajar_10" is not allowed to deploy to github-pages due to environment protection rules.`

### Solusi (Menambahkan Aturan Deployment Branch):
1. Buka pengaturan environment di browser:  
   `https://github.com/[USERNAME]/[REPO_NAME]/settings/environments`
2. Klik nama environment: **`github-pages`**.
3. Scroll ke bawah ke bagian **Deployment branches and tags**:
   - Klik tombol **Add deployment branch or tag rule**.
   - Masukkan nama branch Anda, misalnya: `belajar_10`  
     *(Atau masukkan pola `*` jika ingin mengizinkan semua branch bisa deploy).*
   - Klik **Add rule**.
4. Kembali ke tab **Actions** di GitHub, buka workflow run yang gagal, lalu klik **Re-run all jobs**.

---

## 4. Penyesuaian Path untuk Proyek Monorepo (Banyak Folder Modul)

Jika repositori Anda berisi banyak subfolder modul (`01_...`, `09_devops_github_actions`, `10_github_actions_selenium`):
1. Pastikan folder `.github/workflows/` berada persis di **root repositori**, bukan di dalam subfolder modul.
2. Di file YAML, tambahkan `defaults.run.working-directory` dan arahkan path artifact agar mengeksekusi dan membaca file di folder modul yang sesuai:

```yaml
jobs:
  deploy-report:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./10_github_actions_selenium
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
      - name: Start Flask App
        run: |
          python app.py &
          for i in {1..10}; do
            curl -s http://127.0.0.1:5000 > /dev/null && echo "Server is up!" && break
            sleep 2
          done
      - name: Run Selenium Tests (Behave)
        run: behave -f html-pretty -o index.html
      - name: Upload Report
        if: always()
        uses: actions/upload-pages-artifact@v4
        with:
          path: ./10_github_actions_selenium
```
