![](Aspose.Words.766ef271-7980-495f-8c42-92d130a97e26.001.png)




**myGEMINI**

*Лояльность*




![mygemini2.jpg](Aspose.Words.766ef271-7980-495f-8c42-92d130a97e26.005.jpeg)




Тип документа

**Функциональная спецификация**



[TOC]


## **История изменений**
История изменений и согласований документа.

|01.10.2019|0.1|Драфт|Гапченко Н.|
| - | :-: | :- | :- |
|||||
|||||
|||||
|||||
|||||

#
PAGE   \\* MERGEFORMAT9

# **Введение** 
## ***Цель документа***
Целью документа является описание модуля «Лояльность» системы my|GEMINI 
## ***Область применения***
Эта бизнес-область предназначена для операций с бонусами клиента. Клиенты могут просматривать бонусы, детали бонусов.
## ***Глоссарий***
Таблица содержит важные определения, сокращения и аббревиатуры:

|Термин|Описание|
| - | - |
|**АБС**|Автоматизированная банковская система|
|**myInternet, ИБ** Краткое обозначение системы my|GEMINI Internet (канал интернет-банкинга Системы)|
|**my GEMINI,** **Система** |Интегрированная многоканальная фронтальная банковская система my GEMINI|
|**ESB, шина**| IBM WebSphere Enterprise Service Bus|
|**Оператор (Bank Operator)**|Сотрудник Банка, уполномоченный на выполнение тех или иных функций в консоли администрирования Системы|
|**Клиент (Client)**|Клиент банка – зарегистрированное в одной или нескольких соответствующих АБС физическое, юридическое лицо или индивидуальный предприниматель, пользующееся теми или иными финансовыми или дополнительными услугами (продуктами) банка|
|**Канал**|это комплекс организационно-технических средств, процедур и параметров, позволяющих осуществлять прямое обслуживание конечного клиента банка или пользователя банковской автоматизированной системы. Например, канал Интернет банк.|
|**Пользователь (User)**|Физическое лицо, представляющее Клиента банка в том или ином канале Системы. Пользователь может быть либо представителем Клиента (уполномоченный сотрудник юридического лица, доверенное лицо физического лица), либо самим Клиентом (физическим лицом, индивидуальным предпринимателем).|
|**Программа лояльности**|Бонусная программа, позволяющая держателям карт банка накапливать бонусы, которые впоследствии можно использовать, совершая покупки в точках продаж партнёров программы|
|**Бонусы**|Бонусы лояльности, начисленные согласно программе лояльности участнику данной программы|
## ***Ссылки***
Ссылки на другие документы:

|Номер|Описание|
| :- | :- |
||CommonTypes\_89|
||loyalty-document\_1.0.0\_draft|

# **Требования к серверной части**
## ***IB.GetLoyaltyInfo***
1. url: /rest/private/loyalty/info; method: GET
1. Назначение сервиса: получение суммы начисленных бонусов

##**Структура запроса и ответа**

1. Структура запроса: Нет
1. Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo): 

|**Название параметра**|**Тип**|`	`**Описание/Значение**	|**Обязательность**|
| :-: | :-: | :- | :-: |
|loyaltyContractId|Long|Идентификатор бонусного контракта|Mandatory|
|availableLoyaltyAmount|MoneyMo|Общая сумма начисленных баллов|Mandatory|
##**Основной сценарий (MC)**

1. [Шаг1] myGEMINI Core получает и разбирает запрос к сервису IB.GetLoyaltyInfo.
1. [Шаг2] myGEMINI Core проверяет параметры запроса на соответствие протоколу (Неуспешная проверка: EC-BadRequest[\[1\]](#links))
1. [Шаг3] myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[\[1\]](#links))
1. [Шаг4] myGEMINI Core формирует и отправляет GET запрос к методу getLoyaltyContract МкС LoyaltyContract для получения информации о бонусном счете.

|**Элемент запроса**|**Значение**|
| :-: | :-: |
|clientId|Идентификатор клиента в АБС CIF|
|\_exclude|programs,preliminaryCalculations,agreements|
1. [Шаг5] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.
1. [Шаг6] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception[\[1\]](#links))
   1. HTTP status 404: ([EC-1](#GetLoyaltyInfo_EC))
1. [Шаг7] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная проверка: [EC-2](#GetLoyaltyInfo_EC))
1. [Шаг8] myGEMINI Core сохраняет описание бонусного счета в БД в соответствии с [маппингом](#links) [4]
1. [Шаг9] myGEMINI Core получает сохраненное описание бонусного счета из БД.
1. [Шаг10] myGEMINI Core формирует ответ приложению, значения атрибутов которого заполняются согласно следующим правилам:

|**Элемент запроса**|**Значение**|
| :-: | :-: |
|loyaltyContractId|Идентификатор бонусного контракта в БД [LoyaltyAccount.id](#Mapping)|
|availableLoyaltyAmount|Значение LoyaltyAccount.availableLoyaltyAmount|
|availableLoyaltyAmount/currency|Значение LoyaltyAccount.availableLoyaltyAmountCurrency|
1. [Шаг11] myGEMINI Core возвращает ответ приложению.
1. [Шаг12] Сценарий завершается.

##**Исключительные сценарии (EC)**

**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**

1. [Шаг1] myGEMINI Core формирует описание ошибки:

|HTTP/1.1 400|
| :- |
|<p>{</p><p>` `"errorResponseMo": {</p><p>`        `"errorMsg": "Функциональность временно недоступна",</p><p>`        `"errorCode": "UNKNOWN\_ERROR",</p><p>`        `"errorSubCode": "PRODUCT\_NOT\_FOUND",</p><p>`        `"errorFields": []</p><p>` `}</p><p>}</p>|
1. [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
1. Сценарий завершается.

**EC-2. Статус бонусного баланса неактивный GetLoyaltyContractResponse/bonusBalanceStatus == '0'**

1. [Шаг1] myGEMINI Core формирует описание ошибки:

|HTTP/1.1 400|
| :- |
|<p>{</p><p>` `"errorResponseMo": {</p><p>`        `"errorMsg": " Бонусный счет заблокирован",</p><p>`        `"errorCode": "BUSINESS\_ERROR",</p><p>`        `"errorSubCode": "BONUS\_BALANCE\_BLOCKED",</p><p>`        `"errorFields": []</p><p>` `}</p><p>}</p>|
1. [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
1. Сценарий завершается.

## ***IB.GetLoyaltyInfoDetails***
1. url: /rest/private/loyalty/info/details; method: GET
1. Назначение сервиса: сумма начисленных бонусов 

##**Структура запроса и ответа**

1. Структура запроса: 

|**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|forceRequest|Boolean|Признак необходимости обращения в МкС|Optional|
1. Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo)

|` `**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|loyaltyContractId|Long|Идентификатор бонусного контракта|Mandatory|
|availableLoyaltyAmount|MoneyMo|Общая сумма начисленных баллов|Mandatory|
|loyaltyPrograms|List<[LoyaltyProgramIBMo](#LoyaltyProgramMo)>|Информация о бонусной программе|Optional|
1. Структура LoyaltyProgramIBMo

|**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|loyaltyProgramId|Integer|Идентификатор бонусной программы|Mandatory|
|loyaltyProgramName|String|Наименование бонусной программы|Mandatory|
|totalTransactionAmount|MoneyMo|Общая сумма транзакций|Mandatory|
|percentTariff|String|Проценты для отрисовки на ползунке|Mandatory|
|loyaltyProgramCondition|List <[LoyaltyProgramConditionIBMo](#LoyaltyProgramConditionMo)>|Условия процентной ставки по бонусной программе|Mandatory|
1. Структура LoyaltyProgramConditionIBMo

|**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|counterGreaterThan|String|Значение суммы в рублях от которой действует указанный %|Mandatory|
|percent|String|Действующий процент для указанного интервала|Mandatory|
##**Основной сценарий (MC)**

1. [Шаг1] myGEMINI Core получает и разбирает запрос к сервису GetLoyaltyInfo.
1. [Шаг2] myGEMINI Core проверяет параметры запроса на соответствие протоколу (Неуспешная проверка: EC-BadRequest[\[1\]](#links))
1. [Шаг3] myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[\[1\]](#links))
1. [Шаг4] myGEMINI Core что в рамках сессии еще не выполнялся запрос на получение информации по бонусному счету или в запросе получен параметр request/forceRequest == 'true'. ([AC-1](#GetLoyaltyInfoDetails_AC))
1. [Шаг5] myGEMINI Core формирует и отправляет GET запрос к методу getLoyaltyContract** МкС LoyaltyContract** для получения информации о бонусном счете.

|**Элемент запроса**|**Значение**|
| :-: | :-: |
|clientId|Идентификатор клиента в АБС CIF|
|\_exclude|preliminaryCalculations,agreements|
1. [Шаг6] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract**.** 
1. [Шаг7] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception [\[1\]](#links))
   1. HTTP status 404: ([EC-1](#GetLoyaltyInfoDetails_EC))
1. [Шаг8] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная проверка: [EC-2](#GetLoyaltyInfoDetails_EC))
1. [Шаг9] myGEMINI Core сохраняет описание бонусного счета в БД в соответствии с [маппингом](#links) [4].
1. [Шаг10] myGEMINI Core проверяет, что в IMDG найден бонусный тариф с идентификатором == GetLoyaltyContractResponse/programId, для которого GetLoyaltyContractResponse/programs/summTransaction != 'null' (Неуспешная проверка: [AC-2](#GetLoyaltyInfoDetails_AC))
1. [Шаг11] myGEMINI Core рассчитывает параметры бонусного тарифа:
   1. Список GetLoyaltyTariffResponse/conditions сортируется по возрастанию значения GetLoyaltyTariffResponse/condition/percent
   1. Выполняется сравнение значения GetLoyaltyContractResponse/summTransaction со значением GetLoyaltyTariffResponse/condition/counterGreaterThan
      1. Для сравнения берется значение counterGreaterThan по убыванию
      1. Если summTransaction больше counterGreaterThan, сравнение продолжается.
         1. В случае если summTransaction  больше всех экземпляров counterGreaterThan, сравнение прекращается, в качестве percentTariff выбирается максимоатльное из counterGreaterThan
      1. Если summTransaction меньше counterGreaterThan, то дальнейшее сравнение прекращается.
   1. Значение GetLoyaltyTariffResponse/condition/percent, ассоциированное по вычисленному значению counterGreaterThan, сохраняется как значение percentTariff
1. [Шаг12] myGEMINI Core обновляет описание бонусного счета в БД в соответствии с [маппингом](#links) [4]
1. [Шаг13] myGEMINI Core получает сохраненное описание бонусного счета из БД.
1. [Шаг14] myGEMINI Core формирует ответ приложению согласно следующим правилам:

|` `**Атрибут ответа**|**Значение**|
| :-: | :-: |
|loyaltyContractId – Идентификатор бонусного контракта в БД [LoyaltyAccount.id](#Mapping)|
|availableLoyaltyAmount - Значение [LoyaltyAccount.availableLoyaltyAmount](#Mapping)|
|availableLoyaltyAmount/currency - Значение [LoyaltyAccount.availableLoyaltyAmountCurrency](#Mapping)|
|loyaltyPrograms - Список с описанием бонусной программы||

|**Атрибут объекта**|**Значение**|
| :-: | :-: |
|loyaltyProgramId|Значение LoyaltyAccount.loyaltyProgramId|
|loyaltyProgramName|Значение LoyaltyAccount.loyaltyProgramName|
|totalTransactionAmount|Значение LoyaltyProgramTransactions.totalTransactionAmount|
|totalTransactionAmount/currency|Значение LoyaltyProgramTransactions.totalTransactionAmountCurrency|
|percentTariff|LoyaltyAccount.percentTariff + '%'|
|loyaltyProgramCondition|Список значений GetLoyaltyTariffResponse/conditions отсортированный по возрастанию значения percent|
||counterGreaterThan|Значение LoyaltyProgramCondition.counterGreaterThan|
||percent|Значение LoyaltyProgramCondition.percent|


1. [Шаг15] myGEMINI Core возвращает ответ приложению
1. Сценарий завершается

##**Альтернативные сценарии (AC)**

**AC-1  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**

1. [Шаг1] myGEMINI Core получает сохраненное описание бонусного контракта из БД.
1. Основной сценарий продолжается с шага 14.

**AC-2  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**

1. [Шаг1] myGEMINI Core формирует и отправляет параллельные GET запросы к методу getLoyaltyTariff МкС LoyaltyTariff для получения информации о бонусном тарифе, по каждой программе.

|**Элемент запроса**|**Значение**|
| :-: | :-: |
|programId|<p>Идентификатор программы лояльности</p><p>Значение GetLoyaltyContractResponse/programId</p>|
1. [Шаг2] myGEMINI Core разбирает полученный ответ от мкс LoyaltyTariff.
1. [Шаг3] myGEMINI Core проверяет, что ответ от метода getLoyaltyTariff** микросервиса LoyaltyTariff** содержит http status 200. 
   1. HTTP status 401 или 400 или 500 или вызов завершен по таймауту: ([EC-Exception](https://dit.rencredit.ru/confluence/display/MB/CommonError#CommonError-EC-Exception)[\[1\]](#links)) 
1. [Шаг4] myGEMINI сохраняет данные по тарифу в IMDG, в соответствии с [маппингом](#links) [4]
1. Основной сценарий продолжается с шага 12 - расчет параметров бонусного тарифа.

##**Исключительные сценарии (EC)**

**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**

1. [Шаг1] myGEMINI Core формирует описание ошибки:

|HTTP/1.1 400|
| :- |
|<p>{</p><p>` `"errorResponseMo": {</p><p>`        `"errorMsg": "Функциональность временно недоступна",</p><p>`        `"errorCode": "UNKNOWN\_ERROR",</p><p>`        `"errorSubCode": "PRODUCT\_NOT\_FOUND",</p><p>`        `"errorFields": []</p><p>` `}</p><p>}</p>|
1. ` `[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
1. Сценарий завершается.

**EC-2. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**

1. [Шаг1] myGEMINI Core формирует описание ошибки:

|HTTP/1.1 400|
| :- |
|<p>{</p><p>` `"errorResponseMo": {</p><p>`        `"errorMsg": "Бонусный счет заблокирован",</p><p>`        `"errorCode": "BUSINESS\_ERROR",</p><p>`        `"errorSubCode": "BONUS\_BALANCE\_BLOCKED",</p><p>`        `"errorFields": []</p><p>` `}</p><p>}</p>|
1. ` `[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
1. Сценарий завершается.
***Таблица «История операций по бонусному счету»***
 ***IB.GetLoyaltyInfoDetailsOperation***
1. url: /rest/private/loyalty/info/details/operation; method: GET
1. Назначение сервиса: получение операций по бонусной программе 

1. Наименование таблицы - LoyaltyHistory
1. Структура таблицы представлена ниже:

Структура запроса:

|**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|forceRequest|Boolean|Признак необходимости обращения в МкС|Optional|
1. Структура ответа getLoyaltyHistoryResponse (extends AbstactResponseMo)

|` `**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|loyaltyContractId|Long|Идентификатор бонусного контракта|Mandatory|
|availableLoyaltyAmount|MoneyMo|Общая сумма начисленных баллов|Mandatory|
1. Структура getLoyaltyHistory

|**Название параметра**|**Тип**|**Описание/Значение**|**Обязательность**|
| :-: | :-: | :-: | :-: |
|loyaltyProgramId|Integer|Идентификатор бонусной программы|Mandatory|
|loyaltyProgramName|String|Наименование бонусной программы|Mandatory|
|loyaltyoperationDate|Date|Дата совершения операции|Mandatory|
|loyaltydescription|String|Описание операции|Mandatory|
|loyaltyoperationCurrency|String|Валюта операции|Mandatory|
|bloyaltyonusAmount|String|Сумма бонусов|Mandatory|
|loyaltybonusCurrency|String|Валюта бонусов|Mandatory|

##**Основной сценарий (MC)**

1. [Шаг1] myGEMINI Core получает и разбирает запрос к сервису myGEMINI Core получает и разбирает запрос к сервису IB.GetLoyaltyInfoDetailsOperation
1. ` `IB. LoyaltyHistory.
1. [Шаг2] [Шаг 2] myGEMINI Core проверяет параметры запроса на соответствие протоколу (Неуспешная проверка: EC-BadRequest[\[1\]](#links))
1. [Шаг3][Шаг 3]  myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию …какую то…(Неуспешная проверка: EC-Forbidden[\[1\]](#links))
1. [Шаг4][Шаг 4]  myGEMINI Core формирует и отправляет GET запрос к методу getLoyaltyHistory МкС LoyaltyHistory LoyaltyContract для получения информации о б операции с бонусамибонусном счете.
1. [Шаг 5] myGEMINI Core проверяет, что ответ от метода getLoyaltyHistory микросервиса LoyaltyHistory содержит http status 200. (EC-Exception[\[1\]](#links))
   1. HTTP status 404: ([EC-1](#GetLoyaltyInfo_EC))
1. ` `[Шаг6] myGEMINI Core сохраняет описание операций с  бонусами в БД в соответствии с [маппингом](#links) [4], сортируя по дате
1. ` `myGEMINI Core получает сохраненное описаний бонусноых операций из БД.
1. [Шаг 7] myGEMINI Core формирует ответ приложению, значения атрибутов которого заполняются согласно следующим правилам:

|operationDate|Date|Дата совершения операции|
| - | - | - |
|description|String|Описание операции|
|operationAmount|String|Сумма операции|
|operationCurrency|String|Валюта операции|
|bonusAmount|String|Сумма бонусов|
|bonusCurrency|String|Валюта бонусов|
1. [Шаг 8]

` `myGEMINI Core возвращает ответ приложению

|**Атрибут объекта**|**Значение**|
| :-: | :-: |
|loyaltyProgramId|Значение LoyaltyAccount.loyaltyProgramId|
|loyaltyProgramName|Значение LoyaltyAccount.loyaltyProgramName|
|loyaltyoperationDate|Значение  description|
|loyaltydescription|Значение  operationAmount|
|loyaltyoperationCurrency|Значение  operationCurrency|
|bloyaltyonusAmount|Значение  bonusAmount|
|loyaltybonusCurrency|Значение мbonusCurrency|

1. Сценарий завершается

##**Альтернативные сценарии (AC)**

**AC-1  В рамках сессии ранее был выполнен запрос к методу getLoyaltyHistory МкС LoyaltyContract**

1. [Шаг1] myGEMINI Core получает сохраненное описание бонусного контракта из БД.
1. Основной сценарий продолжается с шага 8.

**Исключительные сценарии (EC)**

**EC-2. Не найдено ни одной операции с бонусами GetLoyaltyHistoryResponse/bonus == '0'**

1. [Шаг1] myGEMINI Core формирует описание ошибки:

|HTTP/1.1 400|
| :- |
|<p>{</p><p>` `"errorResponseMo": {</p><p>`        `"errorMsg": " Не найдено ни одной операции с бонусами",</p><p>`        `"errorCode": "BONUS\_ERROR",</p><p>`        `"errorSubCode": "BONUS\_OPERATION\_BLOCKED",</p><p>`        `"errorFields": []</p><p>` `}</p><p>}</p>|
1. [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
1. Сценарий завершается.
