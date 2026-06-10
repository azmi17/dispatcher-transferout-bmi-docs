[← Kembali ke Halaman Utama](./index.md)

# QA Testing Guide

## Ready To Test

Dispatcher Transfer Out BMI Service sudah dapat dilakukan pengujian pada environment QA / SIT.

Service ini berfungsi sebagai penghubung antara Switching untuk layanan Transfer Out dengan Bank BMI untuk memproses layanan inquiry, transfer, pengecekan status transaksi, dan histori transaksi dengan flow:

```txt
Switching
↓
gRPC
↓
Dispatcher
↓
REST API
↓
Bank Muamalat Indonesia (BMI)
````

Untuk fase QA saat ini, testing dilakukan menggunakan mock response via Mockoon karena akses sandbox dari Bank BMI belum diberikan.

## Important Testing Notes

### 1. Mode Testing Saat Ini

* Testing sementara menggunakan Mockoon sebagai simulasi API Bank BMI.
* Endpoint Bank BMI sandbox belum digunakan karena akses resmi dari Bank BMI belum tersedia.
* Request dan response Bank BMI saat ini mengikuti mock contract yang disiapkan pada Mockoon.
* Data credential yang digunakan saat ini masih menggunakan dummy credential.

### 2. Response Code Guideline

* Setiap HTTP Response Code dan response status code dari Bank BMI wajib mengacu ke List Code yang disediakan pada **doktek BMI halaman 72**.
* Jangan menjadikan Sample Response sebagai acuan utama untuk response code.
* Sample Response hanya digunakan sebagai referensi bentuk payload.
* Acuan mapping tetap menggunakan daftar response code resmi yang tersedia pada dokumen.

### 3. Protocol Testing

Dispatcher menerima request melalui protokol gRPC.

Tool testing yang dapat digunakan:

* Postman (Diutamakan)
* Insomnia

### 4. gRPC Server Reflection

Service ini sudah mengaktifkan fitur **gRPC Server Reflection**.

Secara sederhana, Server Reflection adalah fitur yang membantu tools seperti Postman untuk membaca daftar service dan method gRPC yang tersedia pada server secara otomatis.

Dengan fitur ini, Tester / QA tidak perlu melakukan import file `.proto` secara manual saat melakukan testing, selama koneksi ke server gRPC berhasil dan reflection aktif.

Pada Postman, flow penggunaannya adalah sebagai berikut:

1. Buat request baru dengan tipe **gRPC Request**.
2. Masukkan alamat host dan port gRPC server.
3. Pada bagian pemilihan method, pilih opsi **Using Server Reflection**.
4. Postman akan mencoba membaca daftar service dan method yang tersedia dari server.
5. Pilih method yang akan diuji, misalnya:

   * `GetBalanceInquiry`
   * `GetAccountInquiry`
   * `DoTransfer`
   * `DoTransferSknbi`
   * `DoTransferRtgs`
   * `GetTransactionStatus`
   * `GetBankStatement`
6. Isi metadata/header wajib.
7. Isi request body sesuai kebutuhan test case.
8. Jalankan request.

Jika daftar service atau method tidak muncul otomatis, opsi alternatifnya adalah menggunakan import manual file `.proto`.

### 5. Mandatory Metadata / Header gRPC

Setiap request gRPC wajib membawa metadata / header berikut:

| Metadata Key    | Required |
| --------------- | -------- |
| `X-IKM-ID`      | Yes      |
| `X-SERVICE-ID`  | Yes      |
| `X-EXTERNAL-ID` | Yes      |
| `CHANNEL-ID`    | Yes      |
| `X-TIMESTAMP`   | Yes      |

### 6. Credential & Encryption Note

Data credential yang digunakan saat ini masih menggunakan dummy credential.

Untuk memudahkan testing selama development, proses decrypt credential dapat di-bypass dengan konfigurasi:

```txt
app.decrypt.credential.param = false
```

Jika konfigurasi tersebut bernilai `false`, data dummy dapat digunakan tanpa proses decrypt dengan syarat value yang di inject ke database juga harus value asli tanpa encrypt.

### 7. Development Logging Mode

Jika konfigurasi berikut diset `true`:

```txt
app.devel_mode = true
```

Maka data request dan data response yang sebelumnya dimasking / hide akan tampil lebih lengkap pada log.

Mode ini hanya digunakan untuk kebutuhan development/testing.

Untuk environment non-development, masking data sensitif tetap harus aktif.

## Service / Method To Test

### A. Inquiry & Data Validation

#### 1. Balance Inquiry

gRPC Method:

```txt
GetBalanceInquiry
```

Description:

* Mengecek informasi saldo rekening IKM.
* Flow ini bersifat inquiry/read-only.
* Timeout pada flow ini tidak diperlakukan sebagai pending transaksi.

#### 2. Account Inquiry

gRPC Method:

```txt
GetAccountInquiry
```

Description:

* Mengecek informasi rekening tujuan.
* Mendukung inquiry rekening internal maupun external bank.
* Flow ini bersifat inquiry/read-only.
* Timeout pada flow ini tidak diperlakukan sebagai pending transaksi.

### B. Transfer Execution

#### 3. Transfer

gRPC Method:

```txt
DoTransfer
```

Description:

* Memproses transaksi transfer sesama/antar Bank.
* Mencakup flow transfer intrabank dan interbank sesuai request dan transaction type.
* Jika terjadi timeout saat transaksi berlangsung, response dikembalikan sebagai pending karena status akhir transaksi belum dapat dipastikan.

#### 4. Transfer SKNBI

gRPC Method:

```txt
DoTransferSknbi
```

Description:

* Memproses transaksi transfer SKNBI.
* Jika terjadi timeout saat transaksi berlangsung, response dikembalikan sebagai pending karena status akhir transaksi belum dapat dipastikan.

#### 5. Transfer RTGS

gRPC Method:

```txt
DoTransferRtgs
```

Description:

* Memproses transaksi transfer RTGS.
* Jika terjadi timeout saat transaksi berlangsung, response dikembalikan sebagai pending karena status akhir transaksi belum dapat dipastikan.

### C. Transaction Monitoring

#### 6. Transaction Status

gRPC Method:

```txt
GetTransactionStatus
```

Description:

* Mengecek status transaksi berdasarkan reference transaksi.
* Pending hanya dikembalikan jika status dari Bank BMI memang mengarah ke pending.

### D. Statement / History

#### 7. Bank Statement

gRPC Method:

```txt
GetBankStatement
```

Description:

* Mengambil histori transaksi rekening mitra.
* Flow ini bersifat inquiry/reporting.
* Timeout pada flow ini tidak diperlakukan sebagai pending transaksi.

## Handler / gRPC Method To Test

1. `GetBalanceInquiry`
2. `GetAccountInquiry`
3. `DoTransfer`
4. `DoTransferSknbi`
5. `DoTransferRtgs`
6. `GetTransactionStatus`
7. `GetBankStatement`

## Testing Focus

### A. Metadata / Header Validation

Pastikan setiap request membawa metadata wajib:

* `X-IKM-ID`
* `X-SERVICE-ID`
* `X-EXTERNAL-ID`
* `CHANNEL-ID`
* `X-TIMESTAMP`

Expected:

* Jika metadata valid, request akan diteruskan sebagai transaksi.
* Jika metadata missing/kosong, service mengembalikan gRPC technical error.

### B. Request Body / Message Validation

Pastikan mandatory field pada body request terisi.

Expected:

* Jika body valid, request akan diteruskan sebagai transaksi.
* Jika body invalid/kosong, service mengembalikan protobuf response normal dengan response code internal `BAD_REQUEST`.

### C. Timeout Handling

Timeout pada transaksi financial posting:

* `DoTransfer`
* `DoTransferSknbi`
* `DoTransferRtgs`

Expected:

* Timeout dikembalikan sebagai pending transaction.

Timeout pada inquiry/reporting/status check:

* `GetBalanceInquiry`
* `GetAccountInquiry`
* `GetTransactionStatus`
* `GetBankStatement`

Expected:

* Timeout tidak otomatis dikembalikan sebagai pending transaksi.
* Response mengikuti mapping error internal yang sesuai.

### D. Response Code Mapping

Pastikan response code dari mock Bank BMI dimapping ke response code internal dispatcher.

Expected:

* Switching hanya menerima response code internal dispatcher.
* Response code Bank BMI tidak dikembalikan mentah sebagai final response ke Switching.

### E. Logging & Tracing

Pastikan setiap request memiliki log dengan correlation id.

Contoh expected log flow:

```txt
RECEIVE gRPC | id=1781098074656788600
SEND HTTP request to Bank BMI / Mockoon | id=1781098074656788600
RECV HTTP response from Bank BMI / Mockoon | id=1781098074656788600
RESPONSE TO CLIENT | id=1781098074656788600
```

`correlation id` harus konsisten dari RECEIVE gRPC sampai SEND HTTP Request Ke API Bank BMI.

## Configuration Notes

Recommended development/testing config:

```txt
app.devel_mode = true
app.decrypt.credential.param = false
```

Notes:

* `app.devel_mode = true` digunakan agar request/response log tampil lebih lengkap untuk kebutuhan QA/debugging.
* `app.decrypt.credential.param = false` digunakan agar credential dummy dapat digunakan tanpa decrypt selama development/testing.
* Untuk environment non-development, konfigurasi masking dan decrypt credential harus disesuaikan kembali mengikuti security internal yang sudah disepakati pada dokumen TRD.

## Mock / Sandbox Notes

* Saat ini Bank BMI belum memberikan akses sandbox.
* Testing request ke Bank BMI sementara diarahkan ke Mockoon.
* Data credential masih dummy.
* Response mock harus dibuat mengikuti kontrak payload dan list response code dari dokumen Bank BMI.
* Response code testing tidak boleh mengacu pada Sample Response.

## Document TRD Dispatcher

1. Dokumen Informasi Umum Pengembangan Ekosistem Transfer Out => `https://docs.google.com/document/d/1SgQlTbUchIQ9tXQbfqPTbu9HViTJ8cJr9Awovsf2PHc/edit?pli=1&tab=t.0`
2. Dokumen Teknis Dispatcher Transfer Out BMI => `https://docs.google.com/spreadsheets/d/1yCAXFupyeQrilxAQ4bWKzTZgTrx3DIoBkUxkX3ujZA4/edit?pli=1&gid=226230398#gid=226230398`
3. Dokumen Parameter Mapping Value Dispatcher Ke BMI => `https://docs.google.com/document/d/1BskS6d51QggFCwaT7Hz7PTW9wL7sMIgRe_OjSmdw7_s/edit?tab=t.0#heading=h.lalas9egnfi2`

[← Kembali ke Halaman Utama](./index.md)