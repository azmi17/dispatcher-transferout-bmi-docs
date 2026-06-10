# Dispatcher Transfer Out BMI Service

## Overview

Dispatcher Transfer Out BMI Service adalah service internal yang berfungsi sebagai middleware dispatcher antara sistem Switching Transfer Out dengan layanan Bank BMI.

Service ini menerima request transaksi dari Switching Transfer Out melalui gRPC, melakukan validasi request dan metadata, menerjemahkan request ke format internal usecase, meneruskan transaksi ke API Bank BMI melalui HTTP, lalu mengembalikan response dalam format standar internal dispatcher.

Dokumentasi ini disiapkan untuk kebutuhan QA/UAT, deployment note, dan referensi internal developer.

## Release Information

| Item | Value |
|---|---|
| App Name | Dispatcher Transfer Out BMI |
| Version | 1.0.0 RC1 |
| Release Type | Release Candidate |
| Status | READY TO TEST |
| Target | QA/UAT |
| Protocol | gRPC |
| Partner API | Bank BMI |
| Current Partner Mode | Mockoon |
| Sandbox Status | Bank BMI sandbox access belum tersedia |

## Documentation Index

| Document | Description |
|---|---|
| [QA Testing Guide](./qa-testing-guide.md) | Panduan utama untuk QA/UAT testing. |
| [gRPC Testing Guide](./grpc-testing-guide.md) | Panduan testing gRPC menggunakan Postman, Insomnia, atau grpcurl. |
| [Release Notes](./release-notes.md) | Catatan rilis `1.0.0 RC1`. |

## Supported Services

| Service | gRPC Method | Description |
|---|---|---|
| Balance Inquiry | `GetBalanceInquiry` | Mengecek informasi saldo rekening IKM. |
| Account Inquiry | `GetAccountInquiry` | Mengecek informasi rekening tujuan internal maupun external bank. |
| Transfer | `DoTransfer` | Memproses transaksi transfer sesama / antar Bank. |
| Transfer SKNBI | `DoTransferSknbi` | Memproses transaksi transfer SKNBI. |
| Transfer RTGS | `DoTransferRtgs` | Memproses transaksi transfer RTGS. |
| Transaction Status | `GetTransactionStatus` | Mengecek status transaksi berdasarkan referensi transaksi. |
| Bank Statement | `GetBankStatement` | Mengambil histori transaksi rekening mitra. |

## High Level Flow

```txt
Switching Transfer Out
   |
   | gRPC Request
   v
Dispatcher gRPC Handler
   |
   | Validate Metadata
   | Validate Request Body
   | Map Protobuf Request to Internal Params
   v
Usecase Layer
   |
   | Build Bank BMI Request
   | Resolve Credential
   | Resolve Access Token from Redis
   | Request New Access Token if Redis Token is Missing / Expired
   | Resolve Signature
   v
HTTP Client / API Repository
   |
   | HTTP Request
   v
Bank BMI API / Mockoon
   |
   | HTTP Response
   v
API Repository
   |
   | Map Bank Response
   v
Usecase Layer
   |
   | Map Bank BMI Response Code to Dispatcher Internal Code
   | Build Internal Result
   v
Dispatcher gRPC Handler
   |
   | Map Internal Result to Protobuf Response
   v
Switching Transfer Out
````

## Important Notes

* Untuk fase QA saat ini, outbound API Bank BMI masih menggunakan Mockoon karena akses sandbox Bank BMI belum tersedia.
* Response code testing wajib mengacu ke list code resmi pada dokumen Bank BMI, bukan hanya sample response.
* gRPC Server Reflection sudah diaktifkan untuk memudahkan testing menggunakan Postman atau grpcurl.
* Credential sementara menggunakan dummy credential.
* Untuk development/testing, decrypt credential dapat di-bypass melalui konfigurasi:

```txt
app.decrypt.credential.param = false
```

* Untuk melihat request/response log lebih lengkap selama testing, aktifkan:

```txt
app.devel_mode = true
```

## Related Links

| Resource          | URL                       |
| ----------------- | ------------------------- |
| Repository        | `<github-repo-url>`       |
| TRD Dispatcher    | `<google-docs-url>`       |
| Bank BMI Document | `<bank-bmi-document-url>` |