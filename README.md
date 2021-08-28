[![cover](images/cover.png)](SUMMARY.md)

> Sumber asli buku ini tersedia dalam bahasa Inggris: [Mostly adequate guide to FP (in javascript)](https://github.com/MostlyAdequate/mostly-adequate-guide)

## Tentang buku ini

Ini adalah buku tentang paradigma fungsional secara umum.

Kami akan menggunakan bahasa pemrograman fungsional terpopuler di dunia: JavaScript.

Beberapa diantaranya mungkin merasa ini adalah pilihan yang buruk karena bertentangan dengan selera budaya saat ini, saat ini, terasa begitu penting.

Namun, saya percaya ini adalah cara terbaik untuk belajar FP karena beberapa alasan berikut:

- **Anda mungkin menggunakannya setiap hari di tempat kerja.**

  Hal ini memungkinkan untuk berlatih dan menerapkan pengetahuan yang Anda peroleh setiap hari pada program dunia nyata daripada proyek hewan peliharaan pada malam hari dan akhir pekan dalam bahasa FP esoteris.

- **Kita tidak harus mempelajari semuanya terlebih dahulu untuk mulai menulis program.**

  Dalam bahasa fungsional murni, Anda tidak dapat mencatat variabel atau membaca simpul DOM tanpa menggunakan monads.

  Di sini kita bisa sedikit curang saat kita belajar memurnikan basis kode kita.

  Juga lebih mudah untuk memulai dalam bahasa ini karena ini adalah paradigma campuran dan Anda dapat kembali ke praktik Anda saat ini sementara ada kesenjangan dalam pengetahuan Anda.

- **Bahasa ini sepenuhnya mampu menulis kode fungsional kedudukan tertinggi.**

  Kami memiliki semua fitur yang kami butuhkan untuk meniru bahasa seperti Scala atau Haskell dengan bantuan satu atau dua perpustakaan kecil.

  Pemrograman berorientasi objek saat ini mendominasi industri, tetapi jelas canggung dalam JavaScript.

  Ini mirip dengan berkemah di jalan raya atau menari tap di sepatu karet.

  Kami harus melakukan `bind` ke mana-mana agar `this` tidak berubah dari bawah kami, kami memiliki berbagai solusi untuk perilaku unik ketika kata kunci `new` dilupakan, _private members_ (anggota pribadi) hanya tersedia melalui _closures_ (penutupan). Bagi banyak dari kita, FP terasa lebih alami.

Yang mengatakan, bahasa fungsional yang diketik (_typed_) akan, tanpa diragukan lagi, menjadi tempat terbaik untuk membuat kode dalam gaya yang disajikan oleh buku ini.

JavaScript akan menjadi sarana kami untuk mempelajari suatu paradigma, di mana Anda menerapkannya terserah Anda. Untungnya, antarmukanya matematis dan dengan demikian ada di mana-mana.

Anda akan merasa betah dengan Swiftz, Scalaz, Haskell, PureScript, dan lingkungan yang cenderung matematis lainnya.

## Baca Online

Untuk pengalaman membaca yang lebih baik, [baca secara online melalui Gitbook](https://renomureza.gitbook.io/belajar-functional-programming-javascript/).

- Bilah samping akses cepat
- Latihan dalam browser
- Contoh mendalam

## Bermain-main dengan Kode

Untuk membuat pelatihan menjadi efisien dan tidak terlalu bosan saat saya menceritakan kisah lain, pastikan untuk bermain-main dengan konsep yang diperkenalkan dalam buku ini.

Beberapa mungkin sulit ditangkap pada awalnya dan lebih mudah dipahami dengan mengotori tangan Anda. Semua fungsi dan struktur data aljabar yang disajikan dalam buku ini dikumpulkan dalam lampiran.

Kode yang sesuai juga tersedia sebagai modul npm:

```bash
$ npm i @mostly-adequate/support
```

Atau, latihan setiap bab dapat dijalankan dan diselesaikan di editor Anda! Misalnya, selesaikan `exercise_*.js` di `exercises/ch04` lalu jalankan:

```bash
$ npm run ch04
```

## Unduh

Temukan **PDF** dan **EPUB** yang telah dibuat sebelumnya sebagai [build artifacts versi terbaru](https://github.com/renomureza/mostly-adequate-guide/releases).

atau, buat sendiri.

> ⚠️ ️Penyiapan proyek ini sekarang agak lama dan karenanya, Anda mungkin mengalami berbagai masalah saat membangun ini secara lokal. Kami menyarankan untuk menggunakan node v10.22.1 dan Calibre versi terbaru jika memungkinkan.

```
git clone https://github.com/renomureza/mostly-adequate-guide.git
cd mostly-adequate-guide/
npm install
npm run setup
npm run generate-pdf
npm run generate-epub
```

> Catatan! Untuk menghasilkan versi ebook, Anda harus menginstal `ebook-convert`. [Petunjuk pemasangan](https://gitbookio.gitbooks.io/documentation/content/build/ebookconvert.html).

# Daftar Isi

Lihat [SUMMARY-id.md](SUMMARY-id.md)

### Berkontribusi

Lihat [CONTRIBUTING-id.md](CONTRIBUTING-id.md)

### Terjemahan

Lihat [TRANSLATIONS.md](TRANSLATIONS.md)

### FAQ

Lihat [FAQ.md](FAQ-id.md)

# Rencana untuk masa depan

- **Bagian 1** (Bab 1-7) adalah panduan untuk dasar-dasarnya. Saya memperbarui karena saya menemukan kesalahan karena ini adalah draf awal. Jangan ragu untuk membantu!
- **Bagian 2** (bab 8-13) membahas kelas tipe seperti functor dan monad hingga traversable. Saya berharap untuk memeras transformer dan aplikasi murni.
- **Bagian 3** (bab 14+) akan mulai menarik garis tipis antara pemrograman praktis dan absurditas akademis. Kita akan melihat comonads, f-algebras, free monads, yoneda, dan konstruksi kategorikal lainnya.

---

<p align="center">
  <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
    <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
  </a>
  <br />
  Karya ini dilisensikan di bawah <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
</p>
