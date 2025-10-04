## **Topik 2: Pengenalan GitLab untuk Kontrol Versi dan Kolaborasi**

Setelah memahami *mengapa* kita butuh DevOps di Topik 1, sekarang kita akan fokus pada *bagaimana* kita mengelola aset paling berharga dalam pengembangan perangkat lunak: **kode sumber**. Topik ini akan membahas tuntas GitLab sebagai platform kolaborasi dan Git sebagai mesin kontrol versi yang menjadi fondasinya. Kita akan beralih dari konsep ke praktik langsung, dari perintah dasar hingga menangani skenario kolaborasi yang kompleks.

### **Penting: Membedakan Git dan GitLab**

Sebelum melangkah lebih jauh, mari kita pertegas perbedaan fundamental keduanya karena ini adalah kunci untuk memahami keseluruhan alur kerja.

  * **Git  (Engine)**
    adalah **perangkat lunak (software)** yang berjalan di komputer Anda. Ia tidak peduli dengan siapa Anda, apa tugas Anda, atau kapan tenggat waktu proyek. Git hanya memiliki satu tugas: **mencatat setiap perubahan pada setiap karakter di setiap file dalam proyek Anda dengan sangat efisien**. Ia adalah sistem terdistribusi, artinya setiap developer memiliki salinan penuh dari riwayat proyek.

  * **GitLab  (Platform)**
    adalah **platform DevOps lengkap berbasis web** yang menggunakan Git sebagai intinya. Jika Git adalah mesin, GitLab adalah mobil mewah di sekelilingnya. GitLab menambahkan lapisan **kolaborasi dan manajemen** di atas Git, seperti:

      * Manajemen pengguna dan hak akses.
      * Antarmuka visual untuk melihat riwayat kode.
      * Fitur pelacakan tugas (*Issue Tracking*).
      * Proses *Code Review* melalui *Merge Request*.
      * Otomatisasi CI/CD yang akan kita bahas di Topik 3.

> **Kesimpulan:** Anda menggunakan **Git** di terminal/lokal untuk bekerja dengan kode, dan Anda menggunakan **GitLab** di browser untuk berkolaborasi dengan tim dan mengelola proyek secara keseluruhan.

-----

### **1. Antarmuka dan Fitur Inti GitLab 🖥️**

**Dasar Teori**

Antarmuka GitLab dirancang untuk menjadi satu platform tunggal (*single application*) untuk seluruh siklus hidup DevOps. Saat Anda membuka sebuah proyek, Anda akan menemukan semua yang dibutuhkan tim Anda, yang dapat dikelompokkan ke dalam beberapa area fungsional.

**How To: Menavigasi Fitur-Fitur Kunci**

Di bilah sisi kiri proyek Anda, Anda akan menemukan beberapa menu utama:

  * **Manajemen Proyek (`Manage`):**

      * **Members:** Mengundang anggota tim dan mengatur peran mereka (Guest, Reporter, Developer, Maintainer, Owner).
      * **Issues:** Jantung dari manajemen tugas. Di sini Anda membuat, melacak, dan mendiskusikan bug, fitur baru, atau tugas lainnya.
      * **Epics & Milestones:** (Fitur premium) Untuk mengelola proyek yang lebih besar dengan mengelompokkan beberapa *issue* ke dalam satu tema (Epic) atau target waktu (Milestone).

  * **Manajemen Kode (`Code`):**

      * **Repository:** "Rumah" dari kode Anda. Anda bisa menelusuri file, melihat riwayat *commit*, dan membandingkan *branch*.
      * **Branches:** Melihat semua cabang aktif dalam proyek.
      * **Merge Requests:** Tempat untuk mengajukan, mereview, dan mendiskusikan perubahan kode sebelum digabungkan.
      * **Snippets:** Menyimpan dan berbagi potongan kode tanpa harus menyimpannya di repositori utama.

  * **Otomatisasi (`Build`):**

      * **Pipelines:** Di sinilah Anda akan melihat visualisasi dari alur kerja CI/CD Anda yang berjalan secara otomatis. Ini adalah jembatan langsung ke Topik 3.

  * **Dokumentasi (`Plan`):**

      * **Wiki:** Tempat yang ideal untuk menyimpan dokumentasi proyek yang lebih permanen, seperti panduan instalasi atau arsitektur sistem.

-----

### **2. Dasar-Dasar Git: Siklus Hidup Kode Anda 📂**

**Dasar Teori: Tiga Area Kerja Git**

Untuk benar-benar menguasai Git, pahami tiga "area" virtual tempat kode Anda berada.

1.  **Working Directory**: Folder proyek di komputer Anda. Ini adalah "meja kerja" Anda yang bisa berantakan. Anda bebas mengubah, menambah, atau menghapus file di sini.
2.  **Staging Area (Index)**: Ini adalah "keranjang belanja" atau "map transit". Sebelum Anda menyimpan perubahan secara permanen, Anda harus memasukkannya terlebih dahulu ke *Staging Area*. Ini memungkinkan Anda untuk memilih dengan cermat hanya perubahan yang relevan untuk sebuah *commit*.
3.  **Repository (.git)**: Ini adalah "lemari arsip" lokal Anda. Saat Anda melakukan `commit`, semua yang ada di *Staging Area* akan disimpan sebagai "snapshot" permanen ke dalam folder `.git` di dalam proyek Anda. Ini berisi seluruh riwayat proyek.

**Langkah Praktik: Alur Kerja dari Lokal ke GitLab**

Mari kita ikuti alur kerja developer sehari-hari dengan perintah yang sesuai.

1.  **Setup Awal (Hanya sekali per komputer):** Atur identitas Anda.
    ```bash
    git config --global user.name "Nama Lengkap Anda"
    git config --global user.email "emailanda@contoh.com"
    ```
2.  **Mendapatkan Kode dari GitLab:**
    ```bash
    # Mengkloning repositori dari GitLab ke Working Directory Anda
    git clone [URL_PROYEK_DARI_GITLAB]
    ```
3.  **Membuat Perubahan:** Buka *Working Directory* Anda dan mulailah bekerja (mengedit, membuat file baru).
4.  **Memeriksa Status:** Lihat perubahan apa saja yang telah Anda buat.
    ```bash
    # Git akan memberitahu file mana yang 'modified' atau 'untracked'
    git status
    ```
5.  **Memasukkan ke Staging Area:** Pilih perubahan yang ingin Anda simpan.
    ```bash
    # Memasukkan satu file spesifik ke "keranjang belanja"
    git add nama_file.js

    # Memasukkan semua perubahan saat ini
    git add .
    ```
6.  **Menyimpan ke Repositori Lokal:** Simpan snapshot dari *Staging Area* ke "lemari arsip" lokal.
    ```bash
    # -m adalah singkatan dari message. Pesan commit sangat penting!
    git commit -m "feat: Menambahkan fungsi otentikasi pengguna"
    ```
7.  **Mengirim ke GitLab (Remote Repository):** Bagikan hasil kerja Anda ke tim.
    ```bash
    # Mengirim commit dari repositori lokal ke GitLab
    git push origin nama-branch-anda
    ```
8.  **Mengambil Perubahan dari GitLab:** Selalu update pekerjaan Anda dengan perubahan dari tim.
    ```bash
    # Mengambil dan menggabungkan perubahan terbaru dari GitLab
    git pull origin main
    ```

----
**Praktik : Membuat repo baru di gitlab**

 - A. Dengan Cara init repo dari local
   - buat repo baru, create `readme.md` dan push ke repository baru.

 - B. Clone repo ke local
   - Clone existing repo ke local pc
-----

### **3. Strategi Branching dan Merge Request (MR) 🌱🔀**

**Dasar Teori**

*Branching* (percabangan) adalah fitur paling kuat di Git. Ia memungkinkan pengembangan paralel tanpa mengganggu kode utama yang stabil.

  * **Konsep Utama:** Setiap *branch* adalah sebuah "garis waktu" atau "dunia alternatif" dari riwayat proyek Anda. *Branch* `main` dianggap sebagai sumber kebenaran (*source of truth*) yang selalu stabil dan siap rilis. Semua pekerjaan baru (fitur, perbaikan bug) harus dilakukan di *branch* terpisah.

  * **GitLab Flow:** Strategi *branching* yang sederhana dan efektif. Aturannya:

    1.  *Branch* `main` harus selalu dapat di-*deploy*.
    2.  Untuk memulai pekerjaan baru, buat *branch* baru dari `main` (misal: `feat/user-login`).
    3.  Lakukan *push* ke *branch* tersebut di GitLab dan buka *Merge Request*.
    4.  Setelah disetujui dan lolos CI, *merge* ke `main` dan langsung *deploy*.

  * **Konsep Pesan Commit (Conventional Commits):** Memberi pesan *commit* yang terstruktur bukan hanya soal kerapian, tapi juga memungkinkan otomatisasi. Strukturnya adalah `type(scope): subject`.

      * `feat`: Perubahan yang menambahkan **fitur baru**.
      * `fix`: Perubahan yang **memperbaiki bug**.
      * `chore`: Perubahan yang tidak berhubungan dengan kode produksi (misal: update skrip build, konfigurasi).
      * `docs`: Perubahan pada dokumentasi.
      * `refactor`: Mengubah kode tanpa mengubah fungsionalitasnya.
      * `test`: Menambah atau memperbaiki tes.
      * **Keuntungan:** Memungkinkan pembuatan *changelog* otomatis dan penentuan versi semantik (Semantic Versioning) secara otomatis.

**Langkah Praktik: Alur Kerja dengan Branch dan MR**

1.  Pastikan Anda berada di branch `main` dan sudah update: `git checkout main` dan `git pull origin main`.
2.  Buat *branch* baru untuk fitur Anda:
    ```bash
    git checkout -b feat/tambah-halaman-kontak
    ```
3.  Lakukan pekerjaan Anda: buat file, edit kode.
4.  *Commit* perubahan Anda dengan pesan yang baik:
    ```bash
    git add .
    git commit -m "feat: Menambahkan halaman kontak dengan formulir"
    ```
5.  *Push branch* baru Anda ke GitLab:
    ```bash
    git push -u origin feat/tambah-halaman-kontak
    ```
6.  Buka GitLab. Anda akan melihat notifikasi untuk membuat *Merge Request* (MR). Klik tombol tersebut.
7.  Isi formulir MR: beri judul yang jelas (mengikuti *conventional commit*), deskripsi, tetapkan *reviewer*, dan kirim.
8.  Tim Anda akan mereview, memberikan komentar, dan setelah disetujui, Anda (atau maintainer) bisa menekan tombol **Merge**.

-----

### **4. Studi Kasus: Menghadapi dan Menyelesaikan Merge Conflict 💥**

**Dasar Teori**

*Merge conflict* terjadi saat Git tidak bisa menggabungkan dua perubahan secara otomatis karena keduanya memodifikasi baris yang sama di file yang sama. Git akan menyerah dan berkata, "Saya tidak tahu mana yang benar. Tolong Anda sebagai manusia yang putuskan."

**Langkah Praktik: Simulasi dan Resolusi Konflik**

**Skenario:** Andi dan Budi kembali beraksi di file `index.html`.

1.  **Kondisi Awal (`main`):**

    ```html
    <h1>Selamat Datang di Perusahaan X</h1>
    <p>Kami adalah solusi untuk semua kebutuhan Anda.</p>
    ```

2.  **Pekerjaan Andi (Sudah di-merge ke `main`):** Andi mengubah judul.

      * `main` sekarang berisi:
        ```html
        <h1>Selamat Datang di Website Resmi Perusahaan X</h1>
        <p>Kami adalah solusi untuk semua kebutuhan Anda.</p>
        ```

3.  **Pekerjaan Budi (Dikerjakan sebelum menarik perubahan Andi):** Budi juga mengubah judul, tetapi dengan redaksi berbeda.

      * Di *branch* `feat/slogan-baru`, Budi mengubah `index.html` menjadi:
        ```html
        <h1>Selamat Datang! Kami Perusahaan X</h1>
        <p>Kami adalah solusi untuk semua kebutuhan Anda.</p>
        ```

4.  **Terjadinya Konflik:** Budi melakukan `git push` dan mencoba membuat MR. GitLab akan memberitahunya bahwa ada konflik dan tidak bisa di-*merge* secara otomatis.

5.  **Resolusi (di Lokal Komputer Budi):**
    a. Pindah ke branch Budi: `git checkout feat/slogan-baru`
    b. Tarik perubahan terbaru dari `main` ke branch Budi. Di sinilah Git akan melaporkan konflik.

    ```bash
    git pull origin main
    # Output akan berisi: CONFLICT (content): Merge conflict in index.html
    ```

    c. Buka `index.html`. Anda akan melihat penanda konflik:

    ```html
    <<<<<<< HEAD
    <h1>Selamat Datang! Kami Perusahaan X</h1>
    =======
    <h1>Selamat Datang di Website Resmi Perusahaan X</h1>
    >>>>>>> [hash-commit-dari-main]
    <p>Kami adalah solusi untuk semua kebutuhan Anda.</p>
    ```

      * `<<<<<<< HEAD`: Ini adalah versi Anda (Budi).
      * `=======`: Pemisah.
      * `>>>>>>> ...`: Ini adalah versi yang datang dari `main` (Andi).

    d. **Buat Keputusan:** Diskusikan dengan Andi. Misalkan, diputuskan untuk menggabungkan keduanya. Edit file secara manual, hapus penanda Git, dan buat versi final.

    ```html
    <h1>Selamat Datang di Website Resmi Perusahaan X</h1>
    <p>Kami adalah solusi untuk semua kebutuhan Anda.</p>
    ```

    e. Simpan file, lalu selesaikan proses *merge* dengan *commit* baru.

    ```bash
    git add index.html
    git commit -m "fix: Menyelesaikan konflik merge pada judul halaman"
    ```

    f. *Push* kembali ke GitLab:

    ```bash
    git push origin feat/slogan-baru
    ```

6.  **Selesai:** Periksa kembali MR Anda di GitLab. Notifikasi konflik akan hilang, dan sekarang MR siap untuk di-*merge*\!

-----

### **Tambahan Praktikum: Menggunakan SSH Key untuk Otentikasi GitLab**

Setelah Anda memahami alur kerja dasar Git, Anda akan segera menyadari bahwa mengetikkan *username* dan *password* setiap kali melakukan `git push` atau `git pull` sangat tidak efisien dan kurang aman. GitLab, seperti layanan Git lainnya, menyediakan metode otentikasi yang jauh lebih baik dan aman menggunakan **kunci SSH**.

**Dasar Teori: Apa itu SSH Key? 🔑**

SSH (*Secure Shell*) adalah protokol jaringan kriptografi untuk komunikasi data yang aman. Untuk otentikasi, SSH menggunakan sepasang kunci:

1.  **Kunci Privat (`private key`):** Kunci ini **sangat rahasia** dan hanya boleh ada di komputer Anda. Jangan pernah membagikannya kepada siapa pun. Anggap ini sebagai **kunci rumah** Anda yang sebenarnya.
2.  **Kunci Publik (`public key`):** Kunci ini dapat dibagikan secara bebas. Anggap ini sebagai **gembok** yang Anda berikan kepada layanan seperti GitLab.

**Bagaimana Cara Kerjanya?**
Saat Anda mencoba terhubung ke GitLab melalui SSH, GitLab akan "menantang" komputer Anda menggunakan kunci publik (gembok) yang Anda berikan. Komputer Anda kemudian akan menggunakan kunci privat (kunci rumah) untuk "menjawab" tantangan tersebut. Jika jawabannya cocok, GitLab tahu bahwa koneksi tersebut berasal dari Anda dan memberikan akses tanpa perlu *password*.

**Konsiderasi Keamanan 🛡️**

  * **Keamanan Unggul:** Menggunakan SSH Key jauh lebih aman daripada *password*. Kunci SSH jauh lebih kompleks dan hampir tidak mungkin ditebak (*brute-force*).
  * **Risiko Kunci Privat:** Siapa pun yang memiliki akses ke kunci privat Anda dapat mengakses repositori Anda. **Lindungi kunci privat Anda** seperti Anda melindungi *password* utama Anda. Gunakan *passphrase* (kata sandi untuk kunci itu sendiri) untuk lapisan keamanan ekstra.
  * **Manajemen:** Jika laptop Anda hilang atau dicuri, Anda harus segera **mencabut (*revoke*)** kunci publik yang terkait dari akun GitLab Anda untuk memutus akses.

-----

### **Langkah Praktik: Membuat dan Menambahkan SSH Key di GitLab**

Berikut adalah panduan lengkap, mulai dari pembuatan kunci hingga penggunaannya, sesuai dengan praktik terbaik yang direkomendasikan GitLab.

**Langkah 1: Periksa Keberadaan Kunci SSH yang Ada**

Sebelum membuat kunci baru, periksa apakah Anda sudah memilikinya. Buka terminal (Git Bash di Windows, atau Terminal di macOS/Linux) dan ketik:

```bash
ls -al ~/.ssh
```

Cari file bernama `id_ed25519.pub` atau `id_rsa.pub`. Jika salah satunya ada, Anda bisa langsung ke **Langkah 3**. Jika tidak ada atau direktori `~/.ssh` tidak ditemukan, lanjutkan ke langkah berikutnya.

**Langkah 2: Membuat SSH Key Baru (Rekomendasi: ED25519)**

GitLab merekomendasikan penggunaan algoritma **ED25519** karena lebih aman dan performanya lebih baik dibandingkan RSA yang lebih lama.

1.  Buka terminal Anda.
2.  Jalankan perintah berikut, ganti `email@anda.com` dengan email yang Anda gunakan untuk akun GitLab.
    ```bash
    ssh-keygen -t ed25519 -C "email@anda.com"
    ```
3.  Anda akan ditanya beberapa hal:
      * **"Enter a file in which to save the key..."**: Tekan **Enter** untuk menerima lokasi default (`~/.ssh/id_ed25519`).
      * **"Enter passphrase (empty for no passphrase):"**: Ini adalah langkah keamanan tambahan. Sangat **direkomendasikan** untuk mengisi *passphrase*. Ini adalah kata sandi yang harus Anda masukkan sekali per sesi saat menggunakan kunci ini. Jika tidak ingin, tekan Enter dua kali.

Setelah selesai, Anda akan memiliki dua file baru di direktori `~/.ssh/`: `id_ed25519` (kunci privat) dan `id_ed25519.pub` (kunci publik).

**Langkah 3: Menambahkan Kunci Publik ke Akun GitLab Anda**

Sekarang, Anda perlu memberikan "gembok" (kunci publik) Anda kepada GitLab.

1.  Salin isi dari file kunci publik Anda ke *clipboard*. Gunakan perintah yang sesuai dengan sistem operasi Anda:

      * **macOS:**
        ```bash
        pbcopy < ~/.ssh/id_ed25519.pub
        ```
      * **Linux (membutuhkan `xclip`):**
        ```bash
        sudo apt-get install xclip # Jika belum terinstal
        xclip -selection clipboard < ~/.ssh/id_ed25519.pub
        ```
      * **Windows (Git Bash):**
        ```bash
        cat ~/.ssh/id_ed25519.pub | clip
        ```
      * **Metode Manual:** Buka file `id_ed25519.pub` dengan editor teks (seperti Notepad atau VS Code) dan salin seluruh isinya secara manual. Isinya akan dimulai dengan `ssh-ed25519 ...`

2.  Buka **GitLab** di browser.

3.  Klik avatar profil Anda di pojok kanan atas, lalu pilih **Edit profile**.

4.  Di menu navigasi kiri, pilih **SSH Keys**.

5.  Tempel (*paste*) kunci yang sudah Anda salin ke dalam kotak teks **Key**.

6.  Beri nama yang deskriptif di kolom **Title**, misalnya "Laptop Kantor" atau "MacBook Pribadi". Ini akan membantu Anda mengidentifikasi kunci ini di kemudian hari.

7.  Biarkan **Usage type** pada `Authentication & Signing`.

8.  Klik tombol **Add key**.

**Langkah 4: Menguji Koneksi**

Untuk memastikan semuanya berfungsi, uji koneksi Anda ke GitLab melalui SSH.

```bash
ssh -T git@gitlab.com
```

  * Jika ini pertama kalinya Anda terhubung, Anda mungkin akan melihat peringatan tentang keaslian host. Ketik `yes` dan tekan Enter.
  * Jika Anda menggunakan *passphrase*, Anda akan diminta untuk memasukkannya.
  * Jika berhasil, Anda akan menerima pesan sambutan seperti: **"Welcome to GitLab, @username\!"**.

Selamat\! Anda sekarang siap untuk berinteraksi dengan repositori Anda tanpa *password*.

-----

### **Praktik: Menggunakan SSH pada Repositori yang Ada**

Jika Anda sebelumnya meng-kloning repositori menggunakan **HTTPS**, Anda perlu mengubah URL *remote*-nya agar menggunakan **SSH**.

1.  Masuk ke direktori proyek lokal Anda.
2.  Periksa URL *remote* Anda saat ini:
    ```bash
    git remote -v
    # Outputnya akan terlihat seperti:
    # origin  https://gitlab.com/username/proyek.git (fetch)
    # origin  https://gitlab.com/username/proyek.git (push)
    ```
3.  Ubah URL ke format SSH:
    ```bash
    git remote set-url origin git@gitlab.com:username/proyek.git
    ```
4.  Verifikasi lagi dengan `git remote -v`. URL sekarang seharusnya sudah berubah.
5.  Sekarang, setiap kali Anda melakukan `git pull` atau `git push`, Git akan menggunakan kunci SSH Anda secara otomatis.

-----

### **Manajemen SSH Key di GitLab**

Anda dapat mengelola semua kunci SSH Anda dari halaman **Edit profile \> SSH Keys**.

  * **Menambah Kunci Baru:** Anda bisa menambahkan beberapa kunci SSH. Misalnya, satu untuk laptop kerja dan satu lagi untuk komputer di rumah. Ulangi saja proses di atas.
  * **Mencabut Kunci (*Revoke*):** Jika sebuah kunci tidak lagi aman (misalnya laptop hilang) atau tidak lagi digunakan, Anda **harus** menghapusnya. Cukup klik ikon tempat sampah atau tombol **Revoke** di samping kunci yang relevan. Akses dari perangkat tersebut akan segera terputus.

-----
### **Referensi**

  * [Pro Git Book](https://git-scm.com/book/en/v2) (Sumber belajar Git terlengkap dan gratis)
  * [Dokumentasi Resmi GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
  * [Spesifikasi Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
  * [Tutorial Visual Git dari Atlassian](https://www.atlassian.com/git/tutorials)
  * [Dokumentasi Resmi GitLab tentang SSH Keys](https://www.google.com/search?q=https://docs.gitlab.com/ee/user/ssh.html)
