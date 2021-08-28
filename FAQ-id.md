## FAQ

- [Mengapa cuplikan terkadang ditulis dengan titik koma dan terkadang tanpa?](#mengapa-cuplikan-terkadang-ditulis-dengan-titik-koma-dan-terkadang-tanpa?)
- [Bukankah perpustakaan eksternal seperti ramda atau jquery membuat panggilan tidak murni?](#bukankah-perpustakaan-eksternal-seperti-ramda-atau-jquery-membuat-panggilan-tidak-murni?)
- [Apa yang dimaksud `f a` dengan tanda tangan?](#apa-yang-dimaksud-f-a-dengan-tanda-tangan?)
- [Apakah ada contoh "dunia nyata" yang tersedia?](#apakah-ada-contoh-"dunia-nyata"-yang-tersedia?)
- [Mengapa buku ini menggunakan ES5? Apakah ada versi ES6 yang tersedia?](#mengapa-buku-ini-menggunakan-ES5?-Apakah-ada-versi-ES6-yang-tersedia?)
- [Apa itu fungsi reduce?](#apa-itu-fungsi-reduce?)
- [Bukankah Anda akan menggunakan bahasa Inggris yang disederhanakan daripada gaya saat ini?](#bukankah-Anda-akan-menggunakan-bahasa-Inggris-yang-disederhanakan-daripada-gaya-saat-ini?)
- [Apa Itu Either? Apa Itu Future? Apa Itu Task?](#apa-Itu-Either?-Apa-Itu-Future?-Apa-Itu-Task?)
- [Dari mana method map, filter, compose ... berasal?](#dari-mana-metode-map,-filter,-compose-...-berasal?)

### Mengapa cuplikan terkadang ditulis dengan titik koma dan terkadang tanpa?

> Lihat [#6](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/6)

Ada dua sekolah di JavaScript, orang yang menggunakannya, dan orang yang tidak.

Kami telah membuat pilihan di sini untuk menggunakannya, dan sekarang, kami berusaha untuk konsisten dengan pilihan itu. Jika ada yang hilang, beri tahu kami dan kami akan menangani pengawasannya.

### Bukankah perpustakaan eksternal seperti ramda atau jquery membuat panggilan tidak murni?

> Lihat [#50](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/50)

Ketergantungan tersebut tersedia seolah-olah mereka berada dalam konteks global, bagian dari bahasa.

Jadi, tidak, panggilan masih bisa dianggap murni. Untuk bacaan lebih lanjut, lihat [artikel ini tentang CoEffects](http://tomasp.net/blog/2014/why-coeffects-matter/)

### Apa yang dimaksud `f a` dengan tanda tangan?

> Lihat [#62](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/62)

Dalam tanda tangan, seperti:

`map :: Functor f => (a -> b) -> f a -> f b`

`f` mengacu pada `functor` yang bisa, misalnya, Maybe atau IO.

Dengan demikian, tanda tangan mengabstraksikan pilihan fungsi tersebut dengan menggunakan variabel tipe yang pada dasarnya berarti bahwa setiap fungsi dapat digunakan di mana `f` berdiri selama semua `f` memiliki tipe yang sama (jika `f` yang pertama adalam tanda tangan mewakili `Maybe a`), maka yang kedua **tidak dapat merujuk** pada sebuah `IO b` tetapi harus menjadi `Maybe b`.

Misalnya:

```javascript
let maybeString = Maybe.of("Patate");
let f = function (x) {
  return x.length;
};
let maybeNumber = map(f, maybeString); // Maybe(6)

// With the following 'refined' signature:
// map :: (string -> number) -> Maybe string -> Maybe number
```

### Apakah ada contoh "dunia nyata" yang tersedia?

> lihat [#77](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/77), [#192](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/192)

Jika Anda belum mencapainya, Anda dapat melihat [ch06-id.md](./ch06.md) yang menyajikan aplikasi flickr sederhana. Contoh lain mungkin akan datang nanti. Omong-omong, jangan ragu untuk berbagi pengalaman Anda dengan kami!

### Mengapa buku ini menggunakan ES5? Apakah ada versi ES6 yang tersedia?

> lihat [#83](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/83), [#235](https://github.com/MostlyAdequate/mostly-adequate-guide/pull/235)

Buku ini bertujuan untuk dapat diakses secara luas.

Itu dimulai sebelum ES6 keluar, dan sekarang, karena standar baru semakin diterima, kami mempertimbangkan untuk membuat dua edisi terpisah dengan ES5 dan ES6. Anggota komunitas sudah mengerjakan versi ES6 (lihat [#235](https://github.com/MostlyAdequate/mostly-adequate-guide/pull/235) untuk informasi lebih lanjut).

### Apa itu fungsi reduce?

> lihat [#109](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/109)

Reduce, accumulate, fold, inject adalah semua fungsi biasa dalam pemrograman fungsional yang digunakan untuk menggabungkan elemen-elemen struktur data secara berurutan. Anda mungkin melihat [pembicaraan ini](https://www.youtube.com/watch?v=JZSoPZUoR58&ab_channel=NewCircleTraining) untuk mendapatkan lebih banyak wawasan tentang fungsi reduce.

### Bukankah Anda akan menggunakan bahasa Inggris yang disederhanakan daripada gaya saat ini?

> lihat [#176](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/176)

Buku ini ditulis dengan gayanya sendiri yang berkontribusi untuk membuatnya konsisten secara keseluruhan. Jika Anda tidak terbiasa dengan bahasa Inggris, anggap itu sebagai pelatihan yang baik.

Namun demikian, jika Anda terkadang memerlukan bantuan untuk memahami artinya, sekarang tersedia [beberapa terjemahan](./TRANSLATIONS-id.md) yang mungkin dapat membantu Anda.

### Apa Itu Either? Apa Itu Future? Apa Itu Task?

> lihat [#194](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/194)

Kami memperkenalkan semua struktur itu di seluruh buku ini. Oleh karena itu, Anda tidak akan menemukan penggunaan struktur yang belum pernah didefinisikan sebelumnya.

Jangan ragu untuk membaca kembali beberapa bagian lama jika Anda merasa tidak nyaman dengan tipe-tipe tersebut. Sebuah glosarium/vade mecum akan muncul di akhir untuk mensintesis semua gagasan ini.

### Dari mana metode map, filter, compose ... berasal?

> lihat [#198](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/198)

Sebagian besar waktu, metode tersebut didefinisikan dalam pustaka vendor tertentu seperti `ramda` oatau `underscore`.

Anda juga harus melihat [Lampiran A](./appendix_a-id.md), [Lampiran B](./appendix_b-id.md) dan [Lampiran C](./appendix_c-id.md) di mana kami mendefinisikan beberapa implementasi yang digunakan untuk latihan.

Fungsi-fungsi itu sangat umum dalam pemrograman fungsional dan meskipun implementasinya mungkin sedikit berbeda, artinya tetap cukup konsisten di antara perpustakaan.

[#6]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/6
[#50]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/50
[#62]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/62
[#77]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/77
[#83]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/83
[#109]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/109
[#176]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/176
[#192]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/192
[#194]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/194
[#198]: https://github.com/MostlyAdequate/mostly-adequate-guide/issues/198
[#235]: https://github.com/MostlyAdequate/mostly-adequate-guide/pull/235
