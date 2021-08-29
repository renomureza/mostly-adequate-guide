# Bab 02: First Class Functions (Fungsi Kelas Satu)

## Ulasan Singkat

Ketika kita mengatakan bahwa fungsi adalah "kelas pertama", yang kita maksud adalah mereka sama seperti yang lainnya... jadi dengan kata lain kelas normal.

Kami dapat memperlakukan fungsi seperti tipe data lainnya dan tidak ada yang khusus tentang mereka - mereka dapat disimpan dalam array, diedarkan sebagai parameter fungsi, ditugaskan ke variabel, dan apa pun yang Anda miliki.

Itu adalah JavaScript 101, tetapi perlu disebutkan karena pencarian kode cepat di github akan mengungkapkan penghindaran kolektif, atau mungkin ketidaktahuan yang meluas tentang konsep ini.

Haruskah kita pergi untuk contoh palsu? Kita harus.

```js
const hi = (name) => `Hi ${name}`;
const greeting = (name) => hi(name);
```

Di sini, pembungkus fungsi `hi` di sekitar `greeting` benar-benar berlebihan.

Mengapa? Karena fungsi dapat dipanggil dalam JavaScript.

Ketika `hi` memiliki `()` diakhir, itu akan menjalankan fungsi dan mengembalikan nilai.

Ketika tidak, itu hanya mengembalikan fungsi yang disimpan dalam variabel.

Untuk memastikannya, lihat sendiri:

```js
hi; // name => `Hi ${name}`
hi("jonas"); // "Hi jonas"
```

Karena `greeting` pada gilirannya hanya memanggil `hi` dengan argumen yang sama, kita cukup menulis:

```js
const greeting = hi;
greeting("times"); // "Hi times"
```

Dengan kata lain, `hi` sudah merupakan fungsi yang mengharapkan satu argumen, mengapa menempatkan fungsi lain di sekitarnya yang hanya memanggil `hi` dengan argumen berdarah yang sama? Itu tidak masuk akal.

Ini seperti mengenakan jaket terberat Anda di tengah bulan Juli hanya untuk meledakkan udara dan meminta es lilin.

Ini sangat bertele-tele dan, seperti yang terjadi, praktik buruk untuk mengelilingi suatu fungsi dengan fungsi lain hanya untuk menunda evaluasi (kita akan melihat mengapa sebentar lagi, tetapi itu ada hubungannya dengan pemeliharaan)

Pemahaman yang kuat tentang ini sangat penting sebelum melanjutkan, jadi mari kita periksa beberapa contoh menyenangkan lainnya yang digali dari perpustakaan paket npm.

```js
// ignorant
const getServerStuff = (callback) => ajaxCall((json) => callback(json));

// enlightened
const getServerStuff = ajaxCall;
```

Dunia dipenuhi dengan kode ajax persis seperti ini. Inilah alasan keduanya setara:

```js
// this line
ajaxCall((json) => callback(json));

// is the same as this line
ajaxCall(callback);

// so refactor getServerStuff
const getServerStuff = (callback) => ajaxCall(callback);

// ...which is equivalent to this
const getServerStuff = ajaxCall; // <-- look mum, no ()'s
```

Dan itu, teman-teman, adalah bagaimana hal itu dilakukan. Sekali lagi agar kita mengerti mengapa aku begitu gigih.

```js
const BlogController = {
  index(posts) {
    return Views.index(posts);
  },
  show(post) {
    return Views.show(post);
  },
  create(attrs) {
    return Db.create(attrs);
  },
  update(post, attrs) {
    return Db.update(post, attrs);
  },
  destroy(post) {
    return Db.destroy(post);
  },
};
```

Kontroler konyol ini adalah 99% bulu. Kita bisa menulis ulang sebagai:

```js
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

...atau hapus semuanya karena tidak lebih dari sekadar menggabungkan Tampilan dan Db kita bersama-sama.

## Mengapa Mendukung Kelas Satu?

Oke, mari kita turun ke alasan untuk mendukung fungsi kelas satu. Seperti yang kita lihat di `getServerStuff` dan `BlogController`, mudah untuk menambahkan lapisan tipuan yang tidak memberikan nilai tambah dan hanya meningkatkan jumlah kode yang berlebihan untuk dipelihara dan dicari.

Selain itu, jika fungsi pembungkus yang tidak perlu seperti itu harus diubah, kita juga harus mengubah fungsi pembungkus kita.

```js
httpGet("/post/2", (json) => renderPost(json));
```

Jika `httpGet` diuba agar dapat mengirim kemungkinan `err`, kita harus kembali dan mengganti "lem".

```js
// go back to every httpGet call in the application and explicitly pass err along.
httpGet("/post/2", (json, err) => renderPost(json, err));
```

Seandainya kami menulisnya sebagai fungsi kelas satu, apalagi yang perlu diubah:

```js
// renderPost is called from within httpGet with however many arguments it wants
httpGet("/post/2", renderPost);
```

Selain penghapusan fungsi yang tidak perlu, kita harus memberi nama dan referensi argumen.

Nama adalah sedikit masalah, Anda tahu. Kami memiliki potensi kesalahan penamaan - terutama karena usia dan persyaratan basis kode berubah.

Memiliki banyak nama untuk konsep yang sama adalah sumber kebingungan yang umum dalam proyek.

Ada juga masalah kode generik. Misalnya, kedua fungsi ini melakukan hal yang persis sama, tetapi yang satu terasa jauh lebih umum dan dapat digunakan kembali:

```js
// specific to our current blog
const validArticles = articles =>
  articles.filter(article => article !== null && article !== undefined),

// vastly more relevant for future projects
const compact = xs => xs.filter(x => x !== null && x !== undefined);
```

Dengan menggunakan penamaan tertentu, kita seolah-olah telah mengikat diri kita sendiri pada data tertentu (dalam hal ini `articles`).

Ini terjadi sedikit dan merupakan sumber dari banyak penemuan kembali.

Saya harus menyebutkan bahwa, seperti halnya kode Berorientasi Objek, Anda harus sadar `this` akan menggigit Anda di jugularis. Jika fungsi yang mendasari menggunakan `this` dan kami menyebutnya kelas pertama, kami tunduk pada murka abstraksi bocor ini.

```js
const fs = require("fs");

// scary
fs.readFile("freaky_friday.txt", Db.save);

// less so
fs.readFile("freaky_friday.txt", Db.save.bind(Db));
```

Karena terikat pada dirinya sendiri, `Db` bebas untuk mengakses kode sampah prototipikalnya. Saya menghindari penggunaan `this` seperti popok kotor.

Benar-benar tidak perlu saat menulis kode fungsional. Namun, saat berinteraksi dengan perpustakaan lain, Anda mungkin harus menyetujui dunia gila di sekitar kita.

Beberapa akan berpendapat bahwa `this` diperlukan untuk mengoptimalkan kecepatan. Jika Anda adalah jenis optimasi mikro, silakan tutup buku ini.

Jika Anda tidak bisa mendapatkan uang Anda kembali, mungkin Anda bisa menukarnya dengan sesuatu yang lebih rumit.

Dan dengan itu, kami siap untuk melanjutkan.

[Bab 03: Kebahagiaan Murni dengan Fungsi Murni](ch03-id.md)
