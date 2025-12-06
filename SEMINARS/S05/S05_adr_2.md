# ADR-002: Secure Avatar Upload Validation Strategy
Status: Proposed

## Context
Risk: R-02 Malicious File Upload (L=3, I=5, Score=15)
DFD: Edge: User → API
NFR: NFR-001, NFR-002, NFR-003
Assumptions:
- Загрузка через `multipart/form-data`.
- Разрешены только изображения (JPEG, PNG).
- Файловое хранилище (S3/Disk) не выполняет файлы, но валидация нужна на входе.

## Decision
Реализовать многоуровневую валидацию входного потока файла до его сохранения на постоянный носитель.

- Param/Policy: **Magic Bytes Check** = Использовать библиотеку сигнатур для детекции реального типа файла. Запретить доверие заголовку `Content-Type`.
- Param/Policy: **Strict Allowlist** = Разрешить только подмножество: `image/jpeg`, `image/png`.
- Param/Policy: **Size Limit** = 15MB Max. Обрывать соединение при превышении.
- Param/Policy: **Renaming** = Сохранять файл под сгенерированным UUID. Оригинальное имя — только в метаданных БД.

## Alternatives
- **Check Extension only** - Отклонено. Тривиально обходится переименованием `malware.php` -> `avatar.jpg`.
- **Async Antivirus Scan** - Рассмотрено как дополнительная мера, но не заменяет синхронную валидацию структуры файла.
- **Image Rewriting (Transcoding)** - Отклонено из-за высокой нагрузки на CPU.

## Consequences
+ **Security:** Исключает класс атак RCE via File Upload и XSS через SVG/HTML.
+ **Storage Safety:** Защита от переполнения диска.
- **Performance:** Анализ байтов требует CPU и памяти.
- **Compatibility:** Некоторые валидные, но нестандартные изображения могут быть отклонены валидатором.

## DoD / Acceptance
Given Хакер пытается загрузить файл `exploit.sh` переименованный в `image.png`
When Выполняется POST `/api/files/avatar`
Then Сервис возвращает 400/422 Bad Request, файл не попадает в хранилище.
Checks:
- test: `SecurityUploadTest` (dataset of polyglot files).
- log: `upload_rejected` field `reason="invalid_magic_bytes"`.
- policy: Security Gate в CI проверяет наличие валидатора.

## Rollback / Fallback
В случае проблем с легальными файлами (False Positives) — возможность оперативно отключить проверку Magic Bytes через ENV переменную `UPLOAD_STRICT_MODE=false`, оставив проверку расширений (как временный workaround).

## Trace
- DFD: Edge User → API
- STRIDE: T (Tampering), D (DoS)
- Risk scoring: R-02 (Top-2), R-04 (Top-4)
- NFR: NFR-001, NFR-002

## Ownership & Dates
Date created: 2025-12-06
Last updated: 2025-12-06