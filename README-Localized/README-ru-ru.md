# <a name="word-add-in-javascript-speckit"></a>JavaScript SpecKit для надстроек Word

Узнайте, как создать надстройку, которая считывает и вставляет стандартный текст, и как реализовать простой процесс проверки документов.

## <a name="table-of-contents"></a>Содержание
* [История изменений](#change-history)
* [Необходимые компоненты](#prerequisites)
* [Настройка проекта](#configure-the-project)
* [Запуск проекта](#run-the-project)
* [Разбор кода](#understand-the-code)
* [Вопросы и комментарии](#questions-and-comments)
* [Дополнительные ресурсы](#additional-resources)

## <a name="change-history"></a>История изменений

31 марта 2016 г.:
* Создание исходной версии примера.

## <a name="prerequisites"></a>Необходимые компоненты

* Word 2016 для Windows, сборка 16.0.6727.1000 или более поздней версии.
* [Node и npm](https://nodejs.org/en/)
* [Git Bash](https://git-scm.com/downloads). Используйте последнюю сборку, так как в более ранних сборках может возникать ошибка при создании сертификатов.

## <a name="configure-the-project"></a>Настройка проекта

Выполните приведенные ниже команды из консоли Bash в корне проекта:

1. Клонируйте этот репозиторий на локальный компьютер.
2. ```npm install``` — для установки всех зависимостей в package.json.
3. ```bash gen-cert.sh``` для создания сертификатов, необходимых для запуска этого примера. В репозитории на локальном компьютере дважды щелкните файл ca.crt и выберите элемент **Установить сертификат**. Выберите элемент **Локальный компьютер** и нажмите кнопку **Далее**. Выберите элемент **Поместить все сертификаты в следующее хранилище** и нажмите кнопку **Обзор**.  Выберите **Доверенные корневые центры сертификации** и нажмите кнопку **ОК**. Нажмите кнопки **Далее** и **Готово**. Теперь сертификат из центра сертификации добавлен в хранилище сертификатов.
4. ```npm start``` — для запуска службы.

На этом этапе пример надстройки уже развернут. Теперь необходимо сообщить Microsoft Word, где находится эта надстройка.

1. Создайте сетевую папку или [откройте доступ к папке в сети](https://technet.microsoft.com/en-us/library/cc770880.aspx) и поместите в нее файл манифеста [word-add-in-javascript-speckit-manifest.xml](word-add-in-javascript-speckit-manifest.xml).
3. Запустите Word и откройте документ.
4. Перейдите на вкладку **Файл**, а затем выберите **Параметры**.
5. Выберите **Центр управления безопасностью**, а затем нажмите кнопку **Параметры центра управления безопасностью**.
6. Выберите пункт **Доверенные каталоги надстроек**.
7. В поле **URL-адрес каталога** введите сетевой путь к общей папке, содержащей файл word-add-in-javascript-speckit-manifest.xml, а затем выберите элемент **Добавление каталога**.
8. Установите флажок **Показать в меню** и нажмите кнопку **ОК**.
9. Появится сообщение о том, что параметры будут применены при следующем запуске Microsoft Office. Закройте и перезапустите Word.

## <a name="run-the-project"></a>Запуск проекта

1. Откройте документ Word.
2. На вкладке **Вставка** в Word 2016 выберите **Мои надстройки**.
3. Откройте вкладку **ОБЩАЯ ПАПКА**.
4. Выберите **надстройку Word SpecKit** и нажмите кнопку **ОК**.
5. Если ваша версия Word поддерживает команды надстроек, то в пользовательском интерфейсе отобразятся сведения о том, что надстройка загружена.

### <a name="ribbon-ui"></a>Интерфейс пользователя на основе ленты
На ленте вы можете:
* открыть вкладку **надстройки SpecKit**, чтобы запустить надстройку в пользовательском интерфейсе;
* выбрать элемент **Вставить шаблон спецификации**, чтобы открыть область задач и вставить в документ шаблон спецификации;
* использовать кнопки проверки на ленте или откройте контекстное меню, чтобы сопоставить документ со списком запрещенных слов.

 > Примечание. Если ваша версия Word не поддерживает команды надстроек, то надстройка загрузится в области задач.

### <a name="task-pane-ui"></a>Пользовательский интерфейс области задач
В области задач вы можете:
* Сохранить предложение, наведя на него указатель мыши, указать его имя в поле над кнопкой **Add sentence to boilerplate* (Добавить предложение в стандартный текст) в области задач и нажать кнопку **Add sentence to boilerplate** (Добавить предложение в стандартный текст). То же самое можно сделать для абзацев.
* При сохранении предложений и абзацев стандартный текст также появится в раскрывающемся списке **Insert boilerplate** (Вставить стандартный текст).
* Поместите указатель в документ. Выберите стандартный текст из раскрывающегося списка, и он будет вставлен в документ.
* Настройте свойство *Author* (Автор) документа, изменив имя автора и нажав кнопку **Update author name** (Обновить имя автора). При этом будут обновлены как свойство документа, так и содержимое привязанного элемента управления контентом.

## <a name="understand-the-code"></a>Разбор кода

Во время пробного периода в этом примере используется [набор обязательных компонентов](http://dev.office.com/reference/add-ins/office-add-in-requirement-sets?product=word) версии 1.2, но когда набор обязательных компонентов версии 1.3 поступит в широкий доступ, эта версия станет обязательной.

### <a name="task-pane"></a>Область задач

Функции области задач задаются в файле sample.js. Этот файл предусматривает следующие функции:

* Настройка пользовательского интерфейса и обработчиков событий.
* Получение шаблона спецификации из службы и вставка его в документ.
* Загрузка списка запрещенных слов для проверки документа. В данном примере эти слова считаются неприемлемыми.
* Загрузка стандартного текста по умолчанию из службы и его кэширование в локальном хранилище.
* Скелет кода для тестирования файла функций. Код команды надстройки следует разработать в области задач, прежде чем перемещать его в файл функций, так как в этот файл невозможно вложить отладчик.
* Загрузка имени автора по умолчанию из свойств документа в область задач. Здесь показано, как открыть и изменить пользовательскую часть XML в документе.
* Публикация стандартного текста в службе.

### <a name="document-validation-and-the-dialog-api"></a>Проверка документа и API диалоговых окон

Файл validation.js содержит код для сопоставления документа со списком запрещенных слов. Метод validateContentAgainstBlacklist() использует новый метод splitTextRanges для разделения документа на диапазоны на основе разделителей. Разделители в этой функции определяют слова в документе. Мы определяем совпадения слов в документе и списке запрещенных слов, а затем передаем эти результаты в локальное хранилище. Затем мы используем метод displayDialogAsync, чтобы открыть диалоговое окно (dialog.html). Диалоговое окно получает результаты проверки из локального хранилища и выводит результаты на экран.

### <a name="boilerplate-text-functionality"></a>Функции стандартного текста

Файл boilerplate.js содержит примеры сохранения стандартного текста в локальном хранилище, добавления в раскрывающийся список Fabric пунктов, соответствующих сохраненному стандартному тексту, и вставки выбранного стандартного текста в раскрывающийся список. Этот файл содержит примеры следующих методов:
* splitTextRanges (появился в наборе обязательных компонентов WordApi 1.3). Этот метод будет заменен на split() в будущем выпуске.
* compareLocationWith (появился в наборе требований WordApi 1.3).
* Добавление новых пунктов в раскрывающийся список Fabric.
* Вставка стандартного текста в документ.

### <a name="custom-xml-binding-to-core-document-properties"></a>Привязка пользовательского кода XML к основным свойствам документа.

Файл authorCustomXml.js содержит методы для получения и задания свойств документа по умолчанию.

* Загрузка свойства автора в область задач при ее запуске. Обратите внимание, что документ также содержит значение свойства автора. Это вызвано тем, что шаблон содержит элемент управления контентом, привязанный к свойству документа. Это позволяет задать значения по умолчанию в документе на основе содержимого пользовательской части XML.
* Обновление свойства автора из области задач. При этом будут обновлены свойство документа и элемент управления контентом в документе.

### <a name="add-in-commands"></a>Команды надстроек

Объявления команд надстроек находятся в файле word-add-in-javascript-speckit-manifest.xml. В этом примере показано, как создавать команды надстроек на ленте и в контекстном меню.

## <a name="questions-and-comments"></a>Вопросы и комментарии

Мы будем рады получить от вас отзывы о примере Word SpecKit, подключающемся к Office 365. Вы можете оставлять свои отзывы в разделе *Issues* (Проблемы) этого репозитория.

Общие вопросы о разработке решений для Microsoft Office 365 следует публиковать на сайте [Stack Overflow](http://stackoverflow.com/questions/tagged/office-js+API). Помечайте свои вопросы тегами [office-js] и [API].

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Документация по надстройкам Office](https://msdn.microsoft.com/en-us/library/office/jj220060.aspx)
* [Центр разработки для Office](http://dev.office.com/)
* [Проекты API Office 365 и примеры кода для начинающих](http://msdn.microsoft.com/en-us/office/office365/howto/starter-projects-and-code-samples)

## <a name="copyright"></a>Авторское право
© Корпорация Майкрософт (Microsoft Corporation), 2016. Все права защищены.



Этот проект соответствует [правилам поведения Майкрософт, касающимся обращения с открытым кодом](https://opensource.microsoft.com/codeofconduct/). Дополнительную информацию см. в разделе [часто задаваемых вопросов по правилам поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).
