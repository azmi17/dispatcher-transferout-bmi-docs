# Release Notes

## [1.0.0 RC1] - 2026-06-10 15:30

### Release Summary

First release candidate untuk Dispatcher Transfer Out BMI Service.

Rilis ini mencakup implementasi awal engine dispatcher yang menghubungkan sistem Switching Transfer Out dengan Bank BMI melalui flow gRPC inbound dan HTTP outbound.

Service ini sudah mencakup validasi metadata, validasi body/message, mapping DTO internal, mapping response code, structured logging dengan correlation id, credential caching, token management, signature flow, serta integrasi ke usecase transaksi utama Transfer Out BMI.

Rilis ini ditujukan untuk deployment ke environment QA/UAT sebelum dinyatakan siap untuk production release.

### Added

* Menambahkan gRPC server delivery untuk Dispatcher Transfer Out BMI Service.
* Menambahkan gRPC service handler untuk Balance Inquiry.
* Menambahkan gRPC service handler untuk Account Inquiry.
* Menambahkan gRPC service handler untuk Transfer.
* Menambahkan gRPC service handler untuk Transfer SKNBI.
* Menambahkan gRPC service handler untuk Transfer RTGS.
* Menambahkan gRPC service handler untuk Transaction Status.
* Menambahkan gRPC service handler untuk Bank Statement.
* Menambahkan mapper request protobuf ke DTO internal untuk seluruh service gRPC yang didukung.
* Menambahkan mapper result DTO internal ke response protobuf untuk seluruh service gRPC yang didukung.
* Menambahkan mapper business error response untuk handling response gRPC.
* Menambahkan validasi mandatory metadata gRPC untuk:

  * `X-IKM-ID`
  * `X-SERVICE-ID`
  * `X-EXTERNAL-ID`
  * `CHANNEL-ID`
  * `X-TIMESTAMP`

* Menambahkan validasi body/message request menggunakan validator v10.
* Menambahkan custom validation rule `notblank` untuk field string mandatory.
* Menambahkan structured logging untuk inbound dan outbound gRPC.
* Menambahkan logging untuk business error gRPC.
* Menambahkan internal correlation id `id_req` untuk tracing transaksi end-to-end.
* Menambahkan context propagation untuk correlation id dari gRPC handler ke usecase dan outbound HTTP client.
* Menambahkan logging request/response outbound HTTP untuk integrasi Bank BMI.
* Menambahkan masking untuk sensitive header/body pada mode non-development.
* Menambahkan konfigurasi gRPC server reflection untuk kebutuhan testing/debugging menggunakan tools seperti Postman atau grpcurl.
* Menambahkan mapping response code dari Bank BMI ke response code internal dispatcher.
* Menambahkan integrasi flow credential, token, dan signature untuk outbound API Bank BMI.
* Menambahkan caching credential Bank BMI ke Redis untuk mengurangi direct read ke database.
* Menambahkan token management berbasis Redis dengan TTL mengikuti response expiry token dari Bank BMI.
* Menambahkan implementasi repository MySQL berbasis `sqlx` untuk data access yang lebih ergonomis.
* Menambahkan dokumentasi overview awal untuk Dispatcher Transfer Out BMI Service.

### Changed

* Mengubah behavior delivery gRPC dengan memisahkan technical error dan business/application error.
* Mengubah business/application error agar dikembalikan sebagai normal protobuf response menggunakan `return response, nil`.
* Mengubah penggunaan gRPC status error agar hanya digunakan untuk technical delivery error seperti nil request atau invalid metadata.
* Meningkatkan flow handler agar mengikuti urutan yang konsisten:

  * validasi object request
  * log inbound request
  * validasi metadata
  * mapping protobuf request ke DTO internal
  * validasi body/message
  * pemanggilan usecase
  * mapping result usecase ke protobuf response
  * log outbound response

* Meningkatkan logging outbound HTTP ke Bank BMI dengan shared correlation id.
* Meningkatkan format log agar selaras dengan legacy logging style perusahaan.
* Meningkatkan payload error response dengan mempertahankan field korelasi dari request jika tersedia.
* Meningkatkan struktur package internal untuk delivery gRPC:

  * `grpchandler`
  * `grpcmapper`
  * `grpcmetadata`
  * `grpclogger`

### Fixed

* Memperbaiki mapping response untuk nested response Bank Statement.
* Memperbaiki mapping response data untuk konteks Transfer interbank dengan menggunakan source account number sebagai internal `AccountNo`.
* Memperbaiki logging response gRPC agar menggunakan gRPC status, bukan hardcoded HTTP status code.
* Memperbaiki behavior error response `AdditionalInfo` untuk menghindari struktur response `null` yang tidak diperlukan.
* Memperbaiki urutan logging request/response agar outbound request log tercatat sebelum eksekusi HTTP request.
* Memperbaiki duplicate content-type execution switch pada flow HTTP client dengan memisahkan proses persiapan request body log dan eksekusi request secara lebih clean.

### Notes

* Rilis ini ditujukan untuk validasi QA/UAT.
* Deployment production sebaiknya dilakukan setelah mendapatkan sign-off dari QA.
* Jika terdapat temuan QA, perbaikan dapat dirilis sebagai release candidate berikutnya, misalnya:

  * `1.0.0 RC2`
  * `1.0.0 RC3`

* Final production release dapat ditandai sebagai `1.0.0` setelah seluruh temuan release candidate diselesaikan.