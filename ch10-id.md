# Bab 10: Aplikatif Functors

## Menerapkan Aplikatif

Nama **functor aplikatif** sangat deskriptif mengingat asal-usul fungsionalnya. Pemrogram fungsional terkenal karena memunculkan nama seperti `mappendor` atau `liftA4` yang tampak sangat alami ketika dilihat di lab matematika, tetapi memiliki kejelasan Darth Vader yang ragu-ragu ketika didorong dalam konteks lain.

Bagaimanapun, nama tersebut harus menjelaskan apa yang diberikan antarmuka ini kepada kita: _kemampuan untuk menerapkan functor satu sama lain_.

Sekarang, mengapa orang normal dan rasional seperti Anda menginginkan hal seperti itu? Apa artinya menerapkan satu functor ke functor lainnya?

Untuk menjawab pertanyaan-pertanyaan ini, kita akan mulai dengan situasi yang mungkin pernah Anda alami dalam perjalanan fungsional Anda.

Katakanlah, secara hipotetis, bahwa kita memiliki dua fungsi (dengan tipe yang sama) dan kita ingin memanggil fungsi kedua dengan nilainya sebagai argumen. Sesuatu yang sederhana seperti menambahkan nilai dua `Container`.

```js
// We can't do this because the numbers are bottled up.
add(Container.of(2), Container.of(3));
// NaN

// Let's use our trusty map
const containerOfAdd2 = map(add, Container.of(2));
// Container(add(2))
```

Kami memiliki diri kami sendiri `Container` dengan fungsi yang diterapkan sebagian di dalamnya.

Lebih khusus lagi, kami memiliki `Container(add(2))` dan kami ingin menggunakan `add(2)` ke `3` dalam `Container(3) `untuk menyelesaikan panggilan.

Dengan kata lain, kami ingin menerapkan satu fungsi ke fungsi lainnya.

Sekarang, kebetulan kita sudah memiliki alat untuk menyelesaikan tugas ini. Kami bisa `chain` dan kemudian `map` digunakan sebagian `add(2)` seperti:

```js
Container.of(2).chain((two) => Container.of(3).map(add(two)));
```

Masalahnya di sini adalah bahwa kita terjebak dalam dunia monad yang berurutan di mana tidak ada yang dapat dievaluasi sampai monad sebelumnya menyelesaikan bisnisnya.

Kami memiliki dua nilai independen yang kuat dan saya pikir tidak perlu menunda pembuat `Container(3)` hanya untuk memenuhi tuntutan berurutan monad.

Bahkan, akan sangat menyenangkan jika kita dapat secara ringkas menerapkan konten satu functor ke nilai lain tanpa fungsi dan variabel yang tidak perlu ini jika kita menemukan diri kita dalam toples acar ini.

## Kapal dalam Botol

<img src="images/ship_in_a_bottle.jpg" alt="https://www.deviantart.com/hollycarden" />

`ap` adalah fungsi yang dapat menerapkan konten fungsi dari satu fungsi ke konten nilai yang lain. Katakan itu lima kali dengan cepat.

```js
Container.of(add(2)).ap(Container.of(3));
// Container(5)

// all together now

Container.of(2).map(add).ap(Container.of(3));
// Container(5)
```

Di sana kita, bagus dan rapi. Kabar baik karena `Container(3)` telah dibebaskan dari penjara fungsi monadik bersarang.

Perlu disebutkan lagi bahwa `add`, dalam hal ini, akan diterapkan sebagian selama `map` yang pertama jadi ini hanya berfungsi ketika `add` adalah kari.

Kita dapat mendefinisikan `ap` seperti ini:

```js
Container.prototype.ap = function (otherContainer) {
  return otherContainer.map(this.$value);
};
```

Ingat, `this.$value` akan menjadi fungsi dan kami akan menerima fungsi lain sehingga kami hanya membutuhkan `map`. Dan dengan itu kami memiliki definisi antarmuka kami:

> Functor aplikatif adalah pointed functor dengan method `ap`

Perhatikan ketergantungan pada **pointed**. Antarmuka pointed sangat penting di sini seperti yang akan kita lihat di seluruh contoh berikut.

Sekarang, saya merasakan skeptisisme Anda (atau mungkin kebingungan dan kengerian), tetapi tetaplah berpikiran terbuka; karakter `ap` ini akan terbukti berguna. Sebelum kita masuk ke dalamnya, mari kita jelajahi properti yang bagus.

```js
F.of(x).map(f) === F.of(f).ap(F.of(x));
```

Dalam bahasa Inggris yang tepat, pemetaan `f` setara dengan `ap` functor dari `f`. Atau dalam bahasa Inggris yang lebih tepat, kami dapat menempatkan `x` ke dalam wadah kami dan `map(f)` ATAU kami dapat mengangkat keduanya `f` dan `x` ke dalam wadah `ap`.

Ini memungkinkan kita untuk menulis dari kiri ke kanan:

```js
Maybe.of(add).ap(Maybe.of(2)).ap(Maybe.of(3));
// Maybe(5)

Task.of(add).ap(Task.of(2)).ap(Task.of(3));
// Task(5)
```

Seseorang bahkan mungkin mengenali bentuk samar dari panggilan fungsi normal jika dilihat dengan mata juling.

Kita akan melihat versi pointfree nanti di bab ini, tetapi untuk saat ini, ini adalah cara yang lebih disukai untuk menulis kode tersebut.

Dengan menggunakan `of`, setiap nilai diangkut ke negeri wadah ajaib, ini alam semesta paralel di mana setiap aplikasi dapat menjadi asinkron atau null atau apapun yang Anda miliki dan `ap` akan menerapkan fungsi di tempat yang fantastis ini.

Ini seperti membangun kapal dalam botol.

Apakah Anda melihat di sana? Kami menggunakan `Task` dalam contoh kami. Ini adalah situasi utama di mana fungsi aplikatif menarik beban mereka. Mari kita lihat contoh yang lebih mendalam.

## Motivasi Koordinasi

Katakanlah kami sedang membangun situs perjalanan dan kami ingin mengambil daftar tujuan wisata dan acara lokal.

Masing-masing adalah panggilan api yang terpisah dan berdiri sendiri.

```js
// Http.get :: String -> Task Error HTML

const renderPage = curry((destinations, events) => {
  /* render page */
});

Task.of(renderPage).ap(Http.get("/destinations")).ap(Http.get("/events"));
// Task("<div>some page with dest and events</div>")
```

Kedua panggilan `Http` akan terjadi secara instan dan `renderPage` akan dipanggil ketika keduanya diselesaikan.

Bandingkan ini dengan versi monadik di mana seseorang harus menyelesaikannya `Task` sebelum tembakan berikutnya. Karena kita tidak memerlukan tujuan untuk mengambil peristiwa, kita bebas dari evaluasi urutan.

Sekali lagi, karena kami menggunakan aplikasi parsial untuk mencapai hasil ini, kami harus memastikan bahwa adalah `renderPage` kari atau tidak akan menunggu kedua `Tasks` smapai selesai.

Kebetulan, jika Anda pernah melakukan hal seperti itu secara manual, Anda akan menghargai kesederhanaan yang menakjubkan dari antarmuka ini. Ini adalah jenis kode indah yang membawa kita selangkah lebih dekat ke singularitas.

Mari kita lihat contoh lain.

```js
// $ :: String -> IO DOM
const $ = (selector) => new IO(() => document.querySelector(selector));

// getVal :: String -> IO String
const getVal = compose(map(prop("value")), $);

// signIn :: String -> String -> Bool -> User
const signIn = curry((username, password, rememberMe) => {
  /* signing in */
});

IO.of(signIn).ap(getVal("#email")).ap(getVal("#password")).ap(IO.of(false));
// IO({ id: 3, email: 'gg@allin.com' })
```

`signIn` adalah fungsi kari dari 3 argumen jadi kita harus melakukan `ap`.

Dengan masing-masing `ap`, `signIn` menerima satu argumen lagi hingga selesai dan berjalan.

Kita dapat melanjutkan pola ini dengan argumen sebanyak yang diperlukan. Hal lain yang perlu diperhatikan adalah bahwa dua argumen di `IO` berakhir secara alami sedangkan yang terakhir membutuhkan sedikit bantuan `of` untuk mengangkat `IO` karena `ap` mengharapkan fungsi dan semua argumennya berada dalam tipe yang sama.

## Bro, Apakah Anda Bahkan Mengangkat?

Mari kita periksa cara pointfree untuk menulis panggilan aplikatif ini. Karena kita tahu `map` sama dengan `of/ap`, kita dapat menulis fungsi generik yang akan `ap` sebanyak yang kita tentukan:

```js
const liftA2 = curry((g, f1, f2) => f1.map(g).ap(f2));

const liftA3 = curry((g, f1, f2, f3) => f1.map(g).ap(f2).ap(f3));

// liftA4, etc
```

`liftA2` adalah nama yang aneh. Kedengarannya seperti salah satu elevator di pabrik kumuh atau piring meja rias untuk perusahaan limusin murah.

Namun, setelah tercerahkan, itu cukup jelas: angkat potongan-potongan ini ke dunia functor aplikatif.

Ketika saya pertama kali melihat 2-3-4 omong kosong ini menurut saya jelek dan tidak perlu.

Lagi pula, kita dapat memeriksa arity fungsi dalam JavaScript dan membangunnya secara dinamis.

Namun, seringkali berguna untuk menerapkan `liftA(N)` dirinya sendiri secara parsial, sehingga tidak dapat bervariasi dalam panjang argumen.

Mari kita lihat ini digunakan:

```js
// checkEmail :: User -> Either String Email
// checkName :: User -> Either String String

const user = {
  name: "John Doe",
  email: "blurp_blurp",
};

//  createUser :: Email -> String -> IO User
const createUser = curry((email, name) => {
  /* creating... */
});

Either.of(createUser).ap(checkEmail(user)).ap(checkName(user));
// Left('invalid email')

liftA2(createUser, checkEmail(user), checkName(user));
// Left('invalid email')
```

Karena `createUser` mengambil dua argumen, kami menggunakan yang sesuai `liftA2`.

Kedua pernyataan itu setara, tetapi `liftA2` tidak menyebutkan `Either`. Ini membuatnya lebih umum dan fleksibel karena kita tidak lagi menikah dengan tipe tertentu.

Mari kita lihat contoh sebelumnya yang ditulis seperti ini:

```js
liftA2(add, Maybe.of(2), Maybe.of(3));
// Maybe(5)

liftA2(renderPage, Http.get("/destinations"), Http.get("/events"));
// Task('<div>some page with dest and events</div>')

liftA3(signIn, getVal("#email"), getVal("#password"), IO.of(false));
// IO({ id: 3, email: 'gg@allin.com' })
```

## Operators

Dalam bahasa seperti Haskell, Scala, PureScript, dan Swift, di mana dimungkinkan untuk membuat operator infix Anda sendiri, Anda mungkin melihat sintaks seperti ini:

```hs
-- Haskell / PureScript
add <$> Right 2 <*> Right 3
```

```js
// JavaScript
map(add, Right(2)).ap(Right(3));
```

Ini membantu untuk mengetahui bahwa `<$>` adalah `map` (alias `fmap`) dan `<*>` hanya `ap`. Ini memungkinkan gaya aplikasi fungsi yang lebih alami dan dapat membantu menghapus beberapa tanda kurung.

## Bebas Dapat Membuka Kaleng

<img src="images/canopener.jpg" alt="http://www.breannabeckmeyer.com/" />

Kami belum berbicara banyak tentang fungsi turunan. Melihat semua antarmuka ini dibangun satu sama lain dan mematuhi seperangkat hukum, kita dapat mendefinisikan beberapa antarmuka yang lebih lemah dalam hal yang lebih kuat.

Misalnya, kita tahu bahwa aplikatif pertama-tama adalah functor, jadi jika kita memiliki instance aplikatif, tentu kita dapat mendefinisikan functor untuk tipe kita.

Harmoni komputasi yang sempurna semacam ini dimungkinkan karena kita bekerja dalam kerangka matematika.

Mozart tidak bisa berbuat lebih baik bahkan jika dia telah melakukan torrent pada Ableton sebagai seorang anak.

Saya sebutkan sebelumnya yang `of/ap` setara dengan `map`. Kita dapat menggunakan pengetahuan ini untuk mendefinisikan `map` secara bebas:

```js
// map derived from of/ap
X.prototype.map = function map(f) {
  return this.constructor.of(f).ap(this);
};
```

Monad berada di puncak rantai makanan, jadi jika kita memiliki `chain`, kita mendapatkan functor dan aplikatif secara gratis:

```js
// map derived from chain
X.prototype.map = function map(f) {
  return this.chain((a) => this.constructor.of(f(a)));
};

// ap derived from chain/map
X.prototype.ap = function ap(other) {
  return this.chain((f) => other.map(f));
};
```

Jika kita dapat mendefinisikan monad, kita dapat mendefinisikan antarmuka aplikatif dan functor.

Ini cukup luar biasa karena kami mendapatkan semua pembuka kaleng ini secara gratis. Kami bahkan dapat memeriksa jenis dan mengotomatisasi proses ini.

Harus ditunjukkan bahwa bagian dari daya tarik `ap` adalah kemampuan untuk menjalankan berbagai hal secara bersamaan sehingga mendefinisikan itu melalui `chain` kehilangan pengoptimalan.

Meskipun demikian, ada baiknya untuk memiliki antarmuka yang langsung berfungsi sementara seseorang mengerjakan implementasi sebaik mungkin.

Mengapa tidak menggunakan monad saja dan menyelesaikannya, Anda bertanya? Ini adalah praktik yang baik untuk bekerja dengan tingkat kekuatan yang Anda butuhkan, tidak lebih, tidak kurang.

Ini menjaga beban kognitif seminimal mungkin dengan mengesampingkan fungsionalitas yang mungkin. Untuk alasan ini, ada baiknya memilih aplikatif daripada monad.

Monad memiliki kemampuan unik untuk mengurutkan komputasi, menetapkan variabel, dan menghentikan eksekusi lebih lanjut semua berkat struktur bersarang ke bawah.

Ketika seseorang melihat aplikasi digunakan, mereka tidak perlu menyibukkan diri dengan bisnis itu.

Sekarang, ke legalitas ...

## Hukum

Seperti konstruksi matematika lainnya yang telah kita jelajahi, fungsi aplikatif memiliki beberapa properti yang berguna untuk kita andalkan dalam kode harian kita.

Pertama, Anda harus tahu bahwa aplikatif "tertutup dalam komposisi", artinya `ap` tidak akan pernah mengubah jenis wadah pada kami (alasan lain untuk mendukung monad).

Itu tidak berarti kami tidak dapat memiliki beberapa efek yang berbeda - kami dapat menumpuk jenis kami mengetahui bahwa mereka akan tetap sama selama keseluruhan aplikasi kami.

Untuk menunjukkan:

```js
const tOfM = compose(Task.of, Maybe.of);

liftA2(
  liftA2(concat),
  tOfM("Rainy Days and Mondays"),
  tOfM(" always get me down")
);
// Task(Maybe(Rainy Days and Mondays always get me down))
```

Lihat, tidak perlu khawatir tentang jenis yang berbeda masuk ke dalam campuran.

Saatnya melihat hukum kategoris favorit kita: _identitas_

### Identitas

```js
// identity
A.of(id).ap(v) === v;
```

Benar, jadi menerapkan semua `id` dari dalam functor seharusnya tidak mengubah nilai dalam `v`.

Sebagai contoh:

```js
const v = Identity.of("Pillow Pets");
Identity.of(id).ap(v) === v;
```

`Identity.of(id)` membuatku tertawa melihat kesia-siaannya.

Bagaimanapun, yang menarik adalah bahwa, seperti yang telah kita tetapkan, `of/ap` sama dengan `map` berikut hukum langsung dari identitas functor: `map(id) == id`.

Keindahan dalam menggunakan hukum ini adalah, seperti pelatih olahraga taman kanak-kanak yang militan, mereka memaksa semua antarmuka kita untuk bermain bersama dengan baik.

### Homomorfisme

```js
// homomorphism
A.of(f).ap(A.of(x)) === A.of(f(x));
```

Homomorfisme hanya struktur melestarikan map. Faktanya, functor hanyalah homomorfisme antar kategori karena fungsi tersebut mempertahankan struktur kategori asli di bawah pemetaan.

Kami benar-benar hanya memasukkan fungsi dan nilai normal kami ke dalam wadah dan menjalankan perhitungan di sana sehingga tidak mengherankan bahwa kami akan berakhir dengan hasil yang sama jika kami menerapkan semuanya di dalam wadah (sisi kiri persamaan) atau terapkan di luar, lalu letakkan di sana (sisi kanan).

Contoh cepat:

```js
Either.of(toUpperCase).ap(Either.of("oreos")) ===
  Either.of(toUpperCase("oreos"));
```

### Pertukaran

Interchange (pertukaran) hukum state yang tidak peduli jika kita memilih untuk mengangkat fungsi kita ke sisi kiri atau kanan ap.

```js
// interchange
v.ap(A.of(x)) === A.of((f) => f(x)).ap(v);
```

Berikut ini contohnya:

```js
const v = Task.of(reverse);
const x = "Sparklehorse";

v.ap(Task.of(x)) === Task.of((f) => f(x)).ap(v);
```

### Komposisi

Dan akhirnya komposisi yang hanya merupakan cara untuk memeriksa bahwa komposisi fungsi standar kami berlaku saat digunakan di dalam wadah.

```js
// composition
A.of(compose).ap(u).ap(v).ap(w) === u.ap(v.ap(w));
```

```js
const u = IO.of(toUpperCase);
const v = IO.of(concat("& beyond"));
const w = IO.of("blood bath ");

IO.of(compose).ap(u).ap(v).ap(w) === u.ap(v.ap(w));
```

## Singkatnya

Kasus penggunaan yang baik untuk aplikatif adalah ketika seseorang memiliki beberapa argumen functor.

Mereka memberi kita kemampuan untuk menerapkan fungsi ke argumen semua dalam dunia functor.

Meskipun kita sudah bisa melakukannya dengan monad, kita harus memilih functor aplikatif ketika kita tidak membutuhkan fungsionalitas khusus monadik.

Kami hampir selesai dengan kontainer apis. Kami telah belajar bagaimana `map`, `chain`, dan sekarang fungsi `ap`.

Di bab berikutnya, kita akan mempelajari cara bekerja lebih baik dengan banyak fungsi dan membongkarnya dengan cara yang berprinsip.

[Bab 11: Transformasi Lagi, Secara Alami](ch11-id.md)

## Latihan

{% exercise %}  
Tulis fungsi yang menambahkan dua kemungkinan null bersama-sama menggunakan `Maybe` dan `ap`.

{% initial src="./exercises/ch10/exercise_a.js#L3;" %}

```js
// safeAdd :: Maybe Number -> Maybe Number -> Maybe Number
const safeAdd = undefined;
```

{% solution src="./exercises/ch10/solution_a.js" %}  
{% validation src="./exercises/ch10/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

{% exercise %}  
Tulis ulang `safeAdd` dari exercise_b mengggunakan `liftA2` sebagai pengganti `ap`.

{% initial src="./exercises/ch10/exercise_b.js#L3;" %}

```js
// safeAdd :: Maybe Number -> Maybe Number -> Maybe Number
const safeAdd = undefined;
```

{% solution src="./exercises/ch10/solution_b.js" %}  
{% validation src="./exercises/ch10/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Untuk latihan selanjutnya, kami mempertimbangkan helper berikut:

```js
const localStorage = {
  player1: { id: 1, name: "Albert" },
  player2: { id: 2, name: "Theresa" },
};

// getFromCache :: String -> IO User
const getFromCache = (x) => new IO(() => localStorage[x]);

// game :: User -> User -> String
const game = curry((p1, p2) => `${p1.name} vs ${p2.name}`);
```

{% exercise %}  
Tulis `IO` yang mendapatkan player1 dan player2 dari cache dan mulai permainan.

{% initial src="./exercises/ch10/exercise_c.js#L16;" %}

```js
// startGame :: IO String
const startGame = undefined;
```

{% solution src="./exercises/ch10/solution_c.js" %}  
{% validation src="./exercises/ch10/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
