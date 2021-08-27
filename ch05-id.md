# Bab 05: Coding dengan Composing

## Peternakan Fungsional

Berikut ini `compose`:

```js
const compose =
  (...fns) =>
  (...args) =>
    fns.reduceRight((res, fn) => [fn.call(null, ...res)], args)[0];
```

... Jangan takut!

Ini adalah bentuk komposisi level-9000-super-Saiyan.

Demi alasan, mari kita tinggalkan implementasi variadic dan pertimbangkan bentuk yang lebih sederhana yang dapat menyusun dua fungsi bersama-sama.

Setelah Anda memahaminya, Anda dapat mendorong abstraksi lebih jauh dan menganggapnya hanya berfungsi untuk sejumlah fungsi (kami bahkan dapat membuktikannya)!

Berikut ini adalah komposisi yang lebih ramah untuk Anda para pembaca yang budiman:

```js
const compose2 = (f, g) => (x) => f(g(x));
```

`f` dan `g` adalah fungsi dan `x` merupakan nilai yang "menyalurkan" melalui mereka.

Komposisi terasa seperti fungsi peternakan.

Anda, peternak fungsi, memilih dua dengan sifat yang ingin Anda gabungkan dan tumbuk keduanya untuk menelurkan yang baru. Penggunaannya adalah sebagai berikut:

```js
const toUpperCase = (x) => x.toUpperCase();
const exclaim = (x) => `${x}!`;
const shout = compose(exclaim, toUpperCase);

shout("send in the clowns"); // "SEND IN THE CLOWNS!"
```

Komposisi dua fungsi mengembalikan fungsi baru.

Ini masuk akal: menyusun dua unit dari beberapa jenis (dalam hal ini fungsi) harus menghasilkan unit baru dari jenis itu.

Anda tidak menghubungkan dua lego bersama-sama dan mendapatkan log lincoln. Ada teori di sini, beberapa hukum dasar yang akan kita temukan pada waktunya.

Dalam definisi kami tentang `compose`, `g` akan berjalan sebelum `f`, menciptakan aliran data dari kanan ke kiri.

Ini jauh lebih mudah dibaca daripada menumpuk banyak panggilan fungsi. Tanpa compose, di atas akan menjadi:

```js
const shout = (x) => exclaim(toUpperCase(x));
```

Alih-alih dari dalam ke luar, kita berlari dari kanan ke kiri, yang menurut saya merupakan langkah ke arah kiri (boo!).

Mari kita lihat contoh di mana urutan penting:

```js
const head = (x) => x[0];
const reverse = reduce((acc, x) => [x, ...acc], []);
const last = compose(head, reverse);

last(["jumpkick", "roundhouse", "uppercut"]); // 'uppercut'
```

`reverse` akan membalikkan daftar sambil `head` mengambil item awal.

Ini menghasilkan fungsi yang efektif, meskipun tidak efisien, fungsi `last`.

Urutan fungsi dalam komposisi harus jelas di sini. Kita bisa mendefinisikan versi kiri ke kanan, namun, kita mencerminkan versi matematika lebih dekat seperti yang ada.

Itu benar, komposisi langsung dari buku matematika. Bahkan, mungkin inilah saatnya untuk melihat properti yang berlaku untuk komposisi apa pun.

```js
// associativity
compose(f, compose(g, h)) === compose(compose(f, g), h);
```

Komposisi bersifat asosiatif, artinya tidak masalah bagaimana Anda mengelompokkan keduanya. Jadi, jika kita memilih untuk huruf besar string, kita dapat menulis:

```js
compose(toUpperCase, compose(head, reverse));
// or
compose(compose(toUpperCase, head), reverse);
```

Karena tidak masalah bagaimana kita mengelompokkan panggilan kita ke `compose`, hasilnya akan sama. Itu memungkinkan kita untuk menulis variadic compose dan menggunakannya sebagai berikut:

```js
// previously we'd have to write two composes, but since it's associative,
// we can give compose as many fn's as we like and let it decide how to group them.
const arg = ["jumpkick", "roundhouse", "uppercut"];
const lastUpper = compose(toUpperCase, head, reverse);
const loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

lastUpper(arg); // 'UPPERCUT'
loudLastUpper(arg); // 'UPPERCUT!'
```

Menerapkan properti asosiatif memberi kita fleksibilitas dan ketenangan pikiran bahwa hasilnya akan setara.

Definisi variadic yang sedikit lebih rumit disertakan dengan pustaka dukungan untuk buku ini dan merupakan definisi normal yang akan Anda temukan di pustaka seperti [lodash][lodash-website], [underscore][underscore-website], dan [ramda][ramda-website].

Salah satu manfaat asosiatif yang menyenangkan adalah bahwa setiap kelompok fungsi dapat diekstraksi dan digabungkan bersama dalam komposisinya sendiri.

Mari bermain dengan refactoring contoh kita sebelumnya:

```js
const loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// -- or ---------------------------------------------------------------

const last = compose(head, reverse);
const loudLastUpper = compose(exclaim, toUpperCase, last);

// -- or ---------------------------------------------------------------

const last = compose(head, reverse);
const angry = compose(exclaim, toUpperCase);
const loudLastUpper = compose(angry, last);

// more variations...
```

Tidak ada jawaban benar atau salah - kami hanya menghubungkan lego kami dengan cara apa pun yang kami inginkan.

Biasanya yang terbaik adalah mengelompokkannya dengan cara yang dapat digunakan kembali seperti `last` dan `angry`.

Jika akrab dengan Fowler "[Refactoring](https://martinfowler.com/books/refactoring.html)", orang mungkin mengenali proses ini sebagai "[_extract function_ ](https://refactoring.com/catalog/extractFunction.html)"... tapi tanpa mengkhawatir semua _state_ objek.

## Pointfree

Gaya pointfree berarti tidak pernah harus mengatakan data Anda.

Permisi. Ini berarti fungsi yang tidak pernah menyebutkan data tempat mereka beroperasi.

Fungsi kelas satu, curry, dan komposisi semuanya bermain bersama dengan baik untuk menciptakan gaya ini.

> Petunjuk: Versi pointfree dari `replace` & `toLowerCase` didefinisikan dalam [Lampiran C - Utilitas Pointfree](appendix_c-id.md). Jangan ragu untuk mengintip!

```js
// not pointfree because we mention the data: word
const snakeCase = (word) => word.toLowerCase().replace(/\s+/gi, "_");

// pointfree
const snakeCase = compose(replace(/\s+/gi, "_"), toLowerCase);
```

Lihat bagaimana kami menerapkan sebagian `replace`? Apa yang kami lakukan adalah menyalurkan data kami melalui setiap fungsi dari 1 argumen.

Currying memungkinkan kita untuk mempersiapkan setiap fungsi untuk hanya mengambil datanya, mengoperasikannya, dan meneruskannya.

Hal lain yang perlu diperhatikan adalah bagaimana kita tidak memerlukan data untuk membangun fungsi kita dalam versi pointfree, sedangkan dalam versi pointful, kita harus memiliki `word` sebelum yang lain.

Mari kita lihat contoh lain.

```js
// not pointfree because we mention the data: name
const initials = (name) =>
  name.split(" ").map(compose(toUpperCase, head)).join(". ");

// pointfree
// NOTE: we use 'intercalate' from the appendix instead of 'join' introduced in Chapter 09!
const initials = compose(
  intercalate(". "),
  map(compose(toUpperCase, head)),
  split(" ")
);

initials("hunter stockton thompson"); // 'H. S. T'
```

Lagi kode Pointfree, membantu kami menghapus nama yang tidak perlu dan membuat kami tetap ringkas dan umum.

Pointfree adalah tes lakmus yang baik untuk kode fungsional karena memberi tahu kami bahwa kami memiliki fungsi kecil yang mengambil input ke output.

Seseorang tidak dapat membuat loop sementara, misalnya.

Berhati-hatilah, bagaimanapun, pointfree adalah pedang bermata dua dan terkadang dapat mengaburkan niat.

Tidak semua kode fungsional pointfree dan tidak apa-apa. Kami akan mengambilnya di tempat yang kami bisa dan tetap menggunakan fungsi normal.

## Debugging

Kesalahan umum adalah membuat sesuatu seperti `map`, fungsi dari dua argumen, tanpa terlebih dahulu menerapkannya sebagian.

```js
// wrong - we end up giving angry an array and we partially applied map with who knows what.
const latin = compose(map, angry, reverse);

latin(["frog", "eyes"]); // error

// right - each function expects 1 argument.
const latin = compose(map(angry), reverse);

latin(["frog", "eyes"]); // ['EYES!', 'FROG!'])
```

Jika Anda mengalami masalah dalam men-debug komposisi, kami dapat menggunakan fungsi pelacakan yang bermanfaat tetapi tidak murni, ini untuk melihat apa yang terjadi.

```js
const trace = curry((tag, x) => {
  console.log(tag, x);
  return x;
});

const dasherize = compose(
  intercalate("-"),
  toLower,
  split(" "),
  replace(/\s{2,}/gi, " ")
);

dasherize("The world is a vampire");
// TypeError: Cannot read property 'apply' of undefined
```

Ada yang salah di sini, ayo `trace`

```js
const dasherize = compose(
  intercalate("-"),
  toLower,
  trace("after split"),
  split(" "),
  replace(/\s{2,}/gi, " ")
);

dasherize("The world is a vampire");
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```

Ah! Kita perlu `map` `toLower` ini, karena ini bekerja pada array.

```js
const dasherize = compose(
  intercalate("-"),
  map(toLower),
  split(" "),
  replace(/\s{2,}/gi, " ")
);

dasherize("The world is a vampire"); // 'the-world-is-a-vampire'
```

Fungsi `trace` memungkinkan kita untuk melihat data pada titik tertentu untuk tujuan debugging.

Bahasa seperti Haskell dan PureScript memiliki fungsi serupa untuk kemudahan pengembangan.

Komposisi akan menjadi alat kami untuk membangun program dan, seperti yang diharapkan, didukung oleh teori kuat yang memastikan segala sesuatunya akan berhasil bagi kami. Mari kita periksa teori ini.

## Teori Kategori

Teori kategori adalah cabang abstrak matematika yang dapat memformalkan konsep dari beberapa cabang yang berbeda seperti teori himpunan, teori tipe, teori grup, logika, dan lainnya.

Ini terutama berkaitan dengan objek, morfisme, dan transformasi, yang mencerminkan pemrograman cukup dekat.

Berikut adalah bagan konsep yang sama seperti yang dilihat dari masing-masing teori yang terpisah.

![teori kategori](images/cat_theory.png)

Maaf, aku tidak bermaksud menakutimu.

Saya tidak berharap Anda akrab dengan semua konsep ini.

Maksud saya adalah untuk menunjukkan kepada Anda berapa banyak duplikasi yang kita miliki sehingga Anda dapat melihat mengapa teori kategori bertujuan untuk menyatukan hal-hal ini.

Dalam teori kategori, kita memiliki sesuatu yang disebut... sebuah kategori. Ini didefinisikan sebagai kumpulan dengan komponen berikut:

- Koleksi objek
- Kumpulan morfisme
- Gagasan komposisi pada morfisme
- Sebuah morfisme dibedakan yang disebut identitas

Teori kategori cukup abstrak untuk memodelkan banyak hal, tetapi mari kita terapkan ini pada tipe dan fungsi, yang menjadi perhatian kita saat ini.

**Kumpulan objek**, Objek akan menjadi tipe data. Misalnya, `String`, `Boolean`, `Number`, `Object`, dll.

Kita sering melihat tipe data sebagai kumpulan dari semua nilai yang mungkin. Seseorang dapat melihat `Boolean` sebagai himpunan dari `[true, false]` dan `Number` sebagai himpunan dari semua nilai numerik yang mungkin.

Memperlakukan tipe sebagai himpunan berguna karena kita dapat menggunakan teori himpunan untuk bekerja dengannya.

**Kumpulan morfisme**, Morfisme akan menjadi fungsi murni standar kami setiap hari.

**Gagasan komposisi pada morfisme**, seperti yang mungkin sudah Anda duga, `compose` - adalah mainan baru kami. Kami telah membahas bahwa fungsi `compose` kami adalah asosiatif yang bukan kebetulan karena merupakan properti yang harus dimiliki untuk komposisi apa pun dalam teori kategori.

Berikut adalah gambar yang menunjukkan komposisi:

![komposisi kategori 1](images/cat_comp1.png)
![komposisi kategori 2](images/cat_comp2.png)

Berikut adalah contoh konkret dalam kode:

```js
const g = (x) => x.length;
const f = (x) => x === 4;
const isFourLetterWord = compose(f, g);
```

**Morfisme yang disebut identitas**, mari kita perkenalkan fungsi yang berguna yang disebut `id`.

Fungsi ini hanya mengambil masukan dan mengembalikannya kepada Anda. Lihatlah:

```js
const id = (x) => x;
```

Anda mungkin bertanya pada diri sendiri "Apa gunanya itu?".

Kami akan menggunakan fungsi ini secara ekstensif dalam bab-bab berikutnya, tetapi untuk saat ini anggap itu sebagai fungsi yang dapat menggantikan nilai kami - fungsi yang menyamar sebagai data setiap hari.

`id` harus bermain cantik dengan `compose`. Berikut adalah properti yang selalu berlaku untuk setiap fungsi unary (unary: a one-argument function) f:

```js
// identity
(compose(id, f) === compose(f, id)) === f;
// true
```

Hei, itu seperti properti identitas pada angka! Jika itu tidak jelas, luangkan waktu dengannya. Memahami kesia-siaan.

Kita akan segera melihat `id` digunakan di semua tempat, tetapi untuk sekarang kita melihat itu adalah fungsi yang bertindak sebagai pengganti untuk nilai tertentu.

Ini cukup berguna saat menulis kode pointfree.

Jadi begitulah, jenis dan fungsi kategori.

Jika ini adalah perkenalan pertama Anda, saya rasa Anda masih sedikit bingung tentang apa itu kategori dan mengapa itu berguna. Kami akan membangun pengetahuan ini di seluruh buku ini.

Sampai sekarang, dalam bab ini, pada baris ini, Anda setidaknya dapat melihatnya memberi kami beberapa kebijaksanaan mengenai komposisi - yaitu, sifat asosiatif dan identitas.

Apa saja kategori yang lainnya, Anda bertanya? Nah, kita dapat mendefinisikan satu untuk graf berarah dengan node sebagai objek, edge sebagai morfisme, dan komposisi hanya sebagai rangkaian jalur.

Kita dapat mendefinisikan dengan Number sebagai objek dan `>=` sebagai morfisme (sebenarnya semua urutan parsial atau total dapat menjadi kategori).

Ada banyak kategori, tetapi untuk tujuan buku ini, kami hanya akan membahas kategori yang dijelaskan di atas. Kami telah cukup menelusuri permukaan dan harus melanjutkan.

## Singkatnya

Komposisi menghubungkan fungsi kita bersama seperti rangkaian pipa. Data akan mengalir melalui aplikasi kita sebagaimana mestinya - fungsi murni adalah input ke output, jadi memutus rantai ini akan mengabaikan output, menjadikan perangkat lunak kita tidak berguna.

Kami memegang komposisi sebagai prinsip desain di atas segalanya.

Ini karena aplikasi kami tetap sederhana dan masuk akal. Teori kategori akan memainkan peran besar dalam arsitektur aplikasi, pemodelan efek samping, dan memastikan kebenaran.

Kita sekarang berada pada titik di mana akan bermanfaat bagi kita untuk melihat beberapa dari ini dalam praktik. Mari kita membuat contoh aplikasi.

[Bab 06: Contoh Aplikasi](ch06-id.md)

## Latihan

Dalam setiap latihan berikut, kita akan mempertimbangkan objek Mobil berikut:

```js
{
  name: 'Aston Martin One-77',
  horsepower: 750,
  dollar_value: 1850000,
  in_stock: true,
}
```

{% exercise %}  
Gunakan `compose()` untuk menulis ulang fungsi di bawah ini.

{% initial src="./exercises/ch05/exercise_a.js#L12;" %}

```js
const isLastInStock = (cars) => {
  const lastCar = last(cars);
  return prop("in_stock", lastCar);
};
```

{% solution src="./exercises/ch05/solution_a.js" %}  
{% validation src="./exercises/ch05/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Dengan memperhatikan fungsi berikut:

```js
const average = (xs) => reduce(add, 0, xs) / xs.length;
```

{% exercise %}  
Gunakan fungsi pembantu `average` untuk me-refactor `averageDollarValue` sebagai komposisi.

{% initial src="./exercises/ch05/exercise_b.js#L7;" %}

```js
const averageDollarValue = (cars) => {
  const dollarValues = map((c) => c.dollar_value, cars);
  return average(dollarValues);
};
```

{% solution src="./exercises/ch05/solution_b.js" %}  
{% validation src="./exercises/ch05/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

{% exercise %}  
Refactor `fastestCar` gunakan `compose()` dan fungsi lainnya dalam gaya pointfree. Petunjuk, fungsi `append` mungkin berguna.

{% initial src="./exercises/ch05/exercise_c.js#L4;" %}

```js
const fastestCar = (cars) => {
  const sorted = sortBy((car) => car.horsepower);
  const fastest = last(sorted);
  return concat(fastest.name, " is the fastest");
};
```

{% solution src="./exercises/ch05/solution_c.js" %}  
{% validation src="./exercises/ch05/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

[lodash-website]: https://lodash.com/
[underscore-website]: https://underscorejs.org/
[ramda-website]: https://ramdajs.com/
[refactoring-book]: https://martinfowler.com/books/refactoring.html
[extract-function-refactor]: https://refactoring.com/catalog/extractFunction.html
