## **Topik 6: Mengintegrasikan Jenkins dengan GitLab**

Sejauh ini, kita telah mempelajari GitLab sebagai platform SCM (Topik 2) dan Jenkins sebagai server otomatisasi (Topik 4 & 5). Keduanya adalah *tools* yang hebat secara individu. Namun, kekuatan sesungguhnya muncul saat kita membuat keduanya "berbicara" satu sama lain. Topik ini akan fokus pada bagaimana cara mengonfigurasi komunikasi dua arah antara GitLab dan Jenkins untuk menciptakan *pipeline* yang sepenuhnya otomatis dan terintegrasi.

Tujuan akhirnya adalah mencapai alur kerja berikut:
`Developer melakukan git push ke GitLab` → `GitLab secara otomatis memberi tahu Jenkins` → `Jenkins menjalankan build & test` → `Jenkins melaporkan kembali statusnya ke GitLab`.

### **1. GitLab Webhooks: Sang Pembawa Pesan 📨**

**Dasar Teori**

**Webhook** adalah sebuah mekanisme sederhana namun sangat kuat untuk komunikasi antar aplikasi web. Pada dasarnya, ini adalah sebuah **panggilan HTTP POST otomatis** yang dikirim dari satu aplikasi (sumber) ke aplikasi lain (tujuan) ketika sebuah *event* (kejadian) tertentu terjadi.

Dalam konteks kita:

  * **Aplikasi Sumber:** GitLab.
  * **Aplikasi Tujuan:** Jenkins.
  * **Event:** `Push event` (seorang developer melakukan `git push`), `Merge request event` (sebuah MR dibuat atau di-update), `Tag push event`, dan lain-lain.

Ketika *event* tersebut terjadi, GitLab akan mengemas informasi relevan tentang *event* tersebut ke dalam sebuah *payload* (biasanya dalam format JSON) dan mengirimkannya ke sebuah URL unik yang disediakan oleh Jenkins. Jenkins, yang "mendengarkan" di URL tersebut, akan menerima *payload* ini dan menggunakannya untuk memicu tindakan yang sesuai, seperti menjalankan sebuah *pipeline*.

> **Analogi:** Bayangkan Anda berlangganan notifikasi dari toko online. Anda memberikan nomor telepon Anda (URL Webhook Jenkins) ke toko tersebut (GitLab). Setiap kali ada produk baru yang Anda minati (Push Event), toko online tersebut akan secara otomatis mengirimkan SMS (HTTP POST) kepada Anda. Anda tidak perlu terus-menerus memeriksa situs web toko tersebut.

-----

### **2. Menghubungkan GitLab ke Jenkins: Token dan Kredensial 🔐**

**Dasar Teori**

Webhook adalah komunikasi **satu arah** (dari GitLab ke Jenkins). Namun, kita juga memerlukan komunikasi **arah sebaliknya** (dari Jenkins ke GitLab) untuk fungsionalitas lanjutan, seperti:

  * Jenkins perlu mengkloning kode dari repositori GitLab yang bersifat *private*.
  * Jenkins perlu melaporkan kembali status *build* (misalnya: *pending*, *running*, *success*, *failure*) yang akan muncul di antarmuka *Merge Request* GitLab.

Untuk melakukan ini, Jenkins memerlukan izin untuk mengakses API GitLab. Cara paling aman untuk memberikan izin ini adalah melalui **Personal Access Token (PAT)**. PAT adalah sebuah token acak yang berfungsi seperti kata sandi, tetapi dapat Anda batasi cakupan (*scope*) izinnya dan dapat dicabut (*revoke*) kapan saja tanpa memengaruhi kata sandi utama Anda.

**How To dan Langkah Praktik: Menyiapkan Koneksi Dua Arah**

Proses ini melibatkan dua langkah utama: membuat token di GitLab dan menyimpannya secara aman di Jenkins.

**A. Membuat Personal Access Token di GitLab**

1.  Buka **GitLab**. Klik avatar profil Anda di pojok kanan atas, lalu pilih **Edit profile**.
2.  Di menu navigasi kiri, pilih **Access Tokens**.
3.  Beri nama yang deskriptif pada token Anda di kolom **Token name**, misalnya `jenkins-integration-token`.
4.  Pilih cakupan (*scopes*) izin yang diperlukan. Untuk integrasi penuh, pilih scope **`api`**. Ini akan memberikan Jenkins akses penuh ke API, yang dibutuhkan oleh GitLab Plugin.
5.  Klik **Create personal access token**.
6.  **PENTING:** GitLab akan menampilkan token Anda **hanya sekali**. Segera **salin token ini** dan simpan di tempat yang aman untuk sementara. Jika Anda kehilangan token ini, Anda harus membuat yang baru.

**B. Mengonfigurasi Koneksi di Jenkins**

1.  **Instal GitLab Plugin:** Pastikan Anda sudah menginstal `GitLab Plugin` di Jenkins melalui **Manage Jenkins \> Plugins**.
2.  **Simpan Token sebagai Kredensial:**
      * Di Jenkins, pergi ke **Manage Jenkins \> Credentials**.
      * Di bawah *Stores scoped to Jenkins*, klik domain `(global)`.
      * Klik **Add Credentials** di sebelah kiri.
      * Pada *Kind*, pilih **GitLab API token**.
      * Tempel token yang Anda salin dari GitLab ke kolom **API Token**.
      * Beri **ID** dan **Description** yang mudah diingat, misalnya `gitlab-api-token`. ID ini akan kita gunakan di Jenkinsfile.
      * Klik **Create**.
3.  **Konfigurasi Koneksi GitLab di Sistem Jenkins:**
      * Pergi ke **Manage Jenkins \> System**.
      * Gulir ke bawah hingga menemukan bagian **GitLab**.
      * Klik **Add GitLab Server**.
      * **Name:** Beri nama koneksi, misalnya `GitLab-Connection`.
      * **Server URL:** Masukkan URL GitLab Anda (misalnya `https://gitlab.com`).
      * **Credentials:** Dari menu *dropdown*, pilih kredensial yang baru saja Anda buat (`gitlab-api-token`).
      * Klik **Test Connection**. Anda seharusnya melihat pesan "Success".
      * Klik **Save**.

-----

### **3. Trigger CI dan Job Chaining 🚀**

**Dasar Teori**

Setelah koneksi terjalin, kita bisa mendefinisikan pemicu (*trigger*) otomatis dan bahkan merangkai beberapa *job* menjadi satu alur kerja yang lebih besar (*job chaining*).

  * **Trigger CI:** Ini adalah aturan yang memberitahu Jenkins kapan harus menjalankan sebuah *pipeline*. Aturan ini bisa sangat spesifik.

      * **Contoh Trigger:**
          * Jalankan saat ada *push* ke *branch* mana pun.
          * Jalankan hanya saat ada *push* ke *branch* `main`.
          * Jalankan saat sebuah *Merge Request* dibuat atau di-update.
          * Jalankan saat sebuah *tag* baru dibuat (biasanya untuk rilis).
          * Filter berdasarkan nama *branch* (misalnya, jalankan hanya untuk *branch* yang dimulai dengan `feature/*`).

  * **Job Chaining:** Ini adalah praktik di mana satu *job* Jenkins secara otomatis memicu *job* lain. Ini sangat berguna untuk memisahkan tahapan yang berbeda dalam alur CI/CD.

      * **Contoh Kasus:**
        1.  **Job A (Build & Test):** Dipicu oleh `git push`. Tugasnya hanya membangun dan menjalankan *unit test*.
        2.  **Job B (Deploy to Staging):** Hanya berjalan **jika Job A berhasil**. Tugasnya men-*deploy* aplikasi ke server *staging*.
        3.  **Job C (Deploy to Production):** Dipicu secara manual setelah tim QA selesai melakukan verifikasi di *staging*.

-----

### **Praktik Detail: Otomatisasi Penuh dari Push ke Build**

**Pendahuluan: Skenario Praktik**

Tujuan kita adalah menciptakan sebuah alur kerja CI (*Continuous Integration*) yang sepenuhnya otomatis. Skenario yang akan kita bangun adalah sebagai berikut:

1.  Seorang developer membuat perubahan pada kode di komputernya.
2.  Developer tersebut melakukan `git push` ke sebuah *feature branch* di repositori GitLab.
3.  GitLab secara **otomatis** mendeteksi *push* ini dan mengirimkan notifikasi (via *webhook*) ke Jenkins.
4.  Jenkins secara **otomatis** menerima notifikasi, mencari *job* yang sesuai, mengambil kode terbaru dari *branch* tersebut, dan menjalankan *pipeline* (Build & Test).
5.  Selama proses berjalan, Jenkins **otomatis** melaporkan kembali statusnya (*pending*, *running*) ke GitLab, yang akan terlihat di halaman *commit*.
6.  Setelah selesai, Jenkins **otomatis** melaporkan status finalnya (*success* atau *failed*) ke GitLab.

Ini adalah fondasi dari semua alur kerja DevOps modern.

-----

### **Fase 1: Persiapan Awal**

Sebelum memulai, pastikan semua prasyarat ini sudah terpenuhi. Ini adalah hasil dari topik-topik sebelumnya.

  * ✅ **Server Jenkins Siap:** Jenkins sudah terinstal (misalnya via Docker) dan dapat diakses melalui URL-nya.

  * ✅ **GitLab Plugin Terinstal:** Plugin `GitLab` sudah terpasang di Jenkins.

  * ✅ **Proyek GitLab Siap:** Anda memiliki sebuah proyek di GitLab, bahkan jika hanya berisi file `README.md` dan `Jenkinsfile`.

  * ✅ **Koneksi Jenkins & GitLab Terkonfigurasi:** Ini adalah bagian terpenting. Pastikan Anda sudah:

    1.  Membuat **Personal Access Token** di GitLab.
    2.  Menyimpannya sebagai **kredensial** (`GitLab API token`) di Jenkins.
    3.  Mengonfigurasi koneksi di **Manage Jenkins \> System \> GitLab**.

  * ✅ **Jenkinsfile di Repositori:** Pastikan `Jenkinsfile` berikut sudah ada di *root* repositori GitLab Anda.

    ```groovy
    // File: Jenkinsfile
    pipeline {
        agent any

        // Menghubungkan pipeline ini dengan konfigurasi GitLab di Jenkins System
        options {
            gitLabConnection('GitLab-Connection') // Nama harus cocok dengan di Manage Jenkins > System
        }

        // Mendefinisikan pemicu otomatis dari GitLab
        triggers {
            gitlab(
                triggerOnPush: true,
                triggerOnMergeRequest: true,
                branchFilterType: 'All' // Memicu untuk semua branch
            )
        }

        stages {
            stage('Build') {
                steps {
                    echo "Membangun proyek dari branch: ${env.gitlabBranch}"
                    // Melaporkan status 'pending' ke GitLab untuk stage ini
                    updateGitLabCommitStatus(name: 'Build', state: 'pending')
                    // Simulasi proses build selama 10 detik
                    sh 'sleep 10'
                    echo "Build selesai."
                }
            }
            stage('Test') {
                steps {
                    echo "Menjalankan unit tests..."
                    updateGitLabCommitStatus(name: 'Test', state: 'pending')
                    // Simulasi proses tes selama 15 detik
                    sh 'sleep 15'
                    echo "Tes selesai."
                }
            }
        }
        post {
            success {
                echo "Pipeline berhasil!"
                // Melaporkan status akhir 'success' ke commit di GitLab
                updateGitLabCommitStatus(name: 'Pipeline', state: 'success')
            }
            failure {
                echo "Pipeline GAGAL!"
                // Melaporkan status akhir 'failed' ke commit di GitLab
                updateGitLabCommitStatus(name: 'Pipeline', state: 'failed')
            }
        }
    }
    ```

-----

### **Fase 2: Konfigurasi di Sisi Jenkins**

Di fase ini, kita akan membuat *job* di Jenkins dan mengarahkannya ke repositori GitLab kita.

**Langkah 2.1: Buat Job Pipeline Baru**

1.  Dari *dashboard* Jenkins, klik **New Item**.
2.  Masukkan nama *item* (misalnya: `proyek-utama-ci`).
3.  Pilih **Pipeline** sebagai tipe proyek.
4.  Klik **OK**.

**Langkah 2.2: Konfigurasi Pipeline SCM (Source Code Management)**
Ini adalah langkah paling krusial di sisi Jenkins.

1.  Anda akan diarahkan ke halaman konfigurasi *job*. Gulir ke bawah hingga menemukan bagian **Pipeline**.
2.  Di menu *dropdown* **Definition**, ubah dari `Pipeline script` menjadi **`Pipeline script from SCM`**. Ini memberitahu Jenkins untuk mengambil `Jenkinsfile` dari repositori Git, bukan dari kotak teks.
3.  Bagian **SCM** akan muncul. Pilih **Git**.
4.  **Repository URL:** Masuk ke halaman proyek GitLab Anda, klik tombol biru **Clone**, dan salin URL **Clone with SSH** (misalnya: `git@gitlab.com:username/proyek-anda.git`). Tempel URL ini di sini.
5.  **Credentials:** Pilih kredensial **SSH private key** Anda yang sudah ditambahkan sebelumnya. Ini memungkinkan Jenkins untuk mengkloning repositori *private*.
6.  **Branches to build:** Secara *default*, ini mungkin terisi `*/main`. Untuk memastikan *pipeline* berjalan untuk *branch* apa pun, ubah menjadi `**` (dua tanda bintang).
7.  **Script Path:** Biarkan *default*, yaitu `Jenkinsfile`.

**Langkah 2.3: Simpan Konfigurasi Job**
Gulir ke bawah dan klik tombol **Save**. *Job* Anda sekarang sudah siap, tetapi belum tahu cara untuk terpicu secara otomatis.

-----

### **Fase 3: Konfigurasi di Sisi GitLab**

Sekarang, kita akan memberitahu GitLab untuk mengirim notifikasi ke Jenkins.

**Langkah 3.1: Buka Pengaturan Webhooks**

1.  Buka proyek Anda di GitLab.
2.  Di menu navigasi kiri, pergi ke **Settings \> Webhooks**.

**Langkah 3.2: Tambahkan Webhook Baru**

1.  **URL:** Masukkan URL Jenkins Anda dengan format: `http://<URL_JENKINS_ANDA>/project/<NAMA_JOB_ANDA>`.
      * Contoh: `http://192.168.1.50:8080/project/proyek-utama-ci`
2.  **Secret Token:** (Sangat direkomendasikan untuk keamanan) Anda dapat mengisi token rahasia untuk memastikan hanya GitLab yang bisa memicu *job* Anda. Token ini harus disamakan di konfigurasi *trigger* di Jenkins. Untuk latihan ini, kita bisa mengosongkannya.
3.  **Trigger:** Di bawah bagian "Trigger", pastikan kotak centang **Push events** aktif. Anda juga bisa mengaktifkan **Merge request events** karena `Jenkinsfile` kita juga mendengarkan *event* tersebut.
4.  Klik tombol hijau **Add webhook**.

**Langkah 3.3: Uji Webhook**

1.  Setelah *webhook* ditambahkan, halaman akan memuat ulang. Gulir ke bawah ke daftar "Project Hooks".
2.  Anda akan melihat *webhook* yang baru saja Anda buat. Klik tombol **Test** di sebelah kanannya, lalu pilih **Push events**.
3.  Anda akan melihat pesan di bagian atas halaman. Jika berhasil, pesannya akan "Hook executed successfully: HTTP 200". Ini berarti GitLab berhasil mengirim "ping" ke Jenkins.

-----

### **Fase 4: Uji Coba End-to-End**

Inilah saatnya melihat semua konfigurasi kita bekerja bersama.

**Langkah 4.1: Buat Perubahan Lokal**

1.  Di komputer Anda, buka terminal dan masuk ke direktori proyek Anda.
2.  Buat *branch* baru untuk memastikan kita tidak bekerja langsung di `main`.
    ```bash
    git checkout -b feature/test-otomatisasi
    ```
3.  Buka file `README.md` dan tambahkan satu baris teks, misalnya: "Menambahkan tes untuk integrasi Jenkins."

**Langkah 4.2: Lakukan Push ke GitLab**

1.  Simpan perubahan, lalu jalankan perintah Git berikut:
    ```bash
    git add README.md
    git commit -m "feat: Menambahkan teks untuk menguji webhook"
    git push origin feature/test-otomatisasi
    ```

**Langkah 4.3: Amati Keajaibannya\!**

1.  **Di Jenkins:**
      * Buka *dashboard* Jenkins Anda. Dalam beberapa detik, Anda akan melihat *job* `proyek-utama-ci` muncul di "Build History" dengan status berkedip, yang menandakan ia sedang berjalan.
      * Klik pada *build* tersebut untuk melihat *live console output*. Anda akan melihat setiap *stage* (`Build`, `Test`) dieksekusi.
2.  **Di GitLab:**
      * Buka repositori Anda dan pergi ke halaman **Code \> Commits**.
      * Di samping *commit* terbaru Anda, Anda akan melihat sebuah ikon. Awalnya ikon ini akan berwarna oranye/biru (menandakan *pipeline pending/running*).
      * Setelah beberapa saat (sesuai durasi `sleep` di Jenkinsfile), segarkan halaman. Ikon tersebut akan berubah menjadi **tanda centang hijau** yang menandakan *pipeline* berhasil.
      * Jika Anda mengklik tanda centang hijau tersebut, Anda akan diarahkan langsung ke halaman *build* yang relevan di Jenkins.

**Kesimpulan**
Selamat\! Anda baru saja berhasil mengimplementasikan alur kerja CI otomatis penuh. Setiap perubahan kode yang Anda *push* sekarang akan secara otomatis divalidasi oleh Jenkins, dan Anda mendapatkan umpan balik langsung di GitLab. Ini adalah langkah fundamental yang membuka jalan untuk otomatisasi yang lebih canggih seperti *deployment* dan rilis.


-----

### **Praktik Detail: Job Chaining untuk Alur Rilis Multi-Stage**

**Dasar Teori: Mengapa Job Chaining?**

*Job chaining* adalah praktik memecah sebuah *pipeline* CI/CD yang besar menjadi beberapa *job* yang lebih kecil, terfokus, dan saling terhubung. Pendekatan ini memiliki beberapa keuntungan signifikan:

  * **Pemisahan Tanggung Jawab (*Separation of Concerns*):** Setiap *job* memiliki satu tujuan yang jelas (misalnya, hanya *build*, hanya *deploy-staging*). Ini membuat *pipeline* lebih mudah dipahami dan dikelola.
  * **Dapat Digunakan Ulang (*Reusability*):** *Job* untuk *deployment* dapat dipicu ulang secara independen tanpa harus menjalankan kembali *build* dan *test* dari awal.
  * **Keamanan dan Kontrol Akses:** Anda dapat mengatur izin yang berbeda untuk setiap *job*. Misalnya, semua developer bisa melihat log *build*, tetapi hanya tim Ops yang bisa memicu *job deployment* ke produksi.
  * **Efisiensi *Resource*:** *Job* yang berbeda dapat berjalan di *agent* yang berbeda dengan *tools* yang spesifik, mengoptimalkan penggunaan sumber daya.

**Prasyarat:**
Pastikan *plugin* **`Copy Artifact`** sudah terinstal di Jenkins Anda (**Manage Jenkins \> Plugins \> Available**). Plugin ini sangat penting untuk memindahkan artefak (*file* hasil *build*) dari satu *job* ke *job* lainnya.

-----

### **Job A: `build-and-test` (Pemicu Otomatis)**

**Tujuan:** *Job* ini adalah gerbang utama. Tugasnya adalah mengkompilasi kode, menjalankan *unit test*, dan jika berhasil, menyimpan artefak *build* serta memicu *job* berikutnya.

**Cara Konfigurasi:**

  * *Job* ini dikonfigurasi untuk terpicu secara otomatis oleh *webhook* dari GitLab (`triggers` block).
  * Setelah semua *stage* berhasil, ia akan menggunakan *step* **`build`** di dalam blok **`post { success }`** untuk memulai `deploy-to-staging`.
  * Ia juga menggunakan *step* **`archiveArtifacts`** untuk menyimpan hasil *build* (misalnya file `.jar` atau `.war`) secara resmi di Jenkins.

**Contoh Jenkinsfile (`build-and-test`):**

```groovy
// File: Jenkinsfile di repositori GitLab

pipeline {
    agent any

    options {
        // Hubungkan ke konfigurasi GitLab di Jenkins
        gitLabConnection('GitLab-Connection')
    }

    triggers {
        // Terpicu oleh push dan merge request
        gitlab(triggerOnPush: true, triggerOnMergeRequest: true, branchFilterType: 'All')
    }

    stages {
        stage('Build') {
            steps {
                echo "Membangun aplikasi..."
                // Contoh untuk aplikasi Java Maven
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                echo "Menjalankan unit tests..."
                // Maven secara otomatis menjalankan tes saat packaging
                // Jika tidak, tambahkan perintah tes di sini, misal: sh 'mvn test'
            }
        }
    }
    post {
        success {
            script {
                echo "Build dan Test berhasil. Menyimpan artefak dan memicu deployment ke Staging."

                // 1. Simpan artefak yang dihasilkan (misal, file JAR)
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

                // 2. Memicu job berikutnya (Job B)
                // wait: true berarti job ini tidak akan dianggap selesai sampai job 'deploy-to-staging' selesai.
                build job: 'deploy-to-staging', wait: true
            }
        }
        failure {
            echo "Build atau Test gagal. Tidak melanjutkan ke deployment."
            // Kirim notifikasi kegagalan
        }
    }
}
```

-----

### **Job B: `deploy-to-staging` (Pemicu dari Job A)**

**Tujuan:** Menerima artefak dari *build* yang berhasil dan men-*deploy*-nya ke lingkungan *Staging* untuk diuji oleh tim QA.

**Cara Konfigurasi:**

  * *Job* ini **tidak memiliki `triggers` block** karena tidak dipicu langsung oleh Git. Ia hanya bisa dipicu oleh *job* lain (dalam hal ini, `build-and-test`).
  * Ia menggunakan *step* **`copyArtifacts`** dari *plugin* Copy Artifact untuk mengambil hasil *build* dari `build-and-test`.

**Contoh Jenkinsfile (`deploy-to-staging`):**
*(Anda perlu membuat job pipeline baru di Jenkins bernama `deploy-to-staging` dan memasukkan script ini)*

```groovy
// File: Jenkinsfile untuk job 'deploy-to-staging'

pipeline {
    agent { label 'staging-server' } // Berjalan di agent khusus untuk staging

    stages {
        stage('Get Artifact') {
            steps {
                echo "Mengambil artefak dari job 'build-and-test'..."

                // Salin artefak dari build terakhir yang sukses dari job 'build-and-test'
                copyArtifacts(
                    projectName: 'build-and-test',
                    selector: lastSuccessful()
                )
            }
        }
        stage('Deploy to Staging') {
            steps {
                echo "Men-deploy artefak ke server Staging..."
                // Contoh skrip deployment
                sh 'scp target/*.jar user@staging-server:/opt/app/'
                sh 'ssh user@staging-server "systemctl restart my-app"'
                echo "Deployment ke Staging selesai."
            }
        }
    }
}
```

-----

### **Job C: `deploy-to-production` (Pemicu Manual dengan Parameter)**

**Tujuan:** Men-*deploy* versi aplikasi yang sudah terverifikasi di *Staging* ke lingkungan *Production*. *Job* ini **harus** dipicu secara manual untuk memastikan kontrol dan keamanan.

**Cara Konfigurasi:**

  * *Job* ini menggunakan *directive* **`parameters`** untuk membuat formulir input saat *job* dijalankan. Ini mengubah tombol "Build Now" menjadi "Build with Parameters".
  * Parameter ini memungkinkan pengguna (misalnya, seorang manajer rilis) untuk memilih *build* spesifik mana dari *job Staging* yang ingin di-*promote* ke *Production*.
  * Sebagai lapisan keamanan tambahan, ia menggunakan *step* **`input`** untuk meminta konfirmasi akhir sebelum melakukan tindakan deployment yang sebenarnya.

**Contoh Jenkinsfile (`deploy-to-production`):**
*(Anda perlu membuat job pipeline baru di Jenkins bernama `deploy-to-production` dan memasukkan script ini)*

```groovy
// File: Jenkinsfile untuk job 'deploy-to-production'

pipeline {
    agent { label 'production-server' } // Berjalan di agent khusus untuk production

    parameters {
        // Membuat parameter input untuk memilih build
        string(
            name: 'STAGING_BUILD_NUMBER',
            defaultValue: 'lastSuccessful',
            description: 'Masukkan nomor build dari job "deploy-to-staging" yang akan di-promote ke Production.'
        )
    }

    stages {
        stage('Get Verified Artifact') {
            steps {
                echo "Mengambil artefak dari build #${params.STAGING_BUILD_NUMBER} dari job 'deploy-to-staging'."

                // Salin artefak dari build spesifik yang dipilih melalui parameter
                copyArtifacts(
                    projectName: 'deploy-to-staging',
                    selector: specific(params.STAGING_BUILD_NUMBER)
                )
            }
        }
        stage('Final Confirmation') {
            steps {
                // Meminta persetujuan manual sebelum lanjut ke Production
                input message: "Siap untuk men-deploy build #${params.STAGING_BUILD_NUMBER} ke Production?", ok: 'Deploy!'
            }
        }
        stage('Deploy to Production') {
            steps {
                echo "Men-deploy artefak ke server Production..."
                // Contoh skrip deployment ke production
                sh 'scp target/*.jar user@prod-server:/opt/app/'
                sh 'ssh user@prod-server "systemctl restart my-app-prod"'
                echo "Deployment ke Production SELESAI."
            }
        }
    }
}
```

### **Alur Kerja Keseluruhan**

1.  `git push` → Memicu **Job A (`build-and-test`)**.
2.  Jika Job A **berhasil**:
      * Artefak disimpan.
      * **Job B (`deploy-to-staging`)** dipicu secara otomatis.
      * Aplikasi ter-*deploy* di lingkungan Staging.
3.  Tim QA melakukan pengujian di server Staging.
4.  Setelah disetujui, seorang Manajer Rilis membuka **Job C (`deploy-to-production`)**.
5.  Manajer Rilis mengklik **"Build with Parameters"**, memasukkan nomor *build* dari Job B yang sudah diverifikasi, lalu klik "Build".
6.  *Pipeline* Job C berjalan, meminta konfirmasi akhir (**"Deploy\!"**).
7.  Setelah dikonfirmasi, aplikasi ter-*deploy* di lingkungan Production.

---
### **Referensi**

  * [Dokumentasi `build` step (untuk chaining)](https://www.google.com/search?q=%5Bhttps://www.jenkins.io/doc/pipeline/steps/pipeline-build-step/%5D\(https://www.jenkins.io/doc/pipeline/steps/pipeline-build-step/\))
  * [Dokumentasi Copy Artifact Plugin](https://plugins.jenkins.io/copyartifact/)
  * [Dokumentasi `input` step (untuk persetujuan manual)](https://www.google.com/search?q=%5Bhttps://www.jenkins.io/doc/pipeline/steps/pipeline-input-step/%5D\(https://www.jenkins.io/doc/pipeline/steps/pipeline-input-step/\))
  * [Dokumentasi Parameter di Pipeline](https://www.google.com/search?q=https://www.jenkins.io/doc/book/pipeline/syntax/%23parameters)
  * [Dokumentasi Resmi Jenkins GitLab Plugin](https://plugins.jenkins.io/gitlab-plugin/)
  * [Dokumentasi Resmi GitLab Webhooks](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)
  * [Dokumentasi Resmi GitLab Personal Access Tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)