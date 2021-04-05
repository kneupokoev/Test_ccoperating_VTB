# Введение

##*Цель документа*
Целью документа является описание модуля «Лояльность» системы my|GEMINI

##*Область применения*
Эта бизнес-область предназначена для операций с бонусами клиента. Клиенты могут просматривать бонусы, детали бонусов.

##*Глоссарий*
Таблица содержит важные определения, сокращения и аббревиатуры:

Термин | Описание
------------- | -------------
АБС	| Автоматизированная банковская система
myInternet, ИБ	| Краткое обозначение системы myGEMINI Internet (канал интернет-банкинга Системы)
myGEMINI, Система	| Интегрированная многоканальная фронтальная банковская система myGEMINI
ESB, шина	| IBM WebSphere Enterprise Service Bus
Оператор (Bank Operator) | Сотрудник Банка, уполномоченный на выполнение тех или иных функций в консоли администрирования Системы
Клиент (Client)	| Клиент банка – зарегистрированное в одной или нескольких соответствующих АБС физическое, юридическое лицо или индивидуальный предприниматель, пользующееся теми или иными финансовыми или дополнительными услугами (продуктами) банка
Канал |	это комплекс организационно-технических средств, процедур и параметров, позволяющих осуществлять прямое обслуживание конечного клиента банка или пользователя банковской автоматизированной системы. Например, канал Интернет банк.
Пользователь (User) |	Физическое лицо, представляющее Клиента банка в том или ином канале Системы. Пользователь может быть либо представителем Клиента (уполномоченный сотрудник юридического лица, доверенное лицо физического лица), либо самим Клиентом (физическим лицом, индивидуальным предпринимателем).
Программа лояльности |	Бонусная программа, позволяющая держателям карт банка накапливать бонусы, которые впоследствии можно использовать, совершая покупки в точках продаж партнёров программы
Бонусы | Бонусы лояльности, начисленные согласно программе лояльности участнику данной программы

# Требования к серверной части
##*IB.GetLoyaltyInfo*
1. url: /rest/private/loyalty/info; method: GET
2. Назначение сервиса: получение суммы начисленных бонусов

###Структура запроса и ответа
3. Структура запроса: Нет
4. Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo):

Название параметра | Тип | Описание/Значение | Обязательность
------------- | ------------- | ------------- | -------------
loyaltyContractId | Long | Идентификатор бонусного контракта | Mandatory
availableLoyaltyAmount | MoneyMo | Общая сумма начисленных баллов | Mandatory

###Основной сценарий
5.	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису IB.GetLoyaltyInfo.
6.	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие прото-колу (Неуспешная проверка: EC-BadRequest[1])
7.	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])
8.	[Шаг4] myGEMINI Core формирует и отправляет GET запрос к методу getLoyaltyContract МкС LoyaltyContract для получения информации о бонусном счете.

Элемент запроса	| Значение
--- | ---
clientId | Идентификатор клиента в АБС CIF
_exclude | programs,preliminaryCalculations,agreements

9.	[Шаг5] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.
10.	[Шаг6] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception[1])

    10.1. HTTP status 404: (EC-1)
11.	[Шаг7] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная проверка: EC-2)
12.	[Шаг8] myGEMINI Core сохраняет описание бонусного счета в БД в соответ-ствии с маппингом [4]
13.	[Шаг9] myGEMINI Core получает сохраненное описание бонусного счета из БД.
14.	[Шаг10] myGEMINI Core формирует ответ приложению, значения атрибутов которого заполняются согласно следующим правилам:

Элемент запроса	| Значение
---|---
loyaltyContractId |	Идентификатор бонусного контракта в БД LoyaltyAccount.id
availableLoyaltyAmount | Значение LoyaltyAccount.availableLoyaltyAmount
availableLoyaltyAmount/currency | Значение LoyaltyAccount.availableLoyaltyAmountCurrency
15.	[Шаг11] myGEMINI Core возвращает ответ приложению.
16.	[Шаг12] Сценарий завершается.

###Исключительные сценарии (ЕС)
**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**
17.	[Шаг1] myGEMINI Core формирует описание ошибки:
       HTTP/1.1 400
```json5
{
"errorResponseMo": {
"errorMsg": "Функциональность временно недоступна",
"errorCode": "UNKNOWN_ERROR",
"errorSubCode": "PRODUCT_NOT_FOUND",
"errorFields": []
    }
}
```
18.	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
19.	Сценарий завершается.

**EC-2. Статус бонусного баланса неактивный GetLoyaltyContractResponse/bonusBalanceStatus == '0'**

20.	[Шаг1] myGEMINI Core формирует описание ошибки:
       HTTP/1.1 400
```json5
{
"errorResponseMo": {
"errorMsg": " Бонусный счет заблокирован",
"errorCode": "BUSINESS_ERROR",
"errorSubCode": "BONUS_BALANCE_BLOCKED",
"errorFields": []
    }
}
```
21.	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
22.	Сценарий завершается.


##*IB.GetLoyaltyInfoDetails*
23.	url: /rest/private/loyalty/info/details; method: GET
24.	Назначение сервиса: сумма начисленных бонусов

###Структура запроса и ответа
25.	Структура запроса:

Название параметра | Тип Описание/Значение | Обязательность
---- | ---- | ----
forceRequest	| Boolean	| Признак необходимости обращения в МкС	Optional

26.	Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo)

Название парамет-ра | Тип	| Описание/Значение	| Обязательность
--- | --- | --- | ---
loyaltyContractId	| Long |	Идентификатор бонусного контракта	| Mandatory
availableLoyaltyAmount |	MoneyMo	| Общая сумма начисленных баллов |	Mandatory
loyaltyPrograms	| List<LoyaltyProgramIBMo> | Информация о бонусной программе |	Optional

27.	Структура LoyaltyProgramIBMo

Название параметра |Тип	| Описание/Значение |	Обязательность
--- | --- | ---- | ---
loyaltyProgramId | Integer | Идентификатор бонусной программы	| Mandatory
loyaltyProgramName	| String | Наименование бонусной программы	| Mandatory
totalTransactionAmount |	MoneyMo |Общая сумма транзакций	| Mandatory
percentTariff |	String	| Проценты для отрисовки на ползунке|	Mandatory
loyaltyProgramCondition	| List <LoyaltyProgramConditionIBMo> | Условия процентной ставки по бонусной программе	| Mandatory

28.	Структура LoyaltyProgramConditionIBMo

Название параметра	| Тип	| Описание/Значение| Обязательность
---|---|---|---
counterGreaterThan	| String |Значение суммы в рублях от которой действует указанный %	| Mandatory
percent | String | Действующий процент для указанного интервала	| Mandatory

###Основной сценарий (MC)
29.	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису GetLoyaltyInfo.
30.	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие прото-колу (Неуспешная проверка: EC-BadRequest[1])
31.	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя преду-сматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])
32.	[Шаг4] myGEMINI Core что в рамках сессии еще не выполнялся запрос на по-лучение информации по бонусному счету или в запросе получен параметр request/forceRequest == 'true'. (AC-1)
33.	[Шаг5] myGEMINI Core формирует и отправляет GET запрос к мето-ду getLoyaltyContract МкС LoyaltyContract для получения информации о бонус-ном счете.

Элемент запроса|	Значение
---|---
clientId	|Идентификатор клиента в АБС CIF
_exclude	|preliminaryCalculations,agreements

34.	[Шаг6] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.
35.	[Шаг7] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract мик-росервиса LoyaltyContract содержит http status 200. (EC-Exception [1])

    35.1.	HTTP status 404: (EC-1)
36.	[Шаг8] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная проверка: EC-2)
37.	[Шаг9] myGEMINI Core сохраняет описание бонусного счета в БД в соответ-ствии с маппингом [4].
38.	[Шаг10] myGEMINI Core проверяет, что в IMDG найден бонусный тариф с идентификатором == GetLoyaltyContractResponse/programId, для которого GetLoyaltyContractResponse/programs/summTransaction != 'null' (Неуспешная проверка: AC-2)
39.	[Шаг11] myGEMINI Core рассчитывает параметры бонусного тарифа:

    39.1.	Список GetLoyaltyTariffResponse/conditions сортируется по возраста-нию значения GetLoyaltyTariffResponse/condition/percent
    39.2.	Выполняется сравнение значения GetLoyaltyContractResponse/summTransaction со значением GetLoyaltyTariffResponse/condition/counterGreaterThan
    * Для сравнения берется значение counterGreaterThan по убыванию
    * Если summTransaction больше counterGreaterThan, сравнение продол-жается.
        39.2.1.	В случае если summTransaction  больше всех экземпляров counterGreaterThan, сравнение прекращается, в качестве percentTariff выбирается максимоатльное из counterGreaterThan
    * Если summTransaction меньше counterGreaterThan, то дальнейшее сравнение прекращается.
    39.3.	Значение GetLoyaltyTariffResponse/condition/percent, ассоциирован-ное по вычисленному значению counterGreaterThan, сохраняется как значение percentTariff
40.	[Шаг12] myGEMINI Core обновляет описание бонусного счета в БД в соответ-ствии с маппингом [4]
41.	[Шаг13] myGEMINI Core получает сохраненное описание бонусного счета из БД.
42.	[Шаг14] myGEMINI Core формирует ответ приложению согласно следующим правилам:

Атрибут ответа|	Значение
---|---
loyaltyContractId | Идентификатор бонусного контракта в БД LoyaltyAccount.id
availableLoyaltyAmount | Значение LoyaltyAccount.availableLoyaltyAmount
availableLoyaltyAmount/currency | Значение LoyaltyAccount.availableLoyaltyAmountCurrency
loyaltyPrograms | Список с описанием бонусной программы

loyaltyPrograms:

Атрибут объекта |Значение
---|---
loyaltyProgramId | Значение LoyaltyAccount.loyaltyProgramId
loyaltyProgramName | Значение LoyaltyAccount.loyaltyProgramName
totalTransactionAmount	| Значение LoyaltyProgramTransactions.totalTransactionAmount
totalTransactionAmount/currency	| Значение LoyaltyProgramTransactions.totalTransactionAmountCurrency
percentTariff	| LoyaltyAccount.percentTariff + '%'
loyaltyProgramCondition	| Список значений GetLoyaltyTariffResponse/conditions отсортированный по возрастанию значения percent

loyaltyProgramCondition:

Атрибут объекта | Значение
---|---
counterGreaterThan	| Значение LoyaltyProgramCondition.counterGreaterThan
percent	| Значение LoyaltyProgramCondition.percent

43.	[Шаг15] myGEMINI Core возвращает ответ приложению
44.	Сценарий завершается

###Альтернативные сценарии (AC)

**AC-1  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**
45.	[Шаг1] myGEMINI Core получает сохраненное описание бонусного контракта из БД.
46.	Основной сценарий продолжается с шага 14.

**AC-2  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**
47.	[Шаг1] myGEMINI Core формирует и отправляет параллельные GET запросы к методу getLoyaltyTariff МкС LoyaltyTariff для получения информации о бонус-ном тарифе, по каждой программе.

Элемент запроса|	Значение
---|---
programId|	Идентификатор программы лояльности Значение GetLoyaltyContractResponse/programId

48.	[Шаг2] myGEMINI Core разбирает полученный ответ от мкс LoyaltyTariff.
49.	[Шаг3] myGEMINI Core проверяет, что ответ от метода getLoyaltyTariff микросервиса LoyaltyTariff содержит http status 200.

    49.1.	HTTP status 401 или 400 или 500 или вызов завершен по таймауту: (EC-Exception[1])
50.	[Шаг4] myGEMINI сохраняет данные по тарифу в IMDG, в соответ-ствии с маппингом [4]
51.	Основной сценарий продолжается с шага 12 - расчет параметров бонусного тарифа.

###Исключительные сценарии (EC)

**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**
52.	[Шаг1] myGEMINI Core формирует описание ошибки:

HTTP/1.1 400
```json5
{
"errorResponseMo": {
"errorMsg": "Функциональность временно недоступна",
"errorCode": "UNKNOWN_ERROR",
"errorSubCode": "PRODUCT_NOT_FOUND",
"errorFields": []
    }
}
```
53.	 [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
54.	Сценарий завершается.
       **EC-2. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**
55.	[Шаг1] myGEMINI Core формирует описание ошибки:

HTTP/1.1 400
```json5
{
"errorResponseMo": {
"errorMsg": "Бонусный счет заблокирован",
"errorCode": "BUSINESS_ERROR",
"errorSubCode": "BONUS_BALANCE_BLOCKED",
"errorFields": []
    }
}
```
56.	 [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
57.	Сценарий завершается.
