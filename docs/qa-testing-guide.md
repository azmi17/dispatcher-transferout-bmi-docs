# QA Testing Guide

## App Information

| Item | Value |
|---|---|
| App Name | Dispatcher Transfer Out BMI |
| Version | 1.0.0 RC1 |
| Date | 2026-06-10 |
| Status | READY TO TEST |
| Target | QA/UAT |
| Protocol | gRPC |
| Partner API | Bank BMI |
| Current Partner Mode | Mockoon |

## Ready To Test

Dispatcher Transfer Out BMI Service sudah selesai diimplementasikan secara fungsional mulai dari level repository, usecase, service, sampai delivery gRPC.

Service ini berfungsi sebagai engine dispatcher antara Switching Transfer Out dan Bank BMI dengan flow:

```txt
Switching Transfer Out
-> gRPC Dispatcher
-> Usecase
-> HTTP API Repository
-> Bank BMI API / Mockoon
-> Mapping Response Code
-> gRPC Response ke Switching Transfer Out
````

Untuk fase QA saat ini, testing dilakukan menggunakan mock response via Mockoon karena akses sandbox dari Bank BMI belum diberikan.

## Important Testing Notes

### 1. Mode Testing Saat Ini

* Testing sementara menggunakan Mockoon sebagai simulasi API Bank BMI.
* Endpoint Bank BMI sandbox belum digunakan karena akses resmi dari Bank BMI belum tersedia.
* Request dan response Bank BMI saat ini mengikuti mock contract yang disiapkan pada Mockoon.
* Data credential yang digunakan saat ini masih menggunakan dummy credential.

### 2. Response Code Guideline

* Setiap HTTP Response Code dan response status code dari Bank BMI wajib mengacu ke List Code yang disediakan pada dokumen TRD/Bank BMI halaman terakhir.
* Jangan menjadikan Sample Response sebagai acuan utama untuk response code.
* Sample Response hanya digunakan sebagai referensi bentuk payload.
* Acuan mapping tetap menggunakan daftar response code resmi yang tersedia pada dokumen.

### 3. Protocol Testing

Dispatcher menerima request melalui protokol gRPC.

Tool testing yang dapat digunakan:

* Postman, preferably
* Insomnia
* grpcurl

### 4. gRPC Server Reflection

Engine ini sudah mengaktifkan gRPC Server Reflection.

Dengan Server Reflection, tools seperti Postman atau grpcurl dapat mendeteksi service dan method gRPC secara otomatis tanpa perlu import file `.proto` secara manual.

Opsi lain tetap bisa menggunakan import manual file `.proto` jika dibutuhkan.

### 5. Mandatory Metadata gRPC

Setiap request gRPC wajib membawa metadata berikut:

| Metadata Key    | Required |
| --------------- | -------- |
| `X-IKM-ID`      | Yes      |
| `X-SERVICE-ID`  | Yes      |
| `X-EXTERNAL-ID` | Yes      |
| `CHANNEL-ID`    | Yes      |
| `X-TIMESTAMP`   | Yes      |

Catatan:

Pada implementasi gRPC Go, metadata key akan dibaca dalam lowercase:

| Original Key    | gRPC Go Key     |
| --------------- | --------------- |
| `X-IKM-ID`      | `x-ikm-id`      |
| `X-SERVICE-ID`  | `x-service-id`  |
| `X-EXTERNAL-ID` | `x-external-id` |
| `CHANNEL-ID`    | `channel-id`    |
| `X-TIMESTAMP`   | `x-timestamp`   |

### 6. Credential & Encryption Note

Data credential yang digunakan saat ini masih menggunakan dummy credential.

Untuk memudahkan testing selama development, proses decrypt credential dapat di-bypass dengan konfigurasi:

```txt
app.decrypt.credential.param = false
```

Jika konfigurasi tersebut bernilai `false`, credential dummy dapat digunakan tanpa proses decrypt.

### 7. Development Logging Mode

Jika konfigurasi berikut diset `true`:

```txt
app.devel_mode = true
```

Maka request dan response yang sebelumnya dimasking akan tampil lebih lengkap pada log.

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
* Timeout saat proses check status tidak otomatis dianggap sebagai pending transaksi original.

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

### A. Metadata Validation

Pastikan setiap request membawa metadata wajib:

* `X-IKM-ID`
* `X-SERVICE-ID`
* `X-EXTERNAL-ID`
* `CHANNEL-ID`
* `X-TIMESTAMP`

Expected:

* Jika metadata valid, request diteruskan ke usecase.
* Jika metadata missing/kosong, service mengembalikan gRPC technical error.

### B. Request Body Validation

Pastikan mandatory field pada body request terisi.

Expected:

* Jika body valid, request diteruskan ke usecase.
* Jika body invalid/kosong, service mengembalikan protobuf response normal dengan response code internal `BAD_REQUEST`.

### C. Business Error Handling

Pastikan business error tidak dikembalikan sebagai gRPC error.

Expected:

* Business error dikembalikan sebagai normal protobuf response.
* Format return tetap:

```go
return response, nil
```

### D. Timeout Handling

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

### E. Response Code Mapping

Pastikan response code dari mock Bank BMI dimapping ke response code internal dispatcher.

Expected:

* Switching hanya menerima response code internal dispatcher.
* Response code Bank BMI tidak dikembalikan mentah sebagai final response ke Switching.

### F. Logging & Tracing

Pastikan setiap request memiliki log dengan correlation id.

Expected log flow:

```txt
RECEIVE gRPC request
Metadata log
SEND HTTP request to Bank BMI / Mockoon
RECV HTTP response from Bank BMI / Mockoon
RESPONSE TO CLIENT
```

Correlation id internal:

```txt
id_req
```

`id_req` harus konsisten dari inbound gRPC sampai outbound HTTP.

## Configuration Notes

Recommended development/testing config:

```txt
app.devel_mode = true
app.decrypt.credential.param = false
```

Notes:

* `app.devel_mode = true` digunakan agar request/response log tampil lebih lengkap untuk kebutuhan QA/debugging.
* `app.decrypt.credential.param = false` digunakan agar credential dummy dapat digunakan tanpa decrypt selama development/testing.
* Untuk environment non-development, konfigurasi masking dan decrypt credential harus disesuaikan kembali mengikuti security policy.

## Mock / Sandbox Notes

* Saat ini Bank BMI belum memberikan akses sandbox.
* Testing outbound Bank BMI sementara diarahkan ke Mockoon.
* Data credential masih dummy.
* Response mock harus dibuat mengikuti kontrak payload dan list response code dari dokumen Bank BMI.
* Response code testing tidak boleh hanya mengacu pada Sample Response.

## Document TRD Dispatcher

1. `<google-docs-url>`
2. `<google-docs-url>`
3. `<google-docs-url>`

## Document Bank BMI

1. `<bank-bmi-document-url>`

Catatan:

Gunakan List Code pada halaman terakhir dokumen Bank BMI sebagai acuan response code.

## GitHub

```txt
<github-repo-url>
```

## Deployment Notes

* Build service dilakukan manual sesuai mekanisme deployment saat ini.
* Service siap dideploy ke environment QA/UAT.
* Setelah deployment, QA dapat melakukan testing melalui Postman gRPC dengan Server Reflection atau import manual file proto jika diperlukan.

## Release Status

| Item         | Value             |
| ------------ | ----------------- |
| Status       | READY TO TEST     |
| Target       | QA/UAT Validation |
| Release Type | Release Candidate |
| Version      | 1.0.0 RC1         |
