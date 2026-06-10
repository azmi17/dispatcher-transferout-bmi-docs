# Dispatcher Transfer Out BMI Service

## Overview

Dispatcher Transfer Out BMI Service adalah service internal yang berfungsi sebagai middleware dispatcher antara sistem Switching Transfer Out dengan layanan Bank Muamalat Indonesia (BMI).

Service ini menerima request transaksi dari Switching Transfer Out melalui gRPC, meneruskan transaksi ke API Bank BMI melalui HTTP, lalu mengembalikan response dalam format standar internal dispatcher.

Dokumentasi ini disiapkan untuk kebutuhan QA / SIT, catatan deployment, dan referensi internal ITSD.

## Release Information

| Item                 | Value                                  |
| -------------------- | -------------------------------------- |
| App Name             | Dispatcher Transfer Out BMI            |
| Version              | 1.0.0 RC1                              |
| Status               | READY TO TEST                          |
| Target               | QA/SIT                                 |
| Protocol             | gRPC                                   |
| Partner API          | Bank BMI                               |
| Current Partner Mode | Mockoon                                |
| Sandbox Status       | Bank BMI sandbox access belum tersedia |

## Documentation Index

| Document                                      | Description                                             |
| --------------------------------------------- | ------------------------------------------------------- |
| [QA Testing Guide](./qa-testing-guide.md)     | Panduan utama untuk QA/SIT testing                      |
| [gRPC Testing Guide](./grpc-testing-guide.md) | Panduan testing gRPC menggunakan Postman atau Insomnia  |
| [Release Notes](./release-notes.md)           | Catatan rilis `1.0.0 RC1`                               |

## Supported Services

| Service            | gRPC Method            | Description                                                       |
| ------------------ | ---------------------- | ----------------------------------------------------------------- |
| Balance Inquiry    | `GetBalanceInquiry`    | Mengecek informasi saldo rekening IKM                             |
| Account Inquiry    | `GetAccountInquiry`    | Mengecek informasi rekening tujuan internal maupun external bank  |
| Transfer           | `DoTransfer`           | Memproses transaksi transfer sesama / antar Bank                  |
| Transfer SKNBI     | `DoTransferSknbi`      | Memproses transaksi transfer SKNBI                                |
| Transfer RTGS      | `DoTransferRtgs`       | Memproses transaksi transfer RTGS                                 |
| Transaction Status | `GetTransactionStatus` | Mengecek status transaksi berdasarkan referensi transaksi         |
| Bank Statement     | `GetBankStatement`     | Mengambil histori transaksi rekening mitra                        |

## Dispatcer Transfer Out BMI Flow

Secara umum alur dispatcher BMI Gateway adalah sebagai berikut:

```txt
Switching Transfer Out
   |
   | Mengirim request transaksi melalui gRPC
   v
Dispatcher Transfer Out BMI
   |
   | Menerima dan memvalidasi request
   | Menyiapkan data transaksi, credential, token, dan signature
   | Mengirim request ke Bank BMI / Mockoon
   v
Bank BMI API / Mockoon
   |
   | Mengembalikan response transaksi
   v
Dispatcher Transfer Out BMI
   |
   | Menyesuaikan response Bank BMI ke standar internal dispatcher
   | Mengembalikan response ke Switching Transfer Out
   v
Switching Transfer Out
```

Ringkasnya:

```txt
Switching Transfer Out
   ↓
Dispatcher Transfer Out BMI
   ↓
Bank BMI API / Mockoon
   ↓
Dispatcher Transfer Out BMI
   ↓
Switching Transfer Out
```

## Important Notes

* Untuk fase QA/SIT saat ini, request ke API Bank BMI masih menggunakan Mockoon karena akses sandbox Bank BMI belum tersedia.
* Response code testing wajib mengacu ke list code resmi pada dokumen Bank BMI di file PDF halaman 72, bukan didapat dari sample response.
* Service ini sudah mendukung fitur gRPC Server Reflection, sehingga Postman dapat membaca daftar service dan method secara otomatis melalui opsi **Using Server Reflection** saat membuat request gRPC.
* Credential sementara menggunakan dummy credential, silakan buat data dummy sesuai kebutuhan testing.
* Untuk development/testing, decrypt data credential dari database dapat di-bypass melalui konfigurasi:

```txt
app.decrypt.credential.param = false
```

* Untuk melihat request/response log lebih lengkap selama testing, aktifkan:

```txt
app.devel_mode = true
```
