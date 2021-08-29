# Bab 04: Currying (Kari)

## Tidak Bisa Hidup Jika Hidup Tanpamu

Ayah saya pernah menjelaskan bagaimana ada hal-hal tertentu yang seseorang dapat hidup tanpanya sampai seseorang mendapatkannya.

Microwave adalah salah satunya. Ponsel pintar, lainnya.

Orang tua di antara kita akan mengingat kehidupan yang memuaskan tanpa internet. Bagi saya, kari ada di daftar ini.

Konsepnya sederhana:

Anda dapat **memanggil fungsi dengan argumen yang lebih sedikit daripada yang diharapkan**. Ini **mengembalikan fungsi yang mengambil argumen yang tersisa**.

Anda dapat memilih untuk memanggil semuanya sekaligus atau hanya memberi makan di setiap argumen sedikit-sedikit.

```js
const add = (x) => (y) => x + y;
const increment = add(1);
const addTen = add(10);

increment(2); // 3
addTen(2); // 12
```

Di sini kita telah membuat sebuah fungsi `add` yang mengambil satu argumen dan mengembalikan sebuah fungsi.

Dengan memanggilnya, fungsi yang dikembalikan mengingat argumen pertama sejak saat itu melalui penutupan (_closures_).

Akan tetapi, memanggilnya dengan kedua argumen sekaligus sedikit merepotkan, jadi kita bisa menggunakan fungsi pembantu khusus yang dipanggil `curry` untuk membuat pendefinisian dan pemanggilan fungsi seperti ini lebih mudah.

Mari kita siapkan beberapa fungsi kari untuk kesenangan kita. Mulai sekarang, kita akan memanggil fungsi `curry` kita yang didefinisikan di [Lampiran A: Dukungan Fungsi Esensial](./appendix_a-id.md).

```js
const match = curry((what, s) => s.match(what));
const replace = curry((what, replacement, s) => s.replace(what, replacement));
const filter = curry((f, xs) => xs.filter(f));
const map = curry((f, xs) => xs.map(f));
```

Pola yang saya ikuti sederhana, tetapi penting.

Saya telah memposisikan secara strategis data yang kami operasikan (String, Array) sebagai argumen terakhir. Ini akan menjadi jelas ketika digunakan.

(Sintaksnya `/r/g` adalah ekspresi reguler yang artinya _cocok dengan setiap huruf 'r'_. Baca lebih lanjut tentang [ekspresi reguler](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions) jika Anda mau.)

```js
match(/r/g, "hello world"); // [ 'r' ]

const hasLetterR = match(/r/g); // x => x.match(/r/g)
hasLetterR("hello world"); // [ 'r' ]
hasLetterR("just j and s and t etc"); // null

filter(hasLetterR, ["rock and roll", "smooth jazz"]); // ['rock and roll']

const removeStringsWithoutRs = filter(hasLetterR); // xs => xs.filter(x => x.match(/r/g))
removeStringsWithoutRs(["rock and roll", "smooth jazz", "drum circle"]); // ['rock and roll', 'drum circle']

const noVowels = replace(/[aeiou]/gi); // (r,x) => x.replace(/[aeiou]/ig, r)
const censored = noVowels("*"); // x => x.replace(/[aeiou]/ig, '*')
censored("Chocolate Rain"); // 'Ch*c*l*t* R**n'
```

Apa yang ditunjukkan di sini adalah kemampuan untuk "memuat terlebih dahulu" suatu fungsi dengan satu atau dua argumen untuk menerima fungsi baru yang mengingat argumen tersebut.

Saya mendorong Anda untuk mengkloning repositori: `git clone https://github.com/renomureza/mostly-adequate-guide.git`, salin kode di atas dan coba di REPL. Fungsi kari, serta apa pun yang didefinisikan dalam lampiran, tersedia di modul `support/index.js`.

Atau, lihat versi yang dipublikasikan di npm:

```
npm install @mostly-adequate/support
```

## Lebih Dari Saos / Spesial

Kari berguna untuk banyak hal. Kita dapat membuat fungsi baru hanya dengan memberikan fungsi dasar kita beberapa argumen seperti yang terlihat pada `hasLetterR`, `removeStringsWithoutRs`, dan `censored`.

Kami juga memiliki kemampuan untuk mengubah fungsi apa pun yang bekerja pada elemen tunggal menjadi fungsi yang bekerja pada array hanya dengan membungkusnya dengan `map`:

```js
const getChildren = (x) => x.childNodes;
const allTheChildren = map(getChildren);
```

Memberikan fungsi argumen lebih sedikit daripada yang diharapkan biasanya disebut aplikasi parsial.

Menerapkan sebagian fungsi dapat menghapus banyak kode pelat boiler.

Pertimbangkan apa fungsi `allTheChildrenfungsi` di atas dengan uncurried `map` dari lodash (perhatikan argumennya dalam urutan yang berbeda):

```js
const allTheChildren = (elements) => map(elements, getChildren);
```

Kami biasanya tidak mendefinisikan fungsi yang bekerja pada array, karena kami hanya dapat memanggil inline `map(getChildren)`.

Sama dengan `sort`, `filter`, dan fungsi tingkat tinggi (_higher order function_) lainnya (fungsi tingkat tinggi adalah fungsi yang mengambil atau mengembalikan fungsi).

Ketika kami berbicara tentang fungsi murni, kami mengatakan bahwa mereka mengambil 1 input ke 1 output.

Currying melakukan hal ini: setiap argumen mengembalikan fungsi baru yang mengharapkan argumen yang tersisa. Itu adalah olahraga lama, 1 input ke 1 output.

Tidak masalah jika outputnya adalah fungsi lain - itu memenuhi syarat sebagai murni.

Kami mengizinkan lebih dari satu argumen pada satu waktu, tetapi ini dianggap hanya menghapus tambahan `()` untuk kenyamanan.

## Singkatnya

Currying berguna dan saya sangat menikmati bekerja dengan fungsi kari setiap hari. Ini adalah alat untuk sabuk yang membuat pemrograman fungsional tidak terlalu bertele-tele dan membosankan.

Kami dapat membuat fungsi baru yang berguna dengan cepat hanya dengan mengirimkan beberapa argumen dan sebagai bonus, kami telah mempertahankan definisi fungsi matematika meski beberapa argumen.

Mari dapatkan alat penting lain yang disebut compose.

[Bab 05: Coding dengan Composing](./ch05-id.md)

## Latihan

#### Catatan tentang Latihan

Di sepanjang buku, Anda mungkin menemukan bagian 'Latihan' seperti ini. Latihan dapat dilakukan langsung di browser asalkan Anda membaca dari gitbook (disarankan).

Perhatikan bahwa, untuk semua latihan dalam buku ini, Anda selalu memiliki beberapa fungsi pembantu yang tersedia dalam lingkup global.

Oleh karena itu, apa pun yang didefinisikan dalam [Lampiran A](./appendix_a-id.md),
[Lampiran B](./appendix_b-id.md) dan [Lampiran C](./appendix_c-id.md) tersedia untuk Anda!

Dan, seolah-olah itu tidak cukup, beberapa latihan juga akan mendefinisikan fungsi khusus untuk masalah yang mereka hadirkan; sebagai soal fakta, menganggap mereka tersedia juga.

> Petunjuk: Anda dapat mengirimkan solusi Anda dengan menekan `Ctrl + Enter` di editor tersemat!

#### Latihan Lari di Mesin Anda (opsional)

Jika Anda lebih suka melakukan latihan langsung di file menggunakan editor Anda sendiri:

- Kloning repositori (`git clone https://github.com/renomureza/mostly-adequate-guide.git`)
- Masuk ke bagian _exercises_ (`cd mostly-adequate-guide/exercises`)
- Pasang package yang diperlukan menggunakan [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) (`npm install`)
- Selesaikan jawaban dengan memodifikasi file bernama _exercises\_\*_ di folder bab yang sesuai
- Jalankan koreksi dengan npm (misalnya. `npm run ch04`)

Tes unit akan bertentangan dengan jawaban Anda dan memberikan petunjuk jika terjadi kesalahan. Oleh karena itu, jawaban untuk latihan tersedia dalam file bernama solution\_\* .

#### Ayo berlatih!

{% exercise %}  
Refactor untuk menghapus semua argumen dengan menerapkan sebagian fungsi.

{% initial src="./exercises/ch04/exercise_a.js#L3;" %}

```js
const words = (str) => split(" ", str);
```

{% solution src="./exercises/ch04/solution_a.js" %}  
{% validation src="./exercises/ch04/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

{% exercise %}  
Refactor untuk menghapus semua argumen dengan menerapkan sebagian fungsi.

{% initial src="./exercises/ch04/exercise_b.js#L3;" %}

```js
const filterQs = (xs) => filter((x) => match(/q/i, x), xs);
```

{% solution src="./exercises/ch04/solution_b.js" %}  
{% validation src="./exercises/ch04/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Dengan memperhatikan fungsi berikut:

```js
const keepHighest = (x, y) => (x >= y ? x : y);
```

{% exercise %}  
Refactor `max` untuk tidak mereferensikan argumen apa pun menggunakan fungsi helper `keepHighest`.

{% initial src="./exercises/ch04/exercise_c.js#L7;" %}

```js
const max = (xs) => reduce((acc, x) => (x >= acc ? x : acc), -Infinity, xs);
```

{% solution src="./exercises/ch04/solution_c.js" %}  
{% validation src="./exercises/ch04/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
