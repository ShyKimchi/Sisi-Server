# Simple LMS — Backend Development & Optimization

Simple LMS adalah project backend berbasis Django dengan PostgreSQL sebagai database dan Docker Compose untuk menjalankan aplikasi serta database dalam container.

## Teknologi

- Python 3.12
- Django
- PostgreSQL 17
- Docker dan Docker Compose

## Struktur Project

```text
minggu-4/
├── catalog/
├── config/
├── users/
├── Dockerfile
├── compose.yaml
├── requirements.txt
├── manage.py
├── .env
└── .gitignore
```

- `config/`: konfigurasi dan routing utama Django.
- `catalog/`: model serta fitur Course, Lesson, Enrollment, Assignment, dan Submission.
- `users/`: model pengguna dan role.
- `Dockerfile`: instruksi pembuatan image aplikasi Django.
- `compose.yaml`: konfigurasi container aplikasi dan PostgreSQL.
- `requirements.txt`: daftar dependency Python.
- `.env`: konfigurasi environment dan database lokal; tidak boleh diunggah ke repository.

## Prasyarat

Pastikan Git dan Docker Desktop sudah terpasang dan berjalan.

## Konfigurasi Environment

Buat file `.env` di root project berdasarkan variabel yang dibutuhkan oleh konfigurasi Compose dan Django:

```dotenv
DB_NAME=your_database_name
DB_USER=your_database_user
DB_PASSWORD=your_database_password
```

Sesuaikan nilainya dengan konfigurasi database lokal. Jangan menggunakan kredensial contoh di atas sebagai password produksi.

## Menjalankan Project

Clone repository:

```bash
git clone https://github.com/ShyKimchi/Sisi-Server.git
cd Sisi-Server
```

Jika project berada dalam subfolder, masuk ke direktori yang berisi `compose.yaml`.

Jalankan container:

```bash
docker compose up -d --build
```

Terapkan migration:

```bash
docker compose exec app python manage.py migrate
```

Periksa status container:

```bash
docker compose ps
```

Aplikasi dapat diakses melalui:

http://localhost:8000/admin

## Django Shell

Buka shell Django:

```bash
docker compose exec app python manage.py shell
```

Contoh penggunaan Django ORM:

```python
from catalog.models import Course

Course.objects.all()
Course.objects.filter(code__startswith="PSS")
Course.objects.get(code="PSS01")
```

## Eksperimen Optimasi Query

Pengujian membandingkan pengambilan 100 Course sebelum dan sesudah optimasi Django ORM.

| Eksperimen | Sebelum optimasi | Sesudah optimasi |
|---|---:|---:|
| Course + Lecturer | 101 query | 1 query |
| Course + Lesson | 101 query | 2 query |

### `select_related()`

`select_related("lecturer")` mengambil Course dan Lecturer melalui SQL JOIN. Metode ini sesuai untuk relasi ForeignKey dan mengurangi query berulang ketika mengakses Lecturer dari setiap Course.

### `prefetch_related()`

`prefetch_related("lessons")` mengambil Course dan Lesson menggunakan query terpisah, lalu memasangkan data terkait di sisi Python. Metode ini sesuai untuk relasi satu-ke-banyak.

### `QuerySet.explain()`

`QuerySet.explain()` digunakan untuk memeriksa rencana eksekusi query database. Pada pengujian Course dengan kondisi `is_active=False` dan `title="Performance Course 5000"`, PostgreSQL menggunakan `Index Scan` melalui index pada kolom `is_active` dan `title`.

Jumlah query dan rencana eksekusi dapat berbeda apabila dataset atau konfigurasi database berubah.

## Menutup Project

Untuk menghentikan container:

```bash
docker compose down
```

Data PostgreSQL disimpan pada named volume `postgres_data`, sehingga menghentikan container tidak otomatis menghapus data database.

## Repository

GitHub: https://github.com/ShyKimchi/Sisi-Server
