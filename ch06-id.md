# Bab 06: Contoh Aplikasi

## Coding Deklaratif

Kita akan mengubah pola pikir kita.

Mulai sekarang, kita akan berhenti memberi tahu komputer bagaimana melakukan tugasnya dan sebagai gantinya menulis spesifikasi apa yang kita inginkan sebagai hasilnya.

Saya yakin Anda akan merasa jauh lebih sedikit stres daripada mencoba mengatur semuanya secara mikro setiap saat.

Deklaratif, sebagai lawan dari imperatif, berarti kita akan menulis ekspresi, sebagai lawan dari instruksi langkah demi langkah.

Pikirkan SQL. Tidak ada istilah "pertama lakukan ini, lalu lakukan itu".

Ada satu ekspresi yang menentukan apa yang kita inginkan dari database. Kami tidak memutuskan bagaimana melakukan pekerjaan, itu yang terjadi.

Ketika database ditingkatkan dan mesin SQL dioptimalkan, kita tidak perlu mengubah kueri kita. Ini karena ada banyak cara untuk menginterpretasikan spesifikasi kami dan mencapai hasil yang sama.

Untuk beberapa orang, termasuk saya sendiri, sulit untuk memahami konsep pengkodean deklaratif pada awalnya, jadi mari tunjukkan beberapa contoh untuk merasakannya.

```js
// imperative
const makes = [];
for (let i = 0; i < cars.length; i += 1) {
  makes.push(cars[i].make);
}

// declarative
const makes = cars.map((car) => car.make);
```

Loop imperatif pertama-tama harus membuat instance array. Penerjemah harus mengevaluasi pernyataan ini sebelum melanjutkan.

Kemudian secara langsung beralih melalui daftar mobil, secara manual meningkatkan penghitung dan menunjukkan potongan-potongannya kepada kami dalam tampilan vulgar dari iterasi eksplisit.

Versi `map` adalah salah satu ekspresi. Itu tidak memerlukan urutan evaluasi.

Ada banyak kebebasan di sini untuk bagaimana fungsi `map` berulang dan bagaimana array yang dikembalikan dapat dirakit. Ini menentukan "**apa**", bukan "**bagaimana**". Dengan demikian, ia memakai selempang deklaratif yang mengkilap.

Selain lebih jelas dan ringkas, fungsi `map` dapat dioptimalkan sesuka hati dan kode aplikasi kita yang berharga tidak perlu diubah.

Bagi Anda yang berpikir "Ya, tetapi jauh lebih cepat untuk melakukan loop imperatif", saya sarankan Anda mendidik diri sendiri tentang bagaimana JIT mengoptimalkan kode Anda. Inilah [video hebat yang mungkin bisa memberi pencerahan](https://www.youtube.com/watch?v=g0ek4vV7nEA)

Berikut adalah contoh lain.

```js
// imperative
const authenticate = (form) => {
  const user = toUser(form);
  return logIn(user);
};

// declarative
const authenticate = compose(logIn, toUser);
```

Meskipun tidak ada yang salah dengan versi imperatif, masih ada evaluasi langkah demi langkah yang dikodekan.

Ekspresi `compose` hanya menyatakan fakta: Otentikasi adalah komposisi `toUser` dan `logIn`.

Sekali lagi, ini meninggalkan ruang gerak untuk perubahan kode dukungan dan menghasilkan kode aplikasi kita menjadi spesifikasi tingkat tinggi.

Dalam contoh di atas, urutan evaluasi ditentukan (`toUser` harus dipanggil sebelumnya `logIn`).

Tetapi ada banyak skenario di mana urutannya tidak penting, dan ini mudah ditentukan dengan pengkodean deklaratif (lebih lanjut tentang ini nanti).

Karena kita tidak harus mengkodekan urutan evaluasi, pengkodean deklaratif cocok untuk komputasi paralel.

Ini ditambah dengan fungsi murni adalah mengapa FP (Functional Programming) adalah pilihan yang baik untuk masa depan paralel - kami tidak benar-benar perlu melakukan sesuatu yang istimewa untuk mencapai sistem paralel/bersamaan.

## Flickr dari Pemrograman Fungsional

Sekarang kita akan membangun sebuah contoh aplikasi dengan cara yang deklaratif dan dapat dikomposisi.

Kami masih akan menipu dan menggunakan efek samping untuk saat ini, tetapi kami akan membuatnya minimal dan terpisah dari basis kode murni kami.

Kita akan membuat widget browser yang menyedot gambar flickr dan menampilkannya. Mari kita mulai dengan membuat perancah aplikasi.

Berikut html-nya:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Flickr App</title>
  </head>
  <body>
    <main id="js-main" class="main"></main>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.2.0/require.min.js"></script>
    <script src="main.js"></script>
  </body>
</html>
```

Dan inilah kerangka main.js:

```js
const CDN = (s) => `https://cdnjs.cloudflare.com/ajax/libs/${s}`;
const ramda = CDN("ramda/0.21.0/ramda.min");
const jquery = CDN("jquery/3.0.0-rc1/jquery.min");

requirejs.config({ paths: { ramda, jquery } });
requirejs(["jquery", "ramda"], ($, { compose, curry, map, prop }) => {
  // app goes here
});
```

Kami menggunakan [rambda](https://ramdajs.com/) alih-alih lodash atau perpustakaan utilitas lainnya.

Ini memiliki `compose`, `curry`, dan banyak lagi.

Saya telah menggunakan requirejs, yang mungkin tampak berlebihan, tetapi kami akan menggunakannya di seluruh buku dan konsistensi adalah kuncinya.

Sekarang itu keluar dari jalan, ke spesifikasi. Aplikasi kami akan melakukan 4 hal.

1. Buat url untuk istilah pencarian khusus kami.
2. Lakukan panggilan api flickr.
3. Ubah json yang dihasilkan menjadi gambar html.
4. Tempatkan di layar.

Ada 2 perbuatan _impure_ (tidak murni) yang disebutkan di atas. Apakah kamu melihat mereka? Sedikit tentang mendapatkan data dari api flickr dan menempatkannya di layar.

Mari kita definisikan terlebih dahulu sehingga kita dapat mengkarantina mereka. Juga, saya akan menambahkan fungsi `trace` untuk memudahkan debugging.

```js
const Impure = {
  getJSON: curry((callback, url) => $.getJSON(url, callback)),
  setHtml: curry((sel, html) => $(sel).html(html)),
  trace: curry((tag, x) => {
    console.log(tag, x);
    return x;
  }),
};
```

Di sini kami hanya membungkus method jQuery untuk menjadi kari dan kami telah menukar argumen ke posisi yang lebih menguntungkan.

Saya telah memberi mereka namespace `Impure` sehingga kami tahu ini adalah fungsi yang berbahaya. Dalam contoh yang akan datang, kita akan membuat kedua fungsi ini murni.

Selanjutnya kita harus membuat url untuk diteruskan ke fungsi `Impure.getJSON`.

```js
const host = "api.flickr.com";
const path = "/services/feeds/photos_public.gne";
const query = (t) => `?tags=${t}&format=json&jsoncallback=?`;
const url = (t) => `https://${host}${path}${query(t)}`;
```

Ada cara yang bagus dan terlalu rumit untuk menulis `url` pointfree menggunakan monoid (kita akan mempelajarinya nanti) atau kombinator.

Kami telah memilih untuk tetap menggunakan versi yang dapat dibaca dan merakit string ini dengan cara yang normal.

Mari kita tulis fungsi aplikasi yang membuat panggilan dan menempatkan konten di layar.

```js
const app = compose(Impure.getJSON(Impure.trace("response")), url);
app("cats");
```

Ini memanggil fungsi `url` kita, lalu meneruskan string ke fungsi `getJSON`, yang sebagian telah diterapkan dengan `trace`. Memuat aplikasi akan menampilkan respons dari panggilan api di konsol.

![respon console](images/console_ss.png)

Kami ingin membuat gambar dari json ini. Sepertinya `mediaUrls` tertimbun di `items` kemudian properti masing `media` `m`.

Bagaimanapun, untuk mendapatkan properti bersarang ini kita dapat menggunakan fungsi pengambil universal yang bagus dari ramda yang disebut prop. Ini adalah versi buatan sendiri sehingga Anda dapat melihat apa yang terjadi:

```js
const prop = curry((property, object) => object[property]);
```

Sebenarnya ini cukup membosankan. Kami hanya menggunakan sintaks `[]` untuk mengakses properti pada objek apa pun. Mari kita gunakan ini untuk mendapatkan `mediaUrls`.

```js
const mediaUrl = compose(prop("m"), prop("media"));
const mediaUrls = compose(map(mediaUrl), prop("items"));
```

Setelah kami mengumpulkan `items`, kami harus `map` untuk mengekstrak setiap url media. Ini menghasilkan array yang bagus dari `mediaUrls`.

Mari kita kaitkan ini ke aplikasi kita dan mencetaknya di layar.

```js
const render = compose(Impure.setHtml("#js-main"), mediaUrls);
const app = compose(Impure.getJSON(render), url);
```

Yang telah kita lakukan adalah membuat komposisi baru yang akan memanggil `mediaUrls` dan mengatur `<main>` html dengan mereka.

Kami telah mengganti panggilan `trace` dengan `render` sekarang karena kami memiliki sesuatu yang perlu dirender selain json mentah. Ini secara kasar akan menampilkan `mediaUrls` di dalam body.

Langkah terakhir kami adalah mengubahnya `mediaUrls` menjadi bonafid images.

Dalam aplikasi yang lebih besar, kami akan menggunakan perpustakaan template/dom seperti Handlebars atau React. Untuk aplikasi ini, kita hanya membutuhkan tag img jadi mari kita tetap menggunakan jQuery.

```js
const img = (src) => $("<img />", { src });
```

Method `html` jQuery akan menerima tag array. Kita hanya perlu mengubah `mediaUrls` kita menjadi gambar dan mengirimkannya ke `setHtml`.

```js
const images = compose(map(img), mediaUrls);
const render = compose(Impure.setHtml("#js-main"), images);
const app = compose(Impure.getJSON(render), url);
```

Dan kita sudah selesai!

<img src="" alt="cats grid" />

Ini skrip yang sudah jadi: [exercises/ch06/main.js](./exercises/ch06/main.js)

Sekarang lihat itu. Spesifikasi deklaratif yang indah tentang apa adanya, bukan bagaimana hal itu terjadi.

Kami sekarang melihat setiap baris sebagai persamaan dengan sifat-sifat yang berlaku. Kita dapat menggunakan properti ini untuk alasan tentang aplikasi dan refactor kita.

## Sebuah Refactor Berprinsip

Ada pengoptimalan yang tersedia - kami memetakan setiap item untuk mengubahnya menjadi url media, lalu kami memetakan lagi di atas mediaUrls tersebut untuk mengubahnya menjadi tag img.

Ada hukum tentang map dan komposisi:

```js
// map's composition law
compose(map(f), map(g)) === map(compose(f, g));
```

Kita dapat menggunakan properti ini untuk mengoptimalkan kode kita. Mari kita gunakan refactor berprinsip.

```js
// original code
const mediaUrl = compose(prop("m"), prop("media"));
const mediaUrls = compose(map(mediaUrl), prop("items"));
const images = compose(map(img), mediaUrls);
```

Mari kita sebariskan peta kita. Kita bisa sebariskan panggilan untuk `mediaUrls` di `images` berkat penalaran equational dan kemurnian.

```js
const mediaUrl = compose(prop("m"), prop("media"));
const images = compose(map(img), map(mediaUrl), prop("items"));
```

Sekarang kita telah menyusun `map` kita, kita dapat menerapkan hukum komposisi.

```js
/*
compose(map(f), map(g)) === map(compose(f, g));
compose(map(img), map(mediaUrl)) === map(compose(img, mediaUrl));
*/

const mediaUrl = compose(prop("m"), prop("media"));
const images = compose(map(compose(img, mediaUrl)), prop("items"));
```

Sekarang _bugger_ hanya akan mengulang sekali sambil mengubah setiap item menjadi img. Mari kita membuatnya sedikit lebih mudah dibaca dengan mengekstrak fungsinya.

```js
const mediaUrl = compose(prop("m"), prop("media"));
const mediaToImg = compose(img, mediaUrl);
const images = compose(map(mediaToImg), prop("items"));
```

## In Summary

Kami telah melihat bagaimana menggunakan keterampilan baru kami dengan aplikasi kecil tapi nyata.

Kami telah menggunakan kerangka matematika kami untuk alasan dan refactor kode kami.

Tapi bagaimana dengan penanganan kesalahan dan percabangan kode? Bagaimana kita bisa membuat seluruh aplikasi murni alih-alih hanya memberi nama pada fungsi destruktif?

Bagaimana kita bisa membuat aplikasi kita lebih aman dan lebih ekspresif?

Ini adalah pertanyaan yang akan kita tangani di bab 2.

[Bab 07: Hindley-Milner dan Saya](ch07-id.md)
