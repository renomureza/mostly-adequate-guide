# Bab 11: Transformasi Lagi, Secara Natural

Kami akan membahas _transformasi alami_ dalam konteks utilitas praktis dalam kode setiap hari.

Kebetulan mereka adalah pilar teori kategori dan mutlak diperlukan ketika menerapkan matematika untuk mempertimbangkan dan memperbaiki kode kita.

Karena itu, saya percaya adalah tugas saya untuk memberi tahu Anda tentang ketidakadilan yang menyedihkan yang akan Anda saksikan tanpa diragukan lagi karena ruang lingkup saya yang terbatas.

Mari kita mulai.

## Kutukan Sarang Ini

Saya ingin membahas masalah bersarang.

Bukan dorongan naluriah yang dirasakan oleh orang tua yang akan segera menjadi orang tua di mana mereka merapikan dan mengatur ulang dengan paksaan obsesif, tetapi ... sebenarnya, kalau dipikir-pikir itu tidak jauh dari sasaran seperti yang akan kita lihat di bab-bab mendatang.

... Bagaimanapun, yang saya maksud dengan bersarang adalah memiliki dua atau lebih tipe yang berbeda semua berkumpul bersama di sekitar nilai, seolah-olah menggendongnya seperti bayi yang baru lahir.

```js
Right(Maybe("b"));

IO(Task(IO(1000)));

[Identity("bee thousand")];
```

Sampai sekarang, kami telah berhasil menghindari skenario umum ini dengan contoh yang dibuat dengan hati-hati, tetapi dalam praktiknya, sebagai satu kode, tipe cenderung kusut seperti earbud dalam eksorsisme.

Jika kita tidak mengatur tipe kita dengan cermat saat kita melanjutkan, kode kita akan terbaca lebih berbulu daripada _beatnik_ di kafe kucing.

## Komedi Situasional

```js
// getValue :: Selector -> Task Error (Maybe String)
// postComment :: String -> Task Error Comment
// validate :: String -> Either ValidationError String

// saveComment :: () -> Task Error (Maybe (Either ValidationError (Task Error Comment)))
const saveComment = compose(
  map(map(map(postComment))),
  map(map(validate)),
  getValue("#comment")
);
```

Semua geng ada di sini, banyak yang membuat tanda tangan tipe kami kecewa.

Izinkan saya untuk menjelaskan secara singkat kodenya.

Kita mulai dengan mendapatkan input pengguna `getValue('#comment')` yang merupakan tindakan yang mengambil teks pada suatu elemen.

Sekarang, kesalahan menemukan elemen atau string nilai mungkin tidak ada sehingga mengembalikan `Task Error (Maybe String)`.

Setelah itu, kita harus melewatkan `map` diatas kedua `Task` dan `Maybe` untuk meneruskan teks kita ke validasi, yang pada gilirannya, mengembalikan `Either` `ValidationError` atau String.

Kemudian ke pemetaan selama berhari-hari untuk mengirim `String` dalam konten `Task Error (Maybe (Either ValidationError String))` ke `postComment` mana saja mengembalikan hasil `Task`.

Apa kekacauan yang menakutkan. Kolase tipe abstrak, ekspresionisme tipe amatir, Pollock polimorfik, Mondrian monolitik.

Ada banyak solusi untuk masalah umum ini.

Kita dapat menyusun jenis-jenisnya menjadi satu wadah yang mengerikan, menyortir dan `join` beberapa, menghomogenkannya, mendekonstruksinya, dan sebagainya. Dalam bab ini, kita akan fokus pada homogenisasi mereka melalui transformasi alami .

## Semua Alami

Tranformasi alami adalah "morphism antara functors", yaitu, fungsi yang beroperasi pada wadah sendiri.

Ketik, itu adalah fungsi `(Functor f, Functor g) => f a -> g a`. Yang membuatnya istimewa adalah kami tidak bisa mengintip isi functor kami, dengan alasan apa pun.

Anggap saja sebagai pertukaran informasi yang sangat rahasia - kedua pihak tidak menyadari apa yang ada di dalam amplop manila yang disegel yang dicap "sangat rahasia".

Ini adalah operasi struktural. Perubahan kostum fungsional. Secara formal, transformasi alami adalah setiap fungsi yang berlaku berikut ini:

<img width=600 src="images/natural_transformation.png" alt="natural transformation diagram" />

atau dalam kode:

```js
// nt :: (Functor f, Functor g) => f a -> g a
compose(map(f), nt) === compose(nt, map(f));
```

Baik diagram maupun kodenya mengatakan hal yang sama: Kita dapat menjalankan transformasi natural kita saat itu lalu `map` atau `map` kemudian menjalankan transformasi natural kita dan mendapatkan hasil yang sama.

Kebetulan, itu mengikuti dari teorema bebas meskipun transformasi alami (dan fungsi) tidak terbatas pada fungsi pada tipe.

## Konversi Tipe Berprinsip

Sebagai programmer kita sudah familiar dengan konversi tipe. Kami mengubah tipe seperti `String` menjadi `Boolean` dan `Integer` menjadi `Float` (meskipun JavaScript hanya memiliki Numbers).

Perbedaannya di sini hanyalah bahwa kami bekerja dengan wadah aljabar dan kami memiliki beberapa teori yang kami miliki.

Mari kita lihat beberapa di antaranya sebagai contoh:

```js
// idToMaybe :: Identity a -> Maybe a
const idToMaybe = (x) => Maybe.of(x.$value);

// idToIO :: Identity a -> IO a
const idToIO = (x) => IO.of(x.$value);

// eitherToTask :: Either a b -> Task a b
const eitherToTask = either(Task.rejected, Task.of);

// ioToTask :: IO a -> Task () a
const ioToTask = (x) =>
  new Task((reject, resolve) => resolve(x.unsafePerform()));

// maybeToTask :: Maybe a -> Task () a
const maybeToTask = (x) => (x.isNothing ? Task.rejected() : Task.of(x.$value));

// arrayToMaybe :: [a] -> Maybe a
const arrayToMaybe = (x) => Maybe.of(x[0]);
```

Melihat idenya? Kami hanya mengubah satu functor ke functor lain.

Kami diizinkan untuk kehilangan informasi di sepanjang jalan selama nilai yang didapatkan selama `map` tidak hilang dalam pengocokan perubahan bentuk. Itulah intinya: `map` harus membawa itu, menurut definisi kami, bahkan setelah transformasi.

Salah satu cara untuk melihatnya adalah bahwa kita mengubah efek kita.

Dalam pencerahan itu, kita dapat melihat `ioToTask` sebagai konversi sinkron ke asinkron atau `arrayToMaybe` dari nondeterminisme ke kemungkinan kegagalan.

Perhatikan bahwa kami tidak dapat mengonversi asinkron ke sinkron dalam JavaScript sehingga kami tidak dapat menulis `taskToIO` - itu akan menjadi transformasi supernatural.

## Fitur Envy

Misalkan kita ingin menggunakan beberapa fitur dari tipe lain seperti `sortBy` pada `List`.

Transformasi alami memberikan cara yang bagus untuk mengkonversi ke tipe target mengetahui `map` akan suara kita.

```js
// arrayToList :: [a] -> List a
const arrayToList = List.of;

const doListyThings = compose(sortBy(h), filter(g), arrayToList, map(f));
const doListyThings_ = compose(sortBy(h), filter(g), map(f), arrayToList); // law applied
```

Goyangan hidung kami, tiga ketukan tongkat kami, masuk `arrayToList`, dan voila! [a] adalah `List a` dan kami bisa `sortBy` jika kami mau.

Juga, menjadi lebih mudah untuk mengoptimalkan / menggabungkan operasi dengan memindahkan `map(f)` ke kiri transformasi alami seperti yang ditunjukkan pada `doListyThings_`.

## JavaScript isomorfik

Ketika kita benar-benar bisa bolak-balik tanpa kehilangan informasi apa pun, itu dianggap sebagai isomorfisme.

Itu hanya kata yang bagus untuk "memegang data yang sama". Kami mengatakan bahwa dua jenis isomorfik jika kami dapat memberikan transformasi alami "ke" dan "dari" sebagai bukti:

```js
// promiseToTask :: Promise a b -> Task a b
const promiseToTask = (x) =>
  new Task((reject, resolve) => x.then(resolve).catch(reject));

// taskToPromise :: Task a b -> Promise a b
const taskToPromise = (x) =>
  new Promise((resolve, reject) => x.fork(reject, resolve));

const x = Promise.resolve("ring");
taskToPromise(promiseToTask(x)) === x;

const y = Task.of("rabbit");
promiseToTask(taskToPromise(y)) === y;
```

Q.E.D. `Promise` dan `Task` yang isomorfik.

Kami juga dapat menulis `listToArray` untuk melengkapi `arrayToList` dan menunjukkan bahwa mereka juga.

Sebagai contoh tandingan, `arrayToMaybe` bukan isomorfisme karena kehilangan informasi:

```js
// maybeToArray :: Maybe a -> [a]
const maybeToArray = (x) => (x.isNothing ? [] : [x.$value]);

// arrayToMaybe :: [a] -> Maybe a
const arrayToMaybe = (x) => Maybe.of(x[0]);

const x = ["elvis costello", "the attractions"];

// not isomorphic
maybeToArray(arrayToMaybe(x)); // ['elvis costello']

// but is a natural transformation
compose(arrayToMaybe, map(replace("elvis", "lou")))(x); // Just('lou costello')
// ==
compose(map(replace("elvis", "lou")), arrayToMaybe)(x); // Just('lou costello')
```

Mereka memang transformasi alami, bagaimanapun, karena `map` di kedua sisi menghasilkan hasil yang sama.

Saya menyebutkan isomorfisme di sini, pertengahan bab sementara kita membahasnya, tetapi jangan biarkan hal itu membodohi Anda, mereka adalah konsep yang sangat kuat dan meresap. Bagaimanapun, mari kita lanjutkan.

## Definisi yang Lebih Luas

Fungsi struktural ini tidak terbatas pada konversi tipe dengan cara apa pun.

Berikut adalah beberapa yang berbeda:

```hs
reverse :: [a] -> [a]

join :: (Monad m) => m (m a) -> m a

head :: [a] -> a

of :: a -> f a
```

Hukum transformasi alami berlaku untuk fungsi-fungsi ini juga.

Satu hal yang mungkin membuat Anda tersandung adalah `head :: [a] -> a` dapat dilihat sebagai `head :: [a] -> Identity a`.

Kami bebas untuk menyisipkan di `Identity` mana pun yang kami mau sambil membuktikan hukum karena kami dapat, pada gilirannya, membuktikan bahwa isomorfik ke Identity (lihat, saya katakan kepada Anda bahwa isomorfisme meresap).

## Satu Solusi Bersarang

Kembali ke tanda tangan tipe komedi kami. Kita dapat memercikkan beberapa transformasi alami di seluruh kode panggilan untuk memaksa setiap jenis yang berbeda-beda sehingga mereka seragam dan, oleh karena itu, `join` mampu.

```js
// getValue :: Selector -> Task Error (Maybe String)
// postComment :: String -> Task Error Comment
// validate :: String -> Either ValidationError String

// saveComment :: () -> Task Error Comment
const saveComment = compose(
  chain(postComment),
  chain(eitherToTask),
  map(validate),
  chain(maybeToTask),
  getValue("#comment")
);
```

Jadi apa yang kita miliki di sini? Kami hanya menambahkan `chain(maybeToTask)` dan `chain(eitherToTask)`.

Keduanya memiliki efek yang sama; mereka secara alami mengubah functor yang dipegang `Task` menjadi `Task` yang lain kemudian `join` keduanya.

Seperti paku merpati di langkan jendela, kami menghindari bersarang tepat di sumbernya. Seperti yang mereka katakan di kota cahaya, "Mieux vaut prévenir que guérir" - satu ons pencegahan bernilai satu pon pengobatan.

## Singkatnya

Transformasi alami adalah fungsi pada functors kita sendiri.

Mereka adalah konsep yang sangat penting dalam teori kategori dan akan mulai muncul di mana-mana setelah lebih banyak abstraksi diadopsi, tetapi untuk saat ini, kami telah membatasinya ke beberapa aplikasi konkret.

Seperti yang kita lihat, kita dapat mencapai efek yang berbeda dengan mengonversi tipe dengan jaminan bahwa komposisi kita akan bertahan.

Mereka juga dapat membantu kami pada tipe bersarang, meskipun mereka memiliki efek umum untuk menyeragamkan fungsi kami ke penyebut umum terendah, yang dalam praktiknya, adalah fungsi dengan efek paling tidak stabil (`Task` dalam banyak kasus).

Penyortiran tipe yang terus-menerus dan membosankan ini adalah harga yang kami bayar untuk mewujudkannya - memanggil mereka dari eter.

Tentu saja, efek implisit jauh lebih berbahaya dan jadi di sini kita bertarung dengan baik.

Kami akan membutuhkan beberapa alat lagi dalam tekel kami sebelum kami dapat menggabungkan jenis amalgamasi yang lebih besar.

Selanjutnya, kita akan melihat pengurutan ulang tipe kita dengan Traversable.

[Bab 12: Melintasi Batu](ch12-id.md)

## Latihan

{% exercise %}  
Tulislah transformasi alami yang diubah `Either b a` menjadi `Maybe a`.

{% initial src="./exercises/ch11/exercise_a.js#L3;" %}

```js
// eitherToMaybe :: Either b a -> Maybe a
const eitherToMaybe = undefined;
```

{% solution src="./exercises/ch11/solution_a.js" %}  
{% validation src="./exercises/ch11/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

```js
// eitherToTask :: Either a b -> Task a b
const eitherToTask = either(Task.rejected, Task.of);
```

{% exercise %}  
Menggunakan `eitherToTask`, sederhanakan `findNameById` untuk menghapus nested `Either`.

{% initial src="./exercises/ch11/exercise_b.js#L6;" %}

```js
// findNameById :: Number -> Task Error (Either Error User)
const findNameById = compose(map(map(prop("name"))), findUserById);
```

{% solution src="./exercises/ch11/solution_b.js" %}  
{% validation src="./exercises/ch11/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Sebagai pengingat, fungsi berikut tersedia dalam konteks latihan:

```hs
split :: String -> String -> [String]
intercalate :: String -> [String] -> String
```

{% exercise %}  
Tulis isomorfisme antara String dan [Char].

{% initial src="./exercises/ch11/exercise_c.js#L8;" %}

```js
// strToList :: String -> [Char]
const strToList = undefined;

// listToStr :: [Char] -> String
const listToStr = undefined;
```

{% solution src="./exercises/ch11/solution_c.js" %}  
{% validation src="./exercises/ch11/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
