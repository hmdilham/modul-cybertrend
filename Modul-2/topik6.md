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

**Praktik Berdasarkan Contoh Kasus: Otomatisasi Penuh dari Push ke Build**

Mari kita gabungkan semua konsep di atas.

**Skenario:** Kita akan membuat *pipeline* Jenkins yang secara otomatis terpicu setiap kali ada *push* ke repositori GitLab kita.

1.  **Buat Jenkinsfile di Repositori GitLab Anda:**

    ```groovy
    pipeline {
        agent any

        // Konfigurasi koneksi ke GitLab menggunakan nama yang dibuat di Manage Jenkins > System
        options {
            gitLabConnection('GitLab-Connection')
        }

        // Definisikan trigger
        triggers {
            gitlab(
                triggerOnPush: true,
                triggerOnMergeRequest: true,
                branchFilterType: 'All'
            )
        }

        stages {
            stage('Build') {
                steps {
                    echo "Membangun proyek dari branch ${env.gitlabBranch}..."
                    // Menambahkan status 'pending' ke commit di GitLab
                    updateGitLabCommitStatus(name: 'Build', state: 'pending')
                    // Simulasi proses build
                    sh 'sleep 15'
                }
            }
            stage('Test') {
                steps {
                    echo "Menjalankan tes..."
                    updateGitLabCommitStatus(name: 'Test', state: 'pending')
                    // Simulasi proses tes
                    sh 'sleep 15'
                }
            }
        }
        post {
            success {
                // Update status final ke 'success' di GitLab
                updateGitLabCommitStatus(name: 'Pipeline', state: 'success')
            }
            failure {
                // Update status final ke 'failed' di GitLab
                updateGitLabCommitStatus(name: 'Pipeline', state: 'failed')
            }
        }
    }
    ```

2.  **Buat Job Pipeline di Jenkins:**

      * Buat *job* baru dengan tipe **Pipeline**.
      * Di bagian **Pipeline**, pilih **"Pipeline script from SCM"**.
      * **SCM:** Pilih **Git**.
      * **Repository URL:** Masukkan URL SSH dari repositori GitLab Anda.
      * **Credentials:** Pilih kredensial SSH Anda.
      * **Script Path:** Biarkan `Jenkinsfile` (default).

3.  **Konfigurasi Webhook di GitLab:**

      * Pergi ke repositori GitLab Anda. Navigasi ke **Settings \> Webhooks**.
      * **URL:** Masukkan URL Jenkins Anda diikuti dengan `/project/NAMA_JOB_ANDA`. Contoh: `http://jenkins-url:8080/project/nama-proyek-pipeline`.
      * **Secret Token:** (Opsional, tapi direkomendasikan) Anda dapat menambahkan token rahasia untuk keamanan tambahan.
      * **Trigger:** Pilih **"Push events"** dan **"Merge request events"**.
      * Klik **Add webhook**. Anda bisa mengklik **Test** untuk memastikan GitLab bisa mengirim notifikasi ke Jenkins.

4.  **Uji Coba Alur Kerja:**

      * Lakukan perubahan sederhana pada salah satu file di repositori Anda.
      * Lakukan `git commit` dan `git push`.
      * Buka Jenkins, Anda akan melihat *pipeline* Anda secara otomatis dimulai.
      * Buka GitLab di halaman *commit* terakhir Anda, Anda akan melihat ikon status *pipeline* yang berubah dari *pending* menjadi *success* (atau *failed*).

### **Referensi**

  * [Dokumentasi Resmi Jenkins GitLab Plugin](https://plugins.jenkins.io/gitlab-plugin/)
  * [Dokumentasi Resmi GitLab Webhooks](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)
  * [Dokumentasi Resmi GitLab Personal Access Tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)