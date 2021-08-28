# Bab 13: Monoid menyatukan semuanya

## Kombinasi liar

Dalam bab ini, kita akan memeriksa monoid dengan cara semigrup.

Monoid adalah permen karet di rambut abstraksi matematika.

Mereka menangkap sebuah ide yang mencakup berbagai disiplin ilmu, secara kiasan dan harfiah menyatukan semuanya.

Mereka adalah kekuatan tak menyenangkan yang menghubungkan semua yang menghitung. Oksigen dalam basis kode kami, dasar tempat ia berjalan, keterikatan kuantum dikodekan.

Monoid adalah tentang kombinasi.

Tapi apa itu kombinasi? Itu bisa berarti banyak hal mulai dari akumulasi hingga penggabungan hingga penggandaan hingga pilihan, komposisi, urutan, bahkan evaluasi!

Kita akan melihat banyak contoh di sini, tetapi kita hanya akan berjingkat-jingkat di kaki gunung monoid.

Instancenya banyak dan aplikasinya luas. Tujuan dari bab ini adalah untuk memberikan intuisi yang baik sehingga Anda dapat membuat beberapa monoid Anda sendiri.

## Penjumlahan abstrak

Penjumlahan memiliki beberapa kualitas menarik yang ingin saya diskusikan. Mari kita lihat melalui kacamata abstraksi kita.

Sebagai permulaan, ini adalah operasi biner, yaitu operasi yang mengambil dua nilai dan mengembalikan nilai, semuanya dalam set yang sama.

```js
// a binary operation
1 + 1 = 2
```

Lihat? Dua nilai di domain, satu nilai di codomain, semua set yang sama - angka, seolah-olah.

Beberapa orang mungkin mengatakan angka "ditutup dengan penambahan", yang berarti jenisnya tidak akan pernah berubah tidak peduli yang mana yang dimasukkan ke dalam campuran.

Itu berarti kita dapat membuat rantai operasi karena hasilnya selalu berupa angka lain:

```js
// we can run this on any amount of numbers
1 + 7 + 5 + 4 + ...
```

Selain itu (permainan kata yang diperhitungkan...), kami memiliki asosiatif yang memberi kami kemampuan untuk mengelompokkan operasi sesuka kami.

Kebetulan, operasi biner asosiatif adalah resep untuk komputasi paralel karena kita dapat memotong dan mendistribusikan pekerjaan.

```js
// associativity
(1 + 2) + 3 = 6
1 + (2 + 3) = 6
```

Sekarang, jangan bingung dengan komutatifitas yang memungkinkan kita untuk mengatur ulang urutannya.

Sementara itu berlaku untuk penambahan, kami tidak terlalu tertarik pada properti itu saat ini - terlalu spesifik untuk kebutuhan abstraksi kami.

Kalau dipikir-pikir, properti apa yang seharusnya ada di superclass abstrak kita?

Sifat-sifat apa yang khusus untuk penjumlahan dan sifat-sifat apa yang dapat digeneralisasikan?

Apakah ada abstraksi lain di tengah hierarki ini atau semuanya satu bagian?

Pemikiran seperti inilah yang diterapkan oleh nenek moyang matematika kita ketika memahami antarmuka dalam aljabar abstrak.

Seperti yang terjadi, abstraksionis jadul itu mendarat di konsep grup ketika mengabstraksikan penjumlahan.

Sebuah kelompok memiliki semua lonceng dan peluit termasuk konsep angka negatif.

Di sini, kami hanya tertarik pada operator biner asosiatif itu sehingga kami akan memilih antarmuka Semigroup yang kurang spesifik.

Semigroup adalah tipe dengan method `concat` yang bertindak sebagai operator biner asosiatif kami.

Mari kita implementasikan untuk penambahan dan menyebutnya `Sum`:

```js
const Sum = (x) => ({
  x,
  concat: (other) => Sum(x + other.x),
});
```

Perhatikan kami `concat` dengan `Sum` yang lain dan selalu kembalikan `Sum`.

Saya telah menggunakan objek factory di sini alih-alih upacara prototipe khas kami, terutama karena `Sum` tidak runcing dan kami tidak ingin harus mengetik `new`.

Bagaimanapun, ini dia beraksi:

```js
Sum(1).concat(Sum(3)); // Sum(4)
Sum(4).concat(Sum(37)); // Sum(41)
```

Hanya seperti itu, kita dapat memprogram ke antarmuka, bukan implementasi.

Karena antarmuka ini berasal dari teori grup, ia memiliki literatur yang mendukungnya selama berabad-abad. Dokumen gratis!

Sekarang, seperti yang disebutkan, `Sum` bukan _pointed_, atau functor.

Sebagai latihan, kembali dan periksa hukum untuk mengetahui alasannya.

Oke, saya hanya akan memberi tahu Anda: itu hanya dapat menampung angka, jadi `map` tidak masuk akal di sini karena kami tidak dapat mengubah nilai dasar ke jenis lain. Itu akan menjadi sangat terbatas `map`!

Jadi mengapa ini berguna? Nah, seperti halnya antarmuka apa pun, kami dapat menukar instance kami untuk mencapai hasil yang berbeda:

```js
const Product = (x) => ({ x, concat: (other) => Product(x * other.x) });

const Min = (x) => ({ x, concat: (other) => Min(x < other.x ? x : other.x) });

const Max = (x) => ({ x, concat: (other) => Max(x > other.x ? x : other.x) });
```

Ini tidak terbatas pada angka. Mari kita lihat beberapa jenis lainnya:

```js
const Any = (x) => ({ x, concat: (other) => Any(x || other.x) });
const All = (x) => ({ x, concat: (other) => All(x && other.x) });

Any(false).concat(Any(true)); // Any(true)
Any(false).concat(Any(false)); // Any(false)

All(false).concat(All(true)); // All(false)
All(true)
  .concat(All(true)) // All(true)

  [(1, 2)].concat([3, 4]); // [1,2,3,4]

"miracle grow".concat("n"); // miracle grown"

Map({ day: "night" }).concat(Map({ white: "nikes" })); // Map({day: 'night', white: 'nikes'})
```

Jika Anda menatap ini cukup lama, polanya akan muncul seperti poster mata ajaib.

Itu ada di mana-mana.

Kami menggabungkan struktur data, menggabungkan logika, membangun string ... tampaknya seseorang dapat memasukkan hampir semua tugas ke dalam antarmuka berbasis kombinasi ini.

Saya telah menggunakan `Map` beberapa kali sekarang.

Maaf jika kalian berdua tidak diperkenalkan dengan benar. `Map` hanya membungkus `Object` sehingga kita dapat menghiasinya dengan beberapa method tambahan tanpa mengubah struktur alam semesta.

## Semua functors favorit saya adalah semigrup.

Jenis yang telah kita lihat sejauh ini yang mengimplementasikan antarmuka functor semuanya mengimplementasikan semigroup satu juga. Mari kita lihat `Identity` (artis yang sebelumnya dikenal sebagai Container):

```js
Identity.prototype.concat = function (other) {
  return new Identity(this.__value.concat(other.__value));
};

Identity.of(Sum(4)).concat(Identity.of(Sum(1))); // Identity(Sum(5))
Identity.of(4).concat(Identity.of(1)); // TypeError: this.__value.concat is not a function
```

Ini adalah _semigroup_ jika dan hanya jika `__value` adalah semigroup. Seperti glider gantung berjari mentega, itu adalah satu saat memegangnya.

Jenis lain memiliki perilaku serupa:

```js
// combine with error handling
Right(Sum(2)).concat(Right(Sum(3))); // Right(Sum(5))
Right(Sum(2)).concat(Left("some error")); // Left('some error')

// combine async
Task.of([1, 2]).concat(Task.of([3, 4])); // Task([1,2,3,4])
```

Ini menjadi sangat berguna ketika kita menumpuk semigrup ini ke dalam kombinasi bertingkat:

```js
// formValues :: Selector -> IO (Map String String)
// validate :: Map String String -> Either Error (Map String String)

formValues("#signup").map(validate).concat(formValues("#terms").map(validate)); // IO(Right(Map({username: 'andre3000', accepted: true})))
formValues("#signup").map(validate).concat(formValues("#terms").map(validate)); // IO(Left('one must accept our totalitarian agreement'))

serverA.get("/friends").concat(serverB.get("/friends")); // Task([friend1, friend2])

// loadSetting :: String -> Task Error (Maybe (Map String Boolean))
loadSetting("email").concat(loadSetting("general")); // Task(Maybe(Map({backgroundColor: true, autoSave: false})))
```

Dalam contoh teratas, kami telah menggabungkan _holding_ `IO` _holding_ Either `holding` `Map` untuk memvalidasi dan menggabungkan nilai formulir.

Selanjutnya, kami telah mencapai beberapa server yang berbeda dan menggabungkan hasilnya dengan cara asinkron menggunakan `Task` dan `Array`.

Terakhir, kami telah menumpuk `Task`, `Maybe`, dan `Map` memuat, mengurai, dan menggabungkan beberapa pengaturan.

Ini dapat di-`chain` atau `ap`, tetapi semigrup menangkap apa yang ingin kita lakukan dengan lebih ringkas.

Ini melampaui functors.

Faktanya, ternyata segala sesuatu yang seluruhnya terdiri dari semigrup, adalah dirinya sendiri, semigrup: jika kita dapat menggabungkan kit, maka kita dapat menggabungkan caboodle.

```js
const Analytics = (clicks, path, idleTime) => ({
  clicks,
  path,
  idleTime,
  concat: (other) =>
    Analytics(
      clicks.concat(other.clicks),
      path.concat(other.path),
      idleTime.concat(other.idleTime)
    ),
});

Analytics(Sum(2), ["/home", "/about"], Right(Max(2000))).concat(
  Analytics(Sum(1), ["/contact"], Right(Max(1000)))
);
// Analytics(Sum(3), ['/home', '/about', '/contact'], Right(Max(2000)))
```

Lihat, semuanya tahu bagaimana menggabungkan dirinya dengan baik. Ternyata, kita bisa melakukan hal yang sama secara gratis hanya dengan menggunakan tipe `Map` :

```js
Map({
  clicks: Sum(2),
  path: ["/home", "/about"],
  idleTime: Right(Max(2000)),
}).concat(
  Map({ clicks: Sum(1), path: ["/contact"], idleTime: Right(Max(1000)) })
);
// Map({clicks: Sum(3), path: ['/home', '/about', '/contact'], idleTime: Right(Max(2000))})
```

Kita dapat menumpuk dan menggabungkan sebanyak yang kita mau. Ini hanya masalah menambahkan pohon lain ke hutan, atau api lain ke kebakaran hutan tergantung pada basis kode Anda.

Perilaku default dan intuitif adalah menggabungkan apa yang dipegang oleh suatu tipe, namun, ada kasus di mana kita mengabaikan apa yang ada di dalamnya dan menggabungkan wadah itu sendiri.

Pertimbangkan jenis seperti `Stream`:

```js
const submitStream = Stream.fromEvent("click", $("#submit"));
const enterStream = filter(
  (x) => x.key === "Enter",
  Stream.fromEvent("keydown", $("#myForm"))
);

submitStream.concat(enterStream).map(submitForm); // Stream()
```

Kami dapat menggabungkan aliran acara dengan menangkap acara dari keduanya sebagai satu aliran baru.

Atau, kita bisa menggabungkan mereka dengan bersikeras mereka mengadakan semi-grup.

Sebenarnya, ada banyak kemungkinan contoh untuk setiap jenis. Pertimbangkan `Task`, kita dapat menggabungkannya dengan memilih yang lebih awal atau yang lebih baru dari keduanya.

Kami selalu dapat memilih `Right` yang pertama daripada korsleting `Left` yang memiliki efek mengabaikan kesalahan.

Ada antarmuka yang disebut Alternatif yang mengimplementasikan beberapa di antaranya, contoh alternatif, biasanya berfokus pada pilihan daripada kombinasi cascading.

Layak untuk dilihat jika Anda membutuhkan fungsionalitas seperti itu.

## Monoids for nothing

Kami mengabstraksi penambahan, tetapi seperti orang Babilonia, kami tidak memiliki konsep nol (tidak ada yang menyebutkannya).

Nol bertindak sebagai identitas yang berarti setiap elemen yang ditambahkan dengan `0`, akan mengembalikan elemen yang sama.

Dari segi abstraksi, sangat membantu untuk menganggapnya `0` sebagai elemen netral atau kosong. Penting bahwa itu bertindak dengan cara yang sama di sisi kiri dan kanan operasi biner kita:

```js
// identity
1 + 0 = 1
0 + 1 = 1
```

Mari kita sebut konsep ini `empty` dan buat antarmuka baru dengannya.

Seperti banyak startups, kami akan memilih heinously tidak informatif, namun nyaman untuk _googling_: _Monoid_.

Resep untuk Monoid adalah mengambil semigrup apa pun dan menambahkan elemen identitas khusus. Kami akan mengimplementasikannya dengan fungsi `empty` pada tipe itu sendiri:

```js
Array.empty = () => [];
String.empty = () => "";
Sum.empty = () => Sum(0);
Product.empty = () => Product(1);
Min.empty = () => Min(Infinity);
Max.empty = () => Max(-Infinity);
All.empty = () => All(true);
Any.empty = () => Any(false);
```

Kapan nilai identitas yang kosong terbukti berguna? Itu seperti menanyakan mengapa nol berguna. Seperti tidak bertanya apa-apa...

Saat kita tidak punya apa-apa lagi, siapa yang bisa kita andalkan? Nol.

Berapa banyak bug yang kita inginkan? Nol.

Ini adalah toleransi kami untuk kode yang tidak aman. Sebuah awal baru. Label harga pamungkas.

Itu dapat memusnahkan segala sesuatu di jalannya atau menyelamatkan kita dalam keadaan darurat. Sebuah penyelamat emas dan lubang keputusasaan.

Secara kode, mereka sesuai dengan default yang masuk akal:

```js
const settings = (prefix="", overrides=[], total=0) => ...

const settings = (prefix=String.empty(), overrides=Array.empty(), total=Sum.empty()) => ...
```

Atau untuk mengembalikan nilai yang berguna ketika kita tidak memiliki apa-apa lagi:

```js
sum([]); // 0
```

Mereka juga merupakan nilai awal yang sempurna untuk sebuah akumulator...

## Melipat rumah

Kebetulan `concat` dan `empty` itu sangat cocok di dua slot pertama `reduce`. Kami sebenarnya dapat menurunkan `reduce` array semigroup dengan mengabaikan nilai kosong, tetapi seperti yang Anda lihat, itu mengarah ke situasi genting:

```js
// concat :: Semigroup s => s -> s -> s
const concat = x => y => x.concat(y)

[Sum(1), Sum(2)].reduce(concat) // Sum(3)

[].reduce(concat) // TypeError: Reduce of empty array with no initial value
```

Boom pergi dinamit.

Seperti pergelangan kaki yang terkilir dalam maraton, kami memiliki pengecualian runtime.

JavaScript lebih dari senang untuk membiarkan kita mengikat pistol ke sepatu kita sebelum berlari - itu adalah jenis bahasa yang konservatif, saya kira, tetapi itu menghentikan kita mati di jalur kita ketika array mandul.

Apa yang bisa dikembalikan? `NaN`, `false`, `-1`? Jika kami melanjutkan program kami, kami ingin hasil dari jenis yang tepat.

Itu bisa mengembalikan `Maybe` untuk menunjukkan kemungkinan kegagalan, tetapi kita bisa melakukan yang lebih baik.

Mari gunakan fungsi `reduce` kari kita dan buat versi aman di mana nilai `empty` tidak opsional.

Selanjutnya akan dikenal sebagai `fold`:

```js
// fold :: Monoid m => m -> [m] -> m
const fold = reduce(concat);
```

Inisial `m` adalah nilai `empty` kita - titik awal netral kita, kemudian kita mengambil array `m` dan menghancurkannya menjadi satu nilai seperti berlian yang indah.

```js
fold(Sum.empty(), [Sum(1), Sum(2)]); // Sum(3)
fold(Sum.empty(), []); // Sum(0)

fold(Any.empty(), [Any(false), Any(true)]); // Any(true)
fold(Any.empty(), []); // Any(false)

fold(Either.of(Max.empty()), [Right(Max(3)), Right(Max(21)), Right(Max(11))]); // Right(Max(21))
fold(Either.of(Max.empty()), [
  Right(Max(3)),
  Left("error retrieving value"),
  Right(Max(11)),
]); // Left('error retrieving value')

fold(IO.of([]), [".link", "a"].map($)); // IO([<a>, <button class="link"/>, <a>])
```

Kami telah memberikan nilai `empty` manual untuk dua yang terakhir karena kami tidak dapat menentukan satu pada jenis itu sendiri.

Itu baik-baik saja. Bahasa yang diketik dapat mengetahuinya sendiri, tetapi kita harus meneruskannya di sini.

## Tidak cukup monoid

Ada beberapa semigrup yang tidak bisa menjadi monoid, yaitu memberikan nilai awal. lihat `First`:

```js
const First = (x) => ({ x, concat: (other) => First(x) });

Map({ id: First(123), isPaid: Any(true), points: Sum(13) }).concat(
  Map({ id: First(2242), isPaid: Any(false), points: Sum(1) })
);
// Map({id: First(123), isPaid: Any(true), points: Sum(14)})
```

Kami akan menggabungkan beberapa akun dan mempertahankan id `First`. Tidak ada cara untuk menentukan nilai `empty` untuk itu. Bukan berarti tidak berguna.

## Teori pemersatu agung

## Teori grup atau teori kategori?

Gagasan operasi biner ada di mana-mana dalam aljabar abstrak.

Faktanya, ini adalah operasi utama untuk sebuah kategori.

Namun, kita tidak dapat memodelkan operasi kita dalam teori kategori tanpa identitas.

Ini adalah alasan kita mulai dengan semi-grup dari teori grup, kemudian melompat ke monoid dalam teori kategori setelah kita memiliki kosong.

Monoid membentuk kategori objek tunggal di mana morfisme adalah `concat`, `empty` adalah identitas, dan komposisi dijamin.

### Komposisi sebagai monoid

Fungsi tipe `a -> a`, di mana domain berada dalam himpunan yang sama dengan kodomain, disebut _endomorfisme_ . Kita dapat membuat monoid yang disebut `Endo` yang menangkap ide ini:

```js
const Endo = run => ({
  run,
  concat: other =>
    Endo(compose(run, other.run))
})

Endo.empty = () => Endo(identity)


// in action

// thingDownFlipAndReverse :: Endo [String] -> [String]
const thingDownFlipAndReverse = fold(Endo(() => []), [Endo(reverse), Endo(sort), Endo(append('thing down')])

thingDownFlipAndReverse.run(['let me work it', 'is it worth it?'])
// ['thing down', 'let me work it', 'is it worth it?']
```

Karena semuanya bertipe sama, kita bisa `concat` melalui `compose` dan tipenya selalu berbaris.

### Monad sebagai monoid

Anda mungkin telah memperhatikan bahwa `join` itu adalah operasi yang membutuhkan dua monad (bersarang) dan meremasnya menjadi satu secara asosiatif.

Ini juga merupakan transformasi alami atau "fungsi functor".

Seperti yang dinyatakan sebelumnya, kita dapat membuat kategori functor sebagai objek dengan transformasi alami sebagai morfisme.

Sekarang, jika kita mengkhususkannya pada Endofunctors, yaitu functors dari jenis yang sama, kemudian `join` memberi kita monoid dalam kategori Endofunctors juga dikenal sebagai Monad.

Untuk menunjukkan formulasi yang tepat dalam kode membutuhkan sedikit penyelesaian yang saya anjurkan Anda ke google, tapi itulah ide umumnya.

### Aplikatif sebagai monoid

Bahkan functor aplikatif memiliki formulasi monoid yang dikenal dalam teori kategori sebagai functor monoidal yang longgar. Kami dapat mengimplementasikan antarmuka sebagai monoid dan memulihkan `ap` dari sana:

```js
// concat :: f a -> f b -> f [a, b]
// empty :: () -> f ()

// ap :: Functor f => f (a -> b) -> f a -> f b
const ap = compose(
  map(([f, x]) => f(x)),
  concat
);
```

## Singkatnya

Jadi Anda lihat, semuanya terhubung, atau bisa jadi.

Realisasi mendalam ini menjadikan Monoids alat pemodelan yang kuat untuk petak luas arsitektur aplikasi hingga bagian terkecil dari datum.

Saya mendorong Anda untuk memikirkan monoid setiap kali akumulasi atau kombinasi langsung adalah bagian dari aplikasi Anda, kemudian setelah Anda memahaminya, mulailah memperluas definisi ke lebih banyak aplikasi (Anda akan terkejut betapa banyak yang dapat dimodelkan dengan monoid).

## Exercises
