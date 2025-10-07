## **Topik 4: Konsep Inti dan Instalasi Jenkins**

Setelah kita menguasai GitLab sebagai platform kolaborasi dan CI/CD terintegrasi di Topik 3, Anda mungkin bertanya: "Mengapa kita perlu belajar *tool* CI/CD lain seperti Jenkins?"

Jawabannya terletak pada **fleksibilitas** dan **ekstensibilitas**. Meskipun GitLab CI/CD sangat kuat dan terintegrasi dengan baik, Jenkins adalah "pisau tentara Swiss" dalam dunia DevOps. Ia adalah server otomatisasi *open-source* yang sangat matang dan dapat diintegrasikan dengan hampir semua *tool* yang ada berkat ekosistem *plugin*-nya yang masif. Dalam lingkungan yang heterogen (menggunakan berbagai bahasa pemrograman, *cloud provider*, dan *tools*), Jenkins seringkali menjadi pilihan utama untuk menjadi orkestrator pusat dari semua proses otomatisasi.

### **1. Arsitektur Jenkins (Model Master/Agent) 🏗️**

**Dasar Teori**

Jenkins dirancang dengan arsitektur terdistribusi yang sangat efisien, yang dikenal sebagai model **Master/Agent** (sebelumnya Master/Slave). Memahami model ini adalah kunci untuk menggunakan Jenkins secara efektif dan skalabel.

**Analogi: Manajer Proyek dan Tim Spesialis**

  * **Jenkins Master (Sang Manajer Proyek):** Ini adalah "otak" dari instalasi Jenkins Anda.

      * **Tugasnya:**
          * Menyediakan antarmuka web (UI) untuk pengguna.
          * Menyimpan semua konfigurasi *job* dan *pipeline*.
          * Menjadwalkan kapan sebuah *job* harus dijalankan.
          * **Mendelegasikan** pekerjaan sebenarnya ke para Agent.
          * Memantau status Agent dan mengumpulkan hasil (log dan artefak) setelah pekerjaan selesai.
      * **Poin Penting:** Dalam praktik terbaik, Jenkins Master **tidak boleh** digunakan untuk menjalankan *job* secara langsung (disebut *build on master*). Master harus fokus pada manajemen agar UI tetap responsif dan sistem tetap stabil.

  * **Jenkins Agent (Tim Pekerja Spesialis):** Ini adalah "tangan dan kaki" yang melakukan semua pekerjaan berat.

      * **Tugasnya:**
          * Menerima perintah dari Master.
          * Menjalankan langkah-langkah dalam *job* (misalnya: `git clone`, `mvn install`, `docker build`).
          * Mengirim kembali status, log, dan artefak ke Master.
      * **Kekuatan Terbesarnya:** Agent bisa sangat beragam. Anda bisa memiliki:
          * Agent berbasis **Linux** untuk membangun aplikasi Java atau Node.js.
          * Agent berbasis **Windows** untuk membangun aplikasi .NET.
          * Agent berbasis **macOS** untuk membangun aplikasi iOS.
          * Agent dengan *tools* spesifik terinstal (misalnya, versi Java atau Python yang berbeda).

**How It Works: Alur Kerja Master-Agent**

1.  Seorang pengguna atau sistem lain (misal: GitLab melalui *webhook*) memicu sebuah *job* di Jenkins Master.
2.  Master menerima permintaan dan memasukkannya ke dalam antrian.
3.  Master mencari Agent yang cocok berdasarkan label (misalnya, label `linux` atau `windows`).
4.  Setelah menemukan Agent yang tersedia, Master mengirimkan instruksi *build* ke Agent tersebut.
5.  Agent membuat *workspace* khusus untuk *job* tersebut, mengunduh kode sumber, dan mengeksekusi semua skrip yang diperintahkan.
6.  Selama proses berjalan, Agent terus mengirimkan *log* secara *real-time* ke Master.
7.  Setelah selesai, Agent melaporkan status akhir (berhasil/gagal) dan mengunggah artefak apa pun ke Master untuk disimpan.

-----

### **2. Ekosistem Plugin: Jantung Fungsionalitas Jenkins 🔌**

**Dasar Teori**

Instalasi Jenkins "polos" sebenarnya memiliki fungsionalitas yang sangat dasar. Kekuatan sejatinya datang dari **ekosistem *plugin*** yang dikelola oleh komunitas. Ada lebih dari 1.800 *plugin* yang tersedia, memungkinkan Jenkins untuk berintegrasi dengan hampir semua teknologi dalam siklus hidup DevOps.

> **Analogi:** Jenkins Core adalah sebuah *smartphone* baru. Ia bisa melakukan panggilan dan mengirim pesan. Tetapi **Plugin** adalah *App Store* yang memungkinkan Anda menginstal aplikasi untuk media sosial, perbankan, game, dan lainnya, yang membuatnya benar-benar berguna sesuai kebutuhan Anda.

**How To: Mengelola Plugin**
Anda dapat mengelola *plugin* dari antarmuka web Jenkins:

1.  Buka *dashboard* Jenkins Anda.
2.  Navigasi ke **Manage Jenkins \> Plugins**.
3.  Anda akan melihat empat tab:
      * **Updates:** Menampilkan *plugin* yang sudah terinstal dan memiliki versi baru.
      * **Available:** Daftar semua *plugin* yang dapat Anda instal. Gunakan kolom pencarian di sini.
      * **Installed:** Daftar *plugin* yang saat ini terpasang di Jenkins Anda.
      * **Advanced:** Opsi untuk konfigurasi lanjutan, seperti mengunggah *plugin* secara manual.

**Praktik Berdasarkan Contoh Kasus Integrasi**

Berikut cara *plugin* memungkinkan Jenkins menjadi pusat orkestrasi CI/CD kita:

  * **Integrasi dengan GitLab (`GitLab Plugin`)**

      * **Tujuan:** Menghubungkan GitLab dengan Jenkins agar setiap `git push` atau *Merge Request* di GitLab dapat secara otomatis memicu *pipeline* di Jenkins. Selain itu, status *pipeline* Jenkins (berhasil/gagal) akan dilaporkan kembali ke UI GitLab.
      * **Cara Kerja:** Plugin ini akan membuat sebuah *endpoint* di Jenkins yang dapat menerima *webhook* dari GitLab. Saat *event* terjadi di GitLab, ia akan mengirim notifikasi ke *endpoint* ini, yang kemudian memicu *job* yang sesuai.
      * *Plugin Terkait:* `GitLab`, `Git`.

  * **Integrasi dengan Nexus (`Nexus Artifact Uploader` / `Nexus Platform Plugin`)**

      * **Tujuan:** Setelah Jenkins berhasil membangun aplikasi dan menghasilkan artefak (misal: file `.jar`, `.war`, atau *Docker image*), kita ingin menyimpannya di manajer repositori artefak seperti Nexus untuk manajemen versi dan distribusi.
      * **Cara Kerja:** Plugin ini menyediakan langkah-langkah *pipeline* (*pipeline steps*) yang dapat Anda tambahkan di akhir *job*. Langkah ini akan mengambil artefak yang dihasilkan dan mengunggahnya ke repositori yang benar di Nexus menggunakan API Nexus.
      * *Plugin Terkait:* `Nexus Artifact Uploader`, `Pipeline Utility Steps`.

  * **Integrasi dengan Spinnaker (`Spinnaker Plugin`)**

      * **Tujuan:** Jenkins sangat baik dalam CI (Build & Test), sementara Spinnaker unggul dalam CD (Deployment Lanjutan). Kita ingin setelah Jenkins selesai dan artefak ada di Nexus, Jenkins memberitahu Spinnaker untuk memulai proses *deployment* (misalnya, *Canary* atau *Blue/Green deployment*).
      * **Cara Kerja:** Setelah *job* CI berhasil, Jenkins akan memicu *pipeline* di Spinnaker. Pemicunya bisa berupa notifikasi langsung ke API Spinnaker atau dengan cara Spinnaker yang memantau repositori Nexus dan secara otomatis mendeteksi artefak versi baru.
      * *Plugin Terkait:* `Spinnaker`.

-----

### **3. Instalasi dan Konfigurasi Dasar Jenkins 🛠️**

**Dasar Teori**

  * **Prasyarat Utama:** Jenkins adalah aplikasi berbasis Java, jadi Anda **wajib** memiliki **Java Development Kit (JDK)** terinstal di mesin tempat Jenkins akan berjalan. Versi yang direkomendasikan saat ini adalah JDK 17 atau 21.
  * **Metode Instalasi:**
    1.  **Paket Sistem Operasi:** Menggunakan `apt` untuk Debian/Ubuntu atau `yum`/`dnf` untuk CentOS/RedHat.
    2.  **File `.war`:** Menjalankan file `jenkins.war` secara mandiri atau men-deploy-nya di *servlet container* seperti Apache Tomcat.
    3.  **Kontainer Docker (Direkomendasikan):** Metode yang paling modern, portabel, dan terisolasi. Ini menyederhanakan manajemen dependensi dan memastikan lingkungan yang konsisten.

**Langkah Praktik: Instalasi Jenkins dengan Docker (Metode Terbaik)**

Metode ini mengasumsikan Anda sudah menginstal Docker di mesin Anda.

1.  **Buat Docker Volume untuk Persistensi Data:** Sangat penting untuk menyimpan data `jenkins_home` di luar kontainer. Jika tidak, semua konfigurasi dan data *job* Anda akan hilang saat kontainer di-restart.

    ```bash
    docker volume create jenkins_home
    ```

2.  **Jalankan Kontainer Jenkins:** Kita akan menggunakan *image* resmi Jenkins LTS (Long-Term Support) yang lebih stabil.

    ```bash
    docker run -d --dns="8.8.8.8" --dns="8.8.4.4" -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins-server jenkins/jenkins:lts-jdk17
    ```

      * `-d`: Menjalankan kontainer di *background* (*detached mode*).
      * `-p 8080:8080`: Memetakan port 8080 di *host* ke port 8080 di kontainer untuk akses UI.
      * `-p 50000:50000`: Memetakan port 50000 untuk komunikasi antara Master dan Agent.
      * `-v jenkins_home:/var/jenkins_home`: Memasang (*mount*) volume yang kita buat ke direktori kerja Jenkins di dalam kontainer.
      * `--name jenkins-server`: Memberi nama yang mudah diingat pada kontainer kita.

3.  **Lakukan Konfigurasi Awal (Post-installation Wizard):**
    a. **Buka Jenkins di Browser:** Navigasi ke `http://<ip_server_anda>:8080`.
    b. **Dapatkan Password Admin Awal:** Jenkins akan meminta *password* awal untuk membuka kunci. Dapatkan *password* tersebut dari log kontainer:

    ```bash
    docker logs jenkins-server
    ```

    Anda akan melihat blok teks dengan *password* alfanumerik. Salin *password* tersebut.

    c. **Install Suggested Plugins:** Jenkins akan menawarkan dua opsi. Untuk pemula, pilih **"Install suggested plugins"**. Ini akan menginstal serangkaian *plugin* paling umum yang sangat berguna (seperti Git, Pipeline, dll).
    d. **Buat Pengguna Admin Pertama:** Buat akun admin Anda sendiri agar Anda tidak perlu lagi menggunakan *password* awal.
    e. **Konfigurasi Instansi:** Konfirmasi URL Jenkins Anda. Biasanya sudah terisi dengan benar. Klik **Save and Finish**.
    f. **Selesai\!** Klik **Start using Jenkins**. Anda akan disambut oleh *dashboard* Jenkins yang siap digunakan.

Kini Anda memiliki server Jenkins yang berfungsi penuh, berjalan dengan cara yang modern dan terisolasi, siap untuk diperluas dengan *plugin* dan dihubungkan dengan Agent untuk membangun *pipeline* CI/CD Anda.

-----

### **Referensi**

  * [Dokumentasi Resmi Jenkins](https://www.jenkins.io/doc/)
  * [Halaman Unduh Jenkins (Termasuk Docker Image)](https://www.jenkins.io/download/)
  * [Jenkins Plugin Marketplace](https://plugins.jenkins.io/)
  * [Tutorial tentang Arsitektur Terdistribusi Jenkins](https://www.jenkins.io/doc/book/architecting-for-scale/)