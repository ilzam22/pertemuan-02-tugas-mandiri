# Tugas Mandiri 5 — Membandingkan SQL Mentah dan ORM

## 1. Judul dan Tujuan

Tugas ini bertujuan untuk membandingkan dua cara mengakses database dari Node.js, yaitu SQL mentah dan ORM (Prisma), dengan satu operasi yang sama. Berbeda dari empat tugas sebelumnya, tugas ini berfokus pada database, bukan pengujian HTTP.

## 2. Cara Menjalankan

1. Pilih satu operasi sederhana, yaitu mengambil satu data berdasarkan ID.
2. Pilih satu tabel sebagai contoh, yaitu `jadwal`, dengan kolom `id`, `mataKuliah`, dan `status` (sama seperti data mock di praktikum Express).
3. Tulis operasi itu sebagai query SQL mentah, lengkap dengan contoh pemakaiannya di Node.js.
4. Tulis operasi yang sama memakai Prisma.
5. Bandingkan keduanya, lalu jawab pertanyaan pada Bagian 3.5.

Tugas ini membandingkan cara penulisannya saja. Praktik koneksi database dan schema Prisma yang sebenarnya dikerjakan di Pertemuan 5.

## 3. Hasil Pengujian

### 3.1. Operasi yang dipilih

Operasi yang saya pilih adalah **mengambil satu data jadwal berdasarkan ID**, dengan tabel `jadwal` dan ID yang dicari adalah `1`.

### 3.2. SQL mentah

Query SQL-nya:

```sql
SELECT *
FROM jadwal
WHERE id = ?;
```

Tanda `?` adalah tempat untuk nilai ID yang diisi belakangan. Jika dipakai di Node.js dengan library `mysql2`:

```js
const [rows] = await connection.execute(
  'SELECT * FROM jadwal WHERE id = ?',
  [1]
);

console.log(rows[0]);
```

Proyek praktikum ini memakai PostgreSQL, jadi jika memakai library `pg`, penulisannya sedikit berbeda karena tempat nilainya ditulis `$1`:

```js
const result = await client.query(
  'SELECT * FROM jadwal WHERE id = $1',
  [1]
);

console.log(result.rows[0]);
```

### 3.3. Prisma ORM

Prisma membaca struktur tabel dari satu file schema. Untuk tabel `jadwal`, modelnya kira-kira seperti ini:

```prisma
model Jadwal {
  id         Int    @id @default(autoincrement())
  mataKuliah String
  status     String
}
```

Operasi yang sama kemudian ditulis sebagai pemanggilan method:

```js
const jadwal = await prisma.jadwal.findUnique({
  where: {
    id: 1,
  },
});

console.log(jadwal);
```

Kode ini dibaca sebagai "cari satu data jadwal yang `id`-nya 1". Hasilnya berupa objek, atau `null` kalau datanya tidak ada.

### 3.4. Perbandingan

| Aspek | SQL mentah | Prisma ORM |
|---|---|---|
| Cara menulis | Menulis query SQL langsung | Memanggil method pada objek |
| Kontrol atas query | Penuh, bisa diatur sampai detail | Lebih terbatas karena sudah diabstraksikan |
| Kemudahan | Perlu paham SQL | Lebih mudah, cukup paham method dan schema |
| Pencegahan SQL injection | Harus ingat memakai parameter | Query disusun otomatis dengan parameter |
| Cocok untuk | Query khusus atau kompleks | CRUD dan pengembangan aplikasi sehari-hari |

### 3.5. Jawaban Pertanyaan

**1. Apa perbedaan SQL mentah dan ORM?**

Pada SQL mentah, programmer menulis sendiri perintah SQL dan mengirimkannya ke database. Pada ORM, programmer bekerja dengan objek dan method di bahasa pemrogramannya, lalu ORM yang menyusun perintah SQL-nya di belakang layar.

**2. Apa kelebihan SQL mentah?**

Kontrolnya penuh. Query bisa disusun persis sesuai kebutuhan, termasuk query yang rumit seperti join banyak tabel atau yang perlu dioptimasi khusus. Hasilnya juga mudah diperkirakan karena yang dijalankan di database adalah query yang kita tulis sendiri.

**3. Apa kelebihan ORM?**

Kodenya lebih ringkas dan lebih mudah dibaca karena tidak perlu menulis SQL berulang-ulang untuk operasi dasar. Struktur tabel tercatat di satu schema, sehingga lebih mudah dirawat dan dikerjakan bersama dalam tim. ORM juga membantu menghindari salah ketik pada nama tabel atau kolom.

**4. Apa risiko SQL injection?**

SQL injection terjadi saat input dari pengguna disambung langsung ke teks query, sehingga input itu bisa mengubah isi query. Contoh yang tidak aman:

```js
const id = userInput;
const sql = `SELECT * FROM jadwal WHERE id = ${id}`;
```

Kalau pengguna mengisi `userInput` dengan `1 OR 1=1`, query-nya berubah menjadi `SELECT * FROM jadwal WHERE id = 1 OR 1=1`. Kondisi `1=1` selalu benar, jadi seluruh isi tabel ikut terambil, padahal seharusnya hanya satu baris. Risikonya adalah data bocor, data diubah, atau bahkan dihapus oleh orang yang tidak berhak.

**5. Mengapa penggunaan parameter query dapat mengurangi risiko SQL injection?**

Dengan parameter, struktur query dan nilai dikirim secara terpisah. Query ditulis dengan tempat kosong (`?` atau `$1`), lalu nilainya dikirim sebagai data. Database memperlakukan nilai itu murni sebagai data, bukan sebagai bagian dari perintah SQL, sehingga `1 OR 1=1` hanya dibaca sebagai satu nilai aneh dan tidak mengubah isi query.

```js
const [rows] = await connection.execute(
  'SELECT * FROM jadwal WHERE id = ?',
  [id]
);
```

**6. Bagaimana ORM membantu programmer dalam mengakses database?**

ORM menghubungkan tabel di database dengan objek di kode, sehingga operasi seperti mencari, menambah, mengubah, dan menghapus data bisa ditulis lewat method seperti `findUnique`, `create`, `update`, dan `delete`. Programmer tidak perlu menyusun SQL untuk setiap operasi, dan Prisma menyusun query dengan parameter secara otomatis. Selain itu, schema Prisma menjadi acuan tunggal antara kode dan database.

## 4. Bukti Visual

Tugas ini membandingkan kode, sehingga tidak ada screenshot hasil pengujian. Bukti pengerjaannya berupa potongan kode SQL dan Prisma pada Bagian 3.

## 5. Kesimpulan

SQL mentah memberi kontrol penuh atas query dan cocok untuk kebutuhan yang khusus atau rumit, sedangkan Prisma membuat operasi database sehari-hari lebih singkat dan lebih rapi. Apa pun cara yang dipakai, nilai dari pengguna sebaiknya selalu dikirim sebagai parameter dan tidak disambung langsung ke teks query, supaya aman dari SQL injection.

## 6. Lampiran

- Pull Request: _(isi tautan PR setelah dibuat)_
- Reviewer: _(isi nama rekan yang mereview)_
- Issue terkait: Tidak ada
