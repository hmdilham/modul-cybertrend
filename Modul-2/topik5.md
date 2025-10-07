## **Topik 5: Pipeline Jenkins dengan Jenkinsfile**

Setelah Jenkins berhasil diinstal dan berjalan di Topik 4, pertanyaan berikutnya adalah: "Bagaimana cara mendefinisikan pekerjaan yang harus dilakukan Jenkins?" Dahulu, ini dilakukan dengan membuat "Freestyle project" dan mengklik banyak tombol di antarmuka web. Namun, cara ini tidak skalabel, tidak transparan, dan sulit dikelola.

Selamat datang di era **Pipeline as Code (PaC)**, sebuah praktik fundamental dalam DevOps di mana definisi *pipeline* CI/CD ditulis sebagai kode dan disimpan bersama kode aplikasi di dalam repositori Git. Di Jenkins, ini diwujudkan melalui sebuah file bernama **Jenkinsfile**.

**Mengapa Pipeline as Code?**

  * **Tersimpan di Kontrol Versi:** Seluruh riwayat perubahan pada *pipeline* Anda terlacak. Anda bisa tahu siapa mengubah apa dan kapan.
  * **Dapat Direview:** Perubahan pada alur CI/CD dapat direview dan disetujui melalui *Merge Request*, sama seperti kode aplikasi.
  * **Dapat Digunakan Ulang (*Reusable*):** Mudah untuk membuat templat dan menggunakan ulang logika *pipeline* di berbagai proyek.
  * **Tahan Banting (*Resilient*):** Jika server Jenkins Anda rusak, seluruh definisi *pipeline* Anda aman tersimpan di Git. Cukup arahkan server Jenkins yang baru ke repositori Anda, dan semuanya akan kembali berjalan.

-----

### **1. Pipeline Deklaratif vs. Skrip: Dua Rasa Jenkinsfile 🎨**

**Dasar Teori**

Jenkinsfile dapat ditulis dalam dua jenis sintaks: **Deklaratif** dan **Skrip**. Memilih yang tepat adalah langkah pertama yang penting.

**A. Pipeline Deklaratif (*Declarative Pipeline*)**
Ini adalah cara yang lebih modern dan direkomendasikan untuk menulis Jenkinsfile. Strukturnya lebih kaku dan terdefinisi dengan baik, membuatnya lebih mudah dibaca dan ditulis.

  * **Analogi:** Mengisi sebuah formulir yang sudah jelas bagian-bagiannya (nama, alamat, dll). Anda hanya perlu mengisi nilainya.
  * **Kelebihan:**
      * Sangat mudah dipelajari, bahkan untuk pemula.
      * Sintaks yang bersih dan mudah dibaca.
      * Validasi eror yang lebih baik sebelum *pipeline* dijalankan.
      * Cocok untuk 90% kasus penggunaan CI/CD.

**B. Pipeline Skrip (*Scripted Pipeline*)**
Ini adalah cara orisinal untuk menulis Jenkinsfile dan didasarkan pada bahasa pemrograman **Groovy**. Ia menawarkan fleksibilitas dan kekuatan yang hampir tak terbatas.

  * **Analogi:** Menulis sebuah program atau skrip dari kanvas kosong. Anda bisa melakukan apa pun yang diizinkan oleh bahasa tersebut.
  * **Kelebihan:**
      * Sangat fleksibel untuk alur kerja yang sangat kompleks, dinamis, atau non-standar.
      * Memungkinkan penggunaan logika pemrograman penuh (perulangan, kondisional, *try-catch*).
  * **Kekurangan:**
      * Membutuhkan pemahaman tentang sintaks Groovy.
      * Lebih sulit dibaca dan lebih rentan terhadap kesalahan sintaks.

**Rekomendasi:** **Selalu mulai dengan Pipeline Deklaratif.** Gunakan Pipeline Skrip hanya jika Anda menghadapi kasus yang sangat kompleks yang tidak dapat ditangani oleh sintaks Deklaratif.

**How To: Perbandingan Sintaks Sederhana**

Mari kita lihat "Hello World" dalam kedua sintaks:

  * **Deklaratif:**

    ```groovy
    pipeline {
        agent any
        stages {
            stage('Contoh') {
                steps {
                    echo 'Hello from Declarative Pipeline'
                }
            }
        }
    }
    ```

  * **Skrip:**

    ```groovy
    node {
        stage('Contoh') {
            echo 'Hello from Scripted Pipeline'
        }
    }
    ```

Perhatikan bagaimana versi Deklaratif lebih "cerewet" namun juga lebih terstruktur dan jelas tujuannya.

-----

### **2. Struktur Jenkinsfile (Fokus pada Deklaratif) 🧱**

**Dasar Teori**

Pipeline Deklaratif memiliki struktur hierarkis yang jelas. Mari kita bedah anatominya.

**How To: Keyword dan Blok Penting**

Berikut adalah blok-blok penyusun utama dari Jenkinsfile Deklaratif:

  * **`pipeline { ... }`**: Blok terluar yang wajib ada. Semua definisi pipeline harus berada di dalamnya.

  * **`agent { ... }`**: Menentukan **di mana** *pipeline* atau *stage* tertentu akan dieksekusi.

      * `agent any`: Jalankan di *agent* mana pun yang tersedia.
      * `agent { label 'linux-builder' }`: Jalankan hanya di *agent* yang memiliki label `linux-builder`.
      * `agent { docker { image 'node:18-alpine' } }`: **Sangat powerful\!** Jenkins akan secara dinamis menjalankan *stage* ini di dalam sebuah kontainer Docker yang menggunakan *image* `node:18-alpine`. Ini memastikan lingkungan *build* yang bersih dan konsisten setiap saat.

  * **`stages { ... }`**: Blok yang berisi satu atau lebih tahapan *pipeline*.

  * **`stage('Nama Stage') { ... }`**: Mendefinisikan satu tahapan kerja, misalnya 'Build', 'Test', atau 'Deploy'.

  * **`steps { ... }`**: Di dalam sebuah `stage`, blok ini berisi langkah-langkah atau perintah yang akan dieksekusi secara berurutan.

      * `sh 'npm install'`: Menjalankan perintah *shell*.
      * `echo 'Build complete.'`: Mencetak pesan ke log.
      * `junit '**/target/surefire-reports/*.xml'`: Salah satu contoh langkah dari *plugin* (untuk memproses hasil tes JUnit).

  * **`post { ... }`**: (Opsional) Mendefinisikan aksi yang akan dijalankan **setelah** seluruh *pipeline* selesai. Sangat berguna untuk notifikasi dan pembersihan.

      * `always { ... }`: Selalu dijalankan, baik *build* berhasil maupun gagal.
      * `success { ... }`: Hanya dijalankan jika *build* berhasil.
      * `failure { ... }`: Hanya dijalankan jika *build* gagal.

**Praktik Berdasarkan Contoh Kasus: Jenkinsfile Dasar**

```groovy
pipeline {
    agent any // Jalankan di agent mana saja

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'javac HelloWorld.java' // Contoh perintah build Java
            }
        }
        stage('Test') {
            steps {
                echo 'Testing the application...'
                sh 'java HelloWorld' // Contoh perintah menjalankan aplikasi
            }
        }
    }
    post {
        success {
            echo 'Pipeline finished successfully. Sending notification...'
            // Di sini Anda bisa menambahkan langkah untuk mengirim email atau notifikasi Slack
        }
        failure {
            echo 'Pipeline failed! Please check the logs.'
        }
    }
}
```

-----

### **3. Konsep Lanjutan: Build Multi-Tahap dan Shared Libraries 📚**

**Dasar Teori**

Seiring dengan kompleksitas proyek, Jenkinsfile Anda juga akan menjadi lebih kompleks. Dua konsep berikut membantu Anda mengelolanya dengan efisien.

**A. Build Multi-Tahap dengan Agent Berbeda**
Seringkali, bagian *backend* dan *frontend* dari sebuah aplikasi membutuhkan *tools* dan lingkungan yang berbeda. Jenkinsfile memungkinkan Anda mendefinisikan *agent* yang berbeda untuk setiap *stage*.

**Praktik Berdasarkan Contoh Kasus: Build Backend (Java) & Frontend (Node.js)**

```groovy
pipeline {
    agent none // Tidak ada agent global, didefinisikan per stage

    stages {
        stage('Build Backend') {
            agent { docker { image 'maven:3.8-eclipse-temurin-17' } } // Agent khusus Java/Maven
            steps {
                sh 'mvn clean package'
                // Stash artefak agar bisa digunakan di stage lain
                stash name: 'backend-jar', includes: 'target/*.jar'
            }
        }
        stage('Build Frontend') {
            agent { docker { image 'node:18-alpine' } } // Agent khusus Node.js
            steps {
                sh 'npm install'
                sh 'npm run build'
                stash name: 'frontend-dist', includes: 'dist/**/*'
            }
        }
        stage('Package') {
            agent any // Gunakan agent umum untuk menggabungkan
            steps {
                // Ambil kembali artefak dari stage sebelumnya
                unstash 'backend-jar'
                unstash 'frontend-dist'
                echo 'Packaging backend and frontend into a Docker image...'
                sh 'docker build -t my-app:latest .'
            }
        }
    }
}
```

Perhatikan penggunaan `stash` dan `unstash` untuk "mengoper" file antar *stage* yang berjalan di lingkungan (kontainer) yang berbeda.

**B. Shared Libraries: Prinsip DRY (*Don't Repeat Yourself*)**
Jika Anda memiliki banyak proyek, Anda akan menemukan pola yang sama di banyak Jenkinsfile (misalnya, logika untuk mengirim notifikasi Slack). Menyalin-tempel kode ini adalah praktik yang buruk.

**Shared Libraries** adalah solusinya. Ini adalah sebuah repositori Git terpisah yang berisi skrip Groovy yang dapat digunakan kembali, yang bisa Anda panggil dari Jenkinsfile mana pun.

**How To: Membuat dan Menggunakan Shared Library Sederhana**

1.  **Buat Repositori Git Baru** untuk *shared library* Anda.
2.  **Buat Struktur Direktori:** Di dalam repositori tersebut, buat struktur `vars/`.
3.  **Buat File Skrip:** Di dalam `vars/`, buat file bernama `sendSlackNotification.groovy`.
    ```groovy
    // vars/sendSlackNotification.groovy
    def call(String status, String channel) {
        echo "Sending Slack notification to channel ${channel} with status: ${status}"
        // Di sini Anda akan menggunakan plugin Slack untuk mengirim pesan sungguhan
        // slackSend(channel: channel, message: "Build status: ${status}")
    }
    ```
4.  **Konfigurasi di Jenkins:**
      * Pergi ke **Manage Jenkins \> System**.
      * Cari bagian **Global Pipeline Libraries**.
      * Klik **Add**. Beri nama (misal: `my-shared-library`), dan masukkan URL repositori Git dari *shared library* Anda.
5.  **Panggil dari Jenkinsfile:**
    ```groovy
    // Impor library di bagian atas Jenkinsfile
    @Library('my-shared-library') _

    pipeline {
        agent any
        stages {
            stage('Hello') {
                steps {
                    echo 'Hello World'
                }
            }
        }
        post {
            success {
                // Panggil fungsi dari shared library
                sendSlackNotification(status: 'SUCCESS', channel: '#ci-alerts')
            }
        }
    }
    ```

Dengan cara ini, logika notifikasi Anda terpusat di satu tempat. Jika Anda perlu mengubahnya, Anda hanya perlu mengeditnya di repositori *shared library*, dan semua proyek akan otomatis mendapatkan pembaruan.

-----

### **Studi Kasus: Shared Library untuk Pesan Sambutan Dinamis**

**Tujuan:** Kita akan membuat sebuah fungsi `sapaPengguna` di dalam *shared library*. Fungsi ini akan menerima parameter `nama` dan `tanggal`, lalu mencetak pesan sambutan yang diformat di log konsol Jenkins.

### **Langkah 1: Membuat Repositori untuk Shared Library 📂**

Langkah pertama adalah membuat "sumber" dari *library* kita, yaitu sebuah repositori Git.

1.  **Buat Repositori Git Baru:** Di GitLab, GitHub, atau layanan Git lainnya, buat sebuah repositori baru. Mari kita beri nama `jenkins-custom-libs`.

2.  **Buat Struktur Direktori:** Kloning repositori tersebut ke komputer lokal Anda. Di dalamnya, buat struktur direktori berikut. Aturan Jenkins mengharuskan skrip fungsi global berada di dalam folder `vars`.

    ```
    jenkins-custom-libs/
    └── vars/
        └── sapaPengguna.groovy
    ```

3.  **Tulis Kode Fungsi (Groovy):** Buat file `sapaPengguna.groovy` di dalam folder `vars/`. Nama file ini (`sapaPengguna`) akan menjadi nama fungsi yang bisa kita panggil nanti. Isi file tersebut dengan kode berikut:

    ```groovy
    // File: vars/sapaPengguna.groovy

    /**
     * Mencetak pesan sambutan yang diformat ke konsol.
     * @param config Sebuah Map yang berisi:
     * - nama: Nama pengguna yang akan disapa.
     * - tanggal: Tanggal yang akan ditampilkan dalam pesan.
     */
    def call(Map config) {
        // Validasi input sederhana
        if (!config.nama || !config.tanggal) {
            error("Parameter 'nama' dan 'tanggal' wajib diisi!")
        }

        // Cetak pesan yang sudah diformat menggunakan echo step
        echo "Halo ${config.nama}, sekarang tanggal ${config.tanggal}"
    }
    ```

      * **`def call(Map config)`:** Ini adalah metode utama yang akan dieksekusi Jenkins. Menggunakan `Map` memungkinkan kita memanggil fungsi dengan parameter bernama (misal: `nama: 'Ilham'`), yang membuat *pipeline* lebih mudah dibaca.
      * **`echo "..."`:** Ini adalah *step* standar Jenkins untuk mencetak teks ke *console log*. Tanda `${...}` adalah fitur interpolasi string dari Groovy.

4.  **Commit dan Push:** Simpan perubahan Anda dan lakukan *push* ke repositori Git.

    ```bash
    git add .
    git commit -m "feat: Menambahkan fungsi sapaPengguna"
    git push origin main
    ```

-----

### **Langkah 2: Konfigurasi Shared Library di Jenkins ⚙️**

Sekarang, kita perlu memberitahu Jenkins di mana menemukan *library* yang baru saja kita buat.

1.  Buka *dashboard* Jenkins Anda.

2.  Navigasi ke **Manage Jenkins \> System**.

3.  Gulir ke bawah hingga Anda menemukan bagian **Global Pipeline Libraries**.

4.  Klik **Add**.

5.  Isi formulir sebagai berikut:

      * **Name:** `jenkins-shared-library` (Ini adalah nama yang akan Anda gunakan untuk mengimpor *library* di Jenkinsfile).
      * **Default version:** `main` (atau nama *branch default* repositori Anda).
      * **Retrieval method:** Pilih **Modern SCM**.
      * **Source Code Management:** Pilih **Git**.
      * **Project Repository URL:** Masukkan URL HTTPS atau SSH dari repositori `jenkins-custom-libs` yang Anda buat di Langkah 1.
      * **Credentials:** Jika repositori Anda *private*, pilih kredensial yang sesuai. Jika *public*, pilih `- none -`.

6.  Klik **Save**.

-----

### **Langkah 3: Memanggil Shared Library dari Jenkinsfile 📝**

Kini saatnya menggunakan fungsi yang sudah kita buat di dalam sebuah *pipeline*.

1.  Buat *job* baru di Jenkins dengan tipe **Pipeline**.

2.  Di bagian konfigurasi *pipeline*, pilih **Pipeline script** dan masukkan Jenkinsfile berikut:

    ```groovy
    // Impor shared library yang sudah dikonfigurasi di Jenkins
    // Nama 'jenkins-shared-library' harus cocok dengan yang ada di Manage Jenkins
    @Library('jenkins-shared-library') _

    pipeline {
        agent any

        stages {
            stage('Sapa Pengguna') {
                steps {
                    script {
                        // Mendapatkan tanggal hari ini secara dinamis
                        // Mengatur zona waktu agar sesuai dengan WIB (Waktu Indonesia Barat)
                        def tanggalHariIni = new Date().format('dd MMMM yyyy', TimeZone.getTimeZone('Asia/Jakarta'))

                        // Memanggil fungsi sapaPengguna dari shared library
                        // dengan parameter yang dinamis
                        sapaPengguna(nama: 'Ilham', tanggal: tanggalHariIni)

                        // Contoh pemanggilan dengan nama lain
                        sapaPengguna(nama: 'Budi', tanggal: tanggalHariIni)
                    }
                }
            }
        }
        post {
            always {
                echo 'Pipeline selesai.'
            }
        }
    }
    ```

      * **`@Library('jenkins-shared-library') _`**: Baris ini mengimpor semua fungsi dari *library* yang kita konfigurasikan. Tanda `_` di akhir sangat penting.
      * **`def tanggalHariIni = ...`**: Di sini kita tidak lagi menulis tanggal secara manual. Kita menggunakan sedikit kode Groovy untuk mendapatkan tanggal saat ini secara dinamis dan memformatnya dalam Bahasa Indonesia.
      * **`sapaPengguna(...)`**: Ini adalah pemanggilan fungsi kita. Perhatikan bagaimana kita memberikan parameter `nama` dan `tanggal` dalam format `key: value`.

3.  Klik **Save**, lalu klik **Build Now**.

-----

### **Langkah 4: Melihat Hasil Eksekusi ✅**

Setelah *build* selesai (seharusnya sangat cepat), klik *build number* tersebut dan pilih **Console Output**. Anda akan melihat hasil seperti berikut:

```
[Pipeline] { (Sapa Pengguna)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
Halo Ilham, sekarang tanggal 07 Oktober 2025
[Pipeline] echo
Halo Budi, sekarang tanggal 07 Oktober 2025
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Pipeline selesai.
[Pipeline] }
```

Anda telah berhasil membuat, mengonfigurasi, dan menggunakan Jenkins Shared Library untuk menjalankan kode yang dapat digunakan kembali\!

### **Referensi**

  * [Dokumentasi Resmi Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
  * [Referensi Sintaks Pipeline Deklaratif](https://www.jenkins.io/doc/book/pipeline/syntax/)
  * [Dokumentasi Resmi Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)