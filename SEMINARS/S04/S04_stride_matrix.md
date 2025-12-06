# S04 - STRIDE per element

| Element | Data/Boundary | Threat (S/T/R/I/D/E) | Description | NFR link (ID) | Mitigation idea (ADR later) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Edge:** User → API | Multipart File | **D** (DoS) | Загрузка файлов >15MB или бесконечных стримов, исчерпание диска/RAM. | NFR-001 | Strict Content-Length limit, streaming upload с обрывом соединения. |
| **Edge:** User → API | Multipart File | **T** (Tampering) | Подмена расширения файла (MIME type spoofing), загрузка exe/html как jpg. | NFR-002 | Валидация Magic Bytes (signature), переименование файла при сохранении. |
| **Edge:** User → API | HTTP Requests | **D** (DoS) | Спам запросами на endpoints `/avatar` и `/export` (L7 DDoS). | NFR-003, NFR-004 | Rate Limiter (Token Bucket) на уровне Gateway по UserID/IP. |
| **Node:** API | JWT Token | **S** (Spoofing) | Использование поддельного или истекшего JWT токена. | `need NFR` (AuthN) | Валидация подписи (RS256) и claims (exp, iss) на Gateway. |
| **Node:** API | Error Response | **I** (Info Disclosure) | Утечка стектрейсов или версий фреймворка в 500 ошибках. | NFR-001 (partial) | Global Error Handler, generic messages (RFC7807), скрытие `server` header. |
| **Edge:** API → Logic | DTO / PII | **E** (Elevation of Privilege) | Пользователь запрашивает `/export` для чужого профиля (IDOR). | `need NFR` (AuthZ) | Проверка ownership: `resource.owner_id == token.user_id`. |
| **Node:** Logic | Export Process | **D** (DoS) | "Тяжелый" SQL запрос экспорта блокирует базу данных > 5 сек. | NFR-006 | Kill slow queries, read-replica для аналитики, пагинация. |
| **Edge:** Logic → LogSink | Logs | **I** (Info Disclosure) | Запись PII (email, фио) в логи в открытом виде. | NFR-005 | Библиотека-логгер с авто-маскированием полей по patterns. |
| **Edge:** Logic → Vendor | HTTP / Network | **D** (DoS) | Внешний API "висит", потоки сервиса заканчиваются (Cascading failure). | NFR-008, NFR-009 | Circuit Breaker (open state при 50% errors), Hard Timeout 2s. |
| **Edge:** Logic → Vendor | HTTP Request | **R** (Repudiation) | Невозможно сопоставить ошибку Вендора с конкретным запросом клиента. | NFR-007 | Проброс `X-Correlation-ID` в заголовки запроса к вендору. |
| **Edge:** Logic → Vendor | Traffic | **T** (Tampering) | Перехват данных при обмене с вендором (Man-in-the-Middle). | `need NFR` (Transport) | Принудительный TLS 1.2+, валидация сертификатов. |
| **Node:** DB | File Storage | **T** (Tampering) | Прямой доступ к файлам на диске/S3 в обход прав приложения. | `need NFR` (Infra) | Ограничение прав сервисного аккаунта (Least Privilege), приватный бакет. |
| **Edge:** Logic → DB | SQL Query | **T** (Tampering) | Внедрение SQL-кода через поля фильтрации экспорта (SQL Injection). | NFR-001 (Input validation) | Использование ORM / Prepared Statements, санитайзинг ввода. |

---