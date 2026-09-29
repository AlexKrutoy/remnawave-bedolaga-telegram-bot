:robot: I have created a release *beep* *boop*
---


## [4.16.0](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/compare/v4.15.0...v4.16.0) (2026-09-29)


### New Features

* **broadcast:** атомарные фильтры аудитории рассылок, поиск пользователей и расчёт аудитории в момент отправки ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **cashera:** автопродление подписки через СБП-подписки Cashera в боте и кабинете ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **cashera:** возврат и чарджбэк по зачисленному платежу списывают баланс ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **cashera:** свой экран оплаты — QR прямо в боте и на карточке пополнения кабинета ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **dpichecker:** раздел DPI//CHECKER — запуски и мониторы проверок, цели из панели, итоги админам, ручки кабинета и CSV в Mini App ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **menu:** живое главное меню — бот сам обновляет последнее rich-меню при смене трафика, статуса, устройств и баланса ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **payments:** платёжная система Cashera для пополнения баланса в боте и кабинете — СБП, карта, крипта ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **webapi:** таймлайн активности пользователя и галерея вложений во внешнем API ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* интеграция с внешним антифрод-сервисом ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))


### Bug Fixes

* **abuse:** письма только на подтверждённый адрес, кривой ответ сервиса не роняет экраны ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **balance:** подсказка, как повторить пополнение с неверной суммой ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **broadcast:** аудитория учитывает живые триалы и отрицания, люди из аудитории — только с правом на карточки пользователей ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **cabinet:** документы и правила откатываются на язык по умолчанию, если текста нет ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **links:** внешние ссылки на файлы — схема и хост из заголовков прокси ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **logging:** ошибки stdlib-логгеров доезжают до админ-чата с текстом и traceback ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **panel:** сброс триала в мультитарифе стирает у человека id удалённого аккаунта ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **redis:** ожидающий пул соединений, размер и таймаут пула из .env ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **reminders:** условие «способ входа» не сводит vk_id с пустой строкой, сбой одного напоминания не роняет список и рассылку ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* **tribute:** оплата не теряется — транзакция и зачисление одним коммитом, после зачисления без 5xx, тревога админам ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))
* алерты CodeQL — слабый хеш ключа DPI//CHECKER и кольца импортов ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))


### Performance

* **live-menu:** ключи живых меню через SCAN, а не KEYS ([8a5030e](https://github.com/BEDOLAGA-DEV/remnawave-bedolaga-telegram-bot/commit/8a5030efe990e6e10e92b138b9443bdb83d8eaa1))

---
This PR was generated with [Release Please](https://github.com/googleapis/release-please). See [documentation](https://github.com/googleapis/release-please#release-please).