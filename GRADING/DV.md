# DV - Мини-проект «DevOps-конвейер»

---

## 0) Мета

- **Проект:** учебный шаблон
- **Версия (commit/date):** 2025-12-07

---

## 1) Воспроизводимость локальной сборки и тестов (DV1)

- **Одна команда для сборки/тестов:**

  ```bash
  make ci-s06
  ```

- **Версии инструментов (фиксация):**

1. python >= 3.11, есть проверка в `Makefile`
2. версии библиотек зафиксированы в [pip-freeze.txt](https://github.com/titarenko-a-v/secdev-lite-titarenko/blob/main/EVIDENCE/S06/pip-freeze.txt)

- **Описание шагов (кратко):**

1. Склонировать репозиторий https://github.com/titarenko-a-v/secdev-s06-s08
2. Создать файл .env на основе шаблона .env.example
3. Запустить сборку и тесты `make ci-s06`
4. Запустить приложение `make run`
---

## 2) Контейнеризация (DV2)

- **[Dockerfile](https://github.com/titarenko-a-v/secdev-s06-s08/blob/main/Dockerfile):** добавлены health check и non-root user 
- **Сборка/запуск локально:**

  ```bash
  docker build -t app:local .
  docker run --rm -p 8080:8080 app:local
  ```
  
- **[docker-compose](https://github.com/titarenko-a-v/secdev-s06-s08/blob/main/docker-compose.yml)**

---

## 3) CI: базовый pipeline и стабильный прогон (DV3)

- **Платформа CI:** GitHub Actions
- **Файл конфига CI:** [workflows/ci.yml](https://github.com/titarenko-a-v/secdev-s06-s08/blob/main/.github/workflows/ci.yml)
- **Стадии:** checkout → deps → **build** → **test** → (package)
- **Стабильность:** Запуск зеленый
- **Ссылка/копия лога прогона:** [ci-log.txt](https://github.com/titarenko-a-v/secdev-lite-titarenko/blob/main/EVIDENCE/S08/ci-log.txt)

---

## 4) Артефакты и логи конвейера (DV4)

| Артефакт/лог                    | Путь в `EVIDENCE/`   | Комментарий                       |
|---------------------------------|----------------------|-----------------------------------|
| Лог успешной сборки/тестов (CI) | `S08/ci-log.txt`     | ключевые шаги                     |
| Локальный лог сборки (опц.)     | `S07/build.log`      | билд докера                       |
| Freeze/версии инструментов      | `S06/pip-freeze.txt` | воспроизводимость окружения       |

---

## 5) Секреты и переменные окружения (DV5 - гигиена, без сканеров)

- **Шаблон окружения:** добавлен файл `/.env.example` со списком переменных (без значений), например:
  - APP_NAME=<set_me>
  - DEBUG=<set_me>
  - SECRET_KEY=<set_me>
- **Хранение и передача в CI:**  
  - секреты лежат в настройках репозитория  
  - в pipeline они **не печатаются** в явном виде.
- **Пример использования секрета в job (адаптируйте):**

  ```yaml
  - name: Mask secret
    run: echo "::add-mask::${{ secrets.SECRET_KEY }}"
  ```

- **Быстрая проверка отсутствия секретов в коде (любой простой способ):**

  ```bash
  git grep -nE 'AKIA|SECRET|token=|password=' || true
  ```

  [grep-secrets.txt](https://github.com/titarenko-a-v/secdev-lite-titarenko/blob/main/EVIDENCE/S08/grep-secrets.txt)

- **Памятка по ротации:** Секреты хранятся в репозитории и меняются при подозрении на утечку.

---

## 6) Индекс артефактов DV

_Чтобы преподаватель быстро сверил файлы._

| Тип                            | Файл в `EVIDENCE/`        |
|--------------------------------|---------------------------|
| Логи локального запуска тестов | `S06/logs/pytest.log`     |
| Зависимости pip                | `S06/pip-freeze.txt`      |
| Репорт тестов                  | `S06/test-report.xml`     |
| Лог билда докера               | `S07/build.log`           |
| Лог билда docker-compose       | `S07/compose-up.log`      |
| Размер контейнера              | `S07/image-size.txt`      |
| Код ответа главной страницы    | `S07/http_root_code.json` |
| Health                         | `S07/health.json`         |
| Inspect Web                    | `S07/inspect_web.json`    |
| Non-root                       | `S07/non-root.txt`        |
| Процессы докер                 | `S07/ps.txt`              |
| Лог запуска контейнера         | `S07/run.log`             |
| Лог CI                         | `S08/ci-log.txt`          |
| Ссылка на последний запуск CI  | `S08/ci-run.txt`          |
| Покрытие тестами               | `S08/coverage.xml`        |
| Отчёт по тестам                | `S08/test-report.xml`     |
| Греп секретов гитом            | `S08/grep-secrets.txt`    |

---

## 7) Связь с TM и DS (hook)

- **TM:** этот конвейер обслуживает риски процесса сборки/поставки (например, культура работы с секретами, воспроизводимость).  
- **DS:** сканы/гейты/триаж будут оформлены в `DS.md` с артефактами в `EVIDENCE/`.

---

## 8) Самооценка по рубрике DV (0/1/2)

- **DV1. Воспроизводимость локальной сборки и тестов:** 2  
- **DV2. Контейнеризация (Docker/Compose):** 2  
- **DV3. CI: базовый pipeline и стабильный прогон:** 2  
- **DV4. Артефакты и логи конвейера:** 2  
- **DV5. Секреты и конфигурация окружения (гигиена):** 2  

**Итог DV (сумма):** 10/10
