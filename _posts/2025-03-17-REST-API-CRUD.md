---
title: Tutorial REST API CRUD Sederhana dengan Flask & SQLAlchemy
image:
    path: "/assets/img/rest.png"
categories: [Blogs]

---
Dalam pengembangan aplikasi, komunikasi antar sistem sering dilakukan melalui REST API. Salah satu cara mudah untuk membuat REST API adalah dengan menggunakan Flask, framework Python yang ringan, serta SQLAlchemy sebagai Object Relational Mapper (ORM) untuk mengelola database.

Pada tutorial ini, kita akan membuat REST API CRUD sederhana menggunakan Flask dan SQLAlchemy, di mana API ini memungkinkan kita untuk melakukan operasi Create, Read, Update, dan Delete pada data.

## **Pengenalan API, REST API, dan CRUD**
### Apa itu API?
API (Application Programming Interface) adalah perantara yang memungkinkan aplikasi berkomunikasi satu sama lain. API menerima permintaan (request), memprosesnya, dan mengembalikan respons (response). Salah satu contoh penggunaan API adalah aplikasi cuaca yang mengambil data suhu dan kondisi cuaca terkini dari API penyedia cuaca.

### Apa Itu REST API?
REST (Representational State Transfer) adalah arsitektur API berbasis HTTP yang banyak digunakan. REST API menggunakan metode standar seperti:
- GET → Mengambil data
- POST → Menambah data
- PUT/PATCH → Memperbarui data
- DELETE → Menghapus data

### Apa Itu CRUD?
CRUD adalah singkatan dari Create, Read, Update, dan Delete, yang merupakan empat operasi dasar dalam pengelolaan data. Operasi ini akan kita implementasikan dalam API menggunakan Flask.

## **Langkah-Langkah**
Sebelum melakukan langkah-langkah di bawah ini, pastikan kamu sudah menginstall Python. Jika belum, kamu bisa mengunduhnya dari [python.org](https://www.python.org). Langkah-langkah instalasi Flask yang ada di tutorial ini hanya berlaku untuk sistem operasi Windows. Untuk instruksi instalasi pada sistem operasi lainnya, bisa dicek di [situs resmi Flask](https://flask.palletsprojects.com/en/stable/installation/). 

### 1) Buat Folder Proyek dan Environment
```
> mkdir myproject
> cd myproject
> py -3 -m venv .venv
```
Virtual environment berfungsi untuk mengisolasi dependensi proyek Python agar tidak saling mengganggu dengan proyek lain atau sistem Python secara global.

### 2) Mengaktifkan Virtual Environment
```
.venv\Scripts\activate
```
Jika virtual environment berhasil diaktifkan, maka nama environment akan muncul di awal prompt terminal, misalnya `(.venv)`.

### 3) Instalasi Dependensi
```
pip install flask flask_sqlalchemy 
```
Kedua pustaka ini diperlukan untuk membangun REST API dengan Flask dan mengelola data di database.

### 4) Konfigurasi Aplikasi Flask
Buat file `app.py` dan konfigurasikan aplikasi Flask serta koneksi ke database
```py
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

# Inisialisasi aplikasi Flask
app = Flask(__name__)

# Konfigurasi database SQLite
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Inisialisasi SQLAlchemy
db = SQLAlchemy(app)

# Import model (akan dibuat pada langkah berikutnya)
from models import Book

# Route untuk halaman utama
@app.route('/')
def home():
    return "Selamat datang di REST API Flask!"
```

### 5) Membuat Model Database
Buat file `models.py` untuk mendefinisikan model database. Misalnya, kita akan
membuat model untuk entitas `Book`.
```py
from app import db

# Model untuk tabel "Book" di database dengan atribut id, title, author, dan published_year  
class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    author = db.Column(db.String(100), nullable=False)
    published_year = db.Column(db.Integer, nullable=False)

    # Method untuk mengubah objek Book menjadi dictionary
    # Ini berguna untuk mengembalikan data dalam format JSON saat digunakan di API
    def to_dict(self):
        return {
        'id': self.id,
        'title': self.title,
        'author': self.author,
        'published_year': self.published_year
        }
```

### 6) Membuat Tabel Database
Buat file `init_db.py` lalu jalankan untuk membuat tabel database. 
```py
from app import app, db
with app.app_context():
    db.create_all()
    print("Database tables created successfully!")
```
Jika berhasil, akan terbentuk file `database.db` di direktori proyek.

### 7) Membuat Endpoint REST API
Tambahkan endpoint untuk operasi CRUD (Create, Read, Update, Delete) pada
file `app.py`. Endpoint adalah URL atau rute yang digunakan untuk mengakses fungsi atau operasi tertentu dalam sebuah API. Setiap endpoint mewakili tindakan tertentu, seperti mengambil data (GET), menambah data (POST), memperbarui data (PUT/PATCH), atau menghapus data (DELETE).
##### 1. GET - Mendapatkan semua buku
```py
# Mendapatkan semua buku
@app.route('/books', methods=['GET'])
def get_books():
    books = Book.query.all()
    return jsonify([book.to_dict() for book in books])
```
##### 2. GET -  Mendapatkan satu buku berdasarkan ID
```py
# Mendapatkan satu buku berdasarkan ID
@app.route('/books/<int:id>', methods=['GET'])
def get_book(id):
    book = Book.query.get_or_404(id)
    return jsonify(book.to_dict())
```
##### 3. POST - Menambahkan buku baru
```py
# Menambahkan buku baru
@app.route('/books', methods=['POST'])
def add_book():
    data = request.get_json()
    new_book = Book(
    title=data['title'],
    author=data['author'],
    published_year=data['published_year']
    )
    db.session.add(new_book)
    db.session.commit()
    return jsonify(new_book.to_dict()), 201
```
##### 4. PUT - Memperbarui buku berdasarkan ID
```py
# Memperbarui buku berdasarkan ID
@app.route('/books/<int:id>', methods=['PUT'])
def update_book(id):
    book = Book.query.get_or_404(id)
    data = request.get_json()
    book.title = data.get('title', book.title)
    book.author = data.get('author', book.author)
    book.published_year = data.get('published_year', book.published_year)
    db.session.commit()
    return jsonify(book.to_dict())
```
##### 5. DELETE - Menghapus buku berdasarkan ID
```py
# Menghapus buku berdasarkan ID
@app.route('/books/<int:id>', methods=['DELETE'])
def delete_book(id):
    book = Book.query.get_or_404(id)
    db.session.delete(book)
    db.session.commit()
    return jsonify({'message': 'Buku berhasil dihapus'}), 200
```

### 8) Menjalankan Aplikasi
Jalankan aplikasi Flask dengan perintah `flask --app app run`. Aplikasi akan berjalan di [http://127.0.0.1:5000/](http://127.0.0.1:5000/)

## Contoh Penggunaan REST API
Selanjutnya kita akan melakukan pengujian REST API menggunakan Postman. Postman adalah aplikasi yang digunakan untuk menguji dan mengembangkan API. Dengan Postman, kamu dapat mengirimkan permintaan (request) ke API, mengatur parameter, dan melihat respons yang diterima, sehingga memudahkan pengujian API. Kamu bisa mengunduh Postman [di sini](https://www.postman.com/downloads/).

### 1. Menambah Data (Create)
- Pilih metode POST pada dropdown di sebelah kiri URL input bar.
- Masukkan URL endpoint http://localhost:5000/books.
- Pilih tab Body yang berada di bawah URL input bar dan pilih format raw Pastikan format yang dipilih adalah JSON. 
- Masukkan data yang diperlukan di kolom body. Misalnya:
```json
{
     "title": "Python Programming",
     "author": "John Doe",
     "published_year": 2023
}
```
- Terakhir, kirimkan data JSON dengan mengklik tombol `Send`
- Jika berhasil, kamu akan menerima respons berupa data buku yang sudah ditambahkan dalam format JSON, misalnya:
![Create](/assets/img/rest-api/create.png)

### 2. Membaca Buku (Read)
- Pilih metode GET pada dropdown di sebelah kiri URL input bar.
- Masukkan URL endpoint `http://localhost:5000/books` untuk mendapatkan semua data buku atau `http://localhost:5000/books/<id>` untuk mendapatkan satu buku berdasarkan ID.
- Klik tombol `Send` untuk mengirim permintaan.
- Jika berhasil, kamu akan menerima respons berupa data buku dalam format JSON. Misalnya:
![Read](/assets/img/rest-api/read.png)

### 3. Mengedit Buku (Update)
- Pilih metode PUT pada dropdown di sebelah kiri URL input bar.
- Masukkan URL endpoint `http://localhost:5000/books/<id>` (misalnya `http://localhost:5000/books/1`) untuk memperbarui buku dengan ID tertentu.
- Pilih tab Body dan pastikan format yang dipilih adalah raw dan JSON.
- Masukkan data yang akan diperbarui di kolom body. Misalnya, untuk mengubah judul buku menjadi "Advanced Python":
```json
  {
    "title": "Advanced Python"
  }
```
- Klik tombol `Send` untuk mengirim permintaan.
- Jika berhasil, kamu akan menerima respons berupa data buku yang sudah diperbarui dalam format JSON, misalnya:
![Update](/assets/img/rest-api/update.png)

### 4. Menghapus Buku (Delete)
- Pilih metode DELETE pada dropdown di sebelah kiri URL input bar.
- Masukkan URL endpoint `http://localhost:5000/books/<id>` (misalnya `http://localhost:5000/books/1`) untuk menghapus buku dengan ID tertentu.
- Klik tombol `Send` untuk mengirim permintaan.
- Jika berhasil, kamu akan menerima respons yang mengonfirmasi bahwa buku telah dihapus, misalnya:
![Delete](/assets/img/rest-api/delete.png)

