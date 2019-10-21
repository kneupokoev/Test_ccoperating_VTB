# Test_ccoperating_VTB

[a relative link](Untitled-2.md)

<html>
<body class="theme-default aui-theme-default">
<iframe src='Untitled-2' style='width:100%;height:100%;border:0px;'></iframe>
Фрейм с указанием формата
<iframe src="Untitled-2.md" style='width:100%;height:100%;border:0px;'></iframe>
Фрейм без указания
<iframe src="Untitled-2.md" style='width:100%;height:100%;border:0px;'></iframe>
Ссылка на файл
<a href="Untitled-2.md"> Скачать Документ </a>
</body>
</html>

{{Untitled-2.md}}
#1.Назначение документа
Данный документ описывает доработки сервера интеграции, мобильного приложения для операционных систем iOS и Android по задаче «BR-16395 Отображение инвестиционного портфеля»

**Задачи в Jira**:

Android: [Android. BR- 16395 Отображение инвестиционного портфеля](https://support.bsc-ideas.com/jira/browse/P20009346-11802)

iOS: [iOS. BR- 16395 Отображение инвестиционного портфеля](https://support.bsc-ideas.com/jira/browse/P20009346-11803)

СИ:[IS. BR-16395 Отображение инвестиционного портфеля](https://support.bsc-ideas.com/jira/browse/P20009346-11801)
#2. Изменение взаимодействия СИ с мобильным приложением 
Название параметра    | Название параметра | Значение по умолчанию|
:-------- |:-----:| -------:
sharedSettings.InvestmentSettingsPortfel.Text | Оценку инвестиции в иностранной валюте проводится по биржевому курсу на текущую дату.   | Дисклеймер на форме "Настройки отображения"
sharedSettings.Investment.SaleText   | Для продажи фондов, акций, облигаций, валюты и других активов воспользуйтесь мобильным приложением ВТБ Мои Инвестиции   | Текст на форме о продаже ценных бумаг через ВТБ Мои инвестиции

#3. Описание объектов - интерфейсных форм
Для сохранения обратной совместимости старых Фронтов и новой Минервы в мобильном приложении необходимо добавить BR16395_InvestmentPortfolio в список [FrontFunctions](https://support.bsc-ideas.com/confl/pages/viewpage.action?pageId=81567337)
##3.1 FRM 3 Форма  "Мои продукты"

