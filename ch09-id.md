# Bab 09: Monadic Onions

## Pointy Functor Factory

Sebelum kita melangkah lebih jauh, saya harus membuat pengakuan: Saya belum sepenuhnya jujur ​​tentang method `of` yang telah kami tempatkan pada masing-masing tipe kami.

Ternyata, itu tidak ada untuk menghindari kata kunci `new`, melainkan untuk menempatkan nilai dalam apa yang disebut _default minimal context_.

Ya, `of` sebenarnya tidak menggantikan konstruktor - ini adalah bagian dari antarmuka penting yang kami sebut _Pointed_.

> _pointed functor_ adalah sebuah functor dengan method `of`

Yang penting di sini adalah kemampuan untuk menjatuhkan nilai apa pun dalam tipe kita dan mulai memetakan.

```js
IO.of("tetris").map(concat(" master"));
// IO('tetris master')

Maybe.of(1336).map(add(1));
// Maybe(1337)

Task.of([{ id: 2 }, { id: 3 }]).map(map(prop("id")));
// Task([2,3])

Either.of("The past, present and future walk into a bar...").map(
  concat("it was tense.")
);
// Right('The past, present and future walk into a bar...it was tense.')
```

Jika Anda ingat, konstruktor `IO` dan `Task` mengharapkan fungsi sebagai argumen mereka, tetapi `Maybe` dan `Either` tidak.

Motivasi untuk antarmuka ini adalah cara yang umum dan konsisten untuk menempatkan nilai ke dalam functor kami tanpa kerumitan dan tuntutan khusus dari konstruktor.

Istilah _default minimal context_ kurang presisi, namun menangkap ide dengan baik: kami ingin mengangkat nilai apa pun dalam tipe kami dan `map` seperti biasa dengan perilaku yang diharapkan dari fungsi mana pun.

Salah satu koreksi penting yang harus saya buat pada titik ini, permainan kata-kata, itu adalah `Left.of` tidak masuk akal.

Setiap functor harus memiliki satu cara untuk menempatkan nilai di dalamnya dengan `Either`, yaitu `new Right(x)`. Kami mendefinisikan `of` menggunakan `Right` karena jika tipe kami bisa `map`, seharusnya `map`.

Melihat contoh di atas, kita harus memiliki intuisi tentang bagaimana `of` biasanya akan bekerja dan `Left` memecahkan cetakan itu.

Anda mungkin pernah mendengar fungsi seperti `pure`, `point`, `unit`, dan `return`. Ini adalah berbagai moniker untuk method `of`, fungsi internasional misteri.

`of` akan menjadi penting ketika kita mulai menggunakan monad karena seperti yang akan kita lihat, adalah tanggung jawab kita untuk menempatkan nilai kembali ke dalam tipe secara manual.

Untuk menghindari kata kunci `new`, ada beberapa trik atau pustaka JavaScript standar jadi mari kita gunakan dan gunakan `of` seperti orang dewasa yang bertanggung jawab mulai sekarang.

Saya sarankan menggunakan instance functor dari folktale, ramda atau fantasy-land karena mereka menyediakan method `of` yang benar serta konstruktor bagus yang tidak bergantung pada `new`.

## Mencampur Metafora

<img src="images/onion.png" alt="onion" />

Anda tahu, selain burrito luar angkasa (jika Anda pernah mendengar desas-desus), monad itu seperti bawang. Izinkan saya untuk menunjukkan dengan situasi umum:

```js
const fs = require("fs");

// readFile :: String -> IO String
const readFile = (filename) => new IO(() => fs.readFileSync(filename, "utf-8"));

// print :: String -> IO String
const print = (x) =>
  new IO(() => {
    console.log(x);
    return x;
  });

// cat :: String -> IO (IO String)
const cat = compose(map(print), readFile);

cat(".git/config");
// IO(IO('[core]\nrepositoryformatversion = 0\n'))
```

Apa yang kita dapatkan di sini adalah `IO` terperangkap di dalam `IO` lain karena `print` diperkenalkan IO kedua selama `map`.

Untuk terus bekerja dengan string kita, kita harus `map(map(f))` dan untuk mengamati efeknya, kita harus `unsafePerformIO().unsafePerformIO()`.

```js
// cat :: String -> IO (IO String)
const cat = compose(map(print), readFile);

// catFirstChar :: String -> IO (IO String)
const catFirstChar = compose(map(map(head)), cat);

catFirstChar(".git/config");
// IO(IO('['))
```

Meskipun senang melihat bahwa kami memiliki dua efek yang dikemas dan siap digunakan dalam aplikasi kami, rasanya seperti bekerja dalam dua setelan hazmat dan kami berakhir dengan API yang canggung dan tidak nyaman.

Mari kita lihat situasi lain:

```js
// safeProp :: Key -> {Key: a} -> Maybe a
const safeProp = curry((x, obj) => Maybe.of(obj[x]));

// safeHead :: [a] -> Maybe a
const safeHead = safeProp(0);

// firstAddressStreet :: User -> Maybe (Maybe (Maybe Street))
const firstAddressStreet = compose(
  map(map(safeProp("street"))),
  map(safeHead),
  safeProp("addresses")
);

firstAddressStreet({
  addresses: [{ street: { name: "Mulburry", number: 8402 }, postcode: "WC2N" }],
});
// Maybe(Maybe(Maybe({name: 'Mulburry', number: 8402})))
```

Sekali lagi, kita melihat situasi functor bersarang ini di mana rapi untuk melihat ada tiga kemungkinan kegagalan dalam fungsi kita, tetapi agak lancang untuk mengharapkan pemanggil `map` tiga kali untuk mendapatkan nilai - kita baru saja bertemu.

Pola ini akan muncul berulang kali dan ini adalah situasi utama di mana kita perlu menyinari simbol monad yang perkasa ke langit malam.

Saya mengatakan monad seperti bawang karena air mata keluar saat kita mengupas setiap lapisan functor bersarang dengan `map` untuk mendapatkan nilai bagian dalamnya.

Kita bisa mengeringkan mata, mengambil napas dalam-dalam, dan menggunakan method yang disebut join.

```js
const mmo = Maybe.of(Maybe.of("nunchucks"));
// Maybe(Maybe('nunchucks'))

mmo.join();
// Maybe('nunchucks')

const ioio = IO.of(IO.of("pizza"));
// IO(IO('pizza'))

ioio.join();
// IO('pizza')

const ttt = Task.of(Task.of(Task.of("sewers")));
// Task(Task(Task('sewers')));

ttt.join();
// Task(Task('sewers'))
```

Jika kita memiliki dua lapisan dengan tipe yang sama, kita dapat menghancurkannya bersama-sama dengan `join`. Kemampuan untuk bergabung bersama, perkawinan functor inilah yang membuat monad menjadi monad.

Mari kita beringsut menuju definisi lengkap dengan sesuatu yang sedikit lebih akurat:

> Monad adalah pointed functors yang dapat meratakan

Setiap fungsi yang mendefinisikan method join, memiliki method of, dan mematuhi beberapa hukum adalah monad.

Mendefinisikan `join` tidak terlalu sulit jadi mari kita lakukan untuk `Maybe`:

```js
Maybe.prototype.join = function join() {
  return this.isNothing() ? Maybe.of(null) : this.$value;
};
```

Di sana, sederhana seperti mengonsumsi saudara kembarnya di dalam kandungan. Jika kita memiliki` Maybe(Maybe(x))` maka `.$value` hanya akan menghapus lapisan tambahan yang tidak perlu dan kita dapat `map` dengan aman dari sana.

Jika tidak, kita hanya akan memiliki satu `Maybe` karena tidak ada yang akan dipetakan sejak awal.

Sekarang setelah kita memiliki method join, mari taburkan debu monad ajaib di atas `firstAddressStreet` contoh dan lihat aksinya:

```js
// join :: Monad m => m (m a) -> m a
const join = (mma) => mma.join();

// firstAddressStreet :: User -> Maybe Street
const firstAddressStreet = compose(
  join,
  map(safeProp("street")),
  join,
  map(safeHead),
  safeProp("addresses")
);

firstAddressStreet({
  addresses: [{ street: { name: "Mulburry", number: 8402 }, postcode: "WC2N" }],
});
// Maybe({name: 'Mulburry', number: 8402})
```

Kami menambahkan di `join` mana pun kami menemukan sarang `Maybe` agar tidak lepas kendali. Mari kita lakukan hal yang sama dengan `IO`.

```js
IO.prototype.join = () => this.unsafePerformIO();
```

Sekali lagi, kita cukup menghapus satu layer. Ingat, kami tidak membuang kemurnian, tetapi hanya menghapus satu lapis bungkus susut berlebih.

```js
// log :: a -> IO a
const log = (x) =>
  new IO(() => {
    console.log(x);
    return x;
  });

// setStyle :: Selector -> CSSProps -> IO DOM
const setStyle = curry((sel, props) => new IO(() => jQuery(sel).css(props)));

// getItem :: String -> IO String
const getItem = (key) => new IO(() => localStorage.getItem(key));

// applyPreferences :: String -> IO DOM
const applyPreferences = compose(
  join,
  map(setStyle("#main")),
  join,
  map(log),
  map(JSON.parse),
  getItem
);

applyPreferences("preferences").unsafePerformIO();
// Object {backgroundColor: "green"}
// <div style="background-color: 'green'"/>
```

`getItem` mengembalikan `IO String` jadi `map` menguraikannya. `log` dan `setStyle` mengembalikan `IO` sendiri jadi kita harus `join` untuk menjaga sarang kita tetap terkendali.

## Rantaiku Memukul Dadaku

<img src="images/chain.jpg" alt="chain" />

Anda mungkin telah memperhatikan sebuah pola. Kami sering berakhir memanggil `join` tepat setelah `map`. Mari kita abstraksikan ini menjadi fungsi yang disebut `chain`.

```js
// chain :: Monad m => (a -> m b) -> m a -> m b
const chain = curry((f, m) => m.map(f).join());

// or

// chain :: Monad m => (a -> m b) -> m a -> m b
const chain = (f) => compose(join, map(f));
```

Kami hanya akan menggabungkan kombo map/join ini menjadi satu fungsi.

Jika Anda pernah membaca tentang monad sebelumnya, Anda mungkin pernah melihat `chain` disebut `>>=`(diucapkan mengikat) atau `flatMap` yang semuanya alias untuk konsep yang sama.

Menurut saya pribadi `flatMap` adalah nama yang paling akurat, tetapi kami akan tetap menggunakan `chain` karena itu adalah nama yang diterima secara luas di JS.

Mari kita refactor dua contoh di atas dengan `chain`:

```js
// map/join
const firstAddressStreet = compose(
  join,
  map(safeProp("street")),
  join,
  map(safeHead),
  safeProp("addresses")
);

// chain
const firstAddressStreet = compose(
  chain(safeProp("street")),
  chain(safeHead),
  safeProp("addresses")
);

// map/join
const applyPreferences = compose(
  join,
  map(setStyle("#main")),
  join,
  map(log),
  map(JSON.parse),
  getItem
);

// chain
const applyPreferences = compose(
  chain(setStyle("#main")),
  chain(log),
  map(JSON.parse),
  getItem
);
```

Saya menukar `map`/`join` dengan fungsi `chain` baru kami untuk sedikit merapikannya.

Semuanya bersih dan bagus, tetapi ada lebih dari `chain` yang terlihat - ini lebih seperti tornado daripada ruang hampa. Karena efek `chain` bersarang yang mudah, kami dapat menangkap urutan dan penugasan variabel dengan cara yang murni fungsional.

```js
// getJSON :: Url -> Params -> Task JSON
getJSON("/authenticate", { username: "stale", password: "crackers" }).chain(
  (user) => getJSON("/friends", { user_id: user.id })
);
// Task([{name: 'Seimith', id: 14}, {name: 'Ric', id: 39}]);

// querySelector :: Selector -> IO DOM
querySelector("input.username").chain(({ value: uname }) =>
  querySelector("input.email").chain(({ value: email }) =>
    IO.of(`Welcome ${uname} prepare for spam at ${email}`)
  )
);
// IO('Welcome Olivia prepare for spam at olivia@tremorcontrol.net');

Maybe.of(3).chain((three) => Maybe.of(2).map(add(three)));
// Maybe(5);

Maybe.of(null).chain(safeProp("address")).chain(safeProp("street"));
// Maybe(null);
```

Kita dapat menulis contoh-contoh ini dengan `compose`, tetapi kita memerlukan beberapa fungsi pembantu dan gaya ini cocok untuk penugasan variabel eksplisit melalui _closures_.

Alih-alih, kami menggunakan versi infiks `chain` yang kebetulan dapat diturunkan dari `map` dan `join` untuk jenis apa pun secara otomatis: `t.prototype.chain = function(f) { return this.map(f).join(); }`.

Kita juga dapat mendefinisikan `chain` secara manual jika kita menginginkan rasa kinerja yang salah, meskipun kita harus berhati-hati untuk mempertahankan fungsionalitas yang benar - yaitu, harus sama `map` diikuti oleh `join`.

Fakta yang menarik adalah bahwa kita dapat memperoleh `map` secara gratis jika kita telah membuat `chain` hanya dengan membotolkan nilai kembali ketika kita selesai dengan `of`.

Dengan `chain`, kita juga dapat mendefinisikan `join` sebagai `chain(id)`. Ini mungkin terasa seperti bermain Texas Hold'em dengan pesulap berlian imitasi karena saya hanya menarik sesuatu dari belakang saya.

Tetapi seperti kebanyakan matematika, semua konstruksi berprinsip ini saling terkait. Banyak dari derivasi ini disebutkan dalam repo [fantasyland](https://github.com/fantasyland/fantasy-land), yang merupakan spesifikasi resmi untuk tipe data aljabar dalam JavaScript.

Ngomong-ngomong, mari kita ke contoh di atas.

Dalam contoh pertama, kita melihat dua `Task` dirantai dalam urutan tindakan asinkron - pertama ia mengambil `user`, kemudian menemukan teman dengan id pengguna itu. Kita gunakan `chain` untuk menghindari suatu situasi `Task(Task([Friend]))`.

Selanjutnya, kita gunakan `querySelector` untuk menemukan beberapa input berbeda dan membuat pesan sambutan.

Perhatikan bagaimana kita memiliki akses ke keduanya `uname` dan `email` pada fungsi terdalam - ini adalah penugasan variabel fungsional yang terbaik.

Karena `IO` dengan murah hati meminjamkan nilainya kepada kami, kami bertanggung jawab untuk mengembalikannya seperti yang kami temukan - kami tidak ingin merusak kepercayaannya (dan program kami).

`IO.of` adalah alat yang sempurna untuk pekerjaan itu dan itulah sebabnya Pointed merupakan prasyarat penting untuk antarmuka Monad.

Namun, kami dapat memilih `map` karena itu juga akan mengembalikan jenis yang benar:

```js
querySelector("input.username").chain(({ value: uname }) =>
  querySelector("input.email").map(
    ({ value: email }) => `Welcome ${uname} prepare for spam at ${email}`
  )
);
// IO('Welcome Olivia prepare for spam at olivia@tremorcontrol.net');
```

Akhirnya, kami memiliki dua contoh penggunaan `Maybe`. Karena pemetaan `chain` di bawah tenda, jika ada nilai `null`, kami menghentikan perhitungan mati di jalurnya.

Jangan khawatir jika contoh-contoh ini sulit dipahami pada awalnya. Bermain dengan mereka. Tusuk mereka dengan tongkat. Hancurkan mereka hingga berkeping-keping dan pasang kembali.

Ingatlah saat `map` mengembalikan nilai "normal" dan saat `chain` mengembalikan fungsi lain. Di bab berikutnya, kita akan mendekati `Applicatives` dan melihat trik yang bagus untuk membuat ekspresi semacam ini lebih bagus dan mudah dibaca.

Sebagai pengingat, ini tidak berfungsi dengan dua jenis bersarang yang berbeda. Komposisi functor dan transformer monad, dapat membantu kita dalam situasi itu.

## Power Trip

Pemrograman gaya kontainer terkadang bisa membingungkan. Terkadang kita menemukan diri kita berjuang untuk memahami berapa banyak container dalam suatu nilai atau jika kita membutuhkan `map` atau `chain`(segera kita akan melihat lebih banyak method container).

Kami dapat sangat meningkatkan debugging dengan trik seperti menerapkan `inspect` dan kami akan belajar cara membuat "tumpukan" yang dapat menangani efek apa pun yang kami berikan, tetapi ada kalanya kami mempertanyakan apakah itu sepadan dengan kerumitannya.

Saya ingin mengayunkan pedang monadik yang berapi-api sejenak untuk menunjukkan kekuatan pemrograman dengan cara ini.

Mari kita baca file, lalu unggah langsung setelahnya:

```js
// readFile :: Filename -> Either String (Task Error String)
// httpPost :: String -> String -> Task Error JSON
// upload :: Filename -> Either String (Task Error JSON)
const upload = compose(map(chain(httpPost("/uploads"))), readFile);
```

Di sini, kami mencabangkan kode kami beberapa kali. Melihat tanda tangan tipe, saya dapat melihat bahwa kami melindungi dari 3 kesalahan - penggunaan `readFile` untuk `Either` memvalidasi input (mungkin memastikan nama file ada).

`readFile` mungkin mendapatkan kesalahan saat mengakses file seperti yang dinyatakan dalam parameter tipe pertama `Task`,

Dan unggahan mungkin gagal untuk alasan apapun yang diungkapkan oleh `Error` di `httpPost`. Kami dengan santai melakukan dua tindakan asinkron berurutan bersarang dengan `chain`.

Semua ini dicapai dalam satu aliran linier kiri ke kanan. Ini semua murni dan deklaratif.

Ini memegang penalaran persamaan dan sifat yang dapat diandalkan. Kami tidak dipaksa untuk menambahkan nama variabel yang tidak perlu dan membingungkan.

Fungsi `upload` kami ditulis sesuai antarmuka generik dan bukan api khusus satu kali. Ini satu garis berdarah demi Tuhan.

Sebagai kontras, mari kita lihat cara imperatif standar untuk melakukan ini:

```js
// upload :: Filename -> (String -> a) -> Void
const upload = (filename, callback) => {
  if (!filename) {
    throw new Error("You need a filename!");
  } else {
    readFile(filename, (errF, contents) => {
      if (errF) throw errF;
      httpPost("/uploads", contents, (errH, json) => {
        if (errH) throw errH;
        callback(json);
      });
    });
  }
};
```

Bukankah itu aritmatika iblis. Kami terjepit di labirin kegilaan yang bergejolak.

Bayangkan jika itu adalah aplikasi tipikal yang juga mengubah variabel di sepanjang jalan! Kami benar-benar akan berada di lubang tar.

## Teori

Hukum pertama yang akan kita lihat adalah asosiatif, tetapi mungkin tidak seperti yang biasa Anda lakukan.

```js
// associativity
compose(join, map(join)) === compose(join, join);
```

Hukum-hukum ini mendapatkan sifat monad yang bersarang sehingga associativity berfokus pada menggabungkan tipe dalam atau luar terlebih dahulu untuk mencapai hasil yang sama. Gambar mungkin lebih instruktif:

<img src="images/monad_associativity.png" alt="monad associativity law" />

Dimulai dengan bagian atas kiri bergerak ke bawah, kita bisa `join` luar dua `M` dari `M(M(M a))` pertama maka kapal pesiar diinginkan `M a` dengan `join` yang lain.

Atau, kita dapat membuka tudung dan meratakan dua bagian dalam `M` dengan `map(join)`. Kita berakhir dengan hal yang sama `M a` terlepas dari apakah kita bergabung dengan bagian dalam atau luar `M` terlebih dahulu dan itulah yang dimaksud dengan asosiatif.

Perlu dicatat bahwa `map(join) != join`. Langkah-langkah perantara dapat bervariasi nilainya, tetapi hasil akhir dari langkah terakhir `join` akan sama.

Hukum kedua serupa:

```js
// identity for all (M a)
(compose(join, of) === compose(join, map(of))) === id;
```

It states that, for any monad `M`, `of` and `join` amounts to `id`. We can also `map(of)` and attack it from the inside out. We call this "triangle identity" because it makes such a shape when visualized:

<img src="images/triangle_identity.png" alt="monad identity law" />

Jika kita mulai dari kiri atas menuju kanan, kita dapat melihat bahwa `of` memang menjatuhkan `M a` di wadah `M` lain. Kemudian jika kita bergerak ke bawah dan itu `join`, kita mendapatkan yang sama seolah-olah kita baru saja memanggil `id` di tempat pertama.

Bergerak dari kanan ke kiri, kita melihat bahwa jika kita menyelinap di bawah selimut dengan `map` dan memanggil `of` dari `a` mentah, kita masih akan berakhir `M (M a)` dan `join` akan membawa kita kembali ke titik awal.

Saya harus menyebutkan bahwa saya baru saja menulis `of`, bagaimanapun, itu harus spesifik `M.of` untuk monad apa pun yang kami gunakan.

Sekarang, saya telah melihat hukum, identitas, dan asosiasi ini, di suatu tempat sebelumnya...

Tunggu, saya sedang berpikir... Ya tentu saja! Mereka adalah hukum untuk suatu kategori.

Tapi itu berarti kita membutuhkan fungsi komposisi untuk melengkapi definisi.

Melihat:

```js
const mcompose = (f, g) => compose(chain(f), g);

// left identity
mcompose(M, f) === f;

// right identity
mcompose(f, M) === f;

// associativity
mcompose(mcompose(f, g), h) === mcompose(f, mcompose(g, h));
```

Bagaimanapun, mereka adalah hukum kategori.

Monad membentuk kategori yang disebut "kategori Kleisli" di mana semua objek adalah monad dan morfisme adalah fungsi yang dirantai.

Saya tidak bermaksud mengejek Anda dengan teori kategori bit dan bobs tanpa banyak penjelasan tentang bagaimana jigsaw cocok bersama.

Tujuannya adalah untuk menggores permukaan cukup untuk menunjukkan relevansi dan memicu minat sambil berfokus pada sifat praktis yang dapat kita gunakan setiap hari.

## Singkatnya

Monad memungkinkan kita menelusuri ke bawah ke dalam perhitungan bersarang. Kita dapat menetapkan variabel, menjalankan efek sekuensial, melakukan tugas asinkron, semua tanpa meletakkan satu bata di piramida malapetaka.

Mereka datang untuk menyelamatkan ketika suatu nilai menemukan dirinya dipenjara di beberapa lapisan dengan jenis yang sama. Dengan bantuan sidekick tepercaya "runcing", monads dapat memberi kami nilai tanpa kotak dan tahu kami akan dapat menempatkannya kembali ketika kami selesai.

Ya, monad sangat kuat, namun kami masih membutuhkan beberapa fungsi container tambahan.

Misalnya, bagaimana jika kita ingin menjalankan daftar panggilan api sekaligus, lalu mengumpulkan hasilnya? Kita dapat menyelesaikan tugas ini dengan monad, tetapi kita harus menunggu setiap monad selesai sebelum memanggil monad berikutnya.

Bagaimana dengan menggabungkan beberapa validasi? Kami ingin melanjutkan validasi untuk mengumpulkan daftar kesalahan, tetapi monad akan menghentikan pertunjukan setelah yang pertama `Left` memasuki gambar.

Di bab berikutnya, kita akan melihat bagaimana fungsi aplikatif cocok dengan dunia container dan mengapa kita lebih memilih mereka daripada monad dalam banyak kasus.

[Bab 10: Applicative Functors](ch10-id.md)

## Latihan

Mempertimbangkan objek Pengguna sebagai berikut:

```js
const user = {
  id: 1,
  name: "Albert",
  address: {
    street: {
      number: 22,
      name: "Walnut St",
    },
  },
};
```

{% exercise %}  
Gunakan `safeProp` dan `map`/`join` atau `chain` untuk mendapatkan nama jalan dengan aman saat diberikan kepada pengguna

{% initial src="./exercises/ch09/exercise_a.js#L16;" %}

```js
// getStreetName :: User -> Maybe String
const getStreetName = undefined;
```

{% solution src="./exercises/ch09/solution_a.js" %}  
{% validation src="./exercises/ch09/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Kami sekarang mempertimbangkan item berikut:

```js
// getFile :: IO String
const getFile = IO.of("/home/mostly-adequate/ch09.md");

// pureLog :: String -> IO ()
const pureLog = (str) => new IO(() => console.log(str));
```

{% exercise %}  
Gunakan getFile untuk mendapatkan filepath, hapus direktori dan simpan hanya nama dasar, lalu log murni. Petunjuk: Anda mungkin ingin menggunakan `split` dan `last` mendapatkan nama dasar dari jalur file.

{% initial src="./exercises/ch09/exercise_b.js#L13;" %}

```js
// logFilename :: IO ()
const logFilename = undefined;
```

{% solution src="./exercises/ch09/solution_b.js" %}  
{% validation src="./exercises/ch09/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Untuk latihan ini, kami mempertimbangkan pembantu dengan tanda tangan berikut:

```js
// validateEmail :: Email -> Either String Email
// addToMailingList :: Email -> IO([Email])
// emailBlast :: [Email] -> IO ()
```

{% exercise %}  
Gunakan `validateEmail`, `addToMailingList` dan `emailBlast` untuk membuat fungsi
yang menambahkan email baru ke milis jika valid, dan kemudian memberi tahu seluruh
daftar.

{% initial src="./exercises/ch09/exercise_c.js#L11;" %}

```js
// joinMailingList :: Email -> Either String (IO ())
const joinMailingList = undefined;
```

{% solution src="./exercises/ch09/solution_c.js" %}  
{% validation src="./exercises/ch09/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
