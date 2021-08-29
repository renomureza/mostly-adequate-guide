# Bab 07: Hindley-Milner dan Saya

## Apa Tipe Anda?

Jika Anda baru mengenal dunia fungsional, tidak akan lama sebelum Anda menemukan diri Anda dalam tanda tangan tipe.

Tipe (jenis) adalah meta bahasa yang memungkinkan semua orang dari latar belakang yang berbeda untuk berkomunikasi secara ringkas dan efektif.

Sebagian besar, mereka ditulis dengan sistem yang disebut "Hindley-Milner", yang akan kita bahas bersama dalam bab ini.

Saat bekerja dengan fungsi murni, tanda tangan tipe memiliki kekuatan ekspresif yang bahasa Inggrisnya tidak dapat digunakan.

Tanda tangan ini membisikkan di telinga Anda rahasia intim suatu fungsi. Dalam satu garis yang kompak, mereka mengekspos perilaku dan niat.

Kita dapat menurunkan "teorema bebas" darinya.

Tipe dapat disimpulkan sehingga tidak perlu anotasi tipe eksplisit. Mereka dapat disetel ke presisi titik halus atau dibiarkan umum dan abstrak.

Mereka tidak hanya berguna untuk JIT, tetapi juga menjadi dokumentasi terbaik yang tersedia. Tipe tanda tangan dengan demikian memainkan peran penting dalam pemrograman fungsional - lebih dari yang Anda duga sebelumnya.

JavaScript adalah bahasa yang dinamis, tetapi itu tidak berarti kita menghindari tipe bersama-sama. Kami masih bekerja dengan string, angka, boolean, dan sebagainya.

Hanya saja tidak ada integrasi tingkat bahasa, jadi kami menyimpan informasi ini di kepala kami. Tidak perlu khawatir, karena kami menggunakan tanda tangan untuk dokumentasi, kami dapat menggunakan komentar untuk memenuhi tujuan kami.

Ada alat pengecekan tipe yang tersedia untuk JavaScript seperti [Flow](https://flow.org/) atau dialek yang diketik, [TypeScript](https://www.typescriptlang.org/).

Tujuan buku ini adalah untuk melengkapi seseorang dengan alat untuk menulis kode fungsional sehingga kita akan tetap menggunakan sistem tipe standar yang digunakan di seluruh bahasa FP.

## Cerita dari Cryptic

Dari halaman berdebu buku matematika, melintasi lautan kertas putih yang luas, di antara posting blog Sabtu pagi yang santai, hingga ke kode sumber itu sendiri, kami menemukan tanda tangan tipe Hindley-Milner.

Sistemnya cukup sederhana, tetapi memerlukan penjelasan singkat dan beberapa latihan untuk menyerap sepenuhnya bahasa kecil itu.

```js
// capitalize :: String -> String
const capitalize = (s) => toUpperCase(head(s)) + toLowerCase(tail(s));

capitalize("smurf"); // 'Smurf'
```

Di sini, `capitalize` mengambil `String` dan mengembalikan `String`. Jangankan implementasinya, ini adalah jenis tanda tangan yang kami minati.

Dalam HM (Hindley-Milner), fungsi ditulis sebagai `a -> b` dimana `a` dan `b` merupakan variabel untuk tipe apa pun.

Jadi tanda tangan untuk `capitalize` dapat dibaca sebagai "fungsi dari `String` ke `String`".

Dengan kata lain, dibutuhkan `String` sebagai inputnya dan mengembalikan `String` sebagai outputnya.

Mari kita lihat beberapa tanda tangan fungsi lainnya:

```js
// strLength :: String -> Number
const strLength = (s) => s.length;

// join :: String -> [String] -> String
const join = curry((what, xs) => xs.join(what));

// match :: Regex -> String -> [String]
const match = curry((reg, s) => s.match(reg));

// replace :: Regex -> String -> String -> String
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

`strLength` adalah ide yang sama seperti sebelumnya: kami mengambil `String` dan mengembalikan `Number`.

Yang lain mungkin membingungkan Anda pada pandangan pertama. Tanpa sepenuhnya memahami detailnya, Anda selalu dapat melihat tipe terakhir sebagai nilai kembalian.

Jadi untuk `match`, Anda dapat menafsirkan sebagai: Dibutuhkan `Regex` dan `String` mengembalikan `[String]`.

Tetapi hal yang menarik sedang terjadi di sini yang ingin saya jelaskan sedikit kalau boleh.

Untuk tanda tangan `match` kita bebas mengelompokkan seperti:

```js
// match :: Regex -> (String -> [String])
const match = curry((reg, s) => s.match(reg));
```

Ah ya, mengelompokkan bagian terakhir dalam kurung mengungkapkan lebih banyak informasi. Sekarang terlihat sebagai fungsi yang mengambil `Regex` dan mengembalikan fungsi dari `String` ke `[String]`.

Karena kari ini memang masalahnya: berikan `Regex` dan kami mendapatkan kembali fungsi menunggu `String` argumennya.

Tentu saja, kita tidak perlu memikirkannya seperti ini, tetapi ada baiknya untuk memahami mengapa tipe terakhir adalah yang dikembalikan.

```js
// match :: Regex -> (String -> [String])
// onHoliday :: String -> [String]
const onHoliday = match(/holiday/gi);
```

Setiap argumen memunculkan satu tipe di depan tanda tangan. `onHoliday` adalah `match` yang sudah memiliki Regex.

```js
// replace :: Regex -> (String -> (String -> String))
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

Seperti yang Anda lihat dengan tanda kurung pada `replace`, notasi ekstra bisa menjadi sedikit bising dan berlebihan sehingga kita cukup menghilangkannya.

Kami dapat memberikan semua argumen sekaligus jika kami memilih sehingga lebih mudah untuk menganggapnya sebagai: `replace` mengambil `Regex`, `String`, `String` lain dan mengembalikan `String`.

Beberapa hal terakhir di sini:

```js
// id :: a -> a
const id = (x) => x;

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));
```

Fungsi `id` mengambil setiap tipe lama `a` dan mengembalikan sesuatu dari tipe yang sama dengan `a`.

Kami dapat menggunakan variabel dalam tipe seperti dalam kode. Nama variabel seperti `a` dan `b` merupakan konvensi, tetapi mereka dapat diganti dengan nama apa pun yang Anda inginkan.

Jika mereka adalah variabel yang sama, mereka harus menjadi tipe yang sama. Itu aturan penting jadi mari kita ulangi:

`a -> b` bisa tipe `a` apa saja hingga tipe apa pun `b`, tetapi `a -> a` artinya harus tipe yang sama.

Misalnya, `id` mungkin `String -> String` atau `Number -> Number,` tetapi tidak `String -> Boolean`.

`map` sama menggunakan tipe variabel, tetapi kali ini kami memperkenalkan `b` yang mungkin tidak bertipe sama dengan `a`.

Kita dapat membacanya sebagai: `map` mengambil fungsi dari tipe `a` apa pun ke tipe yang sama atau berbeda dengan `b`, kemudian mengambil array `a` dan mengembalikan array `b`.

Mudah-mudahan, Anda telah dikuasai oleh keindahan ekspresif dalam tanda tangan jenis ini.

Ini benar-benar memberitahu kita apa fungsinya hampir kata demi kata. Itu diberikan fungsi dari `a` ke `b`, array `a`, dan itu memberi kita array `b`.

Satu-satunya hal yang masuk akal untuk dilakukan adalah memanggil fungsi berdarah pada masing-masing `a`. Apa pun akan menjadi kebohongan wajah yang berani.

Mampu bernalar tentang tipe dan implikasinya adalah keterampilan yang akan membawa Anda jauh di dunia fungsional.

Tidak hanya makalah, blog, dokumen, dll, menjadi lebih mudah dicerna, tetapi tanda tangan itu sendiri secara praktis akan mengajari Anda tentang fungsinya.

Dibutuhkan latihan untuk menjadi pembaca yang fasih, tetapi jika Anda tetap melakukannya, banyak informasi akan tersedia untuk Anda tanpa RTFM.

Berikut beberapa lagi hanya untuk melihat apakah Anda dapat menguraikannya sendiri.

```js
// head :: [a] -> a
const head = (xs) => xs[0];

// filter :: (a -> Bool) -> [a] -> [a]
const filter = curry((f, xs) => xs.filter(f));

// reduce :: ((b, a) -> b) -> b -> [a] -> b
const reduce = curry((f, x, xs) => xs.reduce(f, x));
```

`reduce` mungkin yang paling ekspresif dari semuanya. Bagaimanapun ini rumit, jadi jangan merasa tidak mampu jika Anda berjuang dengan itu.

Bagi yang penasaran, saya akan mencoba menjelaskan dalam bahasa Inggris meskipun bekerja melalui tanda tangan sendiri jauh lebih instruktif.

Ahem, tidak ada apa-apa.... melihat tanda tangan, kita melihat argumen pertama adalah fungsi yang mengharapkan `b` dan `a`, dan menghasilkan `b`.

Di mana ia bisa mendapatkan `a` dan `b` ini? Nah, argumen berikut dalam tanda tangan adalah `b` dan array dari `a` sehingga kita hanya dapat berasumsi bahwa `b` an masing-masing `a` akan dimasukkan.

Kita juga melihat bahwa hasil dari fungsinya adalah `b` jadi pemikiran di sini adalah akhir kita mantra dari fungsi yang diteruskan akan menjadi nilai output kami.

Mengetahui apa yang dilakukan pengurangan, kita dapat menyatakan bahwa penyelidikan di atas akurat.

## Mempersempit Kemungkinan

Setelah tipe variabel diperkenalkan, muncul properti aneh yang disebut parametrik _[parametricity](https://en.wikipedia.org/wiki/Parametricity)_. Properti ini menyatakan bahwa suatu fungsi akan bekerja pada semua tipe dengan cara yang seragam.

Mari kita selidiki:

```js
// head :: [a] -> a
```

Lihat `head`, kita melihat bahwa diperlukan `[a]` untuk `a`.

Selain tipe beton `array`, ia tidak memiliki informasi lain yang tersedia, oleh karena itu fungsinya terbatas untuk bekerja pada array saja.

Apa yang mungkin dilakukan dengan variabel `a` jika tidak tahu apa-apa tentangnya? Dengan kata lain, `a` mengatakan itu tidak bisa menjadi tipe spesifik, yang berarti itu bisa menjadi tipe apa pun, memberi kita fungsi yang harus bekerja secara seragam untuk setiap tipe yang mungkin.

Inilah yang dimaksud dengan parametrik.

Menebak implementasinya, satu-satunya asumsi yang masuk akal adalah bahwa dibutuhkan elemen pertama, terakhir, atau acak dari array itu. Nama `head` harus memberi tahu kita.

Ini satu lagi:

```js
// reverse :: [a] -> [a]
```

Dari jenis tanda tangan saja, apa yang mungkin terjadi pada `reverse`? Sekali lagi, itu tidak dapat melakukan sesuatu yang spesifik untuk `a`.

Itu tidak dapat mengubah `a` ke tipe yang berbeda atau kami akan memperkenalkan `b`.

Bisakah itu menyortir? Yah, tidak, itu tidak akan memiliki informasi yang cukup untuk menyortir setiap tipe yang mungkin.

Apakah bisa diatur ulang? Ya, saya kira itu bisa melakukannya, tetapi harus melakukannya dengan cara yang persis sama yang dapat diprediksi.

Kemungkinan lain adalah bahwa ia dapat memutuskan untuk menghapus atau menduplikasi elemen. Bagaimanapun, intinya adalah, perilaku yang mungkin secara besar-besaran dipersempit oleh tipe polimorfiknya.

Penyempitan kemungkinan ini memungkinkan kita menggunakan mesin pencari tipe signature seperti [Hoogle](https://hoogle.haskell.org/) untuk menemukan fungsi yang kita cari. Informasi yang dikemas rapat memang menjadi tanda tangan cukup kuat.

## Bebas seperti pada Teorema

Selain menyimpulkan kemungkinan implementasi, penalaran semacam ini memberikan kita teorema bebas.

Berikut ini adalah beberapa contoh teorema acak yang diangkat langsung dari [makalah Wadler tentang subjek tersebut](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).

```js
// head :: [a] -> a
compose(f, head) === compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```

Anda tidak memerlukan kode apa pun untuk mendapatkan teorema ini, teorema ini mengikuti langsung dari tipenya.

Yang pertama mengatakan bahwa jika kita mendapatkan `head` array, kemudian menjalankan beberapa fungsi `f` di atasnya, yang setara, dan kebetulan jauh lebih cepat, daripada jika kita terlebih dahulu `map(f)` melewati setiap elemen kemudian mengambil `head` untuk hasilnya.

Anda mungkin berpikir, itu hanya akal sehat. Tapi terakhir saya periksa, komputer tidak memiliki akal sehat.

Memang, mereka harus memiliki cara formal untuk mengotomatiskan pengoptimalan kode semacam ini. Matematika memiliki cara untuk memformalkan yang intuitif, yang sangat membantu di tengah medan logika komputer yang kaku.

Teorema `filter` mirip. Dikatakan bahwa jika kita menulis `f` dan `p` memeriksa mana yang harus difilter, kemudian benar-benar menerapkan `f` melalui `map` (ingat `filter` tidak akan mengubah elemen - tanda tangannya memaksa yang `a` tidak akan disentuh), itu akan selalu sama dengan memetakan `f` lalu memfilter hasilnya dengan predikat `p`.

Ini hanyalah dua contoh, tetapi Anda dapat menerapkan alasan ini ke tanda tangan tipe polimorfik apa pun dan itu akan selalu berlaku.

Dalam JavaScript, ada beberapa alat yang tersedia untuk mendeklarasikan aturan penulisan ulang.

Seseorang mungkin juga melakukan ini melalui fungsi `compose` itu sendiri. Buahnya menggantung rendah dan kemungkinannya tidak terbatas.

## Constraints

Satu hal terakhir yang perlu diperhatikan adalah bahwa kita dapat membatasi tipe ke antarmuka.

```js
// sort :: Ord a => [a] -> [a]
```

Apa yang kita lihat di sisi kiri panah gemuk (`=>`) kita di sini adalah pernyataan fakta: `a` pasti `Ord`.

Atau dengan kata lain, `a` harus mengimplementasikan antarmuka `Ord`.

Apa `Ord` dan dari mana asalnya? Dalam bahasa yang diketik (_typed language_), itu akan menjadi antarmuka yang ditentukan yang mengatakan bahwa kita dapat mengurutkan (order) nilai.

Ini tidak hanya memberi tahu kita lebih banyak tentang `a` dan apa fungsi `sort`, tetapi juga membatasi domain. Kami menyebutnya sebagai deklarasi antarmuka _type constraints_.

```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Di sini, kami memiliki dua constraints: `Eq` dan `Show`.

Itu akan memastikan bahwa kami dapat memeriksa kesetaraan `a` dan mencetak perbedaannya jika tidak sama.

Kita akan melihat lebih banyak contoh constraints dan idenya akan lebih terbentuk di bab-bab selanjutnya.

## Singkatnya

Tanda tangan tipe Hindley-Milner ada di mana-mana di dunia fungsional.

Meskipun mudah dibaca dan ditulis, dibutuhkan waktu untuk menguasai teknik memahami program hanya melalui tanda tangan. Kami akan menambahkan tanda tangan tipe ke setiap baris kode mulai sekarang.

[Bab 08: Tupperware](./ch08-id.md)

Referensi lain tentang _type signatures_ Hindley-Milner:

- [Type Signatures - Rambda Wiki Github](https://github.com/ramda/ramda/wiki/Type-Signatures)
- [Hindley–Milner type system - Wikipedia](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)
