# Bab 03: Kebahagiaan Murni dengan Fungsi Murni

## Oh untuk Menjadi Murni Lagi

Satu hal yang perlu kita luruskan adalah gagasan tentang fungsi murni.

> Fungsi murni adalah fungsi dengan input yang sama, akan selalu menghasilkan output yang sama dan tidak memiliki efek samping yang dapat diamati.

Ambil `slice` dan `splice`. Mereka adalah dua fungsi yang melakukan hal yang sama persis - dengan cara yang sangat berbeda, ingatlah, tetapi tetap saja hal yang sama.

Kita mengatakan `slice` ini _murni_ karena ia mengembalikan output yang sama per masukan setiap kali, dijamin.

Di sisi lain `splice` akan mengunyah susunannya dan memuntahkannya kembali selamanya berubah yang merupakan efek yang dapat diamati.

```js
const xs = [1, 2, 3, 4, 5];

// pure
xs.slice(0, 3); // [1,2,3]

xs.slice(0, 3); // [1,2,3]

xs.slice(0, 3); // [1,2,3]

// impure
xs.splice(0, 3); // [1,2,3]

xs.splice(0, 3); // [4,5]

xs.splice(0, 3); // []
```

Dalam pemrograman fungsional, kita tidak suka fungsi berat seperti `splice` yang membuat data _bermutasi_ atau berubah.

Ini tidak akan pernah berhasil karena kami berusaha untuk fungsi yang andal yang mengembalikan hasil yang sama setiap saat, bukan fungsi yang meninggalkan kekacauan seperti `splice`.

Mari kita lihat contoh lain.

```js
// impure
let minimum = 21;
const checkAge = (age) => age >= minimum;

// pure
const checkAge = (age) => {
  const minimum = 21;
  return age >= minimum;
};
```

Di bagian yang tidak murni (_impure_), `checkAge` bergantung pada variabel `minimum` yang dapat diubah untuk menentukan hasilnya.

Dengan kata lain, itu tergantung pada keadaan sistem yang mengecewakan karena meningkatkan [beban kognitif](https://en.wikipedia.org/wiki/Cognitive_load) dengan memperkenalkan lingkungan eksternal.

Ini mungkin tidak tampak seperti banyak dalam contoh ini, tetapi ketergantungan pada keadaan ini adalah salah satu kontributor terbesar untuk kompleksitas sistem: http://curtclifton.net/papers/MoseleyMarks06a.pdf .

`checkAge` mungkin memberikan hasil yang berbeda tergantung pada faktor eksternal input, yang tidak hanya mendiskualifikasinya dari kemurnian, tetapi juga menempatkan pikiran kita melalui pemeras setiap kali kita berpikir tentang perangkat lunak.

Bentuknya yang murni (_pure_), di sisi lain, sepenuhnya mandiri. Kami juga dapat membuat `minimum` tidak dapat diubah, yang menjaga kemurnian karena _state_ tidak akan pernah berubah.

Untuk melakukan ini, kita harus membuat objek untuk dibekukan.

```js
const immutableState = Object.freeze({ minimum: 21 });
```

## Efek Samping (Side Effect) Mungkin Termasuk...

Mari kita lihat lebih lanjut "efek samping" ini untuk meningkatkan intuisi kita.

Jadi apa _efek samping_ yang tidak diragukan lagi ini yang disebutkan dalam definisi _fungsi murni_? Kami akan mengacu pada efek sebagai segala sesuatu yang terjadi dalam perhitungan kami selain perhitungan hasil.

Tidak ada yang secara intrinsik buruk tentang efek dan kami akan menggunakannya di semua tempat di bab-bab yang akan datang.

Bagian _samping_ itulah yang berkonotasi negatif.

Air saja bukanlah inkubator larva yang melekat, itu adalah bagian stagnan yang menghasilkan kawanan, dan saya yakinkan Anda, efek samping adalah tempat berkembang biak yang serupa dalam program Anda sendiri.

> Efek samping adalah perubahan dari sistem _state_ atau interaksi diamati dengan dunia luar yang terjadi selama perhitungan hasilnya.

Efek samping mungkin termasuk, tetapi tidak terbatas pada:

- mengubah sistem file
- memasukkan catatan ke dalam database
- membuat panggilan http
- mutasi
- mencetak ke layar / logging
- mendapatkan masukan pengguna
- menanyakan DOM
- mengakses status sistem

Dan daftarnya terus bertambah.

Setiap interaksi dengan dunia di luar suatu fungsi, adalah efek samping, yang merupakan fakta yang mungkin mendorong Anda untuk mencurigai kepraktisan pemrograman tanpa mereka.

Filosofi pemrograman fungsional mendalilkan bahwa efek samping adalah penyebab utama dari perilaku yang salah.

Bukannya kami dilarang menggunakannya, melainkan kami ingin menahannya dan menjalankannya dengan cara yang terkendali.

Kita akan belajar bagaimana melakukan ini ketika kita membahas fungsi dan monad di bab selanjutnya, tetapi untuk sekarang, mari kita coba memisahkan fungsi berbahaya ini dari fungsi murni kita.

Efek samping mendiskualifikasi fungsi dari yang _murni_.

Dan masuk akal: fungsi murni, menurut definisi, harus selalu mengembalikan output yang sama dengan input yang sama, yang tidak mungkin dijamin ketika menangani hal-hal di luar fungsi lokal kita.

Mari kita lihat lebih dekat mengapa kita menuntut output yang sama per input. Buka kerah Anda, kita akan melihat beberapa matematika kelas 8.

## Matematika kelas 8

Dari mathisfun.com:

> Fungsi adalah hubungan khusus antara nilai: Setiap nilai inputnya mengembalikan tepat satu nilai output.

Dengan kata lain, itu hanya hubungan antara dua nilai: input dan output.

Meskipun setiap input memiliki tepat satu output, output tersebut tidak harus unik per input. Di bawah ini menunjukkan diagram fungsi yang benar-benar valid dari `x` ke `y`;

<img src="images/function-sets.gif" alt="function sets" />(https://www.mathsisfun.com/sets/function.html)

Sebagai kontras, diagram berikut menunjukkan hubungan yang bukan fungsi karena nilai input `5` menunjuk ke beberapa output:

<img src="images/relation-not-function.gif" alt="relation not function" />(https://www.mathsisfun.com/sets/function.html)

Fungsi dapat digambarkan sebagai sekumpulan pasangan dengan posisi (input, output): `[(1,2), (3,6), (5,10)` (Tampaknya fungsi ini menggandakan inputnya).

Atau mungkin tabel:

<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

Atau bahkan sebagai grafik dengan `x` sebagai input dan `y` sebagai output:

<img src="images/fn_graph.png" width="300" height="300" alt="function graph" />

Tidak perlu detail implementasi jika input menentukan output. Karena fungsi hanyalah pemetaan input ke output, seseorang dapat dengan mudah menuliskan objek literal dan menjalankannya dengan `[]` alih - alih `()`.

```js
const toLowerCase = {
  A: "a",
  B: "b",
  C: "c",
  D: "d",
  E: "e",
  F: "f",
};
toLowerCase["C"]; // 'c'

const isPrime = {
  1: false,
  2: true,
  3: true,
  4: false,
  5: true,
  6: false,
};
isPrime[3]; // true
```

Tentu saja, Anda mungkin ingin menghitung daripada menulis secara manual, tetapi ini mengilustrasikan cara berpikir yang berbeda tentang fungsi.

Anda mungkin berpikir "bagaimana dengan fungsi dengan banyak argumen?".

Memang, itu menghadirkan sedikit ketidaknyamanan ketika berpikir dalam hal matematika. Untuk saat ini, kita dapat menggabungkannya dalam sebuah array atau hanya menganggap objek `arguments` sebagai input.

Ketika kita mempelajari tentang _currying_ , kita akan melihat bagaimana kita dapat secara langsung memodelkan definisi matematis dari suatu fungsi.

Inilah pengungkapan dramatisnya: Fungsi murni adalah fungsi matematika dan itulah yang dimaksud dengan pemrograman fungsional.

Pemrograman dengan malaikat kecil ini dapat memberikan manfaat yang sangat besar. Mari kita lihat beberapa alasan mengapa kita rela berusaha keras untuk menjaga kemurnian.

## Kasus untuk Kemurnian

### Dapat disimpan dalam cache

Sebagai permulaan, fungsi murni selalu dapat di-cache dengan input. Ini biasanya dilakukan dengan menggunakan teknik yang disebut memoisasi:

```js
const squareNumber = memoize((x) => x * x);

squareNumber(4); // 16

squareNumber(4); // 16, returns cache for input 4

squareNumber(5); // 25

squareNumber(5); // 25, returns cache for input 5
```

Berikut adalah implementasi yang disederhanakan, meskipun ada banyak versi yang lebih kuat yang tersedia.

```js
const memoize = (f) => {
  const cache = {};

  return (...args) => {
    const argStr = JSON.stringify(args);
    cache[argStr] = cache[argStr] || f(...args);
    return cache[argStr];
  };
};
```

Yang perlu diperhatikan adalah Anda dapat mengubah beberapa fungsi tidak murni menjadi fungsi murni dengan menunda evaluasi:

```js
const pureHttpCall = memoize((url, params) => () => $.getJSON(url, params));
```

Hal yang menarik di sini adalah bahwa kita tidak benar-benar membuat panggilan http - kita malah mengembalikan fungsi yang akan melakukannya saat dipanggil.

Fungsi ini murni karena akan selalu mengembalikan output yang sama dengan input yang sama: fungsi yang akan membuat panggilan http tertentu diberikan `url` dan `params`.

Fungsi `memoize` kami bekerja dengan baik, meskipun tidak men-cache hasil panggilan http, melainkan men-cache fungsi yang dihasilkan.

Ini belum terlalu berguna, tetapi kita akan segera mempelajari beberapa trik yang akan membuatnya begitu. Kesimpulannya adalah kita dapat men-cache setiap fungsi tidak peduli seberapa merusaknya kelihatannya.

### Portabel / Mendokumentasikan diri sendiri

Fungsi murni sepenuhnya mandiri. Semua fungsi yang dibutuhkan diserahkan padanya di atas piring perak.

Renungkan ini sejenak...

Bagaimana ini bisa bermanfaat? Sebagai permulaan, dependensi suatu fungsi bersifat eksplisit dan karenanya lebih mudah dilihat dan dipahami - tidak ada bisnis lucu yang terjadi di bawah tenda.

```js
// impure
const signUp = (attrs) => {
  const user = saveUser(attrs);
  welcomeUser(user);
};

// pure
const signUp = (Db, Email, attrs) => () => {
  const user = saveUser(Db, attrs);
  welcomeUser(Email, user);
};
```

Contoh di sini menunjukkan bahwa fungsi murni harus jujur ​​tentang dependensinya dan, dengan demikian, beri tahu kami apa yang sebenarnya dilakukannya.

Hanya dari tanda tangannya, kita tahu bahwa itu akan menggunakan `Db`, `Email`, dan `attrs` yang harus dikatakan paling sedikit.

Kita akan belajar bagaimana membuat fungsi seperti ini murni tanpa hanya menunda evaluasi, tetapi intinya harus jelas bahwa bentuk murni jauh lebih informatif daripada rekan liciknya yang tidak murni yang terserah siapa yang tahu apa.

Hal lain yang perlu diperhatikan adalah bahwa kami dipaksa untuk "menyuntikkan" dependensi, atau meneruskannya sebagai argumen, yang membuat aplikasi kami jauh lebih fleksibel karena kami telah membuat parameter basis data atau klien email kami atau apa pun yang Anda miliki (jangan khawatir, kita akan melihat cara untuk membuat ini tidak terlalu membosankan daripada kedengarannya).

Jika kita memilih untuk menggunakan Db yang berbeda, kita hanya perlu memanggil fungsi kita dengannya.

Jika kita menemukan diri kita menulis aplikasi baru di mana kita ingin menggunakan kembali fungsi yang dapat diandalkan ini, kita cukup memberikan fungsi ini apa pun Dbdan yang `Email` kita miliki saat itu.

Dalam pengaturan JavaScript, portabilitas dapat berarti membuat serial dan mengirim fungsi melalui soket. Itu bisa berarti menjalankan semua kode aplikasi kami di _web worker_.

Portabilitas adalah sifat yang kuat.

Berlawanan dengan metode dan prosedur "khas" dalam pemrograman imperatif yang berakar jauh di lingkungan mereka melalui status, dependensi, dan efek yang tersedia, fungsi murni dapat dijalankan di mana saja.

Kapan terakhir kali Anda menyalin metode ke aplikasi baru? Salah satu kutipan favorit saya berasal dari pencipta Erlang, Joe Armstrong:

> "Masalah dengan bahasa berorientasi objek adalah mereka memiliki semua lingkungan implisit yang mereka bawa. Anda menginginkan pisang tetapi yang Anda dapatkan adalah gorila yang memegang pisang ... dan seluruh hutan".

### Dapat diuji

Selanjutnya, kami menyadari bahwa fungsi murni membuat pengujian menjadi lebih mudah.

Kami tidak perlu mengejek gateway pembayaran atau pengaturan "nyata" dan menegaskan keadaan dunia setelah setiap pengujian. Kami hanya memberikan input fungsi dan menegaskan output.

Faktanya, kami menemukan komunitas fungsional yang mempelopori alat uji baru yang dapat meledakkan fungsi kami dengan input yang dihasilkan dan menegaskan bahwa properti menahan output.

Ini di luar cakupan buku ini, tetapi saya sangat menganjurkan Anda untuk mencari dan mencoba _Quickcheck_ - alat pengujian yang disesuaikan untuk lingkungan fungsional murni.

### Masuk Akal

Banyak yang percaya bahwa kemenangan terbesar saat bekerja dengan fungsi murni adalah _transparansi referensial_.

Sebuah tempat kode transparan secara referensial ketika dapat diganti dengan nilai yang dievaluasi tanpa mengubah perilaku program.

Karena fungsi murni tidak memiliki efek samping, mereka hanya dapat mempengaruhi perilaku program melalui nilai keluarannya.

Selanjutnya, karena nilai keluarannya dapat dihitung dengan andal hanya dengan menggunakan nilai masukannya, fungsi murni akan selalu menjaga transparansi referensial.

Mari kita lihat contohnya.

```js
const { Map } = require("immutable");

// Aliases: p = player, a = attacker, t = target
const jobe = Map({ name: "Jobe", hp: 20, team: "red" });
const michael = Map({ name: "Michael", hp: 20, team: "green" });
const decrementHP = (p) => p.set("hp", p.get("hp") - 1);
const isSameTeam = (p1, p2) => p1.get("team") === p2.get("team");
const punch = (a, t) => (isSameTeam(a, t) ? t : decrementHP(t));

punch(jobe, michael); // Map({name:'Michael', hp:19, team: 'green'})
```

`decrementHP`, `isSameTeam` dan `punch` semuanya murni dan oleh karena itu transparan secara referensial.

Kita dapat menggunakan teknik yang disebut penalaran persamaan di mana seseorang mengganti "sama dengan sama" dengan alasan tentang kode.

Ini seperti mengevaluasi kode secara manual tanpa memperhitungkan kebiasaan evaluasi terprogram. Menggunakan transparansi referensial, mari kita bermain dengan kode ini sedikit.

Pertama kita akan inline fungsi `isSameTeam`.

```js
const punch = (a, t) => (a.get("team") === t.get("team") ? t : decrementHP(t));
```

Karena data kami tidak dapat diubah, kami cukup mengganti team dengan nilai sebenarnya

```js
const punch = (a, t) => ("red" === "green" ? t : decrementHP(t));
```

Kami melihat bahwa itu salah dalam kasus ini sehingga kami dapat menghapus seluruh cabang if

```js
const punch = (a, t) => decrementHP(t);
```

Dan jika kita inline `decrementHP`, kita melihat bahwa, dalam hal ini, punch menjadi panggilan untuk mengurangi `hp` dengan 1.

```js
const punch = (a, t) => t.set("hp", t.get("hp") - 1);
```

Kemampuan untuk menalar tentang kode ini sangat bagus untuk refactoring dan memahami kode secara umum.

Sebenarnya, kami menggunakan teknik ini untuk memperbaiki program kawanan burung camar kami.

Kami menggunakan penalaran persamaan untuk memanfaatkan sifat-sifat penjumlahan dan perkalian. Memang, kami akan menggunakan teknik ini di seluruh buku ini.

### Kode Paralel

Akhirnya, dan inilah _coup de grâce_, kita dapat menjalankan fungsi murni apa pun secara paralel karena tidak memerlukan akses ke memori bersama dan, menurut definisi, tidak dapat memiliki kondisi balapan karena beberapa efek samping.

Ini sangat mungkin terjadi di lingkungan js sisi server dengan _thread_ serta di browser dengan _web worker_ meskipun budaya saat ini tampaknya menghindarinya karena kerumitan ketika berhadapan dengan fungsi yang tidak murni.

## Singkatnya

Kami telah melihat apa fungsi murni dan mengapa kami sebagai programmer fungsional percaya bahwa itu adalah pakaian malam kucing.

Mulai saat ini, kami akan berusaha untuk menulis semua fungsi kami dengan cara yang murni.

Kami akan memerlukan beberapa alat tambahan untuk membantu kami melakukannya, tetapi sementara itu, kami akan mencoba memisahkan fungsi tidak murni dari sisa kode murni.

Menulis program dengan fungsi murni sedikit melelahkan tanpa beberapa alat tambahan di ikat pinggang kami.

Kami harus menyulap data dengan melewatkan argumen di semua tempat, kami dilarang menggunakan _state_, apalagi efek. Bagaimana cara menulis program masokis ini? Mari kita mendapatkan alat baru yang disebut _curry_ (kari).

[Bab 04: Currying](ch04-id.md)
