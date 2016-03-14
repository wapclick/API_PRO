# API WAPCLICK.ONLINE (ВЕРСИЯ ДЛЯ ПАРТНЁРОВ)

**версия 1.0**

## 1. Описание

WapClick – новая прогрессивная технология мобильных подписок. Абоненту достаточно нажать одну кнопку для получения доступа к платному контенту и совершения подписки. Абонент получает качественный контент, списания с его счёта происходит ежедневно!


### 1.1 Особенности:

1. Абонент соглашается на подписку на странице оператора связи (landing page).
2. Оператор связи списывает средства со счёта абонента. Списание средств и предоставление доступа прекращается при выполнении одного из следующих условий – абонент отписался, закончился срок подписки, нет достаточного колличества средств на счету.
3. Тарификация подписавшегося абонента происходит ежедневно.


## 2. Подключение

Для подключения партнёр предоставляет:

1. Адрес уведомлений.
2. Адрес возврата на сайт (может быть не задан, если партнёр будет задавать адрес возврата для каждого абонента при инициации подписки).

Менеджер сообщает партнёру:

1. Идентификатор заведённой подписки (service_id).
2. Секретный ключ (secret_key).

Для партнёров доступна расширенная схема. Схема используется в тех случаях, когда партнёр не хочет разрешать абоненту взаимодействовать с сайтом wapclick.online и хочет обрабатывать всё взаимодействие с абонентом на своей стороне. Например, это требуется в тех случаях, когда у партнёра реализована собственная версия протокола подписок и он предоставляет её субпартнёрам.

Партнёр делает GET запрос на адрес

```
https://wapclick.mobi/init/sync/[идентификатор подписки].json
```

Обязательные параметры запроса

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| p_data | varchar(100) | Идентификатор подписки в системе партнёра | 077dd9d0-690d-11e5-b533-0d1018f8ac82
| back_url | varchar(1000) | Адрес возврата абонента | https://site.com/content
| ip | varchar(15) | IP абонента | 213.87.249.227
| referer | varchar(1000) | Рефёрер (указывается адрес страницы, на котором была нажата кнопка заказа) | https://site.com/

Пример запроса

```
https://wapclick.mobi/init/sync/12187.json?ip=213.87.249.227&p_data=1&back_url=https%3A%2F%2Fsite.com%2Fcontent
```

В ответ сервис возвращает статус обработки запроса в JSON формате

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| p_data | varchar(100) | Идентификатор подписки в системе партнёра | 077dd9d0-690d-11e5-b533-0d1018f8ac82
| redirect | varchar(1000) | Адрес для редиректа абонента | http://moipodpiski.ssl.mts.ru/lp/?SID=09565fc2-6dcf-11e5-9242-f933a6d11a80&IsMobile=Y
| code | int | Статус подписки (п. 4) | 0
| error | varchar(30) | Расшифровка статуса подписки (п. 4) | ok

Пример

```json
{"code":0,"error":"ok","p_data":"077dd9d0-690d-11e5-b533-0d1018f8ac82","redirect":"http://moipodpiski.ssl.mts.ru/lp/?SID=09565fc2-6dcf-11e5-9242-f933a6d11a80&IsMobile=Y"}
```

Партнёр осуществляет перенаправление абонента на полученный адрес redirect или обрабатывает полученную ошибку.

## 3. Дополнительные параметры подключения

Если требуется предоставлять абоненту несколько видов контента или требуется большая безопасность взаимодействия, то возможно задавать дополнительные параметры

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| landing[operator] | varchar(1000) | Идентификатор лэндинга на стороне оператора | 111

Параметр landing используется для отображения абоненту кастомизированных лэндингов при оформлении подписки на сайте оператора и представляет собой идентификатор лэндинга. Данных параметров может быть несколько, если требуется указать идентификатор лэндинга для нескольких операторов.

Данные параметры не являются обязательными.

## 4. Возврат абонента на сайт партнёра

После операции подписки абонент возвращается на back_url, переданный в запросе инициации или на адрес возврата на сайт, который прописывается при заведении подписки. При возврате к адресу добавляются следующие параметры

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| code | int | Статус подписки | 0
| error | varchar(30) | Расшифровка статуса подписки | ok

Получение данных параметров (успешного статуса подписки) не является гарантией успешного статуса подписки на стороне оператора, т.к. может быть подделано абонентом. Рекомендуется опираться на данные параметры только в простейших сервисах или для предварительного уведомления абонента об успехе или неуспехе.

Для гарантии подписки абонента на стороне оператора требуется обрабатывать уведомление о тарификации и предоставлять абоненту контент после получения успешного уведомления.

Статусы подписок

| code | error | Описание
| --- | --- | ---
| 0 | ok | Подписка оформлена
| 1 | invalid service settings (please, contact us) | Невалидные настройки сервиса, требуется их корректировка на стороне wapclick.com, просьба обратиться к нам
| 2 | operator is not supported | Оператор не поддерживается
| 3 | service not found | Сервис не найден
| 4 | already subscribed | Абонент уже подписан на данный сервис
| 5 | no money | У абонента нет денег
| 6 | session broken | Сессия абонента прервана (таймаут, некорректный запрос)
| 7 | operator internal error | Ошибка на стороне оператора связи
| 8 | service internal error | Ошибка на стороне wapclick.com
| 9 | service blacklisted | Подписка находится в чёрном списке
| 10 | phone blacklisted | Номер абонента находится в чёрном списке

## 5. Уведомления о тарификациях

wapclick.online уведомляет партнера об успешных списаниях денежных средств со счета абонента.

Уведомление - GET запрос с параметрами

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| action | varchar(30) | Тип уведомления - тарификация | charge_report
| status | int | Статус тарификации. 0 - успешная, -1 - неуспешная | 0
| phone | bigint | Номер абонента | 79031234567
| op | varchar(30) | Оператор | beeline
| c_amount | numeric(18,2) | Сумма тарификации в валюте абонента | 100.01
| amount | numeric(18,2) | Сумма тарификации в валюте расчётов с партнёром | 100.01
| c_pay | numeric(18,2) | Сумма выплаты партнёру в валюте абонента | 50.01
| pay | numeric(18,2) | Сумма выплаты партнёру в валюте расчётов с партнёром | 50.01
| c_curr | character(3) | Буквенный код валюты абонента (ISO 4217) | RUB
| curr | character(3) | Буквенный код валюты расчётов с партнёром (ISO 4217) | RUB
| tid | varchar(100) | Уникальный идентификатор транзакции | 08057700-690d-11e5-b610-321018f8ac82
| p_data | varchar(100) | Идентификатор подписки в системе партнёра | 077dd9d0-690d-11e5-b533-0d1018f8ac82
| service_id | integer | Идентификатор подписки | 1234

Пример уведомления

```
https://site.com/subscriptions?action=charge_report&status=0&phone=79031234567&op=beeline&c_amount=100.01&amount=100.01&c_pay=50.01&pay=50.01&c_curr=RUB&curr=RUB&tid=08057700-690d-11e5-b610-321018f8ac82&p_data=077dd9d0-690d-11e5-b533-0d1018f8ac82&service_id=1234
````

## 6. Уведомления об отписках

wapclick.online уведомляет партнера об отписке абонента.

Уведомление - GET запрос с параметрами

| Параметр | Тип | Описание | Пример
| --- | --- | --- | ---
| action | varchar(30) | Тип уведомления - отписка | unsubscribe_report
| service_id | integer | Идентификатор подписки | 1234
| phone | bigint | Номер телефона абонента | 79031234567
| tid | varchar(100) | Идентификатор транзакции | 077dd9d0-690d-11e5-b533-0d1018f8ac82
| p_data | varchar(100) | Идентификатор подписки в системе партнёра | 077dd9d0-690d-11e5-b533-0d1018f8ac82
| sign | char(64) | Подпись запроса sha256(action+service_id+phone+tid+p_data+secret_key). Если p_data не использовался, то он не участвует в формировании подписи | 68e656b251e67e8358bef8483ab0d51c6619f3e7a1a9f0e75838d41ff368f728

Пример уведомления

```
https://site.com/subscriptions?action=unsubscribe_report&service_id=1234&phone=79031234567&tid=077dd9d0-690d-11e5-b533-0d1018f8ac82&p_data=077dd9d0-690d-11e5-b533-0d1018f8ac82&sign=68e656b251e67e8358bef8483ab0d51c6619f3e7a1a9f0e75838d41ff368f728
```

## 7. Оценка конверсии

Система поддерживает UTM метки. Для их использования нужно передавать параметры в запросе инициации подписки (п. 2.1). Метки будут доступны для выборки в личном кабинете.

Поддерживаемые метки:

* utm_source
* utm_medium
* utm_campaign
* utm_term
* utm_content

Пример запроса

```
https://wapclick.mobi/init/sync/12187.json?ip=213.87.249.227&p_data=1&back_url=https%3A%2F%2Fsite.com%2Fcontent&utm_source=1&utm_medium=2&utm_campaign=3&utm_term=4&utm_content=5
```

## 8. Контакты

С вопросами обращайтесь по почте [support@wapclick.online](mailto:support@wapclick.online)

