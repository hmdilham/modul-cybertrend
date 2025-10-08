### **Topik 9: Best Practices untuk Manajemen Artefak**

### **Kebijakan Retensi (*Retention Policies*)**

**Dasar Teori: Apa itu Kebijakan Retensi?**

Secara sederhana, **kebijakan retensi** adalah seperangkat aturan yang menentukan **berapa lama sebuah artefak harus disimpan** di dalam repositori Anda dan **dalam kondisi apa ia boleh dihapus**.

Tujuannya adalah untuk mencapai keseimbangan antara dua hal yang saling bertentangan:
1.  **Kebutuhan untuk Menyimpan:** Menjaga artefak rilis yang penting untuk tujuan audit, kepatuhan (*compliance*), dukungan pelanggan, dan kemampuan untuk melakukan *rollback*.
2.  **Kebutuhan untuk Membersihkan:** Mengelola biaya penyimpanan, menjaga performa sistem, dan memastikan repositori tetap rapi dan mudah dinavigasi.

Tanpa kebijakan yang jelas, Anda akan berakhir dengan salah satu dari dua ekstrem: menghapus sesuatu yang ternyata penting, atau tidak pernah menghapus apa pun hingga repositori Anda membengkak tak terkendali.

**Mengapa Kebijakan Retensi Penting?**
* **Manajemen Biaya 💰:** Penyimpanan, terutama di *cloud*, tidak gratis. Kebijakan retensi yang efektif secara langsung mengurangi biaya infrastruktur.
* **Kepatuhan dan Audit ⚖️:** Beberapa industri (seperti keuangan atau kesehatan) memiliki regulasi yang mengharuskan data dan artefak rilis disimpan untuk jangka waktu tertentu (misalnya, 7 tahun).
* **Performa Sistem 🚀:** Repositori yang lebih ramping berarti pencarian yang lebih cepat, *backup* yang lebih efisien, dan administrasi yang lebih mudah.
* **Kejelasan dan Keteraturan 🧹:** Mencegah repositori menjadi "tempat sampah digital" dan memudahkan developer menemukan artefak yang relevan dan disetujui untuk digunakan.

---

### **Strategi Umum untuk Kebijakan Retensi**

Kebijakan retensi yang baik tidak menerapkan satu aturan untuk semua. Ia harus disesuaikan berdasarkan **jenis dan siklus hidup artefak**. Mari kita hubungkan ini dengan contoh kasus NPM dan Docker kita.

#### **1. Artefak Rilis (Contoh: `aplikasi-web-sederhana:1.2.0`, `@my-org/string-utils:v1.2.0`)**

* **Kebijakan:** **Simpan Selamanya** (atau untuk jangka waktu yang sangat lama sesuai regulasi, misal: 7-10 tahun).
* **Alasan:** Ini adalah versi resmi dari produk atau *library* Anda yang telah dirilis ke pengguna atau digunakan di produksi. Artefak ini bersifat **immutable** (tidak boleh diubah) dan merupakan catatan sejarah dari evolusi produk Anda. Anda akan membutuhkannya untuk:
    * **Rollback:** Kembali ke versi stabil sebelumnya jika rilis baru bermasalah.
    * **Dukungan Pelanggan:** Menganalisis dan mereproduksi *bug* yang dilaporkan pada versi lama.
    * **Audit:** Membuktikan versi apa yang aktif pada tanggal tertentu.
* **Implementasi di Nexus:** Pastikan repositori `releases` Anda (`maven-releases`, `npm-hosted` untuk rilis, `docker-hosted` untuk rilis) **dikecualikan** dari tugas pembersihan otomatis yang agresif. Anda mungkin hanya ingin menghapus versi rilis yang sangat tua (lebih dari 5 tahun) dan sudah tidak didukung.

#### **2. Artefak Pengembangan (Contoh: Versi `SNAPSHOT`, Prarilis)**

* **Kebijakan:** **Simpan untuk Jangka Waktu Pendek** (misalnya, 30 hingga 90 hari).
* **Alasan:** Artefak `SNAPSHOT` atau prarilis (`-beta`, `-rc`) bersifat sementara. Nilainya menurun drastis setelah tidak lagi aktif dikembangkan. Menyimpannya untuk sementara waktu berguna untuk men-debug *build* harian, tetapi menyimpannya selamanya hanya akan membuang-buang ruang.
* **Implementasi di Nexus:** Ini adalah target utama dari **Cleanup Policies**. Buat kebijakan yang secara otomatis menghapus artefak di repositori `snapshots` Anda (`maven-snapshots`, atau artefak prarilis di `npm-hosted`) yang belum diubah (*updated*) atau diunduh selama, katakanlah, 30 hari.

#### **3. Artefak CI/CD (Contoh: `simple-static-website:<build_number>`)**

* **Kebijakan:** **Simpan Berdasarkan Jumlah atau Waktu yang Terbatas** (misalnya, simpan 20 *build* terakhir, atau simpan yang dibuat dalam 14 hari terakhir).
* **Alasan:** Ini adalah artefak yang dihasilkan dari setiap *build* di Jenkins. Anda membutuhkan beberapa versi terakhir untuk *deployment* ke lingkungan *staging* atau untuk *rollback* cepat dari *deployment* yang gagal. Namun, Anda sama sekali tidak membutuhkan artefak dari *build* nomor 237 yang dibuat tiga bulan lalu.
* **Implementasi di Nexus (Contoh Kasus Docker):**
    * Buat *Cleanup Policy* untuk repositori `docker-hosted` Anda.
    * Atur kriteria untuk menghapus *image* yang *tag*-nya hanya berupa angka (RegEx `^\d+$`) dan `Last downloaded`-nya lebih tua dari 14 hari.
    * Pastikan Anda **mengecualikan** *tag* rilis penting (seperti `latest` atau yang cocok dengan pola SemVer `v?\d+\.\d+\.\d+`) dari kebijakan pembersihan ini.

#### **4. Artefak yang Di-cache (*Proxy Cache*)**

* **Kebijakan:** **Simpan Berdasarkan Penggunaan** (misalnya, hapus jika tidak diakses/diunduh selama 180 hari).
* **Alasan:** Tujuan dari repositori *proxy* adalah untuk menyimpan dependensi yang sering digunakan. Jika sebuah dependensi tidak diakses selama 6 bulan, kemungkinan besar ia tidak lagi digunakan oleh proyek aktif mana pun. Menghapusnya dari *cache* akan menghemat ruang. Jika suatu saat dibutuhkan lagi, Nexus akan secara otomatis mengunduhnya kembali.
* **Implementasi di Nexus:** Buat *Cleanup Policy* yang menargetkan repositori *proxy* Anda (`maven-central-proxy`, `npm-proxy`, `docker-hub-proxy`) dengan kriteria `Last downloaded` lebih tua dari 180 hari.

---

### **Saran Tambahan (Agar Lebih Sempurna)**

* **Dokumentasikan Kebijakan Anda:** Tulis kebijakan retensi Anda di Wiki tim atau dokumentasi proyek. Setiap orang harus tahu, misalnya, bahwa artefak *SNAPSHOT* akan hilang setelah 30 hari.
* **Selalu Gunakan "Dry Run":** Sebelum mengaktifkan jadwal tugas pembersihan, jalankan dalam mode **Dry Run** beberapa kali. Ini akan menghasilkan laporan tentang apa yang *akan* dihapus tanpa benar-benar menghapusnya, memungkinkan Anda untuk memverifikasi logika kebijakan Anda.
* **Tinjau Secara Berkala:** Kebutuhan bisnis dan proyek berubah. Tinjau kembali kebijakan retensi Anda setidaknya setahun sekali untuk memastikan kebijakan tersebut masih relevan dan efektif.

Dengan menerapkan kebijakan retensi yang bijaksana, Anda mengubah repositori artefak Anda dari sekadar "tempat penyimpanan" menjadi aset yang terkelola dengan baik, efisien, dan hemat biaya.

---

### **Keamanan dan Kontrol Akses**

**Dasar Teori: Mengapa Keamanan Artefak Penting?**

Manajer repositori artefak seperti Nexus adalah jantung dari ekosistem *build* Anda. Ia menyimpan tidak hanya kekayaan intelektual Anda (artefak internal), tetapi juga bertindak sebagai gerbang untuk semua komponen pihak ketiga (dependensi eksternal).

Kontrol akses yang lemah dapat menyebabkan risiko serius:
* **Pencurian Kode:** Pengguna yang tidak sah dapat mengunduh artefak internal yang sensitif.
* **Sabotase Rilis:** Artefak rilis yang stabil dapat ditimpa atau dihapus, baik secara sengaja maupun tidak sengaja.
* **Injeksi Komponen Berbahaya:** Pengguna jahat dapat mengunggah versi dependensi yang telah dimodifikasi dan berisi *malware* ke repositori internal Anda.

Oleh karena itu, menerapkan model keamanan yang kuat adalah praktik terbaik yang tidak bisa ditawar.

---

### **Model Keamanan di Nexus: Authentication & Authorization**

Keamanan di Nexus dibangun di atas dua konsep fundamental:

1.  **Authentication (Siapa Anda?):** Proses memverifikasi identitas pengguna atau sistem (seperti Jenkins). Nexus harus tahu siapa yang membuat permintaan. Ini biasanya dilakukan melalui *username* dan *password* atau token.
2.  **Authorization (Apa yang Boleh Anda Lakukan?):** Setelah identitas diverifikasi, proses ini menentukan tindakan apa saja yang diizinkan untuk pengguna tersebut. Apakah mereka boleh membaca, menulis, atau menghapus dari repositori tertentu?

Nexus mengelola otorisasi melalui model tiga lapis yang sangat fleksibel: **Privileges**, **Roles**, dan **Users**.



#### **1. Privileges (Hak Istimewa)**

* **Apa itu?** Ini adalah unit izin terkecil dan paling spesifik di Nexus. Sebuah *privilege* mendefinisikan satu tindakan tunggal pada satu target tunggal.
* **Contoh:**
    * `nx-repository-view-maven-releases-read`: Izin untuk **membaca/mengunduh** dari repositori `maven-releases`.
    * `nx-repository-view-npm-hosted-add`: Izin untuk **menambah/mengunggah** ke repositori `npm-hosted`.
    * `nx-repository-view-docker-hosted-delete`: Izin untuk **menghapus** dari repositori `docker-hosted`.
    * `nx-users-create`: Izin untuk membuat pengguna baru.

Anda jarang memberikan *privilege* secara langsung ke pengguna. Sebaliknya, Anda mengumpulkannya ke dalam *Roles*.

#### **2. Roles (Peran)**

* **Apa itu?** Sebuah *Role* adalah kumpulan dari satu atau lebih *Privileges*. Anda membuat peran berdasarkan fungsi pekerjaan atau tugas.
* **Contoh:**
    * **`developer-role`:** Mungkin memiliki *privilege* untuk membaca dari semua repositori, tetapi hanya bisa menulis ke repositori `snapshots`.
    * **`ci-role`:** Peran untuk Jenkins. Memiliki *privilege* untuk membaca dari semua repositori dan menulis ke semua repositori `hosted` (`releases` dan `snapshots`).
    * **`qa-role`:** Mungkin hanya memiliki *privilege* untuk membaca dari repositori `releases` untuk mengambil artefak yang akan diuji.

Dengan menggunakan *Roles*, Anda dapat mengelola izin secara efisien. Jika kebijakan berubah, Anda cukup mengedit *Role*, dan semua pengguna yang memiliki *Role* tersebut akan otomatis mendapatkan pembaruan izin.

#### **3. Users (Pengguna)**

* **Apa itu?** Ini adalah akun untuk individu (developer, admin) atau sistem (Jenkins, Docker client) yang perlu berinteraksi dengan Nexus.
* **Cara Kerja:** Anda memberikan satu atau lebih **Roles** kepada sebuah **User**. Kumpulan semua *privilege* dari semua *role* yang dimiliki pengguna tersebut menentukan apa yang boleh ia lakukan.

---

### **Praktik Terbaik untuk Kontrol Akses**

Berikut adalah beberapa praktik terbaik yang harus Anda terapkan.

**1. Terapkan Prinsip Hak Istimewa Terendah (*Principle of Least Privilege*)**
Berikan pengguna atau sistem hanya izin minimum yang benar-benar mereka butuhkan untuk melakukan pekerjaan mereka.
* **Developer:** Tidak memerlukan izin untuk menghapus artefak rilis atau mengubah konfigurasi sistem. Beri mereka akses tulis hanya ke repositori pengembangan (`snapshots`).
* **Jenkins (Akun Layanan):** Tidak memerlukan izin untuk membuat pengguna baru atau mengubah konfigurasi keamanan.

**2. Pisahkan Akun Pengguna dan Akun Layanan**
Jangan pernah menggunakan akun pribadi developer untuk otomatisasi CI/CD. Buatlah akun layanan khusus untuk Jenkins (misalnya, `jenkins-ci-user`).
* **Keuntungan:**
    * **Audit:** Jejak audit menjadi jelas. Anda tahu persis tindakan mana yang dilakukan oleh Jenkins versus oleh seorang developer.
    * **Keamanan:** Jika kredensial Jenkins bocor, dampaknya terbatas pada izin yang dimiliki oleh akun layanan tersebut, tidak mengekspos akun pribadi.

**3. Nonaktifkan Akses Anonim**
Seperti yang dibahas saat instalasi, menonaktifkan akses anonim adalah langkah pertama yang mudah untuk mengamankan repositori Anda. Ini memaksa semua permintaan untuk diautentikasi, memastikan setiap tindakan dapat dilacak ke pengguna atau layanan tertentu.

**4. Gunakan Peran (*Roles*) Secara Efektif**
Rancang peran Anda berdasarkan fungsi tim.
* **Contoh Kasus: Mengatur Peran untuk Jenkins dan Developer**
    * **Buat Peran `developer-role` dengan *privileges*:**
        * `read` pada `maven-public-group`, `npm-group`, `docker-group`.
        * `read` dan `write` pada `maven-snapshots`.
    * **Buat Peran `jenkins-ci-role` dengan *privileges*:**
        * `read` pada semua repositori `group`.
        * `read`, `write`, dan `delete` (untuk menimpa *snapshots*) pada semua repositori `hosted` (`releases` dan `snapshots`).
    * **Buat Pengguna:**
        * Buat pengguna `ilham` dan berikan `developer-role`.
        * Buat pengguna `jenkins-ci-user` dan berikan `jenkins-ci-role`.

**5. (Lanjutan) Gunakan *Content Selectors***
Untuk kontrol yang lebih granula, Nexus Pro menawarkan fitur *Content Selectors*. Ini memungkinkan Anda membatasi akses tidak hanya pada level repositori, tetapi juga pada **path atau namespace di dalam repositori**.
* **Contoh:** Anda dapat membuat *privilege* yang hanya mengizinkan tim *frontend* untuk mempublikasikan paket di bawah *scope* `@frontend/*` di dalam repositori `npm-hosted` yang sama.

Dengan menerapkan praktik keamanan dan kontrol akses ini, Anda mengubah Nexus dari sekadar tempat penyimpanan menjadi benteng yang aman untuk semua aset perangkat lunak Anda.

-----

### **Mempromosikan Artefak Antar Lingkungan**

**Dasar Teori: Apa itu "Promosi Artefak"?**

**Promosi artefak** adalah proses yang terkontrol untuk memindahkan sebuah versi artefak yang spesifik dan telah terverifikasi dari satu lingkungan (atau tahap siklus hidup) ke lingkungan berikutnya yang lebih tinggi. Contohnya, dari lingkungan `Staging` ke `Production`.

Ini adalah praktik inti dari **Continuous Delivery** dan didasarkan pada prinsip fundamental: **"Build Once, Deploy Many"** (Bangun Sekali, Deploy Berkali-kali).

**Mengapa Menerapkan Prinsip "Build Once, Deploy Many"?**

  * **Konsistensi:** Anda memastikan bahwa *binary* (file `.jar`, `.war`, atau *Docker image*) yang Anda uji di lingkungan QA adalah **persis sama** dengan yang Anda *deploy* ke produksi. Ini menghilangkan risiko munculnya *bug* baru akibat kompilasi ulang atau perbedaan kecil pada proses *build*.
  * **Kepercayaan (*Confidence*):** Proses ini memberikan keyakinan tinggi bahwa apa yang telah lulus pengujian adalah apa yang akan diterima oleh pengguna akhir. Tidak ada "kejutan" di menit terakhir.
  * **Keterlacakan (*Traceability*):** Menciptakan jejak audit yang jelas. Anda dapat dengan pasti melacak bahwa "versi `2.1.5` telah diuji, disetujui pada tanggal X oleh Tim QA, dan dipromosikan ke produksi pada tanggal Y."

-----

### **Bagaimana Cara Kerja Promosi Artefak dengan Nexus dan Jenkins?**

Promosi **bukan berarti mengunggah ulang atau membangun ulang** artefak. Sebaliknya, ini adalah proses mengubah status atau lokasi logis dari artefak yang sudah ada di Nexus. Ada dua pola umum untuk mengimplementasikannya.

#### **Pola 1: Menggunakan Repositori Terpisah untuk Setiap Lingkungan (Paling Umum)**

Ini adalah pendekatan yang paling jelas dan mudah dikelola, terutama untuk artefak seperti Maven atau NPM.

**Konsep:**
Anda membuat repositori `hosted` yang berbeda di Nexus untuk setiap lingkungan kunci.

  * `maven-staging-candidates`: Tempat Jenkins menaruh artefak yang siap untuk diuji di *Staging*.
  * `maven-production-releases`: Tempat artefak yang **sudah lulus QA** dipindahkan. Repositori ini harus sangat terkunci dan terkontrol.

**Alur Kerja Praktis:**

1.  **Build Awal:** *Pipeline* CI (`build-and-test`) membangun artefak (misal: `my-app-1.2.0.jar`) dan mempublikasikannya ke repositori `maven-staging-candidates`.
2.  **Deployment ke Staging:** Proses *deployment* ke server *Staging* dikonfigurasi untuk menarik artefak **hanya** dari `maven-staging-candidates`.
3.  **Verifikasi QA:** Tim QA melakukan pengujian menyeluruh terhadap aplikasi di lingkungan *Staging*.
4.  **Proses Promosi (Setelah Disetujui):**
      * Seorang manajer rilis atau QA lead memicu sebuah *job* Jenkins khusus bernama `promote-to-production`.
      * *Job* ini **tidak mengambil kode sumber atau membangun ulang**.
      * Tugasnya adalah menggunakan API Nexus (melalui *plugin* atau skrip `curl`) untuk **menyalin** artefak `my-app-1.2.0.jar` dari `maven-staging-candidates` ke `maven-production-releases`.
5.  **Deployment ke Produksi:** Proses *deployment* ke server *Production* dikonfigurasi untuk menarik artefak **hanya** dari `maven-production-releases`.

**Contoh Jenkinsfile untuk Job Promosi (`promote-to-production`):**

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'ARTIFACT_VERSION', description: 'Versi artefak yang akan dipromosikan (misal: 1.2.0)')
    }

    stages {
        stage('Promote in Nexus') {
            steps {
                script {
                    echo "Mempromosikan versi ${params.ARTIFACT_VERSION} ke Produksi..."
                    // Contoh menggunakan skrip shell dengan curl untuk memanggil Nexus API
                    // Kredensial harus dikelola dengan Jenkins Credentials
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh """
                        curl -u ${NEXUS_USER}:${NEXUS_PASS} -X POST \
                        'http://<ip_nexus_anda>:8081/service/rest/v1/components?repository=maven-production-releases' \
                        -H 'Content-Type: application/json' \
                        -d '{ "maven2": { "groupId": "com.mycompany", "artifactId": "my-app", "version": "${params.ARTIFACT_VERSION}", "extension": "jar" } }'
                        """
                        // Catatan: Sintaks API Nexus yang lebih modern mungkin diperlukan.
                        // Alternatif yang lebih baik adalah menggunakan plugin seperti Nexus Platform Plugin.
                    }
                }
            }
        }
    }
}
```

#### **Pola 2: Menggunakan Penandaan (*Tagging*) (Umum untuk Docker)**

Untuk Docker, memindahkan *image* antar repositori kurang umum. Sebaliknya, promosi dilakukan dengan **menambahkan *tag* baru** pada *image* yang sudah ada.

**Konsep:**
Sebuah *image* dianggap "dipromosikan" ketika ia diberi *tag* yang stabil dan diakui sebagai rilis produksi.

**Alur Kerja Praktis:**

1.  **Build Awal:** *Pipeline* CI membangun *image* dan mendorongnya ke `docker-hosted` dengan *tag* yang unik dan tidak stabil, seperti nomor *build* Jenkins (misal: `my-app:125`).
2.  **Deployment ke Staging:** Server *Staging* men-*deploy* `my-app:125`.
3.  **Verifikasi QA:** Tim QA menguji aplikasi di *Staging*.
4.  **Proses Promosi (Setelah Disetujui):**
      * *Job* `promote-docker-image` di Jenkins dipicu.
      * *Job* ini melakukan tiga hal:
        a. Menarik (*pull*) *image* spesifik dari Nexus: `docker pull <nexus_url>/my-app:125`.
        b. Memberi *tag* baru yang stabil: `docker tag <nexus_url>/my-app:125 <nexus_url>/my-app:1.2.0`.
        c. Mendorong (*push*) **hanya *tag* baru tersebut** kembali ke Nexus: `docker push <nexus_url>/my-app:1.2.0`.
5.  **Deployment ke Produksi:** Proses *deployment* ke *Production* dikonfigurasi untuk menarik *tag* yang stabil, seperti `my-app:1.2.0`.

**Contoh Jenkinsfile untuk Job Promosi Docker:**

```groovy
pipeline {
    agent { label 'docker-agent' }

    parameters {
        string(name: 'BUILD_TAG', description: 'Tag build yang akan dipromosikan (misal: 125)')
        string(name: 'RELEASE_TAG', description: 'Tag rilis baru (misal: 1.2.0)')
    }

    environment {
        DOCKER_REGISTRY_URL    = '<ip_nexus_anda>:8082'
        DOCKER_CREDENTIALS     = 'nexus-docker-credentials'
        IMAGE_NAME             = 'my-app'
    }

    stages {
        stage('Promote Docker Image') {
            steps {
                script {
                    docker.withRegistry("http://${DOCKER_REGISTRY_URL}", DOCKER_CREDENTIALS) {
                        echo "Menarik image ${IMAGE_NAME}:${params.BUILD_TAG}"
                        sh "docker pull ${DOCKER_REGISTRY_URL}/${IMAGE_NAME}:${params.BUILD_TAG}"

                        echo "Memberi tag baru: ${IMAGE_NAME}:${params.RELEASE_TAG}"
                        sh "docker tag ${DOCKER_REGISTRY_URL}/${IMAGE_NAME}:${params.BUILD_TAG} ${DOCKER_REGISTRY_URL}/${IMAGE_NAME}:${params.RELEASE_TAG}"
                        
                        echo "Mendorong tag rilis baru ke Nexus"
                        sh "docker push ${DOCKER_REGISTRY_URL}/${IMAGE_NAME}:${params.RELEASE_TAG}"
                    }
                }
            }
        }
    }
}
```

-----

### **Catatan Penting dan Praktik Terbaik**

  * **Promosi ke Produksi Seringkali Manual:** Meskipun prosesnya otomatis, pemicu (*trigger*) untuk promosi ke produksi seringkali merupakan tindakan manual (klik tombol di Jenkins) untuk memastikan ada persetujuan dan pengawasan manusia.
  * **Kontrol Akses adalah Kunci:** Pastikan akun layanan Jenkins memiliki izin tulis ke repositori `production-releases`, tetapi akun developer biasa hanya memiliki izin baca. Ini menegakkan alur kerja promosi yang terkontrol.
  * **Log dan Lacak Semuanya:** *Job* promosi harus mencatat dengan jelas artefak versi mana yang dipromosikan, oleh siapa, dan kapan, untuk tujuan audit.

---

### **Referensi**
* [Dokumentasi Resmi Nexus tentang Keamanan](https://help.sonatype.com/repomanager3/security)