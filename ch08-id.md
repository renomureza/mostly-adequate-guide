# Bab 08: Tupperware

## Wadah Perkasa

<img src="images/jar.jpg" alt="http://blog.dwinegar.com/2011/06/another-jar.html" />

Kami telah melihat bagaimana menulis program yang menyalurkan data melalui serangkaian fungsi murni.

Mereka adalah spesifikasi deklaratif dari perilakunya. Tapi bagaimana dengan _control flow_, _error handling_, tindakan asinkron, _state_, berani saya katakan, efek?!

Dalam bab ini, kita akan menemukan fondasi di mana semua abstraksi yang bermanfaat ini dibangun.

Pertama kita akan membuat wadah. Wadah ini harus menampung semua jenis nilai; ziplock yang hanya menampung puding tapioka jarang berguna.

Ini akan menjadi objek, tetapi kami tidak akan memberikan properti dan metode dalam pengertian OO (Object Oriented).

Tidak, kami akan memperlakukannya seperti peti harta karun - kotak khusus yang menampung data berharga kami.

```js
class Container {
  constructor(x) {
    this.$value = x;
  }

  static of(x) {
    return new Container(x);
  }
}
```

Ini wadah pertama kami.

Kami telah menamakannya dengan cermat `Container`. Kami akan menggunakan `Container.of` sebagai konstruktor yang menyelamatkan kami dari keharusan menulis kata kunci `new` yang mengerikan itu di semua tempat.

Ada lebih banyak fungsi `of` daripada yang terlihat, tetapi untuk saat ini, anggap itu sebagai cara yang tepat untuk menempatkan nilai ke dalam wadah kita.

Mari kita periksa kotak baru kami ...

```js
Container.of(3);
// Container(3)

Container.of("hotdogs");
// Container("hotdogs")

Container.of(Container.of({ name: "yoda" }));
// Container(Container({ name: 'yoda' }))
```

Jika Anda menggunakan node, Anda akan melihat `{$value: x}` meskipun kami memiliki `Container(x)`.

Chrome akan menampilkan tipe dengan benar, tetapi tidak masalah; selama kita memahami seperti apa rupa `Container`, kita akan baik-baik saja.

Di beberapa lingkungan Anda dapat menimpa method `inspect` ini jika Anda mau, tetapi kami tidak akan begitu teliti.

Untuk buku ini, kami akan menulis hasil konseptual seolah-olah kami telah ditimpa `inspect` karena jauh lebih instruktif daripada `{$value: x}` alasan pedagogis dan estetika.

Mari kita perjelas beberapa hal sebelum kita melanjutkan:

- `Container` adalah objek dengan satu properti. Banyak kontainer hanya menampung satu hal, meskipun tidak terbatas pada satu. Kami telah secara sewenang-wenang menamai propertinya `$value`.
- `$value` tidak bisa menjadi satu tipe tertentu atau `Container` kami tidak akan sesuai dengan namanya.
- Setelah data masuk ke `Container`, itu tetap di sana. Kita bisa mengeluarkannya dengan menggunakan `.$value`, tetapi itu akan mengalahkan tujuannya.

Alasan kita melakukan ini akan menjadi jelas seperti stoples, tapi untuk saat ini, bersabarlah.

## Functor Pertamaku

Setelah nilai kami, apa pun itu, ada di dalam wadah, kami akan membutuhkan cara untuk menjalankan fungsi di dalamnya.

```js
// (a -> b) -> Container a -> Container b
Container.prototype.map = function (f) {
  return Container.of(f(this.$value));
};
```

Mengapa, itu seperti array `map`, tapi kita memiliki `Container a` alih - alih [a]. Pada dasarnya bekerja dengan cara yang sama:

```js
Container.of(2).map((two) => two + 2);
// Container(4)

Container.of("flamethrowers").map((s) => s.toUpperCase());
// Container('FLAMETHROWERS')

Container.of("bombs").map(append(" away")).map(prop("length"));
// Container(10)
```

Kami dapat bekerja dengan nilai kami tanpa harus meninggalkan `Container`. Ini adalah hal yang luar biasa. Nilai kami di `Container` serahkan ke fungsi `map` sehingga kami bisa repot dengannya dan setelah itu, dikembalikan ke `Container` untuk disimpan dengan aman.

Sebagai hasil dari tidak pernah meninggalkan Container, kita dapat terus `map`, menjalankan fungsi sesuka kita. Kami bahkan dapat mengubah tipenya saat kami melanjutkan seperti yang ditunjukkan pada contoh terakhir dari tiga contoh.

Tunggu sebentar, jika kita terus memanggil `map`, sepertinya itu semacam komposisi! Keajaiban matematika apa yang bekerja di sini? Baiklah, kami baru saja menemukan Functors.

> Functor adalah tipe yang mengimplementasikan `map` dan mematuhi beberapa hukum

Ya, Functor hanyalah antarmuka dengan kontrak. Kita bisa dengan mudah menamakannya _Mappable_, tapi sekarang, di mana kesenangannya?

Fungsi berasal dari teori kategori dan kita akan melihat matematika secara detail menjelang akhir bab ini, tetapi untuk sekarang, mari kita bekerja pada intuisi dan penggunaan praktis untuk antarmuka yang diberi nama aneh ini.

Alasan apa yang mungkin kita miliki untuk membotolkan nilai dan menggunakan `map` untuk mendapatkannya? Jawabannya muncul dengan sendirinya jika kita memilih pertanyaan yang lebih baik: Apa yang kita dapatkan dari meminta wadah kita untuk menerapkan fungsi untuk kita? Nah, abstraksi aplikasi fungsi.

Saat kami menjalankan fungsi `map`, kami meminta jenis wadah untuk menjalankannya untuk kami. Ini adalah konsep yang sangat kuat, memang.

## Mungkin Schrödinger

<img src="images/cat.png" alt="cool cat, need reference" />

`Container` cukup membosankan.

Faktanya, fungsi ini biasanya dipanggil `Identity` dan memiliki dampak yang hampir sama dengan fungsi `id` kita (sekali lagi ada hubungan matematika yang akan kita lihat saat waktunya tepat).

Namun, ada functor lain, yaitu tipe wadah seperti yang memiliki fungsi `map`, yang dapat memberikan perilaku yang berguna saat pemetaan. Mari kita definisikan sekarang.

> Implementasi lengkap ada di [Lampiran B](./appendix_b-id.md#Maybe)

```js
class Maybe {
  static of(x) {
    return new Maybe(x);
  }

  get isNothing() {
    return this.$value === null || this.$value === undefined;
  }

  constructor(x) {
    this.$value = x;
  }

  map(fn) {
    return this.isNothing ? this : Maybe.of(fn(this.$value));
  }

  inspect() {
    return this.isNothing ? "Nothing" : `Just(${inspect(this.$value)})`;
  }
}
```

Sekarang, `Maybe` terlihat sangat mirip `Container` dengan satu perubahan kecil: pertama-tama akan memeriksa untuk melihat apakah ia memiliki nilai sebelum memanggil fungsi yang disediakan.

Ini memiliki efek melangkahi null sial itu seperti map (Perhatikan bahwa implementasi ini disederhanakan untuk pengajaran).

```js
Maybe.of("Malkovich Malkovich").map(match(/a/gi));
// Just(True)

Maybe.of(null).map(match(/a/gi));
// Nothing

Maybe.of({ name: "Boris" }).map(prop("age")).map(add(10));
// Nothing

Maybe.of({ name: "Dinah", age: 14 }).map(prop("age")).map(add(10));
// Just(24)
```

Perhatikan aplikasi kami tidak meledakkan kesalahan saat kami memetakan fungsi di atas nilai null. Ini karena `Maybe` akan berhati-hati untuk memeriksa nilai setiap kali itu menerapkan suatu fungsi.

Sintaks titik ini sangat bagus dan fungsional, tetapi untuk alasan yang disebutkan di Bab 1, kami ingin mempertahankan gaya pointfree. Seperti yang terjadi, `map` dilengkapi sepenuhnya untuk mendelegasikan ke functor apa pun yang diterimanya:

```js
// map :: Functor f => (a -> b) -> f a -> f b
const map = curry((f, anyFunctor) => anyFunctor.map(f));
```

Ini menyenangkan karena kami dapat melanjutkan komposisi seperti biasa dan `map` akan bekerja seperti yang diharapkan.

Ini adalah sama dengan map ramda juga. Kami akan menggunakan notasi titik ketika itu instruktif dan versi pointfree jika itu sesuai.

Apakah Anda memperhatikan itu? Saya diam-diam telah memperkenalkan notasi tambahan ke dalam tanda tangan tipe kami. `Functor f =>` memberitahu kita bahwa `f` harus Functor. Tidak terlalu sulit, tetapi saya merasa harus menyebutkannya.

## Gunakan Kasus

Di alam liar, kita biasanya akan melihat `Maybe` digunakan dalam fungsi yang mungkin gagal mengembalikan hasil.

```js
// safeHead :: [a] -> Maybe(a)
const safeHead = (xs) => Maybe.of(xs[0]);

// streetName :: Object -> Maybe String
const streetName = compose(map(prop("street")), safeHead, prop("addresses"));

streetName({ addresses: [] });
// Nothing

streetName({ addresses: [{ street: "Shady Ln.", number: 4201 }] });
// Just('Shady Ln.')
```

`safeHead` seperti `head` normal, tetapi dengan tambahan keamanan tipe.

Hal yang aneh terjadi ketika `Maybe` dimasukkan ke dalam kode kita; kita dipaksa untuk berurusan dengan nilai-nilai licik null. ungsi `safeHead` jujur dan di depan tentang kegagalan yang mungkin terjadi - ada benar-benar menjadi malu - dan sehingga mengembalikan `Maybe` untuk menginformasikan kepada kami tentang hal ini.

Bagaimanapun lebih dari sekedar informasi, karena kami dipaksa mapuntuk mendapatkan nilai yang kami inginkan karena tersimpan di dalam objek `Maybe`.

Pada dasarnya, ini adalah pemeriksaan `null` yang diberlakukan oleh fungsi `safeHead` itu sendiri.

Kita sekarang bisa tidur lebih nyenyak di malam hari mengetahui nilai `null` tidak akan mengangkat kepalanya yang jelek dan dipenggal ketika kita tidak mengharapkannya.

API seperti ini akan meningkatkan aplikasi tipis dari kertas dan paku ke kayu dan paku. Mereka akan menjamin perangkat lunak yang lebih aman.

Terkadang suatu fungsi dapat mengembalikan `Nothing` eksplisit ke sinyal kegagalan. Contohnya:

```js
// withdraw :: Number -> Account -> Maybe(Account)
const withdraw = curry((amount, { balance }) =>
  Maybe.of(balance >= amount ? { balance: balance - amount } : null)
);

// This function is hypothetical, not implemented here... nor anywhere else.
// updateLedger :: Account -> Account
const updateLedger = (account) => account;

// remainingBalance :: Account -> String
const remainingBalance = ({ balance }) => `Your balance is $${balance}`;

// finishTransaction :: Account -> String
const finishTransaction = compose(remainingBalance, updateLedger);

// getTwenty :: Account -> Maybe(String)
const getTwenty = compose(map(finishTransaction), withdraw(20));

getTwenty({ balance: 200.0 });
// Just('Your balance is $180')

getTwenty({ balance: 10.0 });
// Nothing
```

`withdraw` akan mengarahkan hidungnya pada kita dan mengembalikan `Nothing` jika kita kekurangan uang.

Fungsi ini juga mengomunikasikan ketidakstabilannya dan membuat kita tidak punya pilihan, selain `map` semuanya setelahnya.

Perbedaannya adalah bahwa `null` itu disengaja di sini. Alih-alih `Just('..')`, kami mendapatkan kembalian `Nothing` ke sinyal kegagalan dan aplikasi kami secara efektif berhenti di jalurnya.

Ini penting untuk diperhatikan: jika `withdraw` gagal, maka `map` akan memutuskan sisa perhitungan kita karena tidak pernah menjalankan fungsi yang dipetakan, yaitu `finishTransaction`.

Inilah perilaku yang dimaksudkan karena kami lebih memilih untuk tidak memperbarui _ledger_ atau menunjukkan saldo baru jika tidak berhasil _withdrawn_.

## Melepaskan Nilai

Satu hal yang sering dilewatkan orang adalah bahwa akan selalu ada akhir; beberapa fungsi yang mempengaruhi yang mengirimkan JSON, atau mencetak ke layar, atau mengubah sistem file kami, atau apa pun yang Anda miliki.

Kami tidak dapat mengirimkan output dengan `return`, kami harus menjalankan beberapa fungsi atau lainnya untuk mengirimkannya ke dunia.

Kita dapat mengungkapkannya seperti koan Buddhis Zen: "Jika sebuah program tidak memiliki efek yang dapat diamati, apakah program itu akan berjalan?". Apakah itu berjalan dengan benar untuk kepuasannya sendiri? Saya menduga itu hanya membakar beberapa siklus dan kembali tidur ...

Tugas aplikasi kita adalah mengambil, mengubah, dan membawa data itu hingga tiba waktunya untuk mengucapkan selamat tinggal dan fungsi yang melakukannya dapat dipetakan, sehingga nilainya tidak perlu meninggalkan wadahnya yang hangat.

Memang, kesalahan umum adalah mencoba untuk menghapus nilai dari `Maybe` satu atau lain cara kita seolah-olah nilai yang mungkin di dalam tiba-tiba terwujud dan semua akan dimaafkan.

Kita harus memahami bahwa ini mungkin sebuah cabang kode di mana nilai kita tidak ada untuk memenuhi takdirnya.

Kode kita, seperti kucing Schrödinger, berada di dua status sekaligus dan harus mempertahankan fakta itu hingga fungsi terakhir. Ini memberi kode kita aliran linier meskipun ada percabangan logis.

Namun, ada pintu pelarian. Jika kita lebih suka mengembalikan nilai khusus dan melanjutkan, kita dapat menggunakan pembantu kecil yang disebut `maybe`.

```js
// maybe :: b -> (a -> b) -> Maybe a -> b
const maybe = curry((v, f, m) => {
  if (m.isNothing) {
    return v;
  }

  return f(m.$value);
});

// getTwenty :: Account -> String
const getTwenty = compose(
  maybe("You're broke!", finishTransaction),
  withdraw(20)
);

getTwenty({ balance: 200.0 });
// 'Your balance is $180.00'

getTwenty({ balance: 10.0 });
// 'You\'re broke!'
```

Kami sekarang akan mengembalikan nilai statis (dari jenis yang sama yang dikembalikan `finishTransaction`) atau melanjutkan dengan gembira menyelesaikan transaksi sans `Maybe`. Dengan `maybe`, kita menyaksikan padanan `if/else` _statement_ sedangkan dengan map, analog imperatifnya adalah: `if (x !== null) { return f(x) }`.

Pengenalan awal `Maybe` dapat menyebabkan beberapa ketidaknyamanan. Pengguna `Swift` dan `Scala` akan tahu apa yang saya maksud karena dimasukkan langsung ke perpustakaan inti dengan kedok `Option(al)`.

Ketika didorong untuk berurusan dengan `null` cek sepanjang waktu (dan ada kalanya kita tahu dengan pasti bahwa nilainya ada), kebanyakan orang tidak bisa tidak merasa itu agak melelahkan.

Namun, seiring waktu, itu akan menjadi kebiasaan dan Anda mungkin akan menghargai keamanannya. Lagi pula, sebagian besar waktu itu akan mencegah jalan pintas dan menyelamatkan kulit kita.

Menulis perangkat lunak yang tidak aman seperti berhati-hati untuk mengecat setiap telur dengan pastel sebelum melemparkannya ke lalu lintas; seperti membangun rumah jompo dengan bahan-bahan yang diperingatkan oleh tiga babi kecil.

Ada baiknya kita menempatkan beberapa keamanan ke dalam fungsi kita dan `Maybe` membantu kita melakukan hal itu.

Saya akan lalai jika saya tidak menyebutkan bahwa implementasi "nyata" akan membagi Maybe menjadi dua jenis: satu untuk sesuatu dan yang lainnya untuk apa-apa.

Hal ini memungkinkan kita untuk mematuhi parametrik `map` sehingga nilai-nilai seperti `null` dan `undefined` masih dapat dipetakan dan kualifikasi universal nilai dalam sebuah functor akan dihormati.

Anda akan sering melihat jenis seperti `Some(x) / None` atau `Just(x) / Nothing` alih-alih `Maybe` yang melakukan pemeriksaan `null` pada nilainya.

## Penanganan Kesalahan Murni

<img src="images/fists.jpg" alt="pick a hand... need a reference" />

Ini mungkin datang sebagai kejutan, tetapi `throw/catch` tidak terlalu murni.

Saat terjadi kesalahan, alih-alih mengembalikan nilai keluaran, kami membunyikan alarm!

Fungsinya menyerang, memuntahkan ribuan 0 dan 1 seperti perisai dan tombak dalam pertempuran listrik melawan input pengganggu kami.

Dengan teman baru `Either`, kita bisa berbuat lebih baik daripada menyatakan perang terhadap masukan, kita bisa membalas dengan pesan yang sopan. Mari lihat:

> Implementasi lengkap tersedi di [Lampiran B](./appendix_b-id.md#Either)

```js
class Either {
  static of(x) {
    return new Right(x);
  }

  constructor(x) {
    this.$value = x;
  }
}

class Left extends Either {
  map(f) {
    return this;
  }

  inspect() {
    return `Left(${inspect(this.$value)})`;
  }
}

class Right extends Either {
  map(f) {
    return Either.of(f(this.$value));
  }

  inspect() {
    return `Right(${inspect(this.$value)})`;
  }
}

const left = (x) => new Left(x);
```

`Left` dan `Right` merupakan dua subkelas dari tipe abstrak yang kita sebut `Either`. Saya telah melewatkan upacara pembuatan superclass `Either` karena kita tidak akan pernah menggunakannya, tetapi ada baiknya untuk berhati-hati.

Nah, tidak ada yang baru di sini selain kedua jenis itu. Mari kita lihat bagaimana mereka bertindak:

```js
Either.of("rain").map((str) => `b${str}`);
// Right('brain')

left("rain").map((str) => `It's gonna ${str}, better bring your umbrella!`);
// Left('rain')

Either.of({ host: "localhost", port: 80 }).map(prop("host"));
// Right('localhost')

left("rolls eyes...").map(prop("host"));
// Left('rolls eyes...')
```

`Left` adalah jenis remaja dan mengabaikan permintaan map untuk mengatasinya. `Right` akan bekerja seperti `Container`(alias Identity). Kekuatan berasal dari kemampuan untuk menyematkan pesan kesalahan di dalam `Left`.

Misalkan kita memiliki fungsi yang mungkin tidak berhasil. Bagaimana kalau kita menghitung usia dari tanggal lahir.

Kita dapat menggunakan `Nothing` untuk menandakan kegagalan dan mencabangkan program kita, namun, itu tidak banyak memberi tahu kita. Mungkin, kami ingin tahu mengapa itu gagal.

Mari kita tulis ini menggunakan `Either`.

```js
const moment = require("moment");

// getAge :: Date -> User -> Either(String, Number)
const getAge = curry((now, user) => {
  const birthDate = moment(user.birthDate, "YYYY-MM-DD");

  return birthDate.isValid()
    ? Either.of(now.diff(birthDate, "years"))
    : left("Birth date could not be parsed");
});

getAge(moment(), { birthDate: "2005-12-12" });
// Right(9)

getAge(moment(), { birthDate: "July 4, 2001" });
// Left('Birth date could not be parsed')
```

Sekarang, sama seperti `Nothing`, kami melakukan hubungan arus pendek aplikasi kami ketika kami mengembalikan `Left`. Bedanya, sekarang kita sudah tahu kenapa program kita kandas.

Sesuatu yang perlu diperhatikan adalah bahwa kita mengembalikan `Either(String, Number)`, yang memegang `String` sebagai nilai kirinya dan `Number` sebagai milik `Right`.

Tanda tangan tipe ini agak informal karena kami belum meluangkan waktu untuk mendefinisikan superclass Either yang sebenarnya, namun kami belajar banyak dari tipe tersebut. Ini memberi tahu kami bahwa kami menerima pesan kesalahan atau mengembalikan usia.

```js
// fortune :: Number -> String
const fortune = compose(
  concat("If you survive, you will be "),
  toString,
  add(1)
);

// zoltar :: User -> Either(String, _)
const zoltar = compose(map(console.log), map(fortune), getAge(moment()));

zoltar({ birthDate: "2005-12-12" });
// 'If you survive, you will be 10'
// Right(undefined)

zoltar({ birthDate: "balloons!" });
// Left('Birth date could not be parsed')
```

Ketika `birthDate` valid, program menampilkan keberuntungan mistiknya ke layar untuk kita lihat.

Jika tidak, kami diberi `Left` dengan pesan kesalahan biasa seperti hari meskipun masih tersimpan di wadahnya.

Itu bertindak seolah-olah kita telah melakukan kesalahan, tetapi dengan cara yang tenang dan lembut sebagai lawan dari kehilangan kesabaran dan berteriak seperti anak kecil ketika terjadi kesalahan.

Dalam contoh ini, kami secara logis mencabangkan aliran kontrol kami tergantung pada validitas tanggal lahir, namun membaca sebagai satu gerakan linier dari kanan ke kiri daripada mendaki melalui kurung kurawal pernyataan bersyarat.

Biasanya, kita akan memindahkan fungsi `console.log` `zoltar` dan fungsi `map` itu pada saat pemanggilan, tetapi akan sangat membantu untuk melihat perbedaan cabang `Right`.

Kami menggunakan \_ tanda tangan tipe cabang kanan untuk menunjukkan itu adalah nilai yang harus diabaikan (Di beberapa browser Anda harus menggunakannya `console.log.bind(console)` untuk menggunakan kelas pertama).

Saya ingin mengambil kesempatan ini untuk menunjukkan sesuatu yang mungkin Anda lewatkan: `fortune`, meskipun digunakan dengan `Either` dalam contoh ini, sama sekali tidak mengetahui fungsi apa pun yang berseliweran.

Hal ini juga terjadi pada `finishTransaction` contoh sebelumnya. Pada saat pemanggilan, suatu fungsi dapat dikelilingi oleh `map`, yang mengubahnya dari fungsi non-fungsional menjadi fungsi fungsional, dalam istilah informal.

Kami menyebut proses ini _lifting_. Fungsi cenderung lebih baik bekerja dengan tipe data normal daripada tipe wadah, kemudian diangkat ke wadah yang tepat jika dianggap perlu.

Ini mengarah ke fungsi yang lebih sederhana dan dapat digunakan kembali yang dapat diubah untuk bekerja dengan fungsi apa pun sesuai permintaan.

`Either` sangat bagus untuk kesalahan biasa seperti validasi serta yang lebih serius, menghentikan kesalahan acara seperti file yang hilang atau soket yang rusak. Coba ganti beberapa contoh `Maybe` dengan `Either` untuk memberikan umpan balik yang lebih baik.

Sekarang, saya tidak bisa tidak merasa telah melakukan tindakan `Either` merugikan dengan memperkenalkannya hanya sebagai wadah untuk pesan kesalahan. Ini menangkap disjungsi logis (alias `||`) dalam suatu tipe.

Ini juga mengkodekan ide `Coproduct` dari teori kategori, yang tidak akan disinggung dalam buku ini, tetapi layak dibaca karena ada properti yang bisa dieksploitasi.

Ini adalah tipe jumlah kanonik (atau penyatuan himpunan yang terputus-putus) karena jumlah penghuninya yang mungkin adalah jumlah dari dua tipe yang terkandung (saya tahu itu agak bergelombang, jadi inilah [artikel yang bagus](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/sum-types)).

Ada banyak hal yang bisa dilakukan `Either`, tetapi sebagai functor, digunakan untuk penanganan kesalahannya.

Sama seperti `Maybe`, kami memiliki sedikit `either`, yang berperilaku serupa, tetapi mengambil dua fungsi alih-alih satu dan nilai statis. Setiap fungsi harus mengembalikan tipe yang sama:

```js
// either :: (a -> c) -> (b -> c) -> Either a b -> c
const either = curry((f, g, e) => {
  let result;

  switch (e.constructor) {
    case Left:
      result = f(e.$value);
      break;

    case Right:
      result = g(e.$value);
      break;

    // No Default
  }

  return result;
});

// zoltar :: User -> _
const zoltar = compose(console.log, either(id, fortune), getAge(moment()));

zoltar({ birthDate: "2005-12-12" });
// 'If you survive, you will be 10'
// undefined

zoltar({ birthDate: "balloons!" });
// 'Birth date could not be parsed'
// undefined
```

Akhirnya, penggunaan untuk fungsi `id` misterius itu. Itu hanya mengembalikan nilai dalam `Left` untuk meneruskan pesan kesalahan ke `console.log`.

Kami telah membuat aplikasi meramal kami lebih kuat dengan menerapkan penanganan kesalahan dari dalam `getAge`.

Kami menampar pengguna dengan kebenaran yang keras seperti tos dari pembaca telapak tangan atau kami melanjutkan proses kami. Dan dengan itu, kami siap untuk beralih ke jenis functor yang sama sekali berbeda.

## McDonald Lama Memiliki Efek...

<img src="images/dominoes.jpg" alt="dominoes.. need a reference" />

Dalam bab kita tentang kemurnian, kita melihat contoh khusus dari fungsi murni. Fungsi ini mengandung efek samping, tetapi kami menjulukinya murni dengan membungkus aksinya dalam fungsi lain. Berikut contoh lain dari ini:

```js
// getFromStorage :: String -> (_ -> String)
const getFromStorage = (key) => () => localStorage[key];
```

Seandainya kita tidak mengepung keberanian dalam fungsi lain, `getFromStorage` akan memvariasikan outputnya tergantung pada keadaan eksternal.

Dengan pembungkus kokoh di tempat, kita akan selalu mendapatkan output yang sama per input: fungsi yang, ketika dipanggil, akan mengambil item tertentu dari `localStorage`.

Dan begitu saja (mungkin memberikan beberapa Salam Maria) kita telah membersihkan hati nurani kita dan semuanya diampuni.

Selain itu, ini tidak terlalu berguna sekarang. Seperti koleksi _action figure_ dalam kemasan aslinya, kita tidak bisa benar-benar bermain dengannya.

Kalau saja ada cara untuk mencapai bagian dalam wadah dan mendapatkan isinya... Masuk `IO`.

```js
class IO {
  static of(x) {
    return new IO(() => x);
  }

  constructor(fn) {
    this.$value = fn;
  }

  map(fn) {
    return new IO(compose(fn, this.$value));
  }

  inspect() {
    return `IO(${inspect(this.$value)})`;
  }
}
```

`IO` berbeda dari fungsi sebelumnya karena `$value` selalu merupakan fungsi.

Namun, kami tidak menganggap `$value` sebagai fungsi - itu adalah detail implementasi dan sebaiknya kami mengabaikannya.

Apa yang terjadi persis seperti yang kita lihat dengan contoh `getFromStorage`: `IO` menunda tindakan tidak murni dengan menangkapnya dalam pembungkus fungsi.

Karena itu, kami menganggap `IO` berisi nilai balik dari tindakan yang dibungkus dan bukan pembungkus itu sendiri.

Ini terlihat dalam fungsi `of`: kita memiliki `IO(x)`, `IO(() => x)` itu hanya diperlukan untuk menghindari evaluasi.

Perhatikan bahwa, untuk menyederhanakan bacaan, kami akan menunjukkan nilai hipotetis yang terkandung dalam IO sebagai hasil; namun dalam praktiknya, Anda tidak dapat mengetahui berapa nilai ini sampai Anda benar-benar melepaskan efeknya!

Mari kita lihat penggunaannya:

```js
// ioWindow :: IO Window
const ioWindow = new IO(() => window);

ioWindow.map((win) => win.innerWidth);
// IO(1430)

ioWindow.map(prop("location")).map(prop("href")).map(split("/"));
// IO(['http:', '', 'localhost:8000', 'blog', 'posts'])

// $ :: String -> IO [DOM]
const $ = (selector) => new IO(() => document.querySelectorAll(selector));

$("#myDiv")
  .map(head)
  .map((div) => div.innerHTML);
// IO('I am some inner html')
```

Di sini, `ioWindow` adalah `IO` aktual yang dapat map langsung saat itu juga, sedangkan `$` adalah fungsi yang mengembalikan `IO` setelah dipanggil.

Saya telah menulis nilai pengembalian konseptual untuk mengekspresikan IO, meskipun, pada kenyataannya, itu akan selalu menjadi `{ $value: [Function] }`.

Ketika `map` melewati `IO`, kita menempelkan fungsi itu di akhir komposisi yang pada gilirannya menjadi `$value` yang baru dan seterusnya.

Fungsi kami yang dipetakan tidak berjalan, mereka ditempelkan di akhir komputasi yang kami bangun, fungsi demi fungsi, seperti menempatkan kartu domino dengan hati-hati yang tidak berani kami jungkirbalikkan.

Hasilnya mengingatkan pada pola Gang of Four's atau _queue_.

Luangkan waktu sejenak untuk menyalurkan intuisi functor Anda. Jika kita melihat melewati detail implementasi, kita akan merasa nyaman saat memetakan wadah apa pun, terlepas dari kebiasaan atau keanehannya.

Kami memiliki hukum functor, yang akan kami jelajahi menjelang akhir bab ini, untuk berterima kasih atas kekuatan pseudo-psikis ini.

Bagaimanapun, kita akhirnya bisa bermain dengan nilai-nilai yang tidak murni tanpa mengorbankan kemurnian kita yang berharga.

Sekarang, kami telah mengurung binatang itu, tetapi kami masih harus membebaskannya di beberapa titik. Pemetaan atas `IO` telah membangun perhitungan yang tidak murni dan menjalankannya pasti akan mengganggu kedamaian.

Jadi di mana dan kapan kita bisa menarik pelatuknya?

Apakah mungkin untuk menjalankan kami IOdan masih mengenakan pakaian putih di pernikahan kami?

Jawabannya adalah ya, jika kita menempatkan tanggung jawab pada kode panggilan. Kode murni kami, terlepas dari plot dan skema jahat, mempertahankan kepolosannya dan pemanggillah yang dibebani dengan tanggung jawab untuk benar-benar menjalankan efeknya.

Mari kita lihat contoh untuk membuat ini konkret.

```js
// url :: IO String
const url = new IO(() => window.location.href);

// toPairs :: String -> [[String]]
const toPairs = compose(map(split("=")), split("&"));

// params :: String -> [[String]]
const params = compose(toPairs, last, split("?"));

// findParam :: String -> IO Maybe [String]
const findParam = (key) =>
  map(compose(Maybe.of, find(compose(eq(key), head)), params), url);

// -- Impure calling code ----------------------------------------------

// run it by calling $value()!
findParam("searchTerm").$value();
// Just(['searchTerm', 'wafflehouse'])
```

Perpustakaan kami menjaga tangan yang bersih dengan pembungkus `url` dalam `IO` dan lewat dolar ke pemanggil.

Anda mungkin juga memperhatikan bahwa kami telah menumpuk wadah kami; sangat masuk akal untuk memiliki `IO(Maybe([x]))`, yang merupakan tiga functors dalam (`Array` paling jelas merupakan jenis wadah yang dapat dipetakan) dan sangat ekspresif.

Ada sesuatu yang mengganggu saya dan kita harus segera memperbaikinya: `$value` `IO` tidak benar-benar mengandung nilai, juga bukan milik pribadi.

Ini adalah pin di granat dan dimaksudkan untuk ditarik oleh pemanggil dengan cara yang paling umum. Mari kita ganti nama properti ini `unsafePerformIO` untuk mengingatkan pengguna kita tentang volatilitasnya.

```js
class IO {
  constructor(io) {
    this.unsafePerformIO = io;
  }

  map(fn) {
    return new IO(compose(fn, this.unsafePerformIO));
  }
}
```

Di sana, jauh lebih baik. Sekarang kode panggilan kami menjadi `findParam('searchTerm').unsafePerformIO()`, yang jelas bagi pengguna (dan pembaca) aplikasi.

`IO` akan menjadi teman setia, membantu kami menjinakkan tindakan tidak murni liar itu. Selanjutnya, kita akan melihat tipe yang serupa dalam semangat, tetapi memiliki kasus penggunaan yang sangat berbeda.

## Tugas Asinkron

Callback adalah tangga spiral yang menyempit ke neraka.

Mereka adalah aliran kontrol seperti yang dirancang oleh MC Escher.

Dengan setiap panggilan balik bersarang terjepit di antara gym hutan kurung kurawal dan kurung, mereka merasa seperti limbo dalam oubliette (seberapa rendah kita bisa pergi?!).

Aku menggigil kedinginan hanya dengan memikirkan mereka. Tidak perlu khawatir, kami memiliki cara yang jauh lebih baik untuk menangani kode asinkron dan dimulai dengan "F".

Bagian dalamnya agak terlalu rumit untuk diungkapkan ke seluruh halaman di sini jadi kita akan menggunakan `Data.Task`(sebelumnya `Data.Future`) dari Cerita Rakyat fantastis Quildreen Motta.

Lihatlah beberapa contoh penggunaan:

```js
// -- Node readFile example ------------------------------------------

const fs = require("fs");

// readFile :: String -> Task Error String
const readFile = (filename) =>
  new Task((reject, result) => {
    fs.readFile(filename, (err, data) => (err ? reject(err) : result(data)));
  });

readFile("metamorphosis").map(split("\n")).map(head);
// Task('One morning, as Gregor Samsa was waking up from anxious dreams, he discovered that
// in bed he had been changed into a monstrous verminous bug.')

// -- jQuery getJSON example -----------------------------------------

// getJSON :: String -> {} -> Task Error JSON
const getJSON = curry(
  (url, params) =>
    new Task((reject, result) => {
      $.getJSON(url, params, result).fail(reject);
    })
);

getJSON("/video", { id: 10 }).map(prop("title"));
// Task('Family Matters ep 15')

// -- Default Minimal Context ----------------------------------------

// We can put normal, non futuristic values inside as well
Task.of(3).map((three) => three + 1);
// Task(4)
```

Fungsi yang saya panggil, `reject` dan `result`, merupakan callback kesalahan dan keberhasilan, masing-masing.

Seperti yang Anda lihat, kita hanya `map` atas `Task` untuk bekerja pada nilai masa depan seolah-olah itu benar ada di genggaman kita. Sekarang `map` harus topi tua.

Jika Anda akrab dengan promise, Anda mungkin mengenali fungsi `map` sebagai `then` dengan `Task` memainkan peran janji kami. Jangan khawatir jika Anda tidak terbiasa dengan promise, kami tidak akan menggunakannya karena tidak murni, tetapi analogi tetap berlaku.

Seperti `IO`, `Task` akan sabar menunggu kita memberi lampu hijau sebelum berlari.

Faktanya, karena menunggu perintah kita, `IO` secara efektif dimasukkan oleh Task untuk semua hal yang tidak sinkron; `readFile` dan `getJSON` tidak memerlukan wadah `IO` tambahan untuk menjadi murni.

Terlebih lagi, `Task` bekerja dengan cara yang sama ketika kita map lebih: kita menempatkan instruksi untuk masa depan seperti bagan tugas dalam kapsul waktu - tindakan penundaan teknologi yang canggih.

Untuk menjalankan `Task`, kita harus memanggil metode `fork`. Ini berfungsi seperti `unsafePerformIO`, tetapi seperti namanya, itu akan memotong proses kami dan evaluasi berlanjut tanpa memblokir utas kami.

Ini dapat diimplementasikan dalam banyak cara dengan utas dan semacamnya, tetapi di sini ia bertindak seperti panggilan asinkron normal dan roda besar dari loop acara terus berputar.

Mari kita lihat `fork`:

```js
// -- Pure application -------------------------------------------------
// blogPage :: Posts -> HTML
const blogPage = Handlebars.compile(blogTemplate);

// renderPage :: Posts -> HTML
const renderPage = compose(blogPage, sortBy(prop("date")));

// blog :: Params -> Task Error HTML
const blog = compose(map(renderPage), getJSON("/posts"));

// -- Impure calling code ----------------------------------------------
blog({}).fork(
  (error) => $("#error").html(error.message),
  (page) => $("#main").html(page)
);

$("#spinner").show();
```

Setelah memanggil `fork`, `Task` bergegas untuk menemukan beberapa postingan dan membuat halaman.

Sementara itu, kami menunjukkan _spinner_ karena `fork` tidak menunggu tanggapan. Terakhir, kami akan menampilkan kesalahan atau merender halaman ke layar tergantung apakah panggilan `getJSON` berhasil atau tidak.

Luangkan waktu sejenak untuk mempertimbangkan seberapa linier aliran kontrol di sini. Kami hanya membaca dari bawah ke atas, kanan ke kiri meskipun program sebenarnya akan melompat-lompat sedikit selama eksekusi.

Hal ini membuat pembacaan dan penalaran tentang aplikasi kita menjadi lebih sederhana daripada harus berpindah-pindah antara callback dan blok penanganan kesalahan.

Astaga, maukah Anda melihat itu, `Task` juga telah tertelan `Either`! Itu harus dilakukan untuk menangani kegagalan futuristik karena aliran kontrol normal kami tidak berlaku di dunia async.

Ini semua baik dan bagus karena memberikan penanganan kesalahan yang cukup dan murni di luar kotak.

Bahkan dengan `Task`, `IO` dan `Either` functors tidak keluar dari task. Bersabarlah dengan saya pada contoh cepat yang condong ke sisi yang lebih kompleks dan hipotetis, tetapi berguna untuk tujuan ilustrasi.

```js
// Postgres.connect :: Url -> IO DbConnection
// runQuery :: DbConnection -> ResultSet
// readFile :: String -> Task Error String

// -- Pure application -------------------------------------------------

// dbUrl :: Config -> Either Error Url
const dbUrl = ({ uname, pass, host, db }) => {
  if (uname && pass && host && db) {
    return Either.of(`db:pg://${uname}:${pass}@${host}5432/${db}`);
  }

  return left(Error("Invalid config!"));
};

// connectDb :: Config -> Either Error (IO DbConnection)
const connectDb = compose(map(Postgres.connect), dbUrl);

// getConfig :: Filename -> Task Error (Either Error (IO DbConnection))
const getConfig = compose(map(compose(connectDb, JSON.parse)), readFile);

// -- Impure calling code ----------------------------------------------

getConfig("db.json").fork(
  logErr("couldn't read file"),
  either(console.log, map(runQuery))
);
```

Dalam contoh ini, kami masih menggunakan `Either` dan `IO` dari dalam cabang sukses `readFile`. `Task` menangani ketidakmurnian membaca file secara asinkron, tetapi kami masih berurusan dengan memvalidasi konfigurasi dengan `Either` dan memperdebatkan koneksi db dengan `IO`.

Jadi Anda lihat, kami masih dalam bisnis untuk semua hal yang sinkron.

Aku bisa terus, tapi itu semua ada untuk itu. Sederhana seperti `map`.

Dalam praktiknya, Anda mungkin akan memiliki beberapa tugas asinkron dalam satu alur kerja dan kami belum memperoleh api penampung penuh untuk menangani skenario ini.

Tidak perlu khawatir, kita akan segera melihat monad dan semacamnya, tetapi pertama-tama, kita harus memeriksa matematika yang memungkinkan ini semua.

## Titik Teori

Seperti disebutkan sebelumnya, functors berasal dari teori kategori dan memenuhi beberapa hukum. Pertama-tama mari kita jelajahi properti yang berguna ini.

```js
// identity
map(id) === id;

// composition
compose(map(f), map(g)) === map(compose(f, g));
```

Hukum identitas sederhana, namun penting. Hukum ini adalah bit kode yang dapat dijalankan sehingga kami dapat mencobanya pada fungsi kami sendiri untuk memvalidasi legitimasinya.

```js
const idLaw1 = map(id);
const idLaw2 = id;

idLaw1(Container.of(2)); // Container(2)
idLaw2(Container.of(2)); // Container(2)
```

Anda lihat, mereka setara. Selanjutnya mari kita lihat komposisinya.

```js
const compLaw1 = compose(map(append(" world")), map(append(" cruel")));
const compLaw2 = map(compose(append(" world"), append(" cruel")));

compLaw1(Container.of("Goodbye")); // Container('Goodbye cruel world')
compLaw2(Container.of("Goodbye")); // Container('Goodbye cruel world')
```

Dalam teori kategori, functor mengambil objek dan morfisme dari suatu kategori dan memetakannya ke kategori yang berbeda.

Menurut definisi, kategori baru ini harus memiliki identitas dan kemampuan untuk menyusun morfisme, tetapi kita tidak perlu memeriksanya karena hukum yang disebutkan di atas memastikan ini dipertahankan.

Mungkin definisi kategori kita masih agak kabur.

Anda dapat menganggap kategori sebagai jaringan objek dengan morfisme yang menghubungkannya.

Jadi, functor akan memetakan satu kategori ke kategori lainnya tanpa memutus jaringan.

Jika sebuah objek `a` berada dalam kategori sumber kami `C`, ketika kami memetakannya ke kategori `D` dengan functor `F`, kami menyebut objek itu sebagai `F a` (Jika Anda menggabungkannya, apa artinya itu?!). Mungkin, lebih baik melihat diagram:

<img src="images/catmap.png" alt="Categories mapped" />

Misalnya, `Maybe` memetakan tipe kategori dan fungsi kami ke kategori di mana setiap objek mungkin tidak ada dan setiap morfisme memiliki tanda centang null.

Kami mencapai ini dalam kode dengan mengelilingi setiap fungsi dengan `map` dan setiap jenis dengan functor.

Kita tahu bahwa masing-masing tipe dan fungsi normal kita akan terus menyusun di dunia baru ini.

Secara teknis, setiap fungsi dalam kode kami memetakan ke sub jenis kategori dan fungsi yang membuat semua fungsi menjadi merek tertentu yang disebut endofunctors, tetapi untuk tujuan kami, kami akan menganggapnya sebagai kategori yang berbeda.

Kami juga dapat memvisualisasikan pemetaan morfisme dan objek yang sesuai dengan diagram ini:

<img src="images/functormap.png" alt="functor diagram" />

Selain memvisualisasikan morfisme yang dipetakan dari satu kategori ke kategori lain di bawah functor `F`, kita melihat bahwa diagram itu berubah-ubah, yaitu, jika Anda mengikuti panah, setiap rute menghasilkan hasil yang sama.

Rute yang berbeda berarti perilaku yang berbeda, tetapi kita selalu berakhir pada tipe yang sama.

Formalisme ini memberi kita cara berprinsip untuk menalar tentang kode kita - kita dapat dengan berani menerapkan rumus tanpa harus menguraikan dan memeriksa setiap skenario individu.

Mari kita ambil contoh konkret.

```js
// topRoute :: String -> Maybe String
const topRoute = compose(Maybe.of, reverse);

// bottomRoute :: String -> Maybe String
const bottomRoute = compose(map(reverse), Maybe.of);

topRoute("hi"); // Just('ih')
bottomRoute("hi"); // Just('ih')
```

Atau secara visual:

<img src="images/functormapmaybe.png" alt="functor diagram 2" />

Kita dapat langsung melihat dan memfaktorkan ulang kode berdasarkan properti yang dimiliki oleh semua fungsi.

Functor dapat menumpuk:

```js
const nested = Task.of([Either.of("pillows"), left("no sleep for you")]);

map(map(map(toUpperCase)), nested);
// Task([Right('PILLOWS'), Left('no sleep for you')])
```

Apa yang kita miliki di sini `nested` adalah array elemen masa depan yang mungkin error. Kami `map` mengupas kembali setiap lapisan dan menjalankan fungsi kami pada elemen.

Kami tidak melihat callback, if/else, atau for loop; konteks eksplisit saja. Kami, bagaimanapun, harus `map(map(map(f)))`. Sebagai gantinya, kita dapat menyusun functors. Anda mendengar saya dengan benar:

```js
class Compose {
  constructor(fgx) {
    this.getCompose = fgx;
  }

  static of(fgx) {
    return new Compose(fgx);
  }

  map(fn) {
    return new Compose(map(map(fn), this.getCompose));
  }
}

const tmd = Task.of(Maybe.of("Rock over London"));

const ctmd = Compose.of(tmd);

const ctmd2 = map(append(", rock on, Chicago"), ctmd);
// Compose(Task(Just('Rock over London, rock on, Chicago')))

ctmd2.getCompose;
// Task(Just('Rock over London, rock on, Chicago'))
```

Di sana, satu `map`. Komposisi functor adalah asosiatif dan sebelumnya, kami mendefinisikan Container, yang sebenarnya disebut `Identity` functor.

Jika kita memiliki identitas dan komposisi asosiatif, kita memiliki kategori. Kategori khusus ini memiliki kategori sebagai objek dan berfungsi sebagai morfisme, yang cukup untuk membuat otak seseorang berkeringat.

Kami tidak akan menyelidiki terlalu jauh ke dalam ini, tapi itu bagus untuk menghargai implikasi arsitektural atau bahkan hanya keindahan abstrak sederhana dalam polanya.

## Singkatnya

Kami telah melihat beberapa fungsi yang berbeda, tetapi ada banyak sekali.

Beberapa kelalaian penting adalah struktur data yang dapat diubah seperti _trees_, _lists_, _maps_, _pairs_, apa saja.

Aliran acara dan yang dapat diamati keduanya berfungsi. Yang lain bisa untuk enkapsulasi atau bahkan hanya mengetik pemodelan.

Functor ada di sekitar kita dan kita akan menggunakannya secara ekstensif di sepanjang buku ini.

Bagaimana dengan memanggil fungsi dengan beberapa argumen functor? Bagaimana dengan bekerja dengan urutan tindakan tidak murni atau tidak sinkron? Kami belum memperoleh set alat lengkap untuk bekerja di dunia kotak ini.

Selanjutnya, kita akan memotong langsung ke pengejaran dan melihat monad.

[Bab 09: Monadic Onions](ch09-id.md)

## Exercises

{% exercise %}  
Gunakan `add` dan `map` untuk membuat fungsi yang menaikkan (_increment_) nilai di dalam functor.

{% initial src="./exercises/ch08/exercise_a.js#L3;" %}

```js
// incrF :: Functor f => f Int -> f Int
const incrF = undefined;
```

{% solution src="./exercises/ch08/solution_a.js" %}  
{% validation src="./exercises/ch08/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Diberikan objek `user` berikut:

```js
const user = { id: 2, name: "Albert", active: true };
```

{% exercise %}  
Gunakan `safeProp` dan `head` untuk menemukan inisial pertama dari user.

{% initial src="./exercises/ch08/exercise_b.js#L7;" %}

```js
// initial :: User -> Maybe String
const initial = undefined;
```

{% solution src="./exercises/ch08/solution_b.js" %}  
{% validation src="./exercises/ch08/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Mengingat fungsi pembantu berikut:

```js
// showWelcome :: User -> String
const showWelcome = compose(concat("Welcome "), prop("name"));

// checkActive :: User -> Either String User
const checkActive = function checkActive(user) {
  return user.active ? Either.of(user) : left("Your account is not active");
};
```

{% exercise %}  
Tulis fungsi yang menggunakan `checkActive` dan `showWelcome` untuk memberikan akses atau mengembalikan kesalahan.

{% initial src="./exercises/ch08/exercise_c.js#L15;" %}

```js
// eitherWelcome :: User -> Either String String
const eitherWelcome = undefined;
```

{% solution src="./exercises/ch08/solution_c.js" %}  
{% validation src="./exercises/ch08/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Kami sekarang mempertimbangkan fungsi-fungsi berikut:

```js
// validateUser :: (User -> Either String ()) -> User -> Either String User
const validateUser = curry((validate, user) => validate(user).map((_) => user));

// save :: User -> IO User
const save = (user) => new IO(() => ({ ...user, saved: true }));
```

{% exercise %}  
Tulis fungsi `validateName` yang memeriksa apakah pengguna memiliki nama lebih dari 3 karakter atau mengembalikan pesan kesalahan. Kemudian gunakan `either`, `showWelcome` dan `save` untuk menulis fungsi register untuk mendaftar dan menyambut pengguna saat validasinya ok.

Ingat kedua argumen harus mengembalikan tipe yang sama.

{% initial src="./exercises/ch08/exercise_d.js#L15;" %}

```js
// validateName :: User -> Either String ()
const validateName = undefined;

// register :: User -> IO String
const register = compose(undefined, validateUser(validateName));
```

{% solution src="./exercises/ch08/solution_d.js" %}  
{% validation src="./exercises/ch08/validation_d.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
