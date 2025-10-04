## **Topik 3: Pipeline CI/CD GitLab**

Setelah kita memahami cara mengelola kode dengan Git dan GitLab di Topik 2, sekarang saatnya mengotomatiskan prosesnya. Topik ini adalah jantung dari DevOps di GitLab, di mana kita mengubah alur kerja manual menjadi *pipeline* otomatis yang cepat dan andal. Kita akan belajar bagaimana "memerintahkan" GitLab untuk membangun, menguji, dan bahkan men-deploy aplikasi kita setiap kali ada perubahan kode.

### **1. Konsep CI/CD GitLab ⚙️**

**Dasar Teori**

GitLab CI/CD adalah fitur bawaan yang terintegrasi penuh di dalam GitLab. Ia bekerja dengan cara mencari file khusus bernama `.gitlab-ci.yml` di *root* repositori Anda. Ketika file ini ada, GitLab akan menggunakan **GitLab Runner** untuk menjalankan serangkaian skrip yang Anda definisikan di dalamnya setiap kali ada event Git (seperti `git push` atau pembuatan *Merge Request*).

Untuk memahami alurnya, mari gunakan **analogi lini perakitan di pabrik**:

  * **Pipeline**: Ini adalah keseluruhan **lini perakitan** dari awal hingga akhir. Sebuah *pipeline* dipicu untuk setiap perubahan kode dan berisi semua proses yang harus dilalui.
  * **Stages (Tahapan)**: Ini adalah **stasiun kerja utama** di lini perakitan. Contoh: `build` (perakitan), `test` (kontrol kualitas), `deploy` (pengiriman). *Stages* berjalan secara **berurutan**. Tahap `test` hanya akan dimulai jika semua pekerjaan di tahap `build` berhasil.
  * **Jobs (Pekerjaan)**: Ini adalah **tugas spesifik** yang dilakukan di setiap stasiun kerja. Contoh: di dalam *stage* `test`, mungkin ada *job* `unit-tests`, `integration-tests`, dan `security-scan`. Semua *job* dalam satu *stage* yang sama akan berjalan secara **paralel** (bersamaan) untuk efisiensi.

**How It Works (Alur Kerja)**

1.  Developer melakukan `git push` ke GitLab.
2.  GitLab mendeteksi adanya file `.gitlab-ci.yml` di dalam proyek.
3.  GitLab membaca file tersebut dan membuat sebuah *pipeline* baru berdasarkan *stages* dan *jobs* yang didefinisikan.
4.  GitLab mencari **Runner** yang tersedia dan cocok (berdasarkan *tags*).
5.  Runner yang ditugaskan akan meng-kloning repositori Anda.
6.  Runner menjalankan skrip di setiap *job* satu per satu sesuai urutan *stage*.
7.  Setelah selesai, Runner mengirimkan hasilnya (log dan status berhasil/gagal) kembali ke GitLab.
8.  Developer bisa melihat status pipeline langsung di antarmuka GitLab.

-----

### **2. Menulis File `.gitlab-ci.yml` 📝**

**Dasar Teori**

`.gitlab-ci.yml` adalah file konfigurasi yang ditulis dalam format **YAML** (*YAML Ain't Markup Language*). File ini adalah cetak biru (*blueprint*) dari *pipeline* Anda. Strukturnya yang sederhana dan mudah dibaca memungkinkan Anda mendefinisikan alur kerja yang kompleks dengan beberapa baris kode.

**How To: Struktur dan Keyword Utama**

Berikut adalah keyword paling penting yang perlu Anda ketahui:

  * **`stages`**: (Opsional tapi sangat direkomendasikan) Mendefinisikan urutan semua *stage* yang ada di *pipeline* Anda. Urutan ini sangat penting.

    ```yaml
    stages:
      - build
      - test
      - deploy
    ```

  * **`image`**: Menentukan **Docker image** yang akan digunakan oleh Runner untuk menjalankan *job*. Ini menciptakan lingkungan yang bersih dan konsisten untuk setiap *job*.

    ```yaml
    default: # Bisa juga didefinisikan secara global
      image: node:18-alpine
    ```

  * **`job_name`**: Anda bebas menamai *job* Anda (misal: `build_app`, `run_unit_tests`). Setiap nama *job* harus unik.

      * **`stage`**: Keyword di dalam *job* untuk menempatkan *job* tersebut ke dalam *stage* yang telah didefinisikan.
      * **`script`**: Keyword terpenting. Ini adalah array berisi perintah *shell* (Linux) yang akan dieksekusi oleh Runner.

    <!-- end list -->

    ```yaml
    build_job:
      stage: build
      script:
        - echo "Compiling the code..."
        - npm install
        - echo "Compilation complete."
    ```

  * **`artifacts`**: Digunakan untuk menyimpan file atau direktori yang dihasilkan oleh sebuah *job*. Artefak ini dapat diunduh atau digunakan oleh *job* di *stage* berikutnya. Sangat berguna untuk meneruskan hasil *build* ke tahap *test*.

    ```yaml
    build_job:
      stage: build
      script:
        - npm install
      artifacts:
        paths:
          - node_modules/ # Simpan folder node_modules
        expire_in: 1 hour # Hapus artefak setelah 1 jam
    ```

  * **`rules`**: Aturan yang sangat fleksibel untuk menentukan **kapan** sebuah *job* harus dijalankan. Ini lebih modern dan direkomendasikan daripada `only`/`except`.

    ```yaml
    deploy_to_prod:
      stage: deploy
      script:
        - echo "Deploying to production server..."
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"' # Hanya jalankan job ini jika ada push ke branch 'main'
    ```

-----

### **3. GitLab Runners: Sang Eksekutor Pipeline 🏃‍♂️**

**Dasar Teori**

GitLab Runner adalah aplikasi (ditulis dalam Go) yang Anda pasang di luar GitLab. Fungsinya adalah sebagai **pekerja** yang mengambil dan menjalankan *job* dari GitLab. Tanpa Runner, *pipeline* Anda hanya akan berstatus *pending*.

**How To: Jenis-Jenis Runner dan Setup**

1.  **Shared Runners**: Disediakan oleh admin GitLab (misalnya di GitLab.com) dan dapat digunakan oleh **semua proyek**.

      * **Kelebihan**: Langsung siap pakai, tidak perlu setup.
      * **Kekurangan**: Mungkin lambat karena dipakai bersama, memiliki batasan, dan tidak bisa dipasangi *software* khusus.

2.  **Specific Runners**: Didaftarkan untuk **proyek tertentu**. Anda memiliki kontrol penuh atas Runner ini.

      * **Kelebihan**: Keamanan lebih tinggi, bisa diinstal *software* khusus (misal: lisensi *software*, SDK Android), performa lebih terjamin.
      * **Kekurangan**: Perlu setup dan pemeliharaan sendiri.

3.  **Group Runners**: Didaftarkan untuk **semua proyek dalam satu grup**. Kombinasi antara *Shared* dan *Specific*.

**Langkah Praktik: Setup Specific Runner (Self-Hosted)**

Untuk kontrol penuh, mari kita siapkan Runner sendiri. Anda bisa menemukannya di **Settings \> CI/CD \> Runners** pada proyek GitLab Anda.

**A. Setup di Virtual Machine (Executor: Docker)**
Metode ini umum digunakan untuk setup yang sederhana dan terkontrol.

1.  **Instal GitLab Runner**: Ikuti panduan instalasi resmi untuk OS Linux Anda di [dokumentasi GitLab](https://docs.gitlab.com/runner/install/).
2.  **Instal Docker**: Runner akan menggunakan Docker untuk menjalankan *job* dalam kontainer yang terisolasi. Pastikan Docker sudah terinstal di VM tersebut. `sudo apt-get install docker.io`
3.  **Daftarkan Runner**: Jalankan perintah berikut di terminal VM Anda.
    ```bash
    sudo gitlab-runner register
    ```
4.  **Isi Prompt Interaktif**:
      * **Enter the GitLab instance URL**: Masukkan URL GitLab Anda (misal: `https://gitlab.com/`).
      * **Enter the registration token**: Salin token dari halaman **Settings \> CI/CD \> Runners**.
      * **Enter a description for the runner**: Beri nama deskriptif (misal: `vm-runner-dev`).
      * **Enter tags for the runner**: Beri *tag* untuk memilih Runner ini di file `.yml` (misal: `docker,linux,nodejs`).
      * **Enter optional maintenance note**: Kosongkan saja.
      * **Enter an executor**: Ketik **`docker`**.
      * **Enter the default Docker image**: Ketik gambar default (misal: `alpine:latest`).
5.  Runner Anda sekarang aktif dan siap menerima *job*\!

**B. Setup di Kubernetes (Executor: Kubernetes)**
Ini adalah metode canggih yang memungkinkan *pipeline* Anda berjalan secara dinamis dan dapat diskalakan (*scalable*).

1.  **Prasyarat**: Anda sudah memiliki klaster Kubernetes dan `kubectl` serta `helm` terkonfigurasi.
2.  **Gunakan Helm Chart**: Cara termudah adalah menggunakan GitLab Runner Helm chart resmi.
3.  **Buat file `values.yaml`**: Ini adalah file konfigurasi untuk Helm.
    ```yaml
    # values.yaml
    gitlabUrl: https://gitlab.com/
    runnerRegistrationToken: "TOKEN_ANDA_DARI_HALAMAN_RUNNER"
    rbac:
      create: true
    runners:
      tags: "kubernetes,k8s,dev"
    ```
4.  **Instal Chart**:
    ```bash
    helm repo add gitlab https://charts.gitlab.io
    helm repo update
    helm install --namespace gitlab gitlab-runner -f values.yaml gitlab/gitlab-runner
    ```
5.  Helm akan secara otomatis membuat semua sumber daya Kubernetes yang diperlukan. Runner akan berjalan sebagai *Pod* dan akan membuat *Pod* baru untuk setiap *job* CI/CD, memberikan skalabilitas yang luar biasa.

-----

### **4. Praktik: Membangun Pipeline Sederhana untuk Aplikasi Node.js 🛠️**

**Studi Kasus:** Kita akan membuat *pipeline* untuk aplikasi "Hello World" menggunakan Express.js. Pipeline akan memiliki 3 *stage*:

1.  `build`: Menginstal dependensi.
2.  `test`: Menjalankan *unit test*.
3.  `notify`: Memberi notifikasi sederhana.

**Langkah 1: Persiapan Aplikasi Node.js**
Buat proyek baru di komputer Anda dengan struktur file berikut:

  * **`package.json`**:

    ```json
    {
      "name": "gitlab-ci-demo",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "start": "node index.js",
        "test": "jest"
      },
      "dependencies": {
        "express": "^4.18.2"
      },
      "devDependencies": {
        "jest": "^29.7.0",
        "supertest": "^6.3.3"
      }
    }
    ```

  * **`index.js`**:

    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;

    app.get('/', (req, res) => {
      res.send('Hello, GitLab CI/CD!');
    });

    module.exports = app; // Ekspor untuk testing
    ```

  * **`index.test.js`**:

    ```javascript
    const request = require('supertest');
    const app = require('./index');

    describe('GET /', () => {
      it('should respond with Hello, GitLab CI/CD!', async () => {
        const response = await request(app).get('/');
        expect(response.statusCode).toBe(200);
        expect(response.text).toBe('Hello, GitLab CI/CD!');
      });
    });
    ```

    Jangan lupa jalankan `npm install` di lokal untuk membuat `package-lock.json`.

**Langkah 2: Membuat File `.gitlab-ci.yml`**
Buat file ini di *root* direktori proyek Anda:

```yaml
# Gunakan image Node.js versi 18 sebagai default untuk semua job
default:
  image: node:18-alpine

# Definisikan urutan stage
stages:
  - build
  - test
  - notify

# Job untuk menginstal dependensi
install_dependencies:
  stage: build
  script:
    - echo "Installing dependencies..."
    - npm install
  artifacts:
    paths:
      - node_modules/ # Simpan node_modules untuk job berikutnya
  tags:
    - docker # Pastikan job ini dijalankan oleh runner dengan tag 'docker'

# Job untuk menjalankan unit test
run_unit_tests:
  stage: test
  script:
    - echo "Running unit tests..."
    - npm test
  dependencies:
    - install_dependencies # Job ini butuh artefak dari install_dependencies
  tags:
    - docker

# Job untuk notifikasi jika pipeline sukses
success_notification:
  stage: notify
  script:
    - echo "Pipeline finished successfully!"
  when: on_success # Hanya jalankan jika semua job sebelumnya berhasil
  tags:
    - docker

# Job untuk notifikasi jika pipeline gagal
failure_notification:
  stage: notify
  script:
    - echo "Pipeline failed. Please check the logs."
  when: on_failure # Hanya jalankan jika ada job yang gagal
  tags:
    - docker
```

**Langkah 3: Push ke GitLab dan Lihat Hasilnya**

1.  Buat repositori baru di GitLab.
2.  Hubungkan direktori lokal Anda dengan repositori GitLab.
3.  Lakukan *commit* dan *push* semua file.
    ```bash
    git init
    git remote add origin https://www.myrepository.dev/user/repo.git
    git add .
    git commit -m "feat: Initial commit with nodejs app and gitlab-ci"
    git push -u origin main
    ```
4.  Buka proyek Anda di GitLab dan navigasi ke menu **Build \> Pipelines**. Anda akan melihat *pipeline* Anda berjalan secara otomatis\! Klik untuk melihat detail setiap *stage* dan *job*.

**Eksperimen:** Coba ubah `index.test.js` agar tesnya gagal (misalnya, `expect(response.text).toBe('Teks yang salah');`). Lakukan *push* lagi dan lihat bagaimana *job* `run_unit_tests` gagal, dan hanya *job* `failure_notification` yang berjalan di *stage* `notify`.

-----

### **Referensi**

  * [Dokumentasi Resmi GitLab CI/CD](https://docs.gitlab.com/ee/ci/)
  * [Referensi Keyword `.gitlab-ci.yml`](https://www.google.com/search?q=%5Bhttps://docs.gitlab.com/ee/ci/yaml/index.html%5D\(https://docs.gitlab.com/ee/ci/yaml/index.html\))
  * [Dokumentasi GitLab Runner](https://docs.gitlab.com/runner/)
  * [Helm Chart untuk GitLab Runner](https://docs.gitlab.com/runner/install/kubernetes.html)