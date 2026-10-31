# Ringkasan fitur Launchpad

Panduan ini adalah ringkasan singkat tentang kit awal Launchpad untuk calon pembeli. Panduan ini merangkum apa yang disertakan, apa yang memerlukan layanan eksternal atau konfigurasi, dan apa yang tidak disediakan secara otomatis oleh kit tersebut. Panduan fitur lengkap dan kode sumber proyek yang dibeli tetap menjadi referensi implementasi yang otoritatif.

## Apa yang disertakan dalam Launchpad

Launchpad adalah aplikasi SaaS modular yang dibangun di atas satu basis kode bersama. Modul opsionalnya mencakup penagihan, obrolan AI, organisasi, penerbitan, dukungan, komunitas, notifikasi, pemasaran email, analitik, rujukan, periklanan, SEO, integrasi, dan penyimpanan file privat. Wizard pengaturan memungkinkan Anda memilih modul opsional mana yang akan dipertahankan dan target deployment. Komponen inti akun, keamanan, pengaturan, dan basis data bersama tetap berada di tempatnya.

Dalam checkout Git, pengaturan menggunakan sparse-checkout sehingga file yang dilacak tetapi dikecualikan tetap ada dalam riwayat Git. Dalam ZIP yang dibeli, file modul yang dikecualikan dihapus dari proyek yang diekstrak; file tersebut tidak dapat dipulihkan dari riwayat Git karena ZIP tidak berisi metadata repositori. Ini adalah pengaturan sekali jalan, bukan alat pengalih modul di tempat. Cadangkan ZIP yang diekstrak sebelum dikonfigurasi jika Anda mungkin ingin memulihkan sumber yang dikecualikan kemudian.

Wizard menulis modul yang dipilih dan preferensi deployment ke `launchpad.config.json`, dan rahasia penyedia ke file `.env` yang diabaikan oleh Git. Wizard ini dapat memeriksa kredensial penyedia yang dipilih ketika Anda secara eksplisit memilih **Uji koneksi**. Wizard ini tidak membuat akun penyedia, mendeploy infrastruktur, atau mengonfigurasi dasbor eksternal. Anda masih perlu menyelesaikan dan memelihara layanan-layanan tersebut sendiri.

Area plugin mengekspos seperangkat komponen bawaan yang diizinkan; ini bukan pasar paket yang sewenang-wenang dan tidak mengunduh atau mengeksekusi kode plugin yang diunggah. Layanan relay event opsional dapat dideploy secara terpisah untuk event pembaruan profil akun yang didukung saat ini. Ini bukan registri layanan umum, bus pesan, atau sistem pengakuan per konsumen.

## Akun dan keamanan

Autentikasi mendukung masuk email dan masuk opsional melalui Google dan GitHub. Dalam pengembangan, Mailpit dapat menerima email masuk ketika tidak ada penyedia email yang dikonfigurasi. Lingkungan yang dideploy memerlukan penyedia email untuk masuk email dan email transaksional. Halaman dan API admin menegakkan peran di sisi server.

Autentikasi multi-faktor TOTP opsional bekerja dengan aplikasi autentikator yang kompatibel dan mencakup kode pemulihan sekali pakai. Pengaturan keamanan mencakup manajemen sesi dan aktivitas audit. Kit ini tidak menyertakan masuk berbasis kata sandi atau alur pemulihan kata sandi.

Daftar periksa onboarding akun mencakup pengaturan profil, keamanan akun, dan dasbor; ini opsional dan dapat dikunjungi kembali.

## Penagihan dan akses berbayar

Modul penagihan mendukung Stripe Checkout untuk langganan yang dikonfigurasi dan pembelian Lifetime sekali, plus Stripe Customer Portal. Verifikasi pembelian Apple App Store dan Google Play opsional dapat menyediakan sumber penagihan lain ketika produk yang diperlukan, kredensial, akun toko, dan integrasi klien native dikonfigurasi. Rute toko tidak tersedia hingga pengaturannya disediakan; Stripe dapat terus beroperasi secara independen.

Kit ini memproses event pembayaran dan langganan yang terverifikasi, mencatat riwayat pembayaran, dan menangani pengembalian dana dan sengketa. Kit ini **tidak** secara otomatis melindungi setiap fitur dalam produk Anda hanya karena penagihan diaktifkan. `paymentBlocked` adalah sinyal penagihan, bukan paywall universal. Setiap fitur berbayar harus menegakkan akses di sisi server; menyembunyikan tombol di browser tidak cukup. Tentukan fitur mana yang berbayar dan apa yang masih dapat dilakukan oleh pelanggan yang dibatasi, lalu implementasikan dan uji aturan-aturan tersebut. Kit ini juga tidak menyertakan alur kerja administrator yang siap pakai untuk memverifikasi pembayaran yang dilakukan di luar Stripe.

Email penagihan menggunakan penyedia email yang dikonfigurasi, tetapi penagihan tidak memerlukan email untuk berfungsi. Pengiriman email dapat tertunda, gagal, atau diduplikasi setelah respons jaringan yang tidak pasti; ini bukan sistem pengiriman tepat sekali.

## Email, daftar tunggu, dan pemasaran

Transport email bersama mendukung Resend, SendGrid, dan SMTP. Ini digunakan untuk autentikasi, pemberitahuan penagihan, notifikasi, buletin, dan pesan pemasaran. Identitas pengirim dan kredensial penyedia harus dikonfigurasi. Pemeriksaan SMTP wizard pengaturan mengautentikasi tanpa mengirim pesan.

Daftar tunggu akses awal yang terlokalisasi memerlukan persetujuan eksplisit. Statusnya dikelola secara manual: mengundang atau menandai seseorang sebagai bergabung tidak membuat akun, mengirim undangan, atau memberikan akses. Persetujuan daftar tunggu terpisah dari langganan buletin.

Buletin mendukung pendaftaran berbasis persetujuan, tautan berhenti berlangganan, dan kampanye teks polos yang dibuat oleh administrator. Otomasi pemasaran mendukung urutan email pendek untuk pelanggan buletin baru. Alat-alat ini bukan editor email visual, sistem segmentasi audiens, atau layanan kepatuhan/hukum. Alat-alat ini tidak secara otomatis mendaftarkan entri daftar tunggu atau mengirim pesan melalui notifikasi push dan webhook. Kampanye terjadwal dan otomasi memerlukan penjadwal eksternal serta penyedia email yang dikonfigurasi.

## Asisten AI dan organisasi

Asisten dasbor opsional terhubung ke API teks kompatibel OpenAI yang dikonfigurasi. Asisten ini menyimpan percakapan dalam basis data aplikasi, menegakkan batas permintaan akun, dan mendukung percakapan pribadi dan organisasi. Asisten ini tidak menyertakan model, mengeksekusi alat, atau mengambil tindakan otonom. Prompt dan respons dikirim ke penyedia yang dikonfigurasi dan disimpan oleh aplikasi; konfigurasikan pengaturan privasi dan retensi penyedia dan ungkapkan pemrosesan ini kepada pengguna.

Organisasi menyediakan ruang kerja bersama, peran anggota, dan undangan. Otorisasi diperiksa terhadap keanggotaan organisasi, tetapi ini adalah penyewaan basis data bersama, bukan basis data atau skema terpisah untuk setiap pelanggan. Penagihan, pengaturan akun, dukungan, notifikasi, analitik, buletin, dan konten penerbitan global tidak secara otomatis dibatasi ruang lingkup organisasi.

## Penerbitan, dukungan, dan komunitas

Modul terpisah menyediakan blog Markdown terlokalisasi, feed pembaruan rilis, dan halaman CMS. Administrator mengelola draf dan konten yang diterbitkan. Halaman publik mencakup metadata terlokalisasi dan dukungan sitemap. Alat penerbitan tidak menyertakan penerbitan terjadwal, terjemahan otomatis, unggahan gambar, atau sistem moderasi komentar.

Modul dukungan menyediakan tiket pelanggan dan balasan administrator. Modul komunitas menyediakan forum diskusi. Direktori kurasi memungkinkan administrator mengelola entri publik. Ini adalah fitur awal, bukan integrasi dengan meja bantuan yang dihosting atau jaringan sosial lengkap: lampiran, otomasi penugasan/SLA, reaksi forum, jawaban yang diterima, dan profil pengguna publik tidak disertakan.

## Notifikasi dan data pelanggan

Dasbor memiliki kotak masuk notifikasi persisten dan saluran pengiriman email, Web Push, dan webhook bertanda tangan opsional. Notifikasi push memerlukan kunci VAPID dan langganan browser; konsumen webhook harus memverifikasi tanda tangan dan menduplikasi event. Percobaan ulang otomatis untuk notifikasi, pemasaran, dan pembersihan penyimpanan memerlukan pekerjaan yang dijadwalkan secara eksternal.

Pengaturan akun menyediakan ekspor JSON dari data akun yang didukung dan tautan terpisah ke manajemen penagihan. Ekspor ini mengecualikan kredensial, sesi, rahasia penandatanganan, dan token sensitif lainnya. Ini adalah unduhan yang dialirkan, bukan cadangan basis data lengkap atau layanan arsip latar belakang.

## Analitik, lokalisasi, dan kustomisasi

Dasbor mencakup tampilan akun dan penagihan. Analitik produk opsional mengumpulkan event yang dikendalikan persetujuan dan menyediakan laporan administratif; ini bukan pengganti platform analitik pihak ketiga. Atribusi, pelacakan pemasaran, dan cookie rujukan mungkin memiliki kewajiban persetujuan dalam deployment Anda. Launchpad tidak menyertakan platform manajemen persetujuan.

Antarmuka mendukung 18 lokalisasi: Inggris, Arab, Bengali, Jerman, Spanyol, Prancis, Hindi, Indonesia, Italia, Jepang, Korea, Belanda, Polandia, Portugis, Rusia, Turki, Vietnam, dan Mandarin. Inggris adalah katalog fallback dan bahasa fallback wizard pengaturan. Tinjau terjemahan dan teks hukum untuk audiens Anda sebelum peluncuran.

Pembangun produk mengedit seperangkat bagian aplikasi yang diizinkan, navigasi, salinan terlokalisasi, dan pengaturan merek. Ini tidak menghasilkan kode sewenang-wenang, membuat rute aplikasi, atau mengubah logika bisnis. Preferensi antarmuka pribadi seperti tema, gerakan, dan lainnya disimpan di browser, tidak disinkronkan antar perangkat.

## Penyimpanan, rujukan, periklanan, dan SEO

Penyimpanan file privat opsional menggunakan penyimpanan objek kompatibel S3. Aplikasi memvalidasi jenis dan ukuran file, menggunakan URL unggah/unduh berumur pendek, dan memeriksa kepemilikan file. Konfigurasikan bucket privat dan kredensial yang diperlukan. Modul ini tidak memindai file untuk malware, dan penghapusan akun tidak secara otomatis menghapus objek jarak jauh.

Modul rujukan melacak tautan mitra dan komisi yang diturunkan dari Stripe. Pembayaran dilakukan di luar Launchpad. Modul ini tidak menghitung komisi untuk pembelian App Store atau Google Play. Atribusi rujukan menggunakan cookie 30 hari; Launchpad tidak memiliki manajer persetujuan bawaan, sehingga evaluasi dan implementasikan persyaratan persetujuan yang berlaku.

Alat periklanan mengorganisasi informasi kampanye dan akun dan mengimpor laporan dari CSV. Alat ini tidak terhubung ke jaringan iklan untuk membuat atau mengelola kampanye langsung. Fitur SEO menyediakan metadata terlokalisasi, sitemap, aturan robots, data terstruktur, dan audit basis data aplikasi; ini bukan perayap web, pelacak peringkat, alat riset kata kunci, atau manajer pengalihan.

Katalog integrasi melaporkan apakah konfigurasi yang diharapkan ada. Katalog ini tidak membuktikan bahwa kredensial berfungsi, membuat akun penyedia, atau menjamin pengiriman.

## Deployment dan tanggung jawab operasional

Deployment memerlukan PostgreSQL dan variabel lingkungan yang sesuai, termasuk koneksi basis data dan `AUTH_SECRET`. Konfigurasikan email, Stripe, penagihan toko, penyimpanan objek, AI, push, dan pekerjaan terjadwal hanya untuk fitur yang Anda aktifkan. Migrasi produksi harus diterapkan menggunakan proses deployment yang terdokumentasi; pertahankan lingkungan pratinjau pada basis data yang terisolasi.

Aplikasi mencakup validasi sisi server, pemeriksaan peran, kontrol keamanan, dan pencatatan audit, tetapi ini bukan program operasi keamanan yang lengkap. Aplikasi ini tidak menyediakan WAF, pemindaian malware, pemantauan keamanan eksternal, atau Kebijakan Keamanan Konten yang ketat. Konfigurasikan ingress yang tepercaya dan perlindungan hosting, tinjau model ancaman, dan uji aplikasi yang dideploy sebelum peluncuran.

Untuk konfigurasi terperinci, kepemilikan data, batas, migrasi, dan prosedur operasional, lihat panduan fitur lengkap yang disertakan dengan proyek.
