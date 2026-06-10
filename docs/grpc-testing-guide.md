# gRPC Testing Guide

## Overview

Dispatcher Transfer Out BMI Service menerima request dari Switching Transfer Out melalui protokol gRPC.

Untuk kebutuhan QA/UAT, testing gRPC direkomendasikan menggunakan Postman karena lebih mudah digunakan untuk eksplorasi service, method, metadata, dan request body.

Tool alternatif yang dapat digunakan:

- Postman, preferably
- Insomnia

## gRPC Server Reflection

Service ini sudah mengaktifkan gRPC Server Reflection.

Dengan Server Reflection, tools seperti Postman atau grpcurl dapat mendeteksi service dan method gRPC secara otomatis tanpa perlu import file `.proto` secara manual.

Jika reflection tidak tersedia pada environment tertentu, QA tetap dapat melakukan testing dengan cara import manual file `.proto`.

## Postman gRPC Testing Flow

### 1. Create gRPC Request

Pada Postman:

```txt
New
-> gRPC Request
````

### 2. Input gRPC Server Address

Masukkan host dan port gRPC server.

Contoh:

```txt
localhost:50051
```

atau:

```txt
<qa-server-ip>:<grpc-port>
```

### 3. Configure Connection Mode

Jika server belum menggunakan TLS, gunakan mode plaintext/insecure.

### 4. Load Service via Reflection

Jika Server Reflection aktif, Postman akan membaca daftar service dan method secara otomatis.

Pilih service dan method yang akan diuji, misalnya:

```txt
GetBalanceInquiry
GetAccountInquiry
DoTransfer
DoTransferSknbi
DoTransferRtgs
GetTransactionStatus
GetBankStatement
```

### 5. Fill Request Body

Isi body request sesuai struktur protobuf masing-masing method.

Gunakan generated protobuf contract sebagai acuan field request.

### 6. Fill Metadata

Setiap request wajib membawa metadata berikut:

| Metadata Key    | Example Value               |
| --------------- | --------------------------- |
| `x-ikm-id`      | `1`                         |
| `x-service-id`  | `TRANSFER_OUT`              |
| `x-external-id` | `EXT-20260610-000001`       |
| `channel-id`    | `2`                         |
| `x-timestamp`   | `2026-06-10T15:30:00+07:00` |

Catatan:

Walaupun dokumen bisnis menulis metadata dalam format uppercase seperti `X-IKM-ID`, pada gRPC Go key diproses sebagai lowercase.

Gunakan lowercase saat testing di Postman:

```txt
x-ikm-id
x-service-id
x-external-id
channel-id
x-timestamp
```

### 7. Invoke Request

Klik Invoke.

Expected:

* Jika request valid, response protobuf dikembalikan sebagai normal response.
* Jika metadata tidak valid, service mengembalikan gRPC technical error.
* Jika body/message tidak valid, service mengembalikan protobuf response normal dengan response code internal `BAD_REQUEST`.
* Jika terjadi business error dari Bank BMI/mock, service tetap mengembalikan protobuf response normal.