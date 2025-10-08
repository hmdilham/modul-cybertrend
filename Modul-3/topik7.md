### **Nexus Fundamentals and Installation**
---
### **A. Mengapa Menggunakan Repositori Artefak?**

Secara singkat, repositori artefak digunakan untuk **mengelola semua "bahan bangunan" perangkat lunak (artefak dan dependensi) secara terpusat, andal, dan aman**. Ini adalah komponen infrastruktur yang krusial, sama pentingnya dengan sistem kontrol versi seperti Git.

Untuk benar-benar memahami pentingnya, mari kita bayangkan sebuah dunia *tanpa* repositori artefak dan masalah-masalah yang pasti akan muncul.

#### **Masalah yang Timbul Tanpa Repositori Artefak (Skenario Kacau)**

Bayangkan sebuah tim yang terdiri dari beberapa developer. Mereka tidak menggunakan Nexus atau manajer repositori sejenisnya. Inilah yang akan terjadi:

* **Penyimpanan Artefak di Server CI:** Tim menyimpan hasil *build* (`.jar`, `.war`) langsung di server Jenkins. Tiba-tiba, server Jenkins perlu di-upgrade atau mengalami kerusakan. Semua artefak rilis yang berharga hilang begitu saja. Tidak ada riwayat, tidak ada cadangan.
* **"Neraka Dependensi":** Setiap developer dan server Jenkins mengunduh dependensi (misalnya, `library` dari internet) secara langsung dari sumber publik seperti Maven Central atau npmjs.org. Suatu hari, Maven Central mengalami gangguan. Akibatnya, **tidak ada seorang pun di perusahaan yang bisa melakukan *build***, menyebabkan seluruh proses pengembangan terhenti.
* **Masalah "Works on My Machine":** Developer A menggunakan `library-x` versi 1.2, sementara Developer B menggunakan versi 1.3. Saat kode mereka digabungkan, aplikasi rusak karena perbedaan versi. Tidak ada "sumber kebenaran tunggal" (*single source of truth*) untuk dependensi.
* **Risiko Keamanan yang Tidak Terlihat:** Tim tanpa sadar menggunakan `library` pihak ketiga yang memiliki kerentanan keamanan kritis (seperti Log4j). Karena tidak ada gerbang kontrol, komponen berbahaya ini masuk dengan bebas ke dalam aplikasi dan baru ketahuan setelah terjadi insiden keamanan di produksi.
* **Distribusi Internal yang Berantakan:** Tim *backend* selesai membuat `api-service-v2.jar`. Mereka mengirimkannya ke tim *frontend* melalui email atau Slack. Versi file menjadi tidak terlacak, dan bisa jadi tim *frontend* secara tidak sengaja menggunakan versi yang salah.

Repositori artefak dirancang untuk menyelesaikan semua masalah ini secara sistematis.

---

### **Manfaat Utama Menggunakan Repositori Artefak**

Berikut adalah alasan-alasan mendetail mengapa repositori artefak seperti Nexus adalah pilar penting dalam DevOps.

#### **1. Sumber Kebenaran Tunggal (*Single Source of Truth*) 📚**

Repositori artefak menjadi pusat dari semua komponen perangkat lunak.

* **Untuk Dependensi Eksternal:** Semua dependensi dari luar diambil melalui repositori ini. Ia memastikan bahwa setiap developer, setiap server CI, dan setiap lingkungan (*staging*, *production*) menggunakan **versi dependensi yang sama persis**, yang telah disetujui dan di-cache.
* **Untuk Artefak Internal:** Ini adalah satu-satunya tempat resmi untuk menemukan hasil *build* dari proyek Anda. Jika Anda butuh `reporting-service-v3.1.5.war`, Anda tahu persis di mana mencarinya, lengkap dengan metadata kapan dan oleh siapa ia dibangun.

**Dampaknya:** Menghilangkan ambiguitas dan masalah "works on my machine" yang disebabkan oleh inkonsistensi versi.

#### **2. Kecepatan dan Keandalan Build 🚀**

Fitur *proxy caching* adalah salah satu keunggulan terbesar.

* **Kecepatan:** Saat dependensi pertama kali diminta, Nexus mengunduhnya dari internet dan menyimpannya di *cache*. Permintaan berikutnya untuk dependensi yang sama akan dilayani langsung dari Nexus dengan kecepatan jaringan lokal, bukan dari internet yang lambat. Untuk proyek dengan ratusan dependensi, ini **mempercepat waktu *build* secara signifikan**.
* **Keandalan (*Reliability*):** Jika repositori publik eksternal (npmjs.org, Docker Hub, dll.) sedang *offline* atau lambat, proses *build* Anda tidak akan terpengaruh selama dependensi yang dibutuhkan sudah ada di *cache* Nexus. Ini memberikan **ketahanan dan kemandirian** pada proses pengembangan Anda.

**Dampaknya:** Proses CI menjadi lebih cepat dan lebih tangguh terhadap gangguan eksternal.

#### **3. Keamanan dan Kontrol (*Security and Governance*) 🛡️**

Repositori artefak bertindak sebagai "firewall" untuk komponen perangkat lunak Anda.

* **Gerbang Kontrol:** Anda dapat mengonfigurasi Nexus untuk memblokir proksi dependensi yang memiliki kerentanan keamanan yang diketahui atau yang lisensinya tidak sesuai dengan kebijakan perusahaan. Ini mencegah komponen berbahaya masuk ke dalam ekosistem Anda sejak awal.
* **Audit dan Visibilitas:** Nexus memberikan visibilitas penuh terhadap semua komponen yang digunakan di seluruh aplikasi Anda. Jika suatu hari ditemukan kerentanan baru pada `library` tertentu, Anda dapat dengan cepat mencari tahu proyek mana saja yang menggunakannya dan perlu diperbaiki.
* **Kontrol Akses:** Anda dapat mengatur izin terperinci tentang siapa yang boleh mengunduh dari repositori tertentu dan siapa yang boleh mengunggah artefak ke repositori rilis.

**Dampaknya:** Meningkatkan postur keamanan perangkat lunak secara proaktif dan memberikan kontrol penuh atas rantai pasokan perangkat lunak Anda.

#### **4. Manajemen Rilis yang Efisien 📦**

Nexus membawa struktur dan disiplin pada proses rilis.

* **Pemisahan `SNAPSHOT` dan `Release`:** Nexus memungkinkan Anda membuat repositori terpisah untuk artefak pengembangan (`SNAPSHOTS`) dan artefak stabil (`RELEASES`). Ini memastikan bahwa hanya versi yang sudah stabil dan teruji yang dapat digunakan untuk *deployment* ke produksi.
* **Imutabilitas Rilis:** Repositori `releases` dapat dikonfigurasi agar bersifat *immutable* (tidak dapat diubah). Artinya, sekali `versi-1.0.0` dipublikasikan, ia tidak dapat ditimpa. Ini menjamin integritas dan reproduktifitas *build*.

**Dampaknya:** Proses rilis menjadi lebih terstruktur, dapat diaudit, dan mengurangi risiko men-*deploy* kode yang belum stabil ke produksi.


---

### **B. Jenis-jenis Repositori: Hosted, Proxy, dan Group**

Kita akan membahas kembali teori setiap tipe secara ringkas, lalu langsung masuk ke praktik konfigurasinya di dalam antarmuka Nexus.

* **`hosted` Repository (Gudang Internal):** Tempat Anda mempublikasikan dan menyimpan artefak yang **dibuat oleh tim Anda sendiri**.
* **`proxy` Repository (Gerbang Cache Eksternal):** Bertindak sebagai *cache* lokal untuk repositori publik di internet (seperti Maven Central, npmjs.org).
* **`group` Repository (Pintu Masuk Terpadu):** Menggabungkan beberapa repositori lain di bawah satu URL untuk menyederhanakan konfigurasi di sisi klien (developer, Jenkins).

-----

### **C. Instalasi Nexus 3 dan Pengaturan Awal**

Panduan ini akan berfokus pada metode instalasi yang paling modern dan direkomendasikan: menggunakan **Docker**. Metode ini memastikan proses instalasi yang bersih, terisolasi, dan mudah direproduksi di lingkungan mana pun.

#### **Prasyarat Sistem 🖥️**

Sebelum memulai, pastikan mesin server Anda (baik fisik maupun virtual) memenuhi persyaratan minimum berikut:

  * **Sistem Operasi:** Server berbasis Linux (seperti Ubuntu 20.04/22.04 atau CentOS) sangat direkomendasikan.
  * **Docker:** Pastikan Docker dan Docker Compose sudah terinstal dan layanan Docker sedang berjalan.
  * **Sumber Daya:**
      * **RAM:** **Minimal 4 GB** dialokasikan untuk Nexus. Karena Nexus adalah aplikasi Java, ia membutuhkan memori yang cukup untuk berjalan secara optimal.
      * **CPU:** Minimal 2 vCPU.
      * **Ruang Disk:** Minimal 20 GB ruang kosong untuk volume data Nexus. Kebutuhan ini akan terus bertambah seiring Anda menyimpan lebih banyak artefak.
  * **Jaringan:** Pastikan server memiliki alamat IP statis dan *port* `8081` tidak digunakan oleh aplikasi lain serta dapat diakses dari jaringan Anda.

-----

### **Langkah Praktik: Instalasi dan Konfigurasi**

Kita akan membagi proses ini menjadi beberapa langkah yang jelas, dari menjalankan kontainer hingga konfigurasi awal.

#### **Langkah 1: Buat Volume Persisten untuk Data Nexus**

Ini adalah langkah **paling krusial**. Kita harus menyimpan semua data Nexus (konfigurasi, repositori, dan artefak) di luar kontainer Docker. Jika tidak, semua data Anda akan hilang saat kontainer diperbarui atau dihapus.

Buka terminal di server Anda dan jalankan perintah berikut:

```bash
docker volume create nexus-data
```

Perintah ini akan membuat volume terkelola bernama `nexus-data` yang akan kita gunakan untuk menyimpan semua data Nexus secara permanen.

-----

#### **Langkah 2: Jalankan Kontainer Nexus 3**

Sekarang, kita akan mengunduh *image* Nexus 3 resmi dari Sonatype dan menjalankannya sebagai kontainer.

Jalankan perintah `docker run` berikut di terminal Anda:

```bash
docker run -d -p 8081:8081 -v nexus-data:/nexus-data --name nexus-repo sonatype/nexus3
```

Mari kita bedah perintah ini:

  * **`-d`**: Menjalankan kontainer dalam mode *detached* (di latar belakang).
  * **`-p 8081:8081`**: Memetakan *port* 8081 di mesin *host* Anda ke *port* 8081 di dalam kontainer. Ini memungkinkan kita mengakses antarmuka web Nexus.
  * **`-v nexus-data:/nexus-data`**: Memasang (*mount*) volume `nexus-data` yang kita buat tadi ke direktori `/nexus-data` di dalam kontainer, yang merupakan lokasi penyimpanan data Nexus.
  * **`--name nexus-repo`**: Memberi nama yang mudah diingat pada kontainer kita.
  * **`sonatype/nexus3`**: Nama *image* Docker resmi untuk Nexus Repository 3.

-----

#### **Langkah 3: Verifikasi Instalasi**

Untuk memastikan kontainer berjalan dengan benar, gunakan perintah berikut:

```bash
docker ps
```

Anda akan melihat output yang mirip seperti ini, yang mengonfirmasi bahwa `nexus-repo` sedang berjalan dan *port* 8081 telah dipetakan dengan benar.

```
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                                           NAMES
a1b2c3d4e5f6   sonatype/nexus3      "sh -c ${JAVA_OPTS} …"   2 minutes ago   Up 2 minutes   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp       nexus-repo
```

-----

#### **Langkah 4: Pengaturan Awal di Antarmuka Web**

Sekarang Nexus sudah berjalan, kita perlu melakukan konfigurasi awal melalui browser.

1.  **Akses Nexus:** Buka browser dan navigasi ke `http://<IP_SERVER_ANDA>:8081`.

      * **Catatan:** Nexus mungkin memerlukan waktu **2-3 menit** untuk proses *booting* pertamanya. Jika halaman tidak langsung muncul, tunggu sejenak lalu segarkan kembali.

2.  **Dapatkan Password Admin Awal:**
    Untuk alasan keamanan, *password* admin awal disimpan di dalam sebuah file. Untuk melihatnya, jalankan perintah ini di terminal server Anda:

    ```bash
    docker exec nexus-repo cat /nexus-data/admin.password
    ```

    Salin *password* (string acak) yang muncul.

3.  **Masuk (Sign In):**

      * Di halaman web Nexus, klik tombol **Sign in** di pojok kanan atas.
      * **Username:** `admin`
      * **Password:** Tempel (*paste*) *password* yang baru saja Anda salin.

4.  **Jalankan Setup Wizard:**

      * **Ubah Password:** Layar pertama akan langsung meminta Anda untuk mengatur *password* baru yang lebih aman untuk akun `admin`. Lakukan ini segera.
      * **Konfigurasi Akses Anonim:** Layar berikutnya akan menanyakan apakah Anda ingin mengizinkan akses anonim. Untuk keamanan terbaik, pilih **Disable anonymous access**. Ini akan mengharuskan semua permintaan (termasuk dari Jenkins) untuk diautentikasi.
      * **Selesai:** Klik **Finish**.

Selamat\! Anda telah berhasil menginstal dan mengonfigurasi server Sonatype Nexus Repository. Server ini sekarang siap untuk Anda gunakan untuk membuat berbagai jenis repositori (Maven, Docker, npm) dan diintegrasikan ke dalam alur kerja CI/CD Anda.

---

### **Praktik: Membuat dan Mengonfigurasi Tipe Repositori di Nexus**

Mari kita langsung konfigurasikan satu set repositori standar untuk alur kerja **Maven (Java)**. Ini adalah pola yang paling umum dan dapat diterapkan ke format lain seperti npm atau Docker.

**Asumsi:** Anda sudah berhasil menginstal Nexus dan bisa masuk sebagai `admin`.

**Langkah 1: Membuat `proxy` Repository (Menghubungkan ke Dunia Luar)**

Tujuan kita adalah membuat *cache* lokal untuk Maven Central, repositori Java publik terbesar.

1.  Masuk ke *dashboard* Nexus Anda.
2.  Klik ikon **Gerigi (⚙️) Server Administration and Configuration** di menu atas.
3.  Di menu sebelah kiri, klik **Repositories**.
4.  Klik tombol biru **Create repository**.
5.  Pilih resep (*recipe*) **`maven2 (proxy)`**.
    
6.  Isi formulir konfigurasi:
    * **Name:** Beri nama yang jelas, misalnya `maven-central-proxy`.
    * **Remote storage:** Masukkan URL dari repositori publik yang ingin Anda proksi. Untuk Maven Central, URL-nya adalah: `https://repo1.maven.org/maven2/`.
    * **Blob Store:** Biarkan *default*. Ini adalah lokasi penyimpanan fisik untuk artefak yang di-*cache*.
7.  Gulir ke bawah dan klik **Create repository**.

Anda sekarang memiliki gerbang yang andal dan cepat ke dunia luar untuk semua dependensi Java Anda.

---

**Langkah 2: Membuat `hosted` Repository (Menyimpan Artefak Internal)**

Selanjutnya, kita butuh tempat untuk menyimpan artefak yang akan dihasilkan oleh Jenkins. Praktik terbaik adalah memisahkan artefak rilis (`release`) yang stabil dari artefak pengembangan (`snapshot`) yang sering berubah.

**A. Membuat Repositori `releases`**

1.  Di halaman **Repositories**, klik lagi **Create repository**.
2.  Pilih resep **`maven2 (hosted)`**.
3.  Isi formulir:
    * **Name:** `maven-releases`.
    * **Version policy:** Pilih **Release**. Ini memastikan hanya artefak versi stabil (misal: `1.0`, `2.1.5`) yang bisa disimpan di sini.
    * **Deployment policy:** Pilih **Disable redeploy**. Ini adalah pengaturan keamanan yang sangat penting. Ini akan mencegah siapa pun menimpa (*overwrite*) artefak rilis yang sudah ada, menjamin bahwa versi `1.0` akan selalu sama.
4.  Klik **Create repository**.

**B. Membuat Repositori `snapshots`**

1.  Ulangi prosesnya: Klik **Create repository** dan pilih **`maven2 (hosted)`**.
2.  Isi formulir:
    * **Name:** `maven-snapshots`.
    * **Version policy:** Pilih **Snapshot**. Ini mengizinkan penyimpanan artefak versi pengembangan (misal: `1.0-SNAPSHOT`).
    * **Deployment policy:** Pilih **Allow redeploy**. Ini kebalikan dari *releases*. Developer harus bisa menimpa versi `SNAPSHOT` berkali-kali selama proses pengembangan.
3.  Klik **Create repository**.

Kini Anda memiliki dua "gudang" internal: satu untuk artefak yang stabil dan satu lagi untuk artefak yang sedang dalam pengembangan.

---

**Langkah 3: Membuat `group` Repository (Menyatukan Semuanya)**

Ini adalah langkah terakhir dan paling penting. Kita akan membuat satu "pintu utama" yang akan digunakan oleh semua developer dan *tool* CI.

1.  Di halaman **Repositories**, klik **Create repository**.
2.  Pilih resep **`maven2 (group)`**.
3.  Isi formulir:
    * **Name:** Beri nama yang umum, misalnya `maven-public-group`.
    * **Members:** Ini adalah bagian terpenting. Di sini Anda akan melihat dua kolom: *Available* dan *Members*. Pindahkan repositori dari *Available* ke *Members* dalam urutan berikut:
        1.  `maven-releases`
        2.  `maven-snapshots`
        3.  `maven-central-proxy`
    
    * **Pentingnya Urutan:** Nexus akan mencari artefak berdasarkan urutan ini. Dengan menempatkan `releases` dan `snapshots` di atas `proxy`, Anda memastikan bahwa Nexus akan **selalu mencari artefak internal Anda terlebih dahulu** sebelum mencoba mengunduhnya dari internet. Ini mencegah masalah keamanan dan inkonsistensi.

4.  Klik **Create repository**.

### **Bagaimana Ini Semua Digunakan?**

Anda telah berhasil membuat satu set repositori yang fungsional. Sekarang, di semua konfigurasi *tool* Anda (misalnya, file `settings.xml` Maven atau konfigurasi di Jenkins), Anda **hanya perlu menggunakan satu URL**, yaitu URL dari `maven-public-group`.

Untuk melihat URL-nya, klik repositori `maven-public-group` dari daftar, dan Anda akan melihat URL-nya di bagian *Properties*.



Ketika sebuah *tool* meminta artefak dari URL *group* ini:
* Jika meminta `aplikasi-saya-1.0.jar` (artefak internal), Nexus akan menemukannya di `maven-releases`.
* Jika meminta `spring-boot-2.7.5.jar` (dependensi eksternal), Nexus akan mencarinya di `maven-releases` dan `maven-snapshots` terlebih dahulu. Karena tidak ada, ia akan melanjutkan pencarian ke `maven-central-proxy` dan mengunduhnya dari internet (jika belum ada di *cache*).

Dengan cara ini, kompleksitas manajemen repositori disembunyikan di balik satu URL sederhana, membuat hidup developer dan tim DevOps jauh lebih mudah.


-----

### **Praktik: Membuat Repositori npm di Nexus**

**Tujuan:** Kita akan membuat sebuah *npm registry* pribadi di dalam Nexus untuk:

1.  Menyimpan *cache* paket dari `npmjs.org` (`npm-proxy`).
2.  Mempublikasikan paket-paket atau *library* internal kita (`npm-hosted`).
3.  Menyediakan satu *registry* terpadu untuk semua perintah `npm` (`npm-group`).

-----

### **Langkah 1: Membuat `npm (proxy)` Repository (Cache untuk npmjs.org)**

Langkah ini akan membuat Nexus bertindak sebagai perantara untuk *registry* npm publik, mempercepat *build* dan meningkatkan keandalan.

1.  Di antarmuka Nexus, pergi ke **⚙️ Server Administration \> Repositories**.
2.  Klik **Create repository**.
3.  Pilih resep **`npm (proxy)`**.
4.  Isi formulir:
      * **Name:** `npm-proxy`.
      * **Remote storage:** Masukkan URL untuk *registry* npm resmi: `https://registry.npmjs.org`.
      * Biarkan pengaturan lainnya sebagai *default*.
5.  Klik **Create repository**.

-----

### **Langkah 2: Membuat `npm (hosted)` Repository (Untuk Paket Internal)**

Di sinilah Anda akan mempublikasikan paket-paket pribadi, misalnya *library* komponen UI atau *helper utility* yang digunakan di berbagai proyek internal.

1.  Kembali ke halaman **Repositories** dan klik **Create repository**.
2.  Pilih resep **`npm (hosted)`**.
3.  Isi formulir:
      * **Name:** `npm-hosted`.
      * **Deployment policy:** Anda bisa memilih `Allow redeploy` karena paket internal mungkin sering di-update selama pengembangan.
      * Biarkan pengaturan lainnya sebagai *default*.
4.  Klik **Create repository**.

-----

### **Langkah 3: Membuat `npm (group)` Repository (Satu URL untuk Semua Paket npm)**

Ini adalah "pintu utama" yang akan digunakan oleh semua developer dan server CI untuk menjalankan perintah `npm`.

1.  Kembali ke halaman **Repositories** dan klik **Create repository**.
2.  Pilih resep **`npm (group)`**.
3.  Isi formulir:
      * **Name:** `npm-group`.
      * **Members:** Pindahkan `npm-hosted` dan `npm-proxy` dari kolom *Available* ke kolom *Members*. **Pastikan `npm-hosted` berada di urutan teratas**.
4.  Klik **Create repository**.

-----

### **Langkah 4: Konfigurasi npm Client (Langkah Krusial)**

Sekarang, Anda perlu memberitahu *command-line tool* (CLI) `npm` di komputer Anda (dan di Jenkins Agent) untuk menggunakan *registry* Nexus yang baru, bukan lagi `npmjs.org`.

1.  **Atur Registry:**

      * Di Nexus, klik repositori `npm-group` yang baru Anda buat dan salin URL-nya.
      * Buka terminal di komputer Anda dan jalankan perintah berikut, ganti URL dengan URL yang Anda salin:
        ```bash
        npm config set registry http://<ip_nexus_anda>:8081/repository/npm-group/
        ```

2.  **Konfigurasi Otentikasi (Untuk `npm publish`):**
    Untuk bisa mempublikasikan paket ke `npm-hosted`, Anda perlu login.

      * Pertama, aktifkan "npm Bearer Token Realm" di Nexus. Pergi ke **⚙️ Administration \> Security \> Realms**. Pindahkan **npm Bearer Token Realm** dari *Available* ke *Active*. Klik **Save**.
      * Sekarang, di terminal Anda, jalankan perintah login:
        ```bash
        npm login
        ```
      * Saat diminta, masukkan:
          * **Username:** `admin` (atau username Nexus Anda).
          * **Password:** *Password* Nexus Anda.
          * **Email:** Alamat email Anda.

    Perintah ini akan membuat atau memperbarui file `.npmrc` di direktori *home* Anda dengan token otentikasi.

-----

### **Alur Kerja Praktis dengan npm**

Setelah konfigurasi selesai, alur kerja Anda menjadi seperti ini:

1.  **Menginstal Paket Publik (melalui Proxy):**

    ```bash
    # npm akan meminta 'express' ke Nexus (npm-group)
    # Nexus akan mengunduhnya dari npmjs.org, menyimpannya di cache, lalu memberikannya pada Anda.
    npm install express
    ```

    Instalasi berikutnya dari `express` akan diambil langsung dari *cache* Nexus dan akan jauh lebih cepat.

2.  **Mempublikasikan Paket Internal (ke Hosted):**
    Misalkan Anda memiliki proyek paket npm pribadi.

    ```bash
    # Masuk ke direktori proyek paket Anda
    cd /path/to/my-internal-library

    # Publikasikan paket. Karena Anda sudah login dan mengatur registry,
    # npm akan otomatis mengirimkannya ke Nexus. Nexus akan menyimpannya di 'npm-hosted'.
    npm publish
    ```

3.  **Menginstal Paket Internal di Proyek Lain:**
    Sekarang, di proyek aplikasi utama Anda, Anda bisa menginstal *library* internal tersebut sama seperti paket publik lainnya.

    ```bash
    npm install my-internal-library
    ```

    Nexus akan menemukannya di `npm-hosted` dan menyediakannya untuk Anda.

Dengan setup ini, Anda memiliki kontrol penuh atas seluruh ekosistem JavaScript Anda, meningkatkan kecepatan, keandalan, dan keamanan.

-----

### **Praktik: Membuat Repositori Docker di Nexus**

**Tujuan:** Kita akan membuat sebuah Docker Registry pribadi di dalam Nexus. Ini akan memungkinkan kita untuk:

1.  Menyimpan *cache image* dari Docker Hub (`docker-hub-proxy`).
2.  Menyimpan *image* aplikasi yang kita bangun sendiri (`docker-hosted`).
3.  Menyediakan satu alamat tunggal untuk semua interaksi Docker (`docker-group`).

-----

### **Langkah 1: Membuat `docker (proxy)` Repository (Cache untuk Docker Hub)**

Tujuan langkah ini adalah agar setiap kali Anda `docker pull nginx`, *image* tersebut diunduh melalui Nexus dan disimpan di *cache*. Tarikan berikutnya akan jauh lebih cepat dan tidak bergantung pada koneksi internet ke Docker Hub.

1.  Di antarmuka Nexus, pergi ke **⚙️ Server Administration \> Repositories**.
2.  Klik **Create repository**.
3.  Pilih resep **`docker (proxy)`**.
4.  Isi formulir:
      * **Name:** `docker-hub-proxy`.
      * **Remote storage:** Masukkan URL untuk Docker Hub: `https://registry-1.docker.io`.
      * **Docker Index:** Pilih **Use Docker Hub** dari *dropdown*. Ini memberitahu Nexus untuk menggunakan indeks Docker Hub untuk mencari *image*.
      * Biarkan pengaturan lainnya sebagai *default*.
5.  Klik **Create repository**.

-----

### **Langkah 2: Membuat `docker (hosted)` Repository (Registry Internal Anda)**

Di sinilah Anda akan menyimpan *image* hasil *build* Jenkins, seperti `aplikasi-saya:1.0` atau `backend-service:latest`.

1.  Kembali ke halaman **Repositories** dan klik **Create repository**.
2.  Pilih resep **`docker (hosted)`**.
3.  Isi formulir:
      * **Name:** `docker-hosted`.

      * **HTTP Port (Penting\!):** Di bagian bawah, di bawah "Connector", **centang kotak untuk membuat konektor HTTP**. Masukkan nomor *port* yang unik di server Anda, misalnya **`8082`**. Port ini akan digunakan Nexus secara khusus untuk "mendengarkan" perintah `docker push` ke repositori ini. Pastikan *port* ini tidak digunakan oleh layanan lain.

      * **Force basic authentication:** Biarkan tercentang. Ini memastikan hanya pengguna yang terotentikasi yang dapat mendorong (*push*) *image*.
4.  Klik **Create repository**.

-----

### **Langkah 3: Membuat `docker (group)` Repository (Satu Pintu untuk Docker)**

Langkah terakhir adalah menyatukan *proxy* dan *hosted* di bawah satu alamat yang mudah digunakan.

1.  Kembali ke halaman **Repositories** dan klik **Create repository**.
2.  Pilih resep **`docker (group)`**.
3.  Isi formulir:
      * **Name:** `docker-group`.
      * **HTTP Port (Penting\!):** Sama seperti *hosted*, *group repository* juga memerlukan konektor HTTP-nya sendiri. Masukkan nomor *port* unik lainnya, misalnya **`8083`**. Inilah *port* yang akan digunakan oleh developer dan Jenkins untuk semua perintah Docker (`pull`, `push`, `login`).
      * **Members:** Pindahkan `docker-hosted` dan `docker-hub-proxy` dari kolom *Available* ke kolom *Members*. **Pastikan `docker-hosted` berada di atas `docker-hub-proxy`**. Ini memastikan Nexus mencari *image* internal terlebih dahulu.
4.  Klik **Create repository**.

-----

### **Langkah 4: Konfigurasi Docker Client (Langkah Krusial)**

Secara *default*, Docker Client hanya berkomunikasi dengan registry melalui HTTPS yang aman. Karena kita menjalankan Nexus di lokal dengan HTTP, kita harus memberitahu Docker untuk "mempercayai" alamat registry Nexus kita.

Ini harus dilakukan di **setiap mesin yang akan berinteraksi dengan Nexus Docker Registry** (misalnya, laptop developer, Jenkins Agent).

1.  Buat atau edit file `daemon.json`. Lokasinya biasanya di `/etc/docker/daemon.json` untuk Linux.

2.  Tambahkan konfigurasi berikut. Gunakan **alamat IP server Nexus Anda** dan **port dari `docker-group` (8083)**.

    ```json
    {
      "insecure-registries" : ["<ip_nexus_anda>:8083"]
    }
    ```

      * Contoh: `{ "insecure-registries" : ["192.168.1.50:8083"] }`

3.  **Restart layanan Docker** agar perubahan ini diterapkan.

    ```bash
    sudo systemctl restart docker
    ```

-----

### **Alur Kerja Praktis dengan Docker**

Sekarang, Anda bisa menggunakan registry Nexus Anda.

1.  **Login ke Registry Group:**

    ```bash
    docker login -u admin -p <password_admin_nexus> <ip_nexus_anda>:8083
    ```

2.  **Menarik (*Pull*) Image dari Docker Hub (melalui Proxy):**

    ```bash
    # Tarikan pertama akan mengunduh dari internet dan menyimpannya di cache Nexus
    docker pull <ip_nexus_anda>:8083/nginx:latest

    # Tarikan kedua (dan seterusnya) akan sangat cepat karena diambil dari cache Nexus
    docker pull <ip_nexus_anda>:8083/nginx:latest
    ```

3.  **Mendorong (*Push*) Image Internal (ke Hosted):**

    ```bash
    # 1. Bangun image aplikasi Anda seperti biasa
    docker build -t my-app:1.0 .

    # 2. Beri 'tag' pada image dengan alamat registry Nexus
    docker tag my-app:1.0 <ip_nexus_anda>:8083/my-app:1.0

    # 3. Dorong image ke Nexus. Nexus akan secara otomatis menyimpannya di 'docker-hosted'
    docker push <ip_nexus_anda>:8083/my-app:1.0
    ```

Anda sekarang memiliki Docker Registry yang berfungsi penuh dan terintegrasi di dalam Nexus, siap digunakan dalam *pipeline* CI/CD Anda untuk menyimpan dan mendistribusikan *image* aplikasi Anda.