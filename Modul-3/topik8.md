### **Topik 8: Mempublikasikan dan Mengelola Artefak**

### **Integrasi Maven dan NPM dengan Nexus**

**Pengantar: Menghubungkan Dunia Nyata ke Nexus**

Kita sudah berhasil menginstal dan mengonfigurasi repositori `hosted`, `proxy`, dan `group` di Nexus. Sekarang, bagaimana cara memberitahu *tools* kita (Maven untuk Java, NPM untuk Node.js) untuk menggunakan Nexus sebagai pusat artefak, bukan lagi repositori publik di internet?

Proses ini melibatkan konfigurasi di dua tempat utama:

1.  **Sisi Klien (Lokal Developer):** Agar setiap developer di tim menggunakan sumber yang sama.
2.  **Sisi CI/CD (Jenkins):** Agar proses *build* otomatis kita juga menggunakan Nexus.

-----

### **A. Integrasi Maven (Java) dengan Nexus ☕**

Tujuannya adalah memaksa Maven untuk **mengunduh semua dependensi** dari `maven-public-group` kita dan **mengunggah semua artefak** ke `maven-releases` atau `maven-snapshots`. Ini semua diatur melalui file `settings.xml`.

#### **1. Konfigurasi Sisi Klien (File `settings.xml`)**

File ini biasanya terletak di `~/.m2/settings.xml` (di direktori *home* pengguna). Jika belum ada, Anda bisa membuatnya.

**Langkah 1: Konfigurasi Kredensial Server**
Pertama, kita perlu memberitahu Maven *username* dan *password* untuk bisa mengunggah artefak ke repositori *hosted* kita.

Tambahkan blok `<servers>` ini ke dalam `settings.xml` Anda.

```xml
<settings>
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>password_admin_nexus</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>password_admin_nexus</password>
    </server>
  </servers>
  </settings>
```

  * **`<id>`:** Ini adalah ID unik yang akan kita referensikan nanti di file `pom.xml` proyek.

**Langkah 2: Konfigurasi Mirror (Untuk Mengunduh)**
Selanjutnya, kita akan "memaksa" semua permintaan unduhan untuk melewati Nexus. Ini dilakukan dengan mendefinisikan sebuah *mirror*.

Tambahkan blok `<mirrors>` ini ke dalam `settings.xml` Anda.

```xml
<settings>
  ...
  <mirrors>
    <mirror>
      <id>nexus-mirror</id>
      <mirrorOf>*</mirrorOf>
      <url>http://<ip_nexus_anda>:8081/repository/maven-public-group/</url>
    </mirror>
  </mirrors>
  ...
</settings>
```

**Langkah 3: Konfigurasi Proyek (`pom.xml` untuk Mengunggah)**
Terakhir, di setiap proyek Maven yang ingin Anda publikasikan ke Nexus, Anda perlu menambahkan blok `<distributionManagement>` di dalam file `pom.xml`-nya.

```xml
<project>
  ...
  <distributionManagement>
    <repository>
      <id>nexus-releases</id> <url>http://<ip_nexus_anda>:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
      <id>nexus-snapshots</id> <url>http://<ip_nexus_anda>:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
  ...
</project>
```

Sekarang, saat Anda menjalankan `mvn deploy`, Maven akan secara otomatis mengirim artefak rilis ke `maven-releases` dan artefak *snapshot* ke `maven-snapshots`.

#### **2. Konfigurasi Sisi CI/CD (Jenkins)**

Di Jenkins, kita tidak boleh menyimpan *password* di file teks. Kita akan menggunakan Jenkins Credentials Manager.

**Langkah 1: Simpan Kredensial Nexus**

1.  Di Jenkins, pergi ke **Manage Jenkins \> Credentials**.
2.  Tambah kredensial baru dengan tipe **Username with password**.
3.  Masukkan *username* dan *password* Nexus Anda.
4.  Beri ID yang mudah diingat, misalnya `nexus-credentials`.

**Langkah 2: Gunakan Kredensial di Jenkinsfile**
Di dalam `Jenkinsfile`, kita bisa menggunakan *plugin* seperti `Config File Provider` untuk mengelola `settings.xml` atau menuliskannya secara dinamis. Cara paling umum adalah menggunakan *wrapper* `withMaven` atau `withCredentials`.

```groovy
// Contoh Stage di Jenkinsfile untuk mempublikasikan ke Nexus
stage('Publish to Nexus') {
    steps {
        // Menggunakan wrapper 'withMaven' untuk integrasi yang lebih dalam
        withMaven(mavenSettingsConfig: 'nama-config-settings-xml-anda') {
            // Jenkins akan secara otomatis menyediakan kredensial jika server ID di settings.xml cocok
            sh 'mvn deploy'
        }

        // Alternatif: Menggunakan 'withCredentials'
        withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
            sh "mvn deploy -s settings.xml -DskipTests -DaltDeploymentRepository=nexus-releases::default::http://<ip_nexus_anda>:8081/repository/maven-releases/ -DrepositoryId=nexus-releases"
        }
    }
}
```

-----

### **B. Integrasi NPM (Node.js) dengan Nexus**

Tujuannya sama: mengarahkan semua perintah `npm` (seperti `install` dan `publish`) untuk melewati `npm-group` kita.

#### **1. Konfigurasi Sisi Klien (File `.npmrc`)**

Konfigurasi NPM lebih sederhana dan biasanya dilakukan melalui *command line*, yang akan memodifikasi file `~/.npmrc`.

**Langkah 1: Atur Registry untuk Mengunduh**
Buka terminal dan jalankan perintah ini. Ini memberitahu NPM untuk selamanya menggunakan Nexus sebagai *registry*-nya.

```bash
npm config set registry http://<ip_nexus_anda>:8081/repository/npm-group/
```

Setelah ini, setiap kali Anda menjalankan `npm install`, paket akan diambil melalui Nexus.

**Langkah 2: Login untuk Mengunggah**
Untuk bisa mempublikasikan paket ke repositori *hosted*, Anda perlu login.

1.  Pastikan **npm Bearer Token Realm** aktif di Nexus (**Administration \> Security \> Realms**).
2.  Jalankan perintah login di terminal:
    ```bash
    npm login
    ```
3.  Masukkan *username* (`admin`), *password*, dan email Anda saat diminta.

Setelah berhasil, NPM akan menyimpan token otentikasi di file `.npmrc` Anda. Sekarang Anda siap untuk `npm publish`.

#### **2. Konfigurasi Sisi CI/CD (Jenkins)**

Lagi-lagi, kita akan menggunakan Jenkins Credentials Manager.

**Langkah 1: Simpan Kredensial NPM**
Token otentikasi NPM adalah cara terbaik untuk login di lingkungan CI.

1.  Lihat file `~/.npmrc` Anda setelah login. Anda akan menemukan baris yang terlihat seperti `//<ip_nexus_anda>:8081/repository/npm-group/:_authToken="NpmToken.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`.
2.  Salin nilai `_authToken` (termasuk "NpmToken...").
3.  Di Jenkins, buat kredensial baru dengan tipe **Secret text**.
4.  Tempel token tersebut di kolom **Secret**.
5.  Beri ID, misalnya `nexus-npm-token`.

**Langkah 2: Gunakan Kredensial di Jenkinsfile**
Cara terbaik di Jenkins adalah membuat file `.npmrc` sementara khusus untuk proses *build*.

```groovy
// Contoh Stage di Jenkinsfile untuk mempublikasikan ke Nexus
stage('Publish to Nexus') {
    steps {
        // Gunakan wrapper 'withCredentials' untuk menyuntikkan token secara aman
        withCredentials([string(credentialsId: 'nexus-npm-token', variable: 'NEXUS_NPM_TOKEN')]) {
            script {
                try {
                    // Buat file .npmrc sementara dengan token
                    sh 'echo "//<ip_nexus_anda>:8081/repository/npm-group/:_authToken=${NEXUS_NPM_TOKEN}" > .npmrc'
                    sh 'echo "registry=http://<ip_nexus_anda>:8081/repository/npm-group/" >> .npmrc'

                    // Jalankan perintah publish
                    sh 'npm publish'
                } finally {
                    // Selalu hapus file .npmrc setelah selesai untuk kebersihan
                    sh 'rm .npmrc'
                }
            }
        }
    }
}
```

Pendekatan ini memastikan bahwa setiap *build* di Jenkins terotentikasi dengan benar ke Nexus tanpa harus menyimpan *password* atau token secara terbuka.

-----

### **Contoh Kasus: Integrasi Aplikasi Web (NPM & Docker) ke Nexus**

**Skenario Proyek**

Kita memiliki aplikasi web sederhana menggunakan **Express.js** (sebuah framework Node.js). Aplikasi ini akan di-*containerize* menggunakan **Docker** dengan server web **Nginx**.

**Struktur Proyek di GitLab:**

```
/
├── Dockerfile
├── index.js
├── package.json
└── public/
    └── index.html
```

**Kode Sumber:**

  * **`package.json`**: Mendefinisikan proyek dan dependensinya.

    ```json
    {
      "name": "aplikasi-web-sederhana",
      "version": "1.0.0",
      "description": "Contoh aplikasi web untuk integrasi Nexus",
      "main": "index.js",
      "scripts": {
        "start": "node index.js"
      },
      "dependencies": {
        "express": "^4.18.2"
      }
    }
    ```

  * **`index.js`**: Server Express sederhana (untuk tujuan *packaging* npm).

    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;
    app.use(express.static('public'));
    app.listen(port, () => {
      console.log(`Aplikasi berjalan di http://localhost:${port}`);
    });
    ```

  * **`public/index.html`**: Halaman web statis sederhana.

    ```html
    <!DOCTYPE html>
    <html>
    <head><title>Aplikasi Web Sederhana</title></head>
    <body><h1>Selamat Datang di Aplikasi Terintegrasi Nexus!</h1></body>
    </html>
    ```

  * **`Dockerfile`**: Resep untuk membangun *image* produksi.

    ```dockerfile
    # Stage 1: Build/Install Dependencies (jika ada build step seperti React/Angular)
    # Untuk contoh ini, kita hanya menyalin file.
    FROM node:18-alpine AS builder
    WORKDIR /app
    COPY package*.json ./
    COPY . .

    # Stage 2: Production Image
    FROM nginx:1.25-alpine
    # Salin file web statis dari stage sebelumnya (atau dari direktori lokal)
    COPY --from=builder /app/public /usr/share/nginx/html
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]
    ```

-----

**Tujuan Akhir Pipeline**

Dari satu `git push`, Jenkins akan menjalankan *pipeline* yang menghasilkan **dua artefak berbeda** dan menyimpannya di Nexus:

1.  **Paket NPM (`.tgz`):** Versi *package* dari kode sumber aplikasi kita akan dipublikasikan ke repositori **`npm-hosted`**. Ini berguna jika aplikasi ini juga berfungsi sebagai *library* untuk proyek lain.
2.  **Docker Image:** Sebuah *image* Nginx yang siap jalan dan berisi aplikasi kita akan didorong (*push*) ke repositori **`docker-hosted`**.

-----

### **Konfigurasi `Jenkinsfile` Terpadu**

Ini adalah inti dari studi kasus kita. `Jenkinsfile` ini akan melakukan semua tugas secara berurutan.

```groovy
pipeline {
    agent { label 'docker-agent' } // Agent dengan Docker terinstal

    environment {
        // --- Konfigurasi NPM ---
        NPM_REGISTRY_URL         = "http://<ip_nexus_anda>:8081/repository/npm-group/"
        NPM_HOSTED_URL         = "http://<ip_nexus_anda>:8081/repository/npm-hosted/"
        NPM_CREDENTIALS        = 'nexus-npm-token' // ID Kredensial 'Secret text'

        // --- Konfigurasi Docker ---
        DOCKER_REGISTRY_URL    = '<ip_nexus_anda>:8082' // Port docker-hosted
        DOCKER_CREDENTIALS     = 'nexus-docker-credentials' // ID Kredensial 'Username with password'
        IMAGE_NAME             = "aplikasi-web-sederhana"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'git@gitlab.com:username/proyek-anda.git'
            }
        }

        stage('Install Dependencies via Nexus') {
            steps {
                echo "Menginstal dependensi NPM dari Nexus Proxy..."
                // Menggunakan registry group untuk menarik dependensi
                sh "npm config set registry ${NPM_REGISTRY_URL}"
                sh "npm install"
            }
        }

        stage('Publish NPM Package to Nexus') {
            steps {
                withCredentials([string(credentialsId: NPM_CREDENTIALS, variable: 'NEXUS_NPM_TOKEN')]) {
                    script {
                        try {
                            // Membuat file .npmrc sementara dengan token untuk PUSH
                            // Menunjuk langsung ke hosted repo untuk publish
                            sh 'echo "//<ip_nexus_anda>:8081/repository/npm-hosted/:_authToken=${NEXUS_NPM_TOKEN}" > .npmrc'
                            
                            // Menambahkan versi build Jenkins ke package.json untuk keunikan
                            sh "npm version 1.0.${BUILD_NUMBER}"

                            echo "Mempublikasikan paket NPM ke Nexus Hosted..."
                            sh "npm publish --registry ${NPM_HOSTED_URL}"
                        } finally {
                            sh 'rm -f .npmrc' // Selalu bersihkan
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Membangun Docker image: ${IMAGE_NAME}"
                    def customImage = docker.build(IMAGE_NAME)
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                script {
                    docker.withRegistry("http://${DOCKER_REGISTRY_URL}", DOCKER_CREDENTIALS) {
                        def imageTag = "1.0.${BUILD_NUMBER}"
                        
                        echo "Memberi tag pada image: ${DOCKER_REGISTRY_URL}/${IMAGE_NAME}:${imageTag}"
                        customImage.push(imageTag)
                        customImage.push("latest")
                    }
                }
            }
        }
    }
}
```

-----

### **Verifikasi di Nexus**

Setelah *pipeline* berjalan sukses, Anda dapat memverifikasi bahwa kedua artefak telah dipublikasikan dengan benar.

1.  **Verifikasi Paket NPM:**

      * Buka Nexus di browser.
      * Navigasi ke **Browse \> `npm-hosted`**.
      * Anda akan melihat paket **`aplikasi-web-sederhana`** terdaftar. Jika Anda mengkliknya, Anda akan melihat versi yang sesuai dengan nomor *build* Jenkins (misalnya: `1.0.1`, `1.0.2`, dst).

2.  **Verifikasi Docker Image:**

      * Kembali ke menu **Browse**.
      * Pilih repositori **`docker-hosted`**.
      * Anda akan melihat *image* **`aplikasi-web-sederhana`** terdaftar. Jika Anda mengkliknya, Anda akan melihat *tag* yang sesuai (misalnya, `1.0.1` dan `latest`).

**Kesimpulan Alur Kerja**

Melalui satu `Jenkinsfile` terpadu, kita telah mencapai alur kerja CI yang lengkap:

1.  **Kode dipush ke GitLab.**
2.  **Jenkins mengambil kode.**
3.  **Dependensi ditarik** melalui `npm-proxy` di Nexus.
4.  **Paket NPM dipublikasikan** ke `npm-hosted` di Nexus.
5.  **Docker image dibangun.**
6.  **Docker image dipublikasikan** ke `docker-hosted` di Nexus.

Dengan ini, Anda memiliki satu sumber kebenaran yang terpusat di Nexus untuk semua jenis artefak, siap untuk ditarik oleh proses *deployment* selanjutnya.


-----

### **Mengunggah Artefak Melalui Pipeline CI**

Mengunggah artefak secara otomatis melalui *pipeline* CI adalah jantung dari praktik *Continuous Integration*. Proses ini memastikan bahwa setiap *build* yang berhasil menghasilkan artefak yang langsung disimpan, diberi versi, dan tersedia di lokasi terpusat (Nexus). Ini menghilangkan kebutuhan intervensi manual dan memastikan konsistensi.

Kunci dari proses ini adalah `Jenkinsfile` yang dikonfigurasi dengan benar untuk:

1.  **Mengautentikasi** dirinya ke Nexus secara aman menggunakan Jenkins Credentials.
2.  **Menjalankan perintah** yang sesuai untuk mempublikasikan artefak (*`npm publish`*, *`docker push`*, *`mvn deploy`*).

Mari kita lihat dua contoh kasus sederhana.

-----

### **Contoh Kasus 1: Mengunggah Paket NPM Sederhana**

**Skenario:** Kita memiliki sebuah proyek *library* JavaScript kecil yang berisi satu fungsi utilitas. Tujuannya adalah setiap kali kita membuat *tag* Git baru (misalnya `v1.0.1`), Jenkins akan secara otomatis mempublikasikan versi baru dari paket NPM ini ke Nexus.

**Struktur Proyek:**

```
/
├── index.js
├── package.json
└── Jenkinsfile
```

  * **`package.json`**:

    ```json
    {
      "name": "@my-org/string-utils",
      "version": "1.0.0",
      "description": "Utility library for strings.",
      "main": "index.js"
    }
    ```

  * **`Jenkinsfile`**:

    ```groovy
    pipeline {
        agent any

        environment {
            NPM_HOSTED_URL   = "http://<ip_nexus_anda>:8081/repository/npm-hosted/"
            NPM_CREDENTIALS  = 'nexus-npm-token' // ID Kredensial 'Secret text'
        }

        stages {
            stage('Setup') {
                steps {
                    script {
                        // Membaca versi dari package.json untuk digunakan nanti
                        def packageJSON = readJSON file: 'package.json'
                        env.PACKAGE_VERSION = packageJSON.version
                        echo "Mempersiapkan untuk mempublikasikan paket: ${packageJSON.name} versi ${env.PACKAGE_VERSION}"
                    }
                }
            }
            stage('Install & Test') {
                steps {
                    // Tarik dependensi melalui proxy Nexus
                    sh 'npm install --registry http://<ip_nexus_anda>:8081/repository/npm-group/'
                    // Jalankan tes jika ada
                    // sh 'npm test'
                }
            }
            stage('Publish to Nexus') {
                // Hanya jalankan stage ini jika build dipicu oleh tag Git
                when {
                    tag "v*"
                }
                steps {
                    withCredentials([string(credentialsId: NPM_CREDENTIALS, variable: 'NEXUS_NPM_TOKEN')]) {
                        script {
                            try {
                                echo "Mempublikasikan versi ${env.PACKAGE_VERSION} ke Nexus..."
                                sh 'echo "//<ip_nexus_anda>:8081/repository/npm-hosted/:_authToken=${NEXUS_NPM_TOKEN}" > .npmrc'
                                sh "npm publish --registry ${NPM_HOSTED_URL}"
                            } finally {
                                sh 'rm -f .npmrc'
                            }
                        }
                    }
                }
            }
        }
    }
    ```

**Alur Kerja:**

1.  Developer selesai mengerjakan fitur, menaikkan versi di `package.json` ke `1.0.1`.
2.  Developer membuat *tag* Git: `git tag v1.0.1` dan `git push --tags`.
3.  Jenkins mendeteksi *tag* baru dan menjalankan *pipeline*.
4.  Karena *build* ini memiliki *tag*, *stage* "Publish to Nexus" akan dieksekusi, dan paket versi `1.0.1` akan diunggah ke repositori `npm-hosted`.

**Verifikasi di Nexus:**
Buka **Browse \> `npm-hosted`** di Nexus. Anda akan melihat paket **`@my-org/string-utils`** dengan versi `1.0.1`.

-----

### **Contoh Kasus 2: Mengunggah Docker Image Aplikasi Statis**

**Skenario:** Kita memiliki aplikasi web yang sangat sederhana, hanya satu file HTML. Tujuannya adalah setiap kali ada *push* ke *branch* `main`, Jenkins akan membangun *image* Nginx yang berisi file HTML ini dan mendorongnya ke *registry* Docker di Nexus.

**Struktur Proyek:**

```
/
├── index.html
├── Dockerfile
└── Jenkinsfile
```

  * **`Dockerfile`**:

    ```dockerfile
    FROM nginx:1.25-alpine
    COPY ./index.html /usr/share/nginx/html/index.html
    ```

  * **`Jenkinsfile`**:

    ```groovy
    pipeline {
        agent { label 'docker-agent' } // Agent harus memiliki Docker

        environment {
            DOCKER_REGISTRY_URL    = '<ip_nexus_anda>:8082' // Port docker-hosted
            DOCKER_CREDENTIALS     = 'nexus-docker-credentials'
            IMAGE_NAME             = "simple-static-website"
        }

        stages {
            stage('Build and Push Image') {
                steps {
                    script {
                        // Membangun image Docker dari Dockerfile
                        def customImage = docker.build(IMAGE_NAME)

                        // Login ke registry Nexus dan push image
                        docker.withRegistry("http://${DOCKER_REGISTRY_URL}", DOCKER_CREDENTIALS) {
                            
                            // Memberi tag dengan nomor build Jenkins untuk keunikan
                            def imageTag = "${BUILD_NUMBER}"
                            customImage.push(imageTag)

                            echo "Image berhasil di-push ke Nexus dengan tag: ${imageTag}"
                        }
                    }
                }
            }
        }
    }
    ```

**Alur Kerja:**

1.  Developer mengubah isi `index.html`.
2.  Developer melakukan `git push` ke *branch* `main`.
3.  Jenkins mendeteksi *push* dan menjalankan *pipeline*.
4.  *Stage* "Build and Push Image" akan dieksekusi. Jenkins akan membangun *image* dan mendorongnya ke repositori `docker-hosted` di Nexus dengan *tag* yang unik (misalnya: `1`, `2`, `3`, dst.).

**Verifikasi di Nexus:**
Buka **Browse \> `docker-hosted`** di Nexus. Anda akan melihat *image* **`simple-static-website`**. Klik *image* tersebut, dan Anda akan melihat daftar semua *tag* yang telah diunggah sesuai dengan setiap nomor *build* Jenkins.

---

### **Pembersihan Repositori dan Kebijakan Versioning**

**Pengantar: Masalah Gudang yang Penuh**

Bayangkan gudang Nexus kita. Setiap kali Jenkins menjalankan *build*, ia mengirim artefak baru. Tanpa manajemen, gudang ini akan cepat sekali penuh dengan artefak-artefak lama, versi *snapshot* yang sudah tidak relevan, dan *image* Docker development yang hanya digunakan sekali.

Masalahnya ada dua:
1.  **Biaya dan Performa:** Penyimpanan disk tidak gratis. Repositori yang membengkak akan memperlambat *backup* dan performa Nexus itu sendiri.
2.  **Kebingungan:** Repositori yang berantakan membuatnya sulit untuk menemukan versi artefak yang benar dan "bersih" untuk dirilis.

Solusinya adalah kombinasi dari dua praktik: **kebijakan *versioning* yang disiplin** dan **otomatisasi pembersihan repositori**.

---

### **1. Kebijakan Versioning (Dasar dari Pengelolaan)**

Sebelum kita bisa membersihkan, kita harus bisa membedakan mana artefak yang "penting" dan mana yang "sementara". Di sinilah kebijakan *versioning* berperan. Standar yang paling banyak diadopsi adalah **Semantic Versioning (SemVer)**.

**Teori Semantic Versioning (SemVer)**

SemVer mengusulkan format versi tiga bagian: **MAJOR.MINOR.PATCH**.
* **`MAJOR` (Versi Mayor):** Dinaikkan jika ada perubahan yang **tidak kompatibel ke belakang** (*breaking change*).
* **`MINOR` (Versi Minor):** Dinaikkan jika ada **penambahan fungsionalitas baru** yang tetap **kompatibel ke belakang**.
* **`PATCH` (Versi Patch):** Dinaikkan jika ada **perbaikan *bug*** yang juga **kompatibel ke belakang**.

Contoh: `1.2.5` → Versi mayor 1, minor 2, patch 5.

**Hubungan dengan `SNAPSHOT` dan Prarilis:**
* **`SNAPSHOT` (Maven):** Versi seperti `1.3.0-SNAPSHOT` menandakan bahwa versi ini sedang **aktif dalam pengembangan**. Artefak ini tidak stabil dan bisa diubah kapan saja. Inilah kandidat utama untuk dibersihkan.
* **Prarilis (npm/Docker):** Versi seperti `2.0.0-alpha.1` atau `2.0.0-beta` juga menandakan versi yang belum stabil dan merupakan kandidat untuk dibersihkan.

#### **Contoh Kasus: Menerapkan Versioning**

* **Kasus NPM:**
    * Gunakan perintah `npm version` untuk mengelola versi secara otomatis di `package.json`.
    * Selesai memperbaiki *bug*? Jalankan `npm version patch`. Ini akan mengubah `1.0.0` menjadi `1.0.1`.
    * Menambah fitur baru? Jalankan `npm version minor`. Ini akan mengubah `1.0.1` menjadi `1.1.0`.
    * Perintah ini juga secara otomatis membuat *tag* Git, yang bisa kita gunakan untuk memicu *pipeline* rilis di Jenkins.

* **Kasus Docker Image:**
    * Praktik terbaik adalah memberi *tag* pada *image* produksi dengan versi SemVer penuh, misalnya `my-app:1.2.5`.
    * **Saran Tambahan (Tag Bergerak):** Untuk kemudahan, berikan juga *tag* yang lebih umum. Saat Anda merilis `1.2.5`, beri *tag* pada *image* yang sama dengan:
        * `my-app:1.2.5` (spesifik)
        * `my-app:1.2` (minor terbaru)
        * `my-app:1` (mayor terbaru)
        * `my-app:latest`
    * Ini memungkinkan pengguna lain untuk menargetkan tingkat stabilitas yang berbeda.

---

### **2. Pembersihan Repositori Otomatis di Nexus**

Setelah kita memiliki kebijakan *versioning* yang jelas, kita bisa mengotomatiskan pembersihan artefak-artefak yang "kadaluarsa". Nexus melakukan ini melalui **Cleanup Policies** dan **Tasks**.

#### **How-To: Konfigurasi Cleanup Policies**

1.  **Navigasi:** Pergi ke **⚙️ Administration > System > Cleanup Policies**.
2.  **Buat Kebijakan Baru:** Klik **Create cleanup policy**.
3.  **Isi Formulir:**
    * **Name:** Beri nama deskriptif (misal: `hapus-snapshot-tua`).
    * **Format:** Pilih format repositori (misal: `maven2`, `npm`, `docker`).
    * **Criteria:** Ini adalah bagian terpenting. Anda bisa menambahkan satu atau lebih kriteria. Beberapa yang paling berguna adalah:
        * **`Last downloaded`:** Hapus jika artefak tidak pernah diunduh selama X hari. Sangat baik untuk membersihkan dependensi *cache* yang tidak lagi digunakan.
        * **`Last blob updated`:** Hapus jika artefak (khususnya `SNAPSHOT`) tidak diperbarui selama X hari.
        * **`Is prerelease`:** Hapus jika versinya adalah prarilis (memiliki `-alpha`, `-beta`, atau `-SNAPSHOT`).

#### **Contoh Kasus: Menerapkan Kebijakan Pembersihan**

* **Kasus `SNAPSHOT` (Maven & NPM Prarilis): Membersihkan Artefak Pengembangan**
    * **Tujuan:** Secara otomatis menghapus semua artefak `SNAPSHOT` (Maven) atau prarilis (NPM) yang lebih tua dari 30 hari.
    * **Konfigurasi Policy di Nexus:**
        * **Name:** `hapus-snapshot-npm-maven-30-hari`
        * **Format:** `maven2` (ulangi proses untuk `npm`)
        * **Criteria:**
            * `Last blob updated` is older than `43200` minutes (30 hari).
            * `Is prerelease` (untuk npm) ATAU `Version` matches RegEx `.*-SNAPSHOT` (untuk maven).

* **Kasus Docker Image: Menjaga Kerapian Registry**
    * **Tujuan:** Membersihkan *image* *development* yang diberi *tag* dengan nomor *build* Jenkins, tetapi **tidak pernah** menyentuh *image* yang memiliki *tag* `latest` atau *tag* rilis (misal: `v1.2.5`).
    * **Kebijakan:** "Hapus *image* yang lebih tua dari 60 hari DAN *tag*-nya **bukan** `latest` DAN *tag*-nya **tidak** mengikuti pola SemVer rilis."
    * **Konfigurasi Policy di Nexus:**
        * **Name:** `hapus-docker-dev-images`
        * **Format:** `docker`
        * **Criteria:**
            * `Last downloaded` is older than `86400` minutes (60 hari).
            * **`Tag` does not match RegEx** `^latest$` (Jangan sentuh *tag latest*).
            * **`Tag` does not match RegEx** `^v?\d+\.\d+\.\d+$` (Jangan sentuh *tag* SemVer seperti `1.0.0` atau `v1.0.0`).

#### **Menjalankan Tugas Pembersihan (Task)**

Kebijakan yang kita buat tidak akan berjalan sendiri. Kita perlu membuat **Task** untuk menjalankannya.

1.  Pergi ke **⚙️ Administration > System > Tasks**.
2.  Klik **Create task**.
3.  Pilih tipe **`Admin - Cleanup repositories`**.
4.  Beri nama tugas, pilih *Cleanup Policy* yang sudah Anda buat, dan pilih repositori mana yang akan dibersihkan (misal: `maven-snapshots`).
5.  Atur jadwalnya (*schedule*) agar berjalan secara periodik (misalnya, setiap hari pada tengah malam).

#### **Saran Tambahan (Agar Lebih Sempurna)**

* **Gunakan "Dry Run" Terlebih Dahulu!** Saat membuat tugas pembersihan, ada opsi **Dry Run**. Jika diaktifkan, tugas akan berjalan dan mencatat di log semua artefak yang *akan* dihapus, tetapi **tidak benar-benar menghapusnya**. Selalu gunakan ini terlebih dahulu untuk memastikan kebijakan Anda tidak menghapus sesuatu yang penting.
* **Pisahkan Blob Store:** Untuk performa dan manajemen yang lebih baik, Anda bisa membuat *Blob Store* terpisah untuk repositori `snapshots` dan `releases`. Ini membuat proses pembersihan lebih cepat dan manajemen data lebih mudah.
* **Pembersihan Bukanlah Backup:** Tugas pembersihan bersifat destruktif. Pastikan Anda memiliki strategi *backup* yang solid untuk volume data Nexus Anda, terpisah dari kebijakan pembersihan.

Dengan menggabungkan *versioning* yang disiplin dan kebijakan pembersihan otomatis, Anda memastikan "gudang" Nexus Anda tetap efisien, aman, dan terorganisir dengan baik.