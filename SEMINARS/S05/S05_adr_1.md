# ADR-001: Resilient Integration with Vendor X (Circuit Breaker)
Status: Proposed

## Context
Risk: R-01 Cascading Failure (L=4, I=5, Score=20)
DFD: Edge: Logic → Vendor X API
NFR: NFR-008, NFR-009, NFR-007
Assumptions:
- Интеграция синхронная (REST/HTTP).
- Внешний вендор ненадежен (возможны таймауты и 500-е ошибки).

## Decision
Внедрить паттерн **Circuit Breaker** в комбинации с **Timeouts** и **Retries** на уровне HTTP-клиента, обращающегося к Vendor X.

- Param/Policy: **Hard Timeout** = 2000ms (2s) на запрос (Scope: `GET /api/integrations/vendor-x/*`).
- Param/Policy: **Circuit Breaker** = Open state при >50% ошибок (5xx/timeout) в окне 60 секунд. Minimum requests = 10.
- Param/Policy: **Retry** = Максимум 3 попытки, Exponential Backoff (wait duration: 100ms -> 500ms), только идемпотентные методы.
- Param/Policy: **Fallback** = При открытом CB возвращать 503 Service Unavailable (Fail Fast) или кэшированные данные (если применимо).

## Alternatives
- **No Protection (Plain HTTP Client)** - Отклонено. Высокий риск исчерпания потоков (thread starvation) при деградации вендора.
- **Asynchronous Queue (Fire-and-forget)** - Отклонено. Специфика продукта требует синхронного ответа пользователю в UI.
- **Global Load Balancer timeouts** - Отклонено. Недостаточно гранулярно, не защищает внутренние пулы соединений приложения.

## Consequences
+ **Resilience:** Сервис не зависает и не падает целиком при сбоях партнера.
+ **Fail Fast:** Пользователь сразу получает ошибку, вместо ожидания 30+ секунд.
- **UX impact:** Частичная недоступность функционала в моменты нестабильности сети.
- **Complexity:** Требуется тонкая настройка порогов CB, чтобы избежать ложноположительных срабатываний ("flapping").

## DoD / Acceptance
Given Вендор отвечает с задержкой 5000ms
When Отправляется серия из 20 запросов
Then Первые 2-3 запроса завершаются по таймауту (2s), последующие мгновенно отбиваются CircuitBreaker-ом.
Checks:
- test: Integration Test с использованием `MockServer` (latency simulation).
- log: pattern `circuit_breaker_state_change` (transition to OPEN).
- metric: `upstream_latency_p99` < 2.1s.

## Rollback / Fallback
Вынести конфигурацию (on/off, thresholds) в "горячий" конфиг (ConfigMap/Consul). При проблемах с CB — отключить через Feature Flag (`ENABLE_CB=false`), вернувшись к стандартному таймауту.

## Trace
- DFD: Edge Logic → Vendor
- STRIDE: D (DoS), R (Repudiation)
- Risk scoring: R-01 (Top-1)
- NFR: NFR-007, NFR-008, NFR-009

## Dates
Date created: 2025-12-06
Last updated: 2025-12-06

## Open Questions
- Какой `waitDurationInOpenState` поставить (автоматическая попытка восстановления)? По умолчанию 60s.