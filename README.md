### Nama   : Reyza Syaifullah Sully  
###  NIM    : 1203220045
Kelas      : IF 02-01 
-

Pengantar
-
Dokumen ini akan menjelaskan tentang sistem komunikasi klien-server yang menggunakan soket TCP. Terbagi menjadi dua bagian utama, yaitu sisi server dan sisi klien. Tujuannya adalah memfasilitasi komunikasi klien dengan server dan interaksi dengan sistem file server.

Server :
-
Server berperan dalam menerima perintah dari klien, menjalankan perintah tersebut, dan memberikan respons kepada klien. Berikut adalah deskripsi fitur dan panduan penggunaan Server

Fitur :
- Menerima perintah seperti ls, rm, size, download, upload, dan byebye.
- Melakukan operasi seperti menampilkan daftar file, menghapus file, mendapatkan ukuran file, mengunduh file, dan mengunggah file.
- Menangani berbagai jenis kesalahan seperti file tidak ditemukan atau kesalahan saat membaca/menulis file.

Panduan Penggunaan :
1. Jalankan Server dengan mengeksekusi file server.py.
2. Program akan menunggu koneksi dari klien. (Menunggu client mengirimkan perintah conme)
3. Setelah koneksi terbentuk, server akan menerima perintah dari klien.
4. Klien dapat mengirim perintah seperti ls, rm, size, download, upload, dan byebye.
5. Server akan menanggapi perintah tersebut dan menjalankan operasi yang sesuai pada sistem file server.
6. Klien dapat terus berinteraksi dengan server sampai mengirim perintah byebye untuk menutup koneksi.

Klien :
-
Klien bertugas untuk berkomunikasi dengan server dengan mengirimkan perintah dan menerima respons. Berikut adalah deskripsi fitur dan panduan penggunaan klien:

Fitur :
- Mengirim perintah seperti ls, rm, size, download, upload, dan byebye ke server.
- Menerima respons dari server dan menampilkannya kepada pengguna.
- Mengunggah file ke server dan mengunduh file dari server.

Panduan Penggunaan :
1. Jalankan klien dengan mengeksekusi file client.py.
2. Program akan berjalan dan menampilkan pesan bahwa klien sedang aktif.
3. Gunakan perintah connme untuk menghubungkan klien ke server.
4. Masukkan perintah yang diinginkan, seperti ls, rm, size, download, upload, dan byebye.
5. Klien akan mengirim perintah tersebut ke server dan menampilkan respons yang diterima dari server.
6. Pengguna dapat terus berinteraksi dengan server sampai mengirim perintah byebye untuk menutup koneksi.

Kesimpulan
-
Sistem klien-server ini menyediakan antarmuka sederhana untuk berkomunikasi dengan sistem file server menggunakan soket TCP. Pengguna dapat mengirim perintah ke server dan melakukan operasi seperti menampilkan daftar file, menghapus file, mengunggah file, dan mengunduh file. Tentunya, penggunaan sistem ini dapat ditingkatkan dengan peningkatan fitur keamanan dan penanganan kesalahan yang lebih baik.


Penjelasan Code Program
-

### Server.py :

### Bagian 1: Inisialisasi dan Konfigurasi
```python
import socket
import os

SERVER_ADDRESS = ("localhost", 5002)
BUFFER_SIZE = 4096

server_directory = 'server'
if not os.path.exists(server_directory):
    os.makedirs(server_directory)
```
- **Import Modules**: Mengimpor modul `socket` untuk komunikasi jaringan dan `os` untuk operasi sistem file.
- **Konstanta**:
  - `SERVER_ADDRESS`: Alamat dan port tempat server akan berjalan.
  - `BUFFER_SIZE`: Ukuran buffer untuk menerima data.
- **Direktori Server**: Menetapkan direktori tempat file akan disimpan (`server`). Jika direktori tidak ada, itu akan dibuat dengan `os.makedirs`.

### Bagian 2: Fungsi Utilitas
```python
def list_files():
    files = os.listdir(server_directory)
    return "\n".join(files)

def delete_file(filename):
    path = os.path.join(server_directory, filename)
    try:
        os.remove(path)
        return f"File {filename} berhasil dihapus."
    except FileNotFoundError:
        return "File tidak ditemukan."

def get_file_size(filename):
    try:
        path = os.path.join(server_directory, filename)
        filesize = os.path.getsize(path)
        return f"Size: {filesize / 1024:.2f} KB"
    except FileNotFoundError:
        return "File tidak ditemukan."
```
- **list_files()**: Mengembalikan daftar file dalam direktori server sebagai string yang dipisahkan dengan baris baru.
- **delete_file(filename)**: Menghapus file yang diberikan dari direktori server dan mengembalikan pesan konfirmasi atau pesan kesalahan jika file tidak ditemukan.
- **get_file_size(filename)**: Mengembalikan ukuran file dalam kilobyte atau pesan kesalahan jika file tidak ditemukan.

### Bagian 3: Fungsi Penanganan Perintah
```python
def handle_command(conn, command):
    response = None
    if command.lower().startswith("ls"):
        response = list_files()
    elif command.startswith("rm"):
        _, filename = command.split(maxsplit=1)
        response = delete_file(filename)
    elif command.startswith("size"):
        _, filename = command.split(maxsplit=1)
        response = get_file_size(filename)
    elif command.startswith("download"):
        _, filename = command.split(maxsplit=1)
        file_path = os.path.join(server_directory, filename)
        if os.path.exists(file_path):
            try:
                with open(file_path, "rb") as f:
                    file_data = f.read()
                    file_length = len(file_data)
                    conn.sendall(file_length.to_bytes(4, byteorder='big'))
                    conn.sendall(file_data)
            except Exception as e:
                print(f"Gagal menerima file: {str(e)}")
        else:
            print("File tidak ditemukan pada server.")
    elif command.startswith("upload"):
        _, filename = command.split(maxsplit=1)
        try:
            file_size = int.from_bytes(conn.recv(4), byteorder='big')
            file_data = conn.recv(file_size)
            file_path = os.path.join(server_directory, filename)

            if not os.path.exists(server_directory):
                os.makedirs(server_directory)

            with open(file_path, "wb") as f:
                f.write(file_data)
            print(f"File {filename} berhasil diterima dan disimpan.")
        except Exception as e:
            print(f"Gagal menerima file: {str(e)}")
    elif command == "connme":
        response = "[+] Berhasil Terkoneksi."
    else:
        response = "Perintah tidak valid."
    
    if response:
        conn.send(response.encode())
```
- **handle_command(conn, command)**: Memproses perintah yang diterima dari klien. Perintah yang didukung meliputi:
  - `ls`: Memanggil `list_files()` dan mengirim hasilnya ke klien.
  - `rm <filename>`: Memanggil `delete_file()` dengan `filename` yang diberikan dan mengirim hasilnya ke klien.
  - `size <filename>`: Memanggil `get_file_size()` dengan `filename` yang diberikan dan mengirim hasilnya ke klien.
  - `download <filename>`: Mengirim file yang diminta ke klien jika file ada.
  - `upload <filename>`: Menerima file dari klien dan menyimpannya di direktori server.
  - `connme`: Mengirim pesan konfirmasi koneksi.
  - Perintah lain: Mengirim pesan "Perintah tidak valid."

### Bagian 4: Blok `with` untuk Socket
```python
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind(SERVER_ADDRESS)
    s.listen()
    print("[*] Menunggu Koneksi")
    conn, addr = s.accept()
    print("[+] Berhasil Terkoneksi.")
    with conn:
        while True:
            command = conn.recv(BUFFER_SIZE).decode()
            if not command:
                break
            print(f"Menerima perintah: {command}")
            if command.lower() == "byebye":
                conn.send("Koneksi terputus dengan server.".encode())
                break
            if handle_command(conn, command):
                break
    conn.close()
```
- **Inisialisasi Socket**: Membuat socket TCP dan mengikatnya ke `SERVER_ADDRESS`.
- **Menunggu Koneksi**: Server mendengarkan koneksi masuk menggunakan `s.listen()`.
- **Menerima Koneksi**: Menerima koneksi dari klien dengan `s.accept()`.
- **Loop Utama Server**: 
  - Menerima perintah dari klien.
  - Memproses perintah menggunakan `handle_command()`.
  - Jika perintah `byebye` diterima, mengirim pesan konfirmasi dan memutuskan koneksi.

### Kesimpulan
Program ini berfungsi sebagai server file sederhana yang mendukung perintah untuk mengelola file pada server melalui koneksi TCP. Klien dapat mengirim perintah untuk listing file, menghapus file, mendapatkan ukuran file, mengunduh file, dan mengunggah file. Server memproses perintah tersebut dan mengirimkan balasan yang sesuai kepada klien.

---

### Client.py :

### Bagian 1: Inisialisasi dan Konfigurasi
```python
import socket
import os

SERVER_ADDRESS = ("localhost", 5002)
BUFFER_SIZE = 4096

client_directory = 'client'
if not os.path.exists(client_directory):
    os.makedirs(client_directory)
```
- **Import Modules**: Mengimpor modul `socket` untuk komunikasi jaringan dan `os` untuk operasi sistem file.
- **Konstanta**:
  - `SERVER_ADDRESS`: Alamat dan port tempat server akan berjalan.
  - `BUFFER_SIZE`: Ukuran buffer untuk menerima data.
- **Direktori Klien**: Menetapkan direktori tempat file akan disimpan (`client`). Jika direktori tidak ada, itu akan dibuat dengan `os.makedirs`.

### Bagian 2: Fungsi Utilitas
```python
def receive_file(s, filename):
    try:
        file_size = int.from_bytes(s.recv(4), byteorder='big')
        file_data = s.recv(file_size)
        file_path = os.path.join(client_directory, filename)

        if not os.path.exists(client_directory):
            os.makedirs(client_directory)

        with open(file_path, "wb") as f:
            f.write(file_data)
        print(f"File {filename} berhasil diterima.")
    except FileNotFoundError:
        print("Direktori untuk penyimpanan file tidak ada.")
    except Exception as e:
        print(f"Gagal menerima file: {str(e)}")
```
- **receive_file(s, filename)**: Menerima file dari server dan menyimpannya di direktori klien.
  - Menerima ukuran file terlebih dahulu.
  - Membaca data file sesuai dengan ukuran yang diterima.
  - Menyimpan data file ke direktori klien.
  - Menangani pengecualian untuk kesalahan yang mungkin terjadi selama penerimaan file.

```python
def sent_file(conn, filename):
    file_path = os.path.join(client_directory, filename)
    if os.path.exists(file_path):
        try:
            with open(file_path, "rb") as f:
                file_data = f.read()
                file_length = len(file_data)
                conn.sendall(file_length.to_bytes(4, byteorder='big'))
                conn.sendall(file_data)
            print(f"File {filename} berhasil dikirim.")
        except Exception as e:
            print(f"Gagal mengirim file: {str(e)}")
    else:
        print("File tidak ditemukan pada client.")
```
- **sent_file(conn, filename)**: Mengirim file dari klien ke server.
  - Membaca data file dari direktori klien.
  - Mengirim ukuran file terlebih dahulu.
  - Mengirim data file ke server.
  - Menangani pengecualian untuk kesalahan yang mungkin terjadi selama pengiriman file.

### Bagian 3: Loop Utama Klien
```python
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    print("[*] Client sedang berjalan.")
    connected = False  
    while True:
        command = input("Masukkan perintah: ").strip()
        if not command:
            continue
        
        if command.lower() == 'connme':
            s.connect(SERVER_ADDRESS)
            
            s.send(command.encode())  
            response = s.recv(BUFFER_SIZE).decode()
            print(response)
            connected = True  
        elif connected:
            if command.lower() == 'ls':
                s.send(command.encode())  
                response = s.recv(BUFFER_SIZE).decode()
                print(response)  
            elif command.lower().startswith("rm"):
                s.send(command.encode())
                response = s.recv(BUFFER_SIZE).decode()
                print(response)
            elif command.lower().startswith("size"):
                s.send(command.encode())
                response = s.recv(BUFFER_SIZE).decode()
                print(response)
            elif command.lower().startswith("download"):
                s.send(command.encode()) 
                _, filename = command.split(maxsplit=1)
                receive_file(s, filename)
            elif command.lower().startswith("upload"):
                s.send(command.encode())
                _, filename = command.split(maxsplit=1)
                sent_file(s, filename)
            elif command.lower() == 'byebye':
                s.send(command.encode())
                response = s.recv(BUFFER_SIZE).decode()
                print(response)
                s.close()  
                break
            else:
                response = "Invalid command."
                print(response)
        else:
            print("Client belum terhubung ke server.")
```
- **Inisialisasi Socket**: Membuat socket TCP dan menunggu perintah dari pengguna.
- **Loop Utama**: Menunggu perintah dari pengguna dan memprosesnya.
  - **connme**: Menghubungkan ke server dan mengirim perintah `connme`.
  - **ls**: Mengirim perintah `ls` ke server dan menampilkan respons yang diterima.
  - **rm <filename>**: Mengirim perintah `rm` dengan `filename` ke server dan menampilkan respons yang diterima.
  - **size <filename>**: Mengirim perintah `size` dengan `filename` ke server dan menampilkan respons yang diterima.
  - **download <filename>**: Mengirim perintah `download` dengan `filename` ke server dan menerima file yang diminta.
  - **upload <filename>**: Mengirim perintah `upload` dengan `filename` ke server dan mengirim file yang diminta.
  - **byebye**: Mengirim perintah `byebye` ke server dan menutup koneksi.
  - Perintah tidak valid: Menampilkan pesan "Invalid command."
  - **Koneksi ke Server**: Menangani kondisi di mana klien belum terhubung ke server sebelum mengirim perintah.

### Kesimpulan
Program klien ini berfungsi sebagai klien file sederhana yang mendukung perintah untuk mengelola file pada server melalui koneksi TCP. Klien dapat mengirim perintah untuk listing file, menghapus file, mendapatkan ukuran file, mengunduh file, dan mengunggah file. Program ini juga menangani kondisi koneksi, memastikan bahwa klien terhubung ke server sebelum mengirim perintah.

---
COMMAND DOCUMENTATION
-
**connme & ls**
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/f09a7756-a0f1-4117-899d-888e7c1c0405)
---
**rm <filename>**
-
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/b24b986e-f0f3-4c6f-82f5-477e5a38ba31)
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/4397af77-8122-4123-be20-e7c292ae8411)
---
**size <filename>**
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/29fc5a15-396f-4604-810f-78f20ec297ec)
---
**download <filename>**
-
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/7b628357-9efa-4868-8308-41f337c9e8db)
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/26a0e4d5-c164-499f-8684-c5fa30f2cc5f)
---
**upload <filename>**
-
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/f162452d-32d8-4476-be2c-4528817ad8e2)
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/002087c1-3c7f-4c9d-9252-f1150955147f)
---
**byebye**
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/3583130a-7330-4c79-803e-a7ee461a684d)

---

### SOAL TAMBAHAN

1. Modifikasi agar file yang diterima dimasukkan ke folder tertentu
2. Modifikasi program agar memberikan feedback nama file dan filesize yang diterima.
3. Apa yang terjadi jika pengirim mengirimkan file dengan nama yang sama dengan file yang telah dikirim sebelumnya? Dapat menyebabkan masalah kah ? Lalu bagaimana solusinya? Implementasikan ke dalam program, solusi yang Anda berikan.

---
SOAL 1 
-
### Penjelasan Modifikasi :
**Server :**
```python
   elif command.startswith("upload"):
        _, filename, folder = command.split(maxsplit=2)
        try:
```
- Pada fungsi handle_command, perintah upload sekarang menerima dua argumen tambahan: filename dan folder.
- Direktori folder yang ditentukan dibuat jika belum ada.
- File disimpan ke dalam direktori folder yang ditentukan.
  
**Klien :**
```python
def send_file(conn, filename, folder):
    file_path = os.path.join(client_directory, filename)
```
- Pada perintah upload, pengguna harus memasukkan filename dan folder.
- Fungsi sent_file menerima argumen tambahan folder dan mengirim file ke server, termasuk informasi tentang folder tempat file harus disimpan.

Dengan modifikasi ini, file yang diterima oleh server akan disimpan dalam folder yang ditentukan oleh klien.

---

SOAL 2
-
### Penjelasan Modifikasi :
**Server :**
```python
 print(f"File {unique_filename} berhasil diterima dan disimpan di folder {folder}.")
            response = f"File {unique_filename} berhasil diterima dengan ukuran {file_size / 1024:.2f} KB."
```
- Pada perintah upload, server memberikan umpan balik kepada klien mengenai nama file dan ukuran file yang diterima.
- Setelah menerima file, server mengirimkan pesan kembali kepada klien dengan informasi nama file dan ukurannya.
  
**Klien :**
```python
elif command.lower().startswith("upload"):
                s.send(command.encode())
                _, filename, folder = command.split(maxsplit=2)
                send_file(s, filename, folder)
                response = s.recv(BUFFER_SIZE).decode()
                print(response)
```
- Pada perintah upload, klien sekarang menunggu umpan balik dari server setelah mengirim file.
- Umpan balik ini berisi informasi tentang nama file dan ukuran file yang diterima oleh server, yang kemudian ditampilkan oleh klien.
  
Dengan modifikasi ini, setiap file yang diterima oleh server akan memberikan umpan balik yang mencakup nama file dan ukuran file yang diterima, yang akan ditampilkan oleh klien.

HASIL SOAL 1 & 2
-
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/a0f41f6e-3543-4ccb-8e1b-b1e9075d1fea)
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/6160eb2f-b917-4f46-8fe1-570825527682)

---

SOAL 3
-
### Penjelasan Modifikasi :
**Server :**
```python
def get_unique_filename(directory, filename):
    base, ext = os.path.splitext(filename)
    counter = 1
    new_filename = filename
    while os.path.exists(os.path.join(directory, new_filename)):
        new_filename = f"{base}_{counter}{ext}"
        counter += 1
    return new_filename
```
```python
elif command.startswith("upload"):
        _, filename, folder = command.split(maxsplit=2)
        try:
            file_size = int.from_bytes(conn.recv(4), byteorder='big')
            file_data = conn.recv(file_size)
            folder_path = os.path.join(server_directory, folder)
            if not os.path.exists(folder_path):
                os.makedirs(folder_path)

            unique_filename = get_unique_filename(folder_path, filename)
            file_path = os.path.join(folder_path, unique_filename)
```
- Ditambahkan fungsi get_unique_filename untuk memastikan bahwa file yang diterima memiliki nama unik dengan menambahkan angka urutan di akhir nama file jika file dengan nama yang sama sudah ada.
- Pada perintah upload, server menggunakan get_unique_filename untuk menentukan nama file yang unik sebelum menyimpan file.
- Server mengirimkan umpan balik kepada klien mengenai nama file baru yang disimpan dan ukuran file yang diterima.
  
**Klien :**
```python
elif command.lower().startswith("upload"):
                s.send(command.encode())
                _, filename, folder = command.split(maxsplit=2)
                send_file(s, filename, folder)
                response = s.recv(BUFFER_SIZE).decode()
                print(response)
```
- Klien menerima umpan balik dari server setelah mengirim file dan menampilkan informasi mengenai nama file baru yang disimpan dan ukuran file yang diterima oleh server.
  
Dengan modifikasi ini, setiap kali klien mengirim file dengan nama yang sama, server akan menyimpan file tersebut dengan nama yang berbeda untuk menghindari penimpaaan file yang sudah ada.

HASIL SOAL 3
-
![image](https://github.com/reyzass/Tugas-2-ProgJar/assets/162030249/fd1ebd1a-c725-4f5b-8702-8a49b4b2868b)









