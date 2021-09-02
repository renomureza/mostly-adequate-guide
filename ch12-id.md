# Bab 12: Melintasi Batu

Sejauh ini, di _cirque du conteneur_, Anda telah melihat kami menjinakkan [functor](./ch08-id.md#functor-pertamaku) yang ganas, membengkokkannya sesuai keinginan kami untuk melakukan operasi apa pun yang sesuai dengan keinginan.

Anda telah terpesona oleh juggling banyak efek berbahaya sekaligus menggunakan fungsi [aplikasi](./ch10-id.md) untuk mengumpulkan hasilnya.

Duduk di sana dengan takjub saat kontainer menghilang di udara tipis dengan menyatukannya.

Pada sideshow efek samping, kami telah melihat mereka dikomposisikan menjadi satu.

Dan baru-baru ini, kami telah berkelana melampaui apa yang alami dan [mengubah](./ch11-id.md) satu jenis menjadi jenis lain di depan mata Anda.

Dan sekarang untuk trik berikutnya, kita akan melihat traversal.

Kita akan melihat tipe-tipe melambung satu sama lain seolah-olah mereka adalah seniman trapeze yang memegang nilai kita secara utuh.

Kami akan menyusun ulang efek seperti troli dengan gerakan miring. Ketika wadah kami terjalin seperti anggota badan manusia karet, kami dapat menggunakan antarmuka ini untuk meluruskan semuanya.

Kami akan menyaksikan efek yang berbeda dengan urutan yang berbeda. Ambilkan saya pantalon dan peluit geser, mari kita mulai.

## Tipe dan Tipe

Ayo menggila:

```js
// readFile :: FileName -> Task Error String

// firstWords :: String -> String
const firstWords = compose(intercalate(" "), take(3), split(" "));

// tldr :: FileName -> Task Error String
const tldr = compose(map(firstWords), readFile);

map(tldr, ["file1", "file2"]);
// [Task('hail the monarchy'), Task('smash the patriarchy')]
```

Di sini kita membaca banyak file dan berakhir dengan serangkaian tugas yang tidak berguna.

Bagaimana kita bisa memotong masing-masing dari ini? Akan sangat menyenangkan jika kita dapat mengganti tipe menjadi have `Task Error [String]` alih - alih `[Task Error String]`.

Dengan begitu, kita akan memiliki satu nilai masa depan yang menampung semua hasil, yang jauh lebih sesuai dengan kebutuhan asinkron kita daripada beberapa nilai masa depan yang tiba di waktu luang mereka.

Inilah satu contoh terakhir dari situasi yang sulit:

```js
// getAttribute :: String -> Node -> Maybe String
// $ :: Selector -> IO Node

// getControlNode :: Selector -> IO (Maybe (IO Node))
const getControlNode = compose(
  map(map($)),
  map(getAttribute("aria-controls")),
  $
);
```

Lihatlah kerinduan `IO` mereka untuk bersama.

Itu akan menyenangkan bagi `join`, biarkan mereka menari pipi ke pipi, tapi sayangnya `Maybe` berdiri di antara mereka seperti pendamping di prom.

Langkah terbaik kami di sini adalah menggeser posisi mereka di samping satu sama lain, dengan cara itu setiap tipe akhirnya dapat bersama dan tanda tangan kami dapat disederhanakan menjadi `IO (Maybe Node)`.

## Tipe Feng Shui

Antarmuka _Traversable_ terdiri dari dua fungsi yang mulia: `sequence` dan `traverse`.

Mari kita atur ulang tipe kita menggunakan `sequence`:

```js
sequence(List.of, Maybe.of(["the facts"])); // [Just('the facts')]
sequence(Task.of, new Map({ a: Task.of(1), b: Task.of(2) })); // Task(Map({ a: 1, b: 2 }))
sequence(IO.of, Either.of(IO.of("buckle my shoe"))); // IO(Right('buckle my shoe'))
sequence(Either.of, [Either.of("wing")]); // Right(['wing'])
sequence(Task.of, left("wing")); // Task(Left('wing'))
```

Lihat apa yang terjadi di sini? Tipe bersarang kami dibolak-balik seperti celana kulit di malam musim panas yang lembap.

Fungsi di dalam digeser ke luar dan sebaliknya. Perlu diketahui bahwa `sequence` agak khusus tentang argumennya.

Ini terlihat seperti ini:

```js
// sequence :: (Traversable t, Applicative f) => (a -> f a) -> t (f a) -> f (t a)
const sequence = curry((of, x) => x.sequence(of));
```

Mari kita mulai dengan argumen kedua.

Itu pasti Traversable yang memegang Applicative, yang terdengar sangat membatasi, tetapi kebetulan lebih sering daripada tidak.

Ini adalah `t (f a)` yang akan berubah menjadi `f (t a)`.

Bukankah itu ekspresif? Jelas sekali kedua jenis itu melakukan do-si-do satu sama lain.

Argumen pertama itu hanyalah penopang dan hanya diperlukan dalam bahasa yang tidak diketik. Ini adalah konstruktor tipe yang disediakan dari kami sehingga kita bisa membalikkan jenis _map-reluctant_ seperti `Left` - lebih pada bahwa dalam satu menit.

Dengan menggunakan `sequence`, kita dapat menggeser tipe di sekitar dengan presisi thimblerigger trotoar.

Tapi bagaimana cara kerjanya? Mari kita lihat bagaimana sebuah tipe, katakanlah `Either`, akan mengimplementasikannya:

```js
class Right extends Either {
  // ...
  sequence(of) {
    return this.$value.map(Either.of);
  }
}
```

Ah ya, jika `$value` adalah functor (sebenarnya pasti aplikatif), kita cukup menggunakan constructor untuk `map` melompati tipe katak.

Anda mungkin telah memperhatikan bahwa kami telah mengabaikan seluruh `of`. Itu diteruskan untuk kesempatan di mana pemetaan sia-sia, seperti halnya dengan Left:

```js
class Left extends Either {
  // ...
  sequence(of) {
    return of(this);
  }
}
```

Kami ingin tipe selalu berakhir dalam pengaturan yang sama, oleh karena itu perlu tipe seperti `Left` yang tidak benar-benar memegang aplikatif batin kita untuk mendapatkan sedikit bantuan dalam melakukannya.

Antarmuka Aplikatif mengharuskan kita pertama memiliki Functor Pointed jadi kita akan selalu memiliki `of` untuk lulus.

Dalam bahasa dengan sistem tipe, tipe luar dapat disimpulkan dari tanda tangan dan tidak perlu secara eksplisit diberikan.

## Berbagai macam efek

Urutan yang berbeda memiliki hasil yang berbeda dalam hal kontainer kami.

Jika saya memiliki `[Maybe a]`, itu adalah kumpulan nilai yang mungkin, sedangkan jika saya memiliki `Maybe [a]`, itu adalah kumpulan nilai yang mungkin.

Yang pertama menunjukkan bahwa kita akan memaafkan dan mempertahankan "yang baik", sedangkan yang kedua berarti itu adalah tipe situasi "semua atau tidak sama sekali".

Demikian juga, `Either Error (Task Error a)` bisa mewakili validasi sisi klien dan `Task Error (Either Error a)` bisa menjadi sisi server. Tipe dapat ditukar untuk memberi kita efek yang berbeda.

```js
// fromPredicate :: (a -> Bool) -> a -> Either e a

// partition :: (a -> Bool) -> [a] -> [Either e a]
const partition = (f) => map(fromPredicate(f));

// validate :: (a -> Bool) -> [a] -> Either e [a]
const validate = (f) => traverse(Either.of, fromPredicate(f));
```

Di sini kita memiliki dua fungsi yang berbeda berdasarkan jika kita `map` atau `traverse`.

Yang pertama, `partition` akan memberi kita array `Left` dan `Right` sesuai dengan fungsi predikat.

Ini berguna untuk menyimpan data berharga untuk digunakan di masa mendatang daripada menyaringnya dengan air mandi.

`validate` sebagai gantinya akan memberi kita item pertama yang gagal predikat di `Left`, atau semua item di `Right` jika semuanya keren dory.

Dengan memilih tipe urutan yang berbeda, kami mendapatkan perilaku yang berbeda.

Mari kita lihat `traverse` fungsi dari List, untuk melihat bagaimana method `validate` tersebut dibuat.

```js
traverse(of, fn) {
    return this.$value.reduce(
      (f, a) => fn(a).map(b => bs => bs.concat(b)).ap(f),
      of(new List([])),
    );
  }
```

Ini hanya menjalankan `reduce` dalam daftar.

Fungsi reduce adalah `(f, a) => fn(a).map(b => bs => bs.concat(b)).ap(f)`, yang terlihat agak menakutkan, jadi mari kita melangkah melaluinya.

1. `reduce(..., ...)`

   Ingat tanda tangan dari `reduce :: [a] -> (f -> a -> f) -> f -> f`. Argumen pertama sebenarnya disediakan oleh notasi titik pada `$value`, jadi ini adalah daftar objek. Kemudian kita membutuhkan fungsi dari `f` (akumulator) dan `a` (iteree) untuk mengembalikan akumulator baru kepada kita.

2. `of(new List([]))`

   Nilai benih adalah `of(new List([]))`, yang dalam kasus kami adalah `Right([]) :: Either e [a]`. Perhatikan bahwa `Either e [a]` itu juga akan menjadi tipe hasil akhir kita!

3. `fn :: Applicative f => a -> f a`

   Jika kita menerapkannya pada contoh kita di atas, `fn` sebenarnya `fromPredicate(f) :: a -> Either e a`.

   > fn(a) :: Either e a

4. `.map(b => bs => bs.concat(b))`

   Ketika `Right`, `Either.map` meneruskan nilai yang tepat ke fungsi dan mengembalikan `Right` yang baru dengan hasilnya.

   Dalam hal ini fungsi memiliki satu parameter `(b)`, dan mengembalikan fungsi lain (`bs => bs.concat(b)`, di mana `b` dalam ruang lingkup karena closures). Ketika `Left`, nilai kiri dikembalikan.

   > fn(a).map(b => bs => bs.concat(b)) :: Either e ([a] -> [a])

5. `.ap(f)`

   Ingat `f` itu adalah Aplikatif di sini, jadi kita bisa menerapkan fungsi `bs => bs.concat(b)` ke nilai `bs :: [a]` apa pun yang ada di `f`.

   Untungnya bagi kami, `f` berasal dari benih awal kami dan memiliki tipe berikut: `f :: Either e [a]` yang omong-omong diawetkan saat kami melamar `bs => bs.concat(b)`.

   Ketika `f` adalah `Right`, panggilan ini `bs => bs.concat(b)`, yang mengembalikan `Right` dengan item yang ditambahkan ke daftar.

   Ketika `Left`, nilai kiri (dari langkah sebelumnya atau iterasi sebelumnya masing-masing) dikembalikan.

   > fn(a).map(b => bs => bs.concat(b)).ap(f) :: Either e [a]

Transformasi yang tampaknya ajaib ini dicapai hanya dengan 6 baris kode yang sangat sedikit di `List.traverse`, dan diselesaikan dengan `of`, `map` dan `ap`, jadi akan berfungsi untuk semua Fungsi Aplikatif.

Ini adalah contoh yang bagus tentang bagaimana abstraksi tersebut dapat membantu menulis kode yang sangat umum hanya dengan beberapa asumsi (yang dapat, kebetulan, dideklarasikan dan diperiksa pada level tipe!).

## Tipe Waltz

Saatnya meninjau kembali dan membersihkan contoh awal kita.

```js
// readFile :: FileName -> Task Error String

// firstWords :: String -> String
const firstWords = compose(intercalate(" "), take(3), split(" "));

// tldr :: FileName -> Task Error String
const tldr = compose(map(firstWords), readFile);

traverse(Task.of, tldr, ["file1", "file2"]);
// Task(['hail the monarchy', 'smash the patriarchy']);
```

Menggunakan `traverse` alih-alih `map`, kami telah berhasil menggiring `Task` yang sulit diatur itu ke dalam array hasil terkoordinasi yang bagus.

Ini seperti `Promise.all()`, jika Anda terbiasa, kecuali itu bukan hanya fungsi khusus satu kali, tidak, ini berfungsi untuk semua jenis yang dapat dilalui.

Api matematika ini cenderung menangkap sebagian besar hal yang ingin kami lakukan dengan cara yang dapat dioperasikan dan dapat digunakan kembali, daripada setiap perpustakaan menciptakan kembali fungsi-fungsi ini untuk satu jenis.

Mari kita bersihkan contoh terakhir untuk closure (tidak, bukan semacam itu):

```js
// getAttribute :: String -> Node -> Maybe String
// $ :: Selector -> IO Node

// getControlNode :: Selector -> IO (Maybe Node)
const getControlNode = compose(
  chain(traverse(IO.of, $)),
  map(getAttribute("aria-controls")),
  $
);
```

Alih-alih `map(map($))` kita memiliki `chain(traverse(IO.of, $))` yang membalikkan tipe kita saat memetakan, kemudian meratakan kedua `IO` melalui `chain`.

## Tidak Ada Hukum dan Urutan

Nah sekarang, sebelum Anda mendapatkan semua penilaian dan menekan tombol spasi mundur seperti palu untuk mundur dari bab ini, luangkan waktu sejenak untuk menyadari bahwa hukum ini adalah jaminan kode yang berguna.

Ini adalah dugaan saya bahwa tujuan dari sebagian besar arsitektur program adalah upaya untuk menempatkan batasan yang berguna pada kode kami untuk mempersempit kemungkinan, untuk memandu kami ke dalam jawaban sebagai desainer dan pembaca.

Antarmuka tanpa hukum hanyalah tipuan.

Seperti struktur matematika lainnya, kita harus mengekspos properti untuk kewarasan kita sendiri.

Ini memiliki efek yang sama dengan enkapsulasi karena melindungi data, memungkinkan kita untuk menukar antarmuka dengan state lain yang taat hukum.

Ayo sekarang, kita punya beberapa hukum untuk dipecahkan.

### Identity

```js
const identity1 = compose(sequence(Identity.of), map(Identity.of));
const identity2 = Identity.of;

// test it out with Right
identity1(Either.of("stuff"));
// Identity(Right('stuff'))

identity2(Either.of("stuff"));
// Identity(Right('stuff'))
```

Ini harus langsung.

Jika kita menempatkan sebuah `Identity` di functor kita, kemudian membalikkannya ke luar dengan `sequence` itu sama saja dengan menempatkannya di luar untuk memulai.

Kami memilih `Right` sebagai kelinci percobaan kami karena mudah untuk diadili dan diperiksa.

Sebuah functor sewenang-wenang di sana normal, namun penggunaan functor konkret di sini, yaitu `Identity` dalam hukum itu sendiri mungkin menimbulkan beberapa alis.

Ingat suatu [kategori](./ch05-id.md#teori-kategori) ditentukan oleh morfisme antara objek-objeknya yang memiliki komposisi dan identitas asosiatif.

Ketika berhadapan dengan kategori functors, transformasi alami adalah morfisme dan `Identity` identitasnya.

Fungsi `Identity` itu sebagai fundamental dalam mendemonstrasikan hukum sebagai fungsi `compose` kita.

Faktanya, kita harus melepaskan hantu itu dan mengikutinya dengan tipe [Compose](./ch08.md#titik-teori):

### Komposisi

```js
const comp1 = compose(sequence(Compose.of), map(Compose.of));
const comp2 = (Fof, Gof) =>
  compose(Compose.of, map(sequence(Gof)), sequence(Fof));

// Test it out with some types we have lying around
comp1(Identity(Right([true])));
// Compose(Right([Identity(true)]))

comp2(Either.of, Array)(Identity(Right([true])));
// Compose(Right([Identity(true)]))
```

Hukum ini mempertahankan komposisi seperti yang diharapkan: jika kita menukar komposisi fungsi, kita seharusnya tidak melihat kejutan karena komposisi adalah fungsi itu sendiri.

Kami sewenang-wenang memilih `true`, `Right`, `Identity`, dan `Array` untuk mengujinya.

Perpustakaan seperti [quickcheck](https://hackage.haskell.org/package/QuickCheck) atau [jsverify](http://jsverify.github.io/) dapat membantu kami menguji hukum dengan menguji input secara kabur.

Sebagai konsekuensi alami dari hukum di atas, kita mendapatkan kemampuan untuk [menggabungkan traversals](https://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf), yang bagus dari sudut pandang kinerja.

### Kealamian

```js
const natLaw1 = (of, nt) => compose(nt, sequence(of));
const natLaw2 = (of, nt) => compose(sequence(of), map(nt));

// test with a random natural transformation and our friendly Identity/Right functors.

// maybeToEither :: Maybe a -> Either () a
const maybeToEither = (x) => (x.$value ? new Right(x.$value) : new Left());

natLaw1(Maybe.of, maybeToEither)(Identity.of(Maybe.of("barlow one")));
// Right(Identity('barlow one'))

natLaw2(Either.of, maybeToEither)(Identity.of(Maybe.of("barlow one")));
// Right(Identity('barlow one'))
```

Ini mirip dengan hukum identitas kita.

Jika kita mengayunkan tipenya pertama kali kemudian menjalankan transformasi alami di luar, itu seharusnya sama dengan memetakan transformasi alami, lalu membalik tipenya.

Akibat yang wajar dari hukum ini adalah:

```js
traverse(A.of, A.of) === A.of;
```

Yang, sekali lagi, bagus dari sudut pandang kinerja.

## Singkatnya

Traversable adalah antarmuka yang kuat yang memberi kita kemampuan untuk mengatur ulang tipe kita dengan kemudahan dekorator interior telekinetik.

Kita dapat mencapai efek yang berbeda dengan urutan yang berbeda serta menghilangkan tipe kerutan yang membuat `join` tidak dapat mengatasinya.

Selanjutnya, kita akan mengambil sedikit jalan memutar untuk melihat salah satu antarmuka pemrograman fungsional yang paling kuat dan bahkan mungkin aljabar itu sendiri: [Monoid menyatukan semuanya](./ch13-id.md)

## Latihan

Dengan memperhatikan unsur-unsur berikut:

```js
// httpGet :: Route -> Task Error JSON

// routes :: Map Route Route
const routes = new Map({ "/": "/", "/about": "/about" });
```

{% exercise %}  
Gunakan antarmuka yang dapat dilalui untuk mengubah jenis tanda tangan `getJsons` ke Map Route Route → Task Error (Map Route JSON)

{% initial src="./exercises/ch12/exercise_a.js#L11;" %}

```js
// getJsons :: Map Route Route -> Map Route (Task Error JSON)
const getJsons = map(httpGet);
```

{% solution src="./exercises/ch12/solution_a.js" %}  
{% validation src="./exercises/ch12/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Kami sekarang mendefinisikan fungsi validasi berikut:

```js
// validate :: Player -> Either String Player
const validate = (player) =>
  player.name ? Either.of(player) : left("must have name");
```

{% exercise %}  
Menggunakan traversable, dan fungsi validate, perbarui `startGame` (dan tanda tangannya) untuk hanya memulai permainan jika semua pemain valid

{% initial src="./exercises/ch12/exercise_b.js#L7;" %}

```js
// startGame :: [Player] -> [Either Error String]
const startGame = compose(map(map(always("game started!"))), map(validate));
```

{% solution src="./exercises/ch12/solution_b.js" %}  
{% validation src="./exercises/ch12/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}

---

Akhirnya, kami mempertimbangkan beberapa helper sistem file:

```js
// readfile :: String -> String -> Task Error String
// readdir :: String -> Task Error [String]
```

{% exercise %}  
Gunakan traversable untuk mengatur ulang dan meratakan Tasks & Maybe bersarang

{% initial src="./exercises/ch12/exercise_c.js#L8;" %}

```js
// readFirst :: String -> Task Error (Maybe (Task Error String))
const readFirst = compose(map(map(readfile("utf-8"))), map(safeHead), readdir);
```

{% solution src="./exercises/ch12/solution_c.js" %}  
{% validation src="./exercises/ch12/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}
