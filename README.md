# Inventaris Buku Mobile App

- **Nama:** Dimas Kendika Fazrulfalah
- **NIM:** H1D023083  
- **Shift Baru:** C
- **Shift Asal:** A

## Video Demo Aplikasi
![Demo](Investaris Buku - H1D023083.mp4)

## Deskripsi Aplikasi
Aplikasi Inventaris Buku adalah aplikasi mobile berbasis Flutter yang terhubung dengan REST API CodeIgniter 4 untuk mengelola inventaris buku. Aplikasi ini memungkinkan pengguna untuk melakukan operasi CRUD (Create, Read, Update, Delete) pada data buku dengan fitur autentikasi.

## Spesifikasi API

### Base URL
```
http://192.168.100.17:8080
```

### A. Autentikasi

#### 1. Registrasi
**Endpoint:** `/registrasi`  
**Method:** `POST`  
**Header:** 
- Content-Type: application/x-www-form-urlencoded

**Body:**
```json
{
    "username": "string",
    "email": "string, unique", 
    "password": "string"
}
```

**Response:**
```json
{
    "code": "integer",
    "status": "boolean", 
    "data": "string"
}
```

#### 2. Login
**Endpoint:** `/login`  
**Method:** `POST`  
**Header:**
- Content-Type: application/x-www-form-urlencoded

**Body:**
```json
{
    "username": "string (bisa email atau username)",
    "password": "string"
}
```

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": {
        "token": "string",
        "user": {
            "id": "integer",
            "username": "string", 
            "email": "string"
        }
    }
}
```

### B. Inventaris Buku

#### 1. List Buku
**Endpoint:** `/buku`  
**Method:** `GET`  
**Header:**
- Authorization: Bearer {token}

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": [
        {
            "id": "integer",
            "judul": "string",
            "harga": "integer", 
            "jumlah": "integer",
            "tanggal_masuk": "date",
            "volume": "integer",
            "penulis": "string",
            "penerbit": "string"
        }
    ]
}
```

#### 2. Create Buku
**Endpoint:** `/buku`  
**Method:** `POST`  
**Header:**
- Authorization: Bearer {token}
- Content-Type: application/x-www-form-urlencoded

**Body:**
```json
{
    "judul": "string",
    "harga": "integer",
    "jumlah": "integer", 
    "tanggal_masuk": "date (YYYY-MM-DD)",
    "volume": "integer",
    "penulis": "string",
    "penerbit": "string"
}
```

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": {
        "id": "integer",
        "judul": "string",
        "harga": "integer",
        "jumlah": "integer",
        "tanggal_masuk": "date",
        "volume": "integer", 
        "penulis": "string",
        "penerbit": "string"
    }
}
```

#### 3. Update Buku
**Endpoint:** `/buku/{id}`  
**Method:** `POST` (dengan method spoofing)  
**Header:**
- Authorization: Bearer {token}
- Content-Type: application/x-www-form-urlencoded

**Body:**
```json
{
    "_method": "PUT",
    "judul": "string",
    "harga": "integer",
    "jumlah": "integer",
    "tanggal_masuk": "date (YYYY-MM-DD)", 
    "volume": "integer",
    "penulis": "string",
    "penerbit": "string"
}
```

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": {
        "id": "integer",
        "judul": "string", 
        "harga": "integer",
        "jumlah": "integer",
        "tanggal_masuk": "date",
        "volume": "integer",
        "penulis": "string",
        "penerbit": "string"
    }
}
```

#### 4. Show Buku
**Endpoint:** `/buku/{id}`  
**Method:** `GET`  
**Header:**
- Authorization: Bearer {token}

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": {
        "id": "integer",
        "judul": "string",
        "harga": "integer", 
        "jumlah": "integer",
        "tanggal_masuk": "date",
        "volume": "integer",
        "penulis": "string",
        "penerbit": "string"
    }
}
```

#### 5. Delete Buku
**Endpoint:** `/buku/{id}`  
**Method:** `DELETE`  
**Header:**
- Authorization: Bearer {token}

**Response:**
```json
{
    "code": "integer",
    "status": "boolean", 
    "data": "boolean"
}
```

#### 6. Statistik Buku
**Endpoint:** `/buku/statistik`  
**Method:** `GET`  
**Header:**
- Authorization: Bearer {token}

**Response:**
```json
{
    "code": "integer",
    "status": "boolean",
    "data": {
        "total_judul": "integer",
        "total_jumlah": "integer", 
        "total_nilai": "integer"
    }
}
```

## Penjelasan Kode

### 1. Model Classes

#### Book Model (`lib/model/book.dart`)
```dart
class Book {
  String? id;
  String judul;
  int harga;
  int jumlah;
  String tanggalMasuk;
  int volume;
  String penulis;
  String penerbit;
}
```
**Fungsi:** Model data untuk representasi buku dengan mapping field database ke Dart objects. Menggunakan `fromJson()` untuk parsing response API dan `toJson()` untuk serialisasi data ke API.

#### Login Model (`lib/model/login.dart`)
```dart
class Login {
  final int? code;
  final bool status;
  final String? message;
  final LoginData? data;
}
```
**Fungsi:** Model untuk menangani response login dari API dengan struktur nested data yang berisi token dan informasi user.

#### Registrasi Model (`lib/model/registrasi.dart`)
```dart
class Registrasi {
  final int? code;
  final bool status;
  final String? message;
}
```
**Fungsi:** Model untuk menangani response registrasi dengan status dan pesan dari server.

### 2. API Helper (`lib/helpers/api.dart`)

#### API Class
```dart
class Api {
  Future<dynamic> post(dynamic url, dynamic data) async {
    // Convert data to form-encoded format
    Map<String, String> formData = {};
    if (data is Map<String, dynamic>) {
      data.forEach((key, value) {
        formData[key] = value?.toString() ?? '';
      });
    }
    
    final response = await http.post(Uri.parse(url),
        body: formData,
        headers: {HttpHeaders.authorizationHeader: "Bearer $token"});
  }
}
```
**Fungsi:** Mengelola komunikasi HTTP dengan API. Mengkonversi data ke format form-encoded yang kompatibel dengan CodeIgniter 4, menambahkan Bearer token untuk autentikasi, dan menangani error responses.

### 3. Business Logic Layer (BLoC)

#### BookBloc (`lib/bloc/book_bloc.dart`)
```dart
class BookBloc {
  static Future<List<Book>> getBooks() async {
    final response = await Api().get(ApiUrl.listBook);
    var jsonObj = json.decode(response.body);
    
    if (jsonObj['status'] == true && jsonObj['code'] == 200) {
      List<dynamic> data = jsonObj['data'] ?? [];
      return data.map((json) => Book.fromJson(json)).toList();
    }
    return [];
  }
  
  static Future<Map<String, dynamic>> updateBook(Book book) async {
    // Update menggunakan POST dengan method spoofing
    var updateData = book.toJson();
    updateData['_method'] = 'PUT';
    final response = await Api().post(ApiUrl.updateBook(book.id!), updateData);
  }
}
```
**Fungsi:** Layer bisnis logic yang mengelola operasi CRUD buku. Menangani konversi data, error handling, dan komunikasi dengan API. Menggunakan method spoofing untuk update (POST dengan `_method=PUT`) karena CodeIgniter 4 memiliki limitasi dalam parsing form data pada HTTP PUT.

#### LoginBloc (`lib/bloc/login_bloc.dart`)
**Fungsi:** Mengelola proses autentikasi login, menyimpan token ke local storage, dan menangani response login dari server.

#### RegistrasiBloc (`lib/bloc/registrasi_bloc.dart`)
**Fungsi:** Mengelola proses registrasi user baru dengan validasi data dan error handling.

### 4. User Interface

#### LoginPage (`lib/ui/login_page.dart`)
**Fungsi:** Halaman login dengan form validasi, menangani proses login, menyimpan token, dan navigasi ke halaman utama setelah login berhasil.

#### RegistrasiPage (`lib/ui/registrasi_page.dart`)
**Fungsi:** Halaman registrasi dengan form validasi (username, email, password, konfirmasi password), menangani proses registrasi dan feedback ke user.

#### BookPage (`lib/ui/book_page.dart`)
**Fungsi:** Halaman utama yang menampilkan daftar buku dalam bentuk list dengan fitur pull-to-refresh, search, dan navigasi ke detail/form buku.

#### BookForm (`lib/ui/book_form.dart`)
**Fungsi:** Form untuk menambah/edit buku dengan validasi input, date picker untuk tanggal masuk, dan handling untuk mode create/update.

#### BookDetail (`lib/ui/book_detail.dart`)
**Fungsi:** Halaman detail buku yang menampilkan informasi lengkap buku dengan opsi edit dan delete. Menggunakan AlertDialog untuk konfirmasi delete.

### 5. Utilities

#### UserInfo (`lib/helpers/user_info.dart`)
**Fungsi:** Mengelola penyimpanan data user (token, user ID) menggunakan SharedPreferences untuk persistent storage antar session aplikasi.

#### ApiUrl (`lib/helpers/api_url.dart`)
**Fungsi:** Centralized configuration untuk URL endpoint API, memudahkan maintenance dan perubahan base URL.

## Fitur Aplikasi

1. **Autentikasi User**
   - Login dengan username/email dan password
   - Registrasi user baru dengan validasi
   - Token-based authentication dengan Bearer token
   - Auto-logout dan session management

2. **Manajemen Inventaris Buku**
   - Tampil list buku dengan informasi ringkas
   - Tambah buku baru dengan form lengkap
   - Edit data buku dengan pre-filled form
   - Hapus buku dengan konfirmasi
   - Detail buku dengan informasi lengkap

3. **User Experience**
   - Loading indicators untuk operasi async
   - Error handling dengan user-friendly messages
   - Pull-to-refresh untuk update data
   - Responsive UI dengan Material Design
   - Form validasi real-time

## Teknologi yang Digunakan

### Frontend (Flutter)
- **Framework:** Flutter 3.x
- **State Management:** StatefulWidget dengan setState
- **HTTP Client:** http package
- **Local Storage:** shared_preferences
- **UI Components:** Material Design widgets

### Backend (CodeIgniter 4)
- **Framework:** CodeIgniter 4
- **Database:** MySQL
- **Authentication:** Custom token-based auth
- **API Architecture:** RESTful API
- **Response Format:** JSON

## Cara Menjalankan Aplikasi

### Prerequisites
1. Flutter SDK (versi 3.x atau lebih baru)
2. Android Studio atau VS Code
3. Web browser (Chrome) atau device Android/iOS
4. XAMPP atau web server dengan PHP 8.x
5. MySQL database

### Setup Backend (CodeIgniter 4)
1. Clone/download project CodeIgniter 4
2. Import database dari file migration
3. Konfigurasi database di `app/Config/Database.php`
4. Jalankan `php spark serve` atau setup di XAMPP
5. Gunakan IP Address pribadi, misal saya di : `http://192.168.100.17:8080`

### Setup Frontend (Flutter)
1. Clone repository ini
2. Buka terminal di direktori project
3. Jalankan `flutter pub get` untuk install dependencies
4. Untuk web development: `flutter run -d chrome --web-browser-flag="--disable-web-security" --web-browser-flag="--user-data-dir=D:/temp"`
5. Untuk mobile: `flutter run` (pastikan device/emulator terhubung)

## Struktur Project
```
lib/
├── bloc/              # Business Logic Components
│   ├── book_bloc.dart
│   ├── login_bloc.dart
│   └── registrasi_bloc.dart
├── helpers/           # Utility classes
│   ├── api.dart
│   ├── api_url.dart
│   ├── user_info.dart
│   └── app_exception.dart
├── model/             # Data models
│   ├── book.dart
│   ├── login.dart
│   └── registrasi.dart
├── ui/                # User Interface
│   ├── book_page.dart
│   ├── book_form.dart
│   ├── book_detail.dart
│   ├── login_page.dart
│   └── registrasi_page.dart
└── main.dart          # Entry point
```
