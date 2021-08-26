# Bab 01: Apa Yang Pernah Kita Lakukan?

## Perkenalan

Hai, yang di sana! Saya Profesor Franklin Frisby. Senang membuat kenalan Anda.

Kita akan menghabiskan waktu bersama, karena saya seharusnya mengajari Anda sedikit tentang pemrograman fungsional.

Tapi cukup tentang saya, bagaimana dengan Anda? Saya harap Anda setidaknya sedikit akrab dengan bahasa JavaScript, memiliki sedikit pengalaman Berorientasi Objek, dan menyukai diri Anda sebagai programmer kelas pekerja.

Anda tidak perlu memiliki gelar PhD di bidang Entomologi, Anda hanya perlu tahu cara menemukan dan membunuh beberapa bug.

Saya tidak akan berasumsi bahwa Anda memiliki pengetahuan pemrograman fungsional sebelumnya, karena kami berdua tahu apa yang terjadi ketika Anda berasumsi.

Namun, saya akan mengharapkan Anda mengalami beberapa situasi yang tidak menguntungkan yang muncul ketika bekerja dengan keadaan yang dapat berubah, efek samping yang tidak terbatas, dan desain yang tidak berprinsip.

Sekarang kita telah diperkenalkan dengan benar, mari kita lanjutkan.

Tujuan bab ini adalah untuk memberi Anda gambaran tentang apa yang kita cari ketika kita menulis program fungsional.

Agar dapat memahami bab-bab berikut, kita harus memiliki beberapa gagasan tentang apa yang membuat suatu program berfungsi.

Kalau tidak, kita akan mendapati diri kita mencoret-coret tanpa tujuan, menghindari objek dengan cara apa pun - usaha yang kikuk memang.

Kami membutuhkan sasaran yang jelas untuk melemparkan kode kami, beberapa kompas surgawi ketika air menjadi kasar.

Sekarang, ada beberapa prinsip pemrograman umum - berbagai akronim kredo yang memandu kita melalui terowongan gelap aplikasi apa pun:

- DRY (_don't repeat yourself_).
- YAGNI (_ya ain't gonna need it_).
- Kopling longgar kohesi tinggi (_loose coupling high cohesion_).
- Prinsip paling tidak mengejutkan (_the principle of least surprise_).
- Tanggung jawab tunggal (_single responsibility_).
- ...dan sebagainya.

Saya tidak akan mengganggu Anda dengan mendaftar setiap pedoman yang saya dengar selama bertahun-tahun...

Intinya adalah bahwa mereka bertahan dalam pengaturan fungsional, meskipun mereka hanya bersinggungan dengan tujuan akhir kita.

Apa yang saya ingin Anda rasakan untuk saat ini, sebelum kita melangkah lebih jauh, adalah niat kita ketika kita menyodok dan menekan keyboard; Xanadu fungsional kami.

<!--BREAK-->

## Pertemuan Singkat

Mari kita mulai dengan sentuhan kegilaan.

Berikut adalah aplikasi burung camar.

Ketika kawanan bergabung, mereka menjadi kawanan yang lebih besar, dan ketika mereka berkembang biak, mereka bertambah dengan jumlah burung camar dengan siapa mereka berkembang biak.

Sekarang, ini tidak dimaksudkan untuk menjadi kode Berorientasi Objek yang baik, ingatlah, ini di sini untuk menyoroti bahaya pendekatan berbasis penugasan modern kami.

Lihat:

```js
class Flock {
  constructor(n) {
    this.seagulls = n;
  }

  conjoin(other) {
    this.seagulls += other.seagulls;
    return this;
  }

  breed(other) {
    this.seagulls = this.seagulls * other.seagulls;
    return this;
  }
}

const flockA = new Flock(4);
const flockB = new Flock(2);
const flockC = new Flock(0);
const result = flockA
  .conjoin(flockC)
  .breed(flockB)
  .conjoin(flockA.breed(flockB)).seagulls;
// 32
```

Siapa yang akan membuat kekejian yang begitu mengerikan? Sangat sulit untuk melacak keadaan internal yang bermutasi.

Dan, astaga, jawabannya bahkan salah! Seharusnya `16`, tetapi `flockA` akhirnya diubah secara permanen dalam prosesnya.

Buruk `flockA`. Ini adalah anarki di TI! Ini adalah aritmatika hewan liar!

Jika Anda tidak memahami program ini, tidak apa-apa, saya juga tidak. Yang perlu diingat di sini adalah bahwa status dan nilai yang dapat diubah sulit untuk diikuti, bahkan dalam contoh kecil.

Mari kita coba lagi, kali ini menggunakan pendekatan yang lebih fungsional:

```js
const conjoin = (flockX, flockY) => flockX + flockY;
const breed = (flockX, flockY) => flockX * flockY;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result = conjoin(
  breed(flockB, conjoin(flockA, flockC)),
  breed(flockA, flockB)
);
// 16
```

Nah, kali ini kami mendapat jawaban yang tepat. Dengan kode yang jauh lebih sedikit.

Fungsi bersarang agak membingungkan... (kami akan memperbaiki situasi ini di bab 5).

Ini lebih baik, tapi mari kita gali sedikit lebih dalam.

Ada manfaat untuk menyebut sekop sekop. Seandainya kami meneliti fungsi kustom kami lebih dekat, kami akan menemukan bahwa kami hanya bekerja dengan penambahan sederhana (`conjoin`) dan perkalian (`breed`).

Sebenarnya tidak ada yang istimewa dari kedua fungsi ini selain namanya. Mari kita ganti nama fungsi kustom kita menjadi `multiply` dan `add` untuk mengungkapkan identitas aslinya.

```js
const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result = add(
  multiply(flockB, add(flockA, flockC)),
  multiply(flockA, flockB)
);
// 16
```

Dan dengan itu, kita mendapatkan pengetahuan orang dahulu:

```js
// associative
add(add(x, y), z) === add(x, add(y, z));

// commutative
add(x, y) === add(y, x);

// identity
add(x, 0) === x;

// distributive
multiply(x, add(y, z)) === add(multiply(x, y), multiply(x, z));
```

Ah ya, sifat matematika lama yang setia itu seharusnya berguna.

Jangan khawatir jika Anda tidak tahu mereka langsung dari atas kepala Anda.

Bagi banyak dari kita, sudah lama sejak kita belajar tentang hukum aritmatika ini.

Mari kita lihat apakah kita dapat menggunakan properti ini untuk menyederhanakan program burung camar kecil kita.

```js
// Original line
add(multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB));

// Apply the identity property to remove the extra add
// (add(flockA, flockC) == flockA)
add(multiply(flockB, flockA), multiply(flockA, flockB));

// Apply distributive property to achieve our result
multiply(flockB, add(flockA, flockA));
```

Cemerlang! Kami tidak perlu menulis kode khusus selain fungsi panggilan kami.

Kami mendefinisi `add` dan `multiply` di sini untuk kelengkapan, tetapi sebenarnya tidak perlu menulisnya - kami pasti memiliki `add` dan `multiply` yang disediakan oleh beberapa perpustakaan yang ada.

Anda mungkin berpikir "betapa bodohnya Anda untuk menempatkan contoh matematika seperti itu di depan".

Atau "program nyata tidak sesederhana ini dan tidak dapat dipikirkan sedemikian rupa."

Saya memilih contoh ini karena kebanyakan dari kita sudah tahu tentang penjumlahan dan perkalian, jadi mudah untuk melihat bagaimana matematika sangat berguna bagi kita di sini.

Jangan putus asa - di sepanjang buku ini, kita akan menaburkan beberapa teori kategori, teori himpunan, dan kalkulus lambda dan menulis contoh dunia nyata yang mencapai kesederhanaan dan hasil elegan yang sama seperti contoh kawanan burung camar kita.

Anda juga tidak perlu menjadi ahli matematika. Ini akan terasa alami dan mudah, sama seperti Anda menggunakan kerangka kerja atau API "normal".

Mungkin mengejutkan mendengar bahwa kita dapat menulis aplikasi sehari-hari yang lengkap di sepanjang garis analog fungsional di atas.

Program yang memiliki sifat suara. Program yang singkat, namun mudah untuk dipikirkan. Program yang tidak menemukan kembali roda di setiap belokan.

Pelanggaran hukum itu baik jika Anda seorang penjahat, tetapi dalam buku ini, kita ingin mengakui dan mematuhi hukum matematika.

Kami ingin menggunakan teori di mana setiap bagian cenderung cocok satu sama lain dengan sangat sopan.

Kami ingin mewakili masalah khusus kami dalam hal bit generik yang dapat dikomposisi dan kemudian mengeksploitasi propertinya untuk keuntungan egois kami sendiri.

Ini akan membutuhkan sedikit lebih banyak disiplin daripada pendekatan "apa saja" dari pemrograman imperatif (kita akan membahas definisi yang tepat dari "imperatif" nanti di buku ini, tetapi untuk saat ini pertimbangkan apa pun selain pemrograman fungsional).

Hasil dari bekerja dalam kerangka matematika yang berprinsip akan benar-benar mengejutkan Anda.

Kami telah melihat kerlip bintang utara fungsional kami, tetapi ada beberapa konsep konkret yang harus dipahami sebelum kami benar-benar dapat memulai perjalanan kami.

[Bab 02: Fungsi Kelas Satu (First Class Function)](ch02-id.md)
