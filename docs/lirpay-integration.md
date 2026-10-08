# LirPay — приём платежей через hosted checkout (lirpay.org)

> Интеграция для [Remnawave Bedolaga Bot](../README.md). Провайдер: [LirPay](https://lirpay.org), Integration API v2 — `https://lirpay.org/api/v2/integration`. Документация провайдера: <https://lirpay.org/docs>.

LirPay — платёжная система с hosted checkout: в боте и кабинете одна кнопка «LirPay», покупатель уходит на страницу `https://lirpay.org/pay/{public_id}`, сам выбирает способ оплаты (СБП, банковская карта, криптовалюта) и платит. Результат приходит на вебхук, подписанный HMAC-SHA256.

## Как работает интеграция

```
Покупатель в боте/кабинете (одна кнопка «LirPay»)
        │  сумма ≥ LIRPAY_MIN_AMOUNT_KOPEKS
        ▼
POST /payment-links ──► LirPay создаёт ссылку (method_mode=multi)
        │   Idempotency-Key = наш order_id (lp{tg_id}_{hex})
        │   customer_id = telegram_id
        ▼
Запись lirpay_payments (status=pending, expires_at)
        │
        ▼
Покупатель платит на lirpay.org/pay/{public_id}
        │
        ├─── ВЕБХУК (живой режим): payment.succeeded ──► проверка подписи ──► зачисление
        │
        └─── ФОНОВАЯ СВЕРКА (страховка): GET /payment-links/{public_id} ──► зачисление
```

Оба пути сходятся в `_finalize_lirpay_payment`: FOR UPDATE-блокировка строки →
сверка суммы → транзакция `deposit` → пополнение баланса → уведомления.
Одновременный приход вебхука и сверки не задвоит зачисление
(идемпотентность по `transaction_id` + `balance_credited` в метаданных).

## Настройка

### 1. Мерчант и проект

Зарегистрируйтесь на [lirpay.org](https://lirpay.org), создайте проект и дождитесь модерации. UUID одобренного проекта — это `LIRPAY_PROJECT_ID` (можно получить через `GET /projects`).

### 2. Ключи API

В кабинете (`lirpay.org/merchant/api-keys`) создайте пару ключей:

| Ключ | Формат | Назначение |
|---|---|---|
| Публичный | `lpk_live_…` / `lpk_test_…` | заголовок `X-Lirpay-Public-Key` |
| Секретный | `lsk_live_…` / `lsk_test_…` | заголовок `X-Lirpay-Secret-Key`, показывается **один раз** |

**Обязательные scopes:** `payments:write`, `payments:read`, `projects:read`.
Если вебхук настраивается через API (шаг 3, вариант Б) — добавьте `webhooks:write` и `webhooks:read`.

Ключ целиком определяет окружение: `lpk_test_` работает с песочницей, `lpk_live_` — с реальными деньгами. Отдельного флага режима нет.

### 3. Вебхук

**Вариант А — через кабинет LirPay:**
- URL: `https://{ваш-домен}/lirpay-webhook` (только HTTPS)
- События: `payment.succeeded`, `payment.failed`, `payment.expired`, `payment.refunded`

**Вариант Б — через API** (нужен scope `webhooks:write`):

```bash
curl -X PUT https://lirpay.org/api/v2/integration/webhook \
  -H "X-Lirpay-Public-Key: lpk_live_…" \
  -H "X-Lirpay-Secret-Key: lsk_live_…" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://bot.example.com/lirpay-webhook",
    "event_types": ["payment.succeeded", "payment.failed", "payment.expired", "payment.refunded"],
    "enabled": true
  }'
```

При сохранении выдаётся **секрет вебхука** `whsec_…` — он **отдельный** от ключей API и показывается один раз. Это `LIRPAY_WEBHOOK_SECRET`.

Проверить доставку можно тестовым событием (бот ответит `200 {"status":"ok"}`):

```bash
curl -X POST https://lirpay.org/api/v2/integration/webhook/test \
  -H "X-Lirpay-Public-Key: lpk_live_…" -H "X-Lirpay-Secret-Key: lsk_live_…" \
  -H "Idempotency-Key: check-001" -d '{}'
```

### 4. Переменные окружения

```dotenv
LIRPAY_ENABLED=true
LIRPAY_PUBLIC_KEY=lpk_live_…
LIRPAY_SECRET_KEY=lsk_live_…
LIRPAY_WEBHOOK_SECRET=whsec_…
LIRPAY_PROJECT_ID=00000000-0000-0000-0000-000000000000
```

Способ включается только при заполненных **всех четырёх** секретах — с пустым секретом вебхука подпись подделывается тривиально.

#### Все переменные

| Переменная | По умолчанию | Описание |
|---|---|---|
| `LIRPAY_ENABLED` | `false` | Включить способ пополнения |
| `LIRPAY_PUBLIC_KEY` | — | Публичный ключ API (`lpk_…`), обязателен |
| `LIRPAY_SECRET_KEY` | — | Секретный ключ API (`lsk_…`), обязателен, показывается один раз |
| `LIRPAY_WEBHOOK_SECRET` | — | Секрет подписи вебхуков (`whsec_…`), обязателен, выдаётся при PUT /webhook |
| `LIRPAY_PROJECT_ID` | — | UUID одобренного проекта, обязателен (GET /projects) |
| `LIRPAY_BASE_URL` | `https://lirpay.org` | Базовый URL API |
| `LIRPAY_DISPLAY_NAME` | `LirPay` | Имя на кнопке пополнения |
| `LIRPAY_CURRENCY` | `RUB` | Валюта счёта: `RUB`, `USD` или `EUR` |
| `LIRPAY_MIN_AMOUNT_KOPEKS` | `10000` (100₽) | Минимальная сумма пополнения |
| `LIRPAY_MAX_AMOUNT_KOPEKS` | `10000000` (100 000₽) | Максимальная сумма пополнения |
| `LIRPAY_WEBHOOK_PATH` | `/lirpay-webhook` | Путь вебхука — должен совпадать с настроенным у LirPay |
Способ оплаты (СБП, карта, крипта) покупатель выбирает **на странице LirPay** — в боте и кабинете одна кнопка «LirPay». Крипто-методы имеют свои минимумы (~5.5 USD): при меньшей сумме ссылка создаётся, но соответствующие методы будут скрыты на странице (провайдер возвращает `below_minimum` в `warnings`).

## Специфика окружений (проверено живым прогоном)

| | TEST (`lpk_test_`) | LIVE (`lpk_live_`) |
|---|---|---|
| Ссылки создаются | ✅ | ✅ |
| Оплата | эмулятором, без денег | реальные деньги |
| Вебхук за оплату | ❌ **не приходит** (только `POST /webhook/test`-эвенты) | ✅ `payment.succeeded` с подписью |
| Зачисление баланса | ❌ помечается `error` — защита от бесплатных пополнений | ✅ мгновенно по вебхуку |
| Фоновая сверка | ✅ находит оплату, но `test_mode` → не начисляет | ✅ доначисляет, если вебхук потерялся |

Для тестирования включите автосверку — она покажет, что счёт оплатился, и корректно откажется начислять:

```dotenv
PAYMENT_VERIFICATION_AUTO_CHECK_ENABLED=true
PAYMENT_VERIFICATION_AUTO_CHECK_INTERVAL_MINUTES=1
```

## Безопасность

- **Подпись обязательна**: HMAC-SHA256 от сырого тела запроса, заголовок `X-Lirpay-Signature`. Без секрета проверка невозможна — webhook возвращает 400.
- **Сумма сверяется до зачисления**: расхождение → статус `amount_mismatch`, деньги не зачислены, доставка вебхука повторится (не-2xx).
- **Тестовые события не начисляют баланс**: режим берётся из заголовка `X-Lirpay-Mode` и полей `test`/`livemode` — событие помечается `error`.
- **Возвраты и чарджбэки** (`payment.refunded`, `payment.chargeback`): баланс не списывается автоматически — платёж помечается `refunded`, требуется ручная сверка.
- **Идемпотентность**: повторная доставка вебхука и одновременная фоновая сверка не задвоят зачисление (`FOR UPDATE` + `transaction_id` + `balance_credited`).
- **Idempotency-Key = order_id** при создании ссылки: сетевой повтор не создаст второй счёт.

## Диагностика

| Симптом | Причина | Решение |
|---|---|---|
| Вебхук LirPay получает 502 | бот не поднял `/lirpay-webhook` | проверить `is_lirpay_enabled()` (все 4 секрета) и лог единого веб-сервера |
| `подпись не совпала` в логе | секрет вебхука не совпадает с окружением | тестовый и live-секреты разные — проверьте `LIRPAY_WEBHOOK_SECRET` |
| Оплата прошла, баланс не пополнился (test) | тестовое окружение не шлёт вебхуки | включить автосверку; в test-режиме начисления не будет — это норма |
| Оплата прошла, баланс не пополнился (live) | вебхук потерялся | автосверка доначислит в течение интервала; проверить `/webhooks/lirpay` health |
| Ссылка создаётся, метод недоступен | ниже минимума метода (~5.5 USD для крипты) | поднять сумму — метод скрыт на странице оплаты сам |

Health-check вебхука: `GET https://{домен}/lirpay-webhook` →
`{"status":"ok","service":"lirpay_webhook","enabled":true}`.

## Архитектура в коде

| Файл | Роль |
|---|---|
| `app/services/lirpay_service.py` | клиент API: создание/статус/отмена ссылок, `verify_webhook_signature` |
| `app/services/payment/lirpay.py` | миксин `PaymentService`: `create_lirpay_payment`, `process_lirpay_callback`, `check_lirpay_payment_status`, зачисление |
| `app/database/crud/lirpay.py` | CRUD `lirpay_payments` c FOR UPDATE |
| `migrations/alembic/versions/0132_create_lirpay_payments.py` | таблица |
| `app/webserver/payments.py` | маршрут `POST /lirpay-webhook` + health на GET |
| `app/handlers/balance/lirpay.py` | экраны пополнения в боте |
| `tests/services/test_payment_service_lirpay.py` | 22 теста: создание, вебхук, подпись, суммы |
