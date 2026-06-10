[← Kembali ke Halaman Utama](./index.md)

# gRPC Testing Guide

## Overview

Dispatcher Transfer Out BMI Service menerima request dari Switching Transfer Out menggunakan protokol gRPC.

Untuk kebutuhan QA / SIT, testing gRPC direkomendasikan menggunakan Postman karena lebih mudah digunakan untuk memilih service, memilih method, mengisi metadata / header, mengisi request body, dan melihat response.

Tool yang dapat digunakan:

* Postman (Diutamakan)
* Insomnia

## gRPC Server Reflection

Service ini sudah mengaktifkan fitur **gRPC Server Reflection**.

Secara sederhana, Server Reflection adalah fitur yang membantu Postman membaca daftar service dan method gRPC yang tersedia pada server secara otomatis.

Dengan fitur ini, QA tidak perlu melakukan import file `.proto` secara manual selama koneksi ke server gRPC berhasil dan Server Reflection aktif.

Pada Postman, gunakan opsi **Using Server Reflection** saat memilih method gRPC.

Jika daftar service atau method tidak muncul otomatis, opsi alternatifnya adalah melakukan import manual file `.proto`.

## Postman gRPC Testing Flow

### 1. Create gRPC Request

Pada Postman:

```txt
New
-> gRPC Request
```

### 2. Input gRPC Server Address

Masukkan host dan port gRPC server.

Contoh local:

```txt
localhost:50051
```

Contoh environment QA / SIT:

```txt
<server-devel-ip>:<grpc-port>
```

### 3. Select Method Using Server Reflection

Pada bagian pemilihan method, pilih opsi:

```txt
Using Server Reflection
```

Postman akan mencoba membaca daftar service dan method yang tersedia dari server.

Jika berhasil, pilih method yang akan diuji, misalnya:

```txt
GetBalanceInquiry
GetAccountInquiry
DoTransfer
DoTransferSknbi
DoTransferRtgs
GetTransactionStatus
GetBankStatement
```

Jika method tidak muncul, pastikan:

* alamat host dan port sudah benar
* service gRPC sudah running
* Server Reflection aktif pada environment tersebut
* koneksi ke server tidak terblokir network/firewall

Jika tetap tidak muncul, QA dapat menggunakan opsi import manual file `.proto`.

### 4. Fill Request Body

Isi request body sesuai kebutuhan test case pada masing-masing method.

Gunakan contoh request yang sudah disediakan pada dokumen teknis engine Dispatcher Transfer Out BMI.

Atau dapat menggunakan fitur **Use Example Message** pada Postman untuk membuat contoh struktur request body secara otomatis berdasarkan method gRPC yang dipilih dan tinggal sesuaikan value request-nya.

Pastikan seluruh field mandatory sudah terisi sebelum request dijalankan.

### 5. Fill Metadata / Header

Setiap request wajib membawa metadata / header berikut:

| Metadata Key    | Example Value               |
| --------------- | --------------------------- |
| `x-ikm-id`      | `1`                         |
| `x-service-id`  | `TRANSFER_OUT`              |
| `x-external-id` | `EXT-20260610-000001`       |
| `channel-id`    | `2`                         |
| `x-timestamp`   | `2026-06-10T15:30:00+07:00` |

Catatan:

Walaupun pada dokumen kontrak metadata ditulis seperti `X-IKM-ID`, pada testing gRPC di Postman disarankan menggunakan lowercase seperti contoh di atas.

### 6. Invoke Request

Klik:

```txt
Invoke
```

Setelah request dijalankan, periksa response yang diterima dari service.

## Expected Result

Secara umum, hasil testing yang diharapkan adalah sebagai berikut:

* Jika metadata / header dan body request valid, service akan mengembalikan response normal.
* Jika metadata / header wajib tidak dikirim atau kosong, service akan mengembalikan gRPC error.
* Jika body request tidak valid atau mandatory field kosong, service akan mengembalikan response normal dengan response code internal `BAD_REQUEST`.
* Jika terjadi error dari Bank BMI / Mockoon, service tetap mengembalikan response normal sesuai mapping response code internal dispatcher.
* Response code yang diterima oleh Switching / QA adalah response code internal dispatcher, bukan response code mentah dari Bank BMI.

## Mandatory Metadata / Header Checklist

Sebelum menjalankan request, pastikan metadata berikut sudah diisi:

* [ ] `x-ikm-id`
* [ ] `x-service-id`
* [ ] `x-external-id`
* [ ] `channel-id`
* [ ] `x-timestamp`

## Method Checklist

Method yang perlu diuji:

* [ ] `GetBalanceInquiry`
* [ ] `GetAccountInquiry`
* [ ] `DoTransfer`
* [ ] `DoTransferSknbi`
* [ ] `DoTransferRtgs`
* [ ] `GetTransactionStatus`
* [ ] `GetBankStatement`

## Testing Notes

* Untuk fase QA / SIT saat ini, request ke Bank BMI masih diarahkan ke Mockoon.
* Akses sandbox Bank BMI belum tersedia.
* Data credential yang digunakan masih dummy.
* Response mock harus mengikuti kontrak payload dan list response code dari dokumen Bank BMI.
* Jangan mengacu pada sample response untuk menentukan response code.
* Setiap HTTP Response Code dan response status code dari Bank BMI wajib mengacu ke List Code yang disediakan **pada doktek BMI halaman 72**.

[← Kembali ke Halaman Utama](./index.md)