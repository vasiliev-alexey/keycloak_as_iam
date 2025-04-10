# Авторизация доступа с помощью OAuth 2.0

В этом тексте вы получите более глубокое понимание того, как Keycloak позволяет вам авторизовать доступ к REST API и другим сервисам, используя стандарт OAuth 2.0. Мы рассмотрим взаимодействие между приложением и Keycloak для получения токена доступа, который можно использовать для безопасного вызова сервиса.

Мы начнем с запуска тестового приложения, затем используем его для получения токена от Keycloak, чтобы безопасно вызвать REST API. После этого мы углубимся в тему получения согласия пользователя перед предоставлением доступа приложению, а также в ограничение доступа, предоставленного приложению. В конце рассмотрим, как REST API проверяет токен, чтобы определить, следует ли предоставлять доступ.

Вы узнаете о следующих основных моментах:

- Запуск тестового приложения OAuth 2.0.
- Получение токена доступа.
- Требование согласия пользователя.
- Ограничение доступа, предоставляемого токенами доступа.
- Проверка токенов доступа.

## Получение токена доступа

Наиболее распространенный способ получения токена доступа для вызова защищенного REST API — это использование типа гранта Authorization Code (Код авторизации) в OAuth 2.0.

Вкратце, чтобы получить токен доступа, приложение перенаправляет запрос на Keycloak, который аутентифицирует пользователя и, возможно, просит пользователя предоставить доступ приложению или нет, после чего возвращает токен доступа приложению. Приложение может включать этот токен в запросы к REST API, что позволит API проверить, следует ли предоставлять доступ.

Шаги процесса следующие:

1. Пользователь выполняет действие, которое требует отправки запроса к внешнему REST API.
2. Приложение формирует запрос на авторизацию.
3. Запрос на авторизацию отправляется пользовательскому агенту в виде 302 редиректа, инструктируя браузер перенаправиться на конечную точку авторизации, предоставленную Keycloak.
4. Пользовательский агент открывает конечную точку авторизации с параметрами запроса, указанными приложением.
5. Если пользователь еще не аутентифицирован в Keycloak, ему отображается страница входа.
6. Если приложение требует согласия для доступа к REST API, пользователю показывается страница согласия с вопросом, хочет ли он предоставить доступ приложению.
7. Keycloak возвращает код авторизации приложению.
8. Приложение обменивает код авторизации на токен доступа, а также токен обновления.
9. Теперь приложение может использовать токен доступа для вызова REST API.

Давайте попробуем это с помощью тестового приложения OAuth 2.0. Откройте приложение по адресу http://localhost:8000. Сначала загрузите конфигурацию провайдера OAuth 2.0, нажав кнопку "Load OAuth 2.0 Provide Configuration". После этого нажмите на кнопку "2 - Authorization". Вы можете оставить значения client_id и scope без изменений, затем нажмите на кнопку "Send Authorization Request".

Вы будете перенаправлены на страницу входа в Keycloak. Войдите с помощью пользователя, созданного ранее. После входа и перенаправления обратно в тестовое приложение токен доступа будет отображен в разделе Access Token. Поскольку Keycloak использует JWT в качестве формата токена по умолчанию, тестовое приложение может напрямую анализировать и просматривать содержимое токена доступа.

Рассмотрим некоторые из значений внутри токена доступа:

- **aud**: Это список сервисов, которым предназначен данный токен.
- **realm_access**: Это список глобальных ролей, которые предоставляет токен. Он представляет собой объединение ролей, предоставленных пользователю, и ролей, к которым приложение имеет доступ.
- **resource_access**: Это список клиентских ролей, которые предоставляет токен.
- **scope**: Это область действия, включенная в токен доступа.

Теперь, когда тестовое приложение получило токен доступа, попробуем вызвать REST API. Нажмите на кнопку "3 - Invoke Service", затем нажмите "Invoke". Вы должны увидеть ответ "Secret message!".

## Требование согласия пользователя

Когда приложение хочет получить доступ к стороннему сервису от имени пользователя, пользователю обычно задают вопрос, хочет ли он предоставить доступ приложению. Без этого шага пользователь мог бы не знать, какой доступ получает приложение, и если пользователь уже аутентифицирован с сервером авторизации, он может даже не заметить, что приложение получило доступ.

В Keycloak приложения могут быть настроены как требующие согласия, так и не требующие его. Для внешнего приложения всегда следует требовать согласие, но для доверенного приложения администратор может решить не требовать согласия, что фактически означает, что он доверяет приложению и предоставляет доступ от имени пользователей.

Чтобы попробовать это самостоятельно, откройте административную консоль Keycloak и перейдите к клиенту oauth-playground. В настройках входа для клиента включите опцию "Consent required".

После включения этой опции вернитесь в тестовое приложение и получите новый токен, нажав кнопку "2 - Authorization", а затем "Send Authorization Request". Теперь вы должны увидеть экран, где будут перечислены все области (scopes), к которым приложение запрашивает доступ от имени пользователя.

Одна интересная особенность областей заключается в том, что приложение может сперва запросить ограниченный доступ. По мере использования различных функций приложения оно может запрашивать дополнительный доступ по мере необходимости. Это может быть менее пугающим для пользователя, поскольку становится яснее, почему приложение запрашивает этот доступ.

Давайте попробуем создать новую клиентскую область и запросить эту дополнительную область в тестовом приложении.

Вернитесь в административную консоль Keycloak и нажмите "Client scopes" в меню слева. Затем нажмите "Create client scope". Заполните форму следующими значениями:

- Имя: albums
- Отображать на экране согласия: ON
- Текст на экране согласия: Просмотр ваших фотоальбомов

Создайте клиентскую область, затем снова перейдите к клиенту oauth-playground, нажмите "Client scopes". Добавьте созданную клиентскую область и установите ее как Optional.

Теперь вернитесь в тестовое приложение, нажмите "2 - Authorization". В поле scope введите albums, затем нажмите "Send Authorization Request". На этот раз вас должны попросить предоставить доступ для просмотра фотоальбомов.

Keycloak запоминает согласие, которое пользователь дал конкретному приложению, что означает, что в следующий раз, когда приложение запросит ту же область, пользователь не будет снова запрашиваться.

Через консоль учетной записи пользователь может удалить доступ для приложения, если пожелает.

## Ограничение доступа, предоставленного токенам доступа

Поскольку токены доступа передаются от приложения к сервисам, важно ограничить предоставленный доступ. В противном случае любой токен доступа потенциально может использоваться для доступа к любому ресурсу, к которому пользователь имеет доступ.

Существуют различные стратегии, которые можно использовать для ограничения доступа для конкретного токена доступа:

- Аудитория (audience): Позволяет указывать поставщиков ресурсов, которые должны принимать токен доступа.
- Роли: Контролируя роли, к которым имеет доступ клиент, можно контролировать, какие роли приложение может использовать от имени пользователя.
- Области (scopes): В Keycloak области создаются через клиентские области, и приложение может иметь доступ только к определенному списку областей. Кроме того, когда приложения требуют согласия, пользователь также должен предоставить доступ к области.

### Использование аудитории для ограничения доступа токена

На данный момент токены доступа, выданные для фронтенд-части тестового приложения, не включают бэкенд в аудиторию. Причина, по которой это работает, заключается в том, что бэкенд не был настроен на проверку аудитории в токене.

Давайте начнем с настройки бэкенда для проверки аудитории. Остановите бэкенд, затем откройте файл `keycloak.json` в текстовом редакторе. Измените значение поля verify-token-audience на true.

Одна вещь, которую стоит отметить в этом файле, это поле resource, которое является значением, которое бэкенд будет искать в поле aud, чтобы знать, следует ли принимать токен.

Запустите бэкенд снова. После запуска вернитесь в тестовое приложение и получите новый токен доступа. Если вы посмотрите на значения токена доступа, вы увидите поле aud, и заметите, что oauth-backend не включен.

Если теперь попытаться вызвать сервис через тестовое приложение, вы получите ответ, сообщающий, что доступ запрещен. Бэкенд теперь отклоняет токен доступа.

В Keycloak есть два разных способа включения клиента в аудиторию. Это можно сделать вручную, добавив конкретного клиента в аудиторию с помощью протокольного маппера (добавленного напрямую к клиенту или через клиентскую область), или автоматически, если клиент имеет область на роли другого клиента.

Давайте попробуем добавить аудиторию вручную с помощью протокольного маппера. Создайте нового клиента с Client ID значением oauth-backend.

Теперь вернитесь в список клиентов и откройте клиент oauth-playground. Нажмите "Client scopes", затем выберите клиентскую область oauth-playground-dedicate. Нажмите "Configure a new mapper", затем выберите Audience. Заполните форму следующими значениями:

- Имя: backend audience
- Включенная клиентская аудитория: oauth-backend

Затем выберите "Add to access token" и нажмите "Save". Вернитесь в тестовое приложение и получите новый токен доступа. Теперь oauth-backend включен в поле aud, и если снова попытаться вызвать сервис через тестовое приложение, вы получите успешный ответ.

### Использование ролей для ограничения доступа токена

Keycloak имеет встроенную поддержку ролей, которые могут использоваться для предоставления пользователям прав. Роли также являются очень полезным инструментом для ограничения разрешений приложения, поскольку вы можете настроить, какие роли включаются в токен доступа для данного приложения.

Пользователь имеет отображения ролей на множество ролей, предоставляющих пользователю права, которые дает роль. Клиент, с другой стороны, не имеет ролей, назначенных напрямую, а вместо этого имеет область на набор ролей, которая контролирует, какие роли могут быть включены в токены, отправляемые клиенту.

Это означает, что роли, включенные в токены, представляют собой пересечение ролей, которые есть у пользователя, и ролей, к которым клиент имеет доступ.

Давайте попробуем это в тестовом приложении. Перед внесением изменений получите новый токен доступа и посмотрите на поля aud, realm_access и resource_access. Пример токена доступа с удаленными нерелевантными полями:

```json
{
  "aud": [
    "oauth-backend",
    "account"
  ],
  "realm_access": {
    "roles": [
      "default-roles-myrealm",
      "offline_access",
      "uma_authorization",
      "myrole"
    ]
  },
  "resource_access": {
    "account": {
      "roles": [
        "manage-account",
        "manage-account-links",
        "view-profile"
      ]
    }
  }
}
```

В поле aud вы можете видеть двух клиентов. Клиент oauth-backend включен, поскольку мы явно включили этого клиента в аудиторию в предыдущем разделе. Клиент account, с другой стороны, включен, поскольку токен включает роли для клиента account, что по умолчанию приводит к тому, что клиент автоматически добавляется в аудиторию токена.

Теперь давайте попробуем ограничить область ролей для клиента oauth-playground, чтобы ограничить, что включено в токен. Откройте административную консоль Keycloak и перейдите к клиенту oauth-playground. Нажмите на вкладку "Client scopes", выберите клиентскую область oauth-playground-dedicated, и нажмите на вкладку Scope. Вы заметите, что опция Full Scope Allowed включена. Именно эта функция по умолчанию включает все роли пользователя в токены, отправляемые этому клиенту.

Отключите Full Scope Allowed, затем вернитесь в тестовое приложение и получите новый токен доступа. В новом токене вы заметите, что больше нет никаких ролей, а поле aud теперь включает только клиента oauth-backend. Если теперь попытаться вызвать сервис этим токеном, вы получите сообщение об отказе в доступе. Это связано с тем, что сервис разрешает запросы только с ролью myrole.

Вернитесь в административную консоль Keycloak и снова откройте вкладку Scope для клиентской области oauth-playground-dedicated. Выберите Assign role, выберите роль myrole и нажмите Assign. Вернитесь в тестовое приложение и получите новый токен доступа, и теперь вы увидите, что роль myrole включена в поле realm_access.

### Использование области для ограничения доступа токена

Стандартный механизм в OAuth 2.0 для ограничения разрешений для токена доступа — это использование областей. Использование областей особенно полезно с приложениями третьих сторон, где пользователи должны согласиться на предоставление приложениям доступа к ресурсам от их имени.

В Keycloak область в OAuth 2.0 отображается на клиентскую область. Если вы хотите иметь область, которую приложение может запрашивать, которая затем используется поставщиком ресурсов для предоставления ограниченного доступа к ресурсам, вы можете просто определить пустую клиентскую область, которая не имеет протокольных мапперов и не имеет доступа ни к каким ролям.

При определении областей важно не переусердствовать и ограничить количество областей, а также учитывать, как область представляется конечному пользователю, который должен понимать, что означает предоставление разрешений для этой области, и не путаться при запросе приложением большого количества областей.

Области также должны быть уникальными для всех приложений в вашей организации, поэтому вы можете захотеть добавить префикс с именем службы или даже использовать URL службы в качестве префикса.

Примеры областей:

- albums:view
- albums:create
- albums:delete
- https://api.acme.org/bombs/bombs.purchase
- https://api.acme.org/bombs/bombs.detonate

Нет стандарта для определения областей, поэтому вам нужно определить свои собственные. Полезно посмотреть на области, определенные Google, GitHub, Twitter и другими для вдохновения.

Давайте попробуем это с тестовым приложением, представив, что у нас есть сервис фотоальбомов, предоставляющий доступ для просмотра, создания и удаления альбомов. Предположим, что тестовое приложение предлагает функциональность для просмотра и управления фотоальбомами.

Создайте следующие три клиентские области через административную консоль Keycloak:

- albums:view
- albums:create
- albums:delete

Добавьте область просмотра как область по умолчанию, поскольку мы предполагаем, что тестовое приложение всегда требует просмотра альбомов. С другой стороны, возможность создавать и удалять альбомы сделана опциональной. Это иногда называют поэтапной авторизацией, где приложение запрашивает дополнительные разрешения только тогда, когда пользователь начинает использовать часть приложения, которая требует дополнительных разрешений.

Перед продолжением убедитесь, что клиент oauth-playground требует согласия.

Теперь вернитесь в тестовое приложение и удалите любое значение в поле Scope перед нажатием кнопки Send Authorization Request. Keycloak теперь должен попросить вас предоставить приложению oauth-playground доступ для просмотра фотоальбомов.

После нажатия на Yes токен доступа, отображаемый в тестовом приложении, будет включать albums:view в поле scope. Представим, что пользователь хочет создать новый фотоальбом через тестовое приложение, поэтому оно должно иметь доступ также для создания альбомов. Установите значение поля scope на albums:create и нажмите Send Authorization Request снова. На этот раз вас попросят предоставить доступ для создания фотоальбомов. После нажатия Yes вы увидите, что поле scope в токене доступа теперь включает как albums:view, так и albums:create.

## Проверка токенов доступа

У вас есть два варианта проверки токена доступа: либо вызывая конечную точку интроспекции токенов, предоставленную Keycloak, либо напрямую проверяя токен.

Использование конечной точки интроспекции токенов — это самый простой подход, и он также делает ваши приложения менее зависимыми от Keycloak как сервера авторизации. OAuth 2.0 не определяет стандартный формат для токенов доступа, и они должны считаться непрозрачными для приложения. Вместо этого он определяет стандартную конечную точку интроспекции токенов, которую можно использовать для запроса состояния токена и связанных с ним утверждений. Это также позволяет токенам не быть самодостаточными, то есть не вся соответствующая информация о токене закодирована в самом токене, а токен служит только ссылкой на информацию.

Один из недостатков использования конечной точки интроспекции токенов заключается в том, что это вводит дополнительную задержку при обработке запроса, а также дополнительную нагрузку на сервер авторизации. Общий метод здесь — иметь кэш, который помнит ранее проверенные токены, предотвращая повторную проверку уже проверенных токенов в течение настраиваемого промежутка времени. Время между повторной проверкой токена должно быть довольно коротким, обычно в пределах нескольких минут.

Вы можете попробовать вызвать конечную точку интроспекции токенов с помощью curl или любого другого инструмента, который позволяет отправлять HTTP-запросы.

Сначала нам нужны две вещи: учетные данные для клиента oauth-backend и закодированный токен доступа.

Чтобы получить учетные данные для клиента oauth-backend, перейдите в административную консоль Keycloak и откройте клиент oauth-backend. В разделе Capability config включите аутентификацию клиента и нажмите Save. Теперь в верхней части страницы нажмите на вкладку Credentials и скопируйте значение поля Client secret.

Откройте терминал и установите секрет в переменную окружения:

```bash
$ export SECRET=b1e0073d-3f2b-4ea4-bec0-a35d1983d5b6
```

Оставьте этот терминал открытым, затем откройте тестовое приложение и получите новый токен доступа. Внизу поля Encoded вы увидите закодированный токен доступа. Скопируйте это значение, затем установите переменную окружения в терминале, который вы открыли ранее:

```bash
$ export TOKEN=eyJhbGciOiJSUzI1NiIsInR5c...
```

Теперь вы можете вызвать конечную точку интроспекции токенов, выполнив следующую команду в том же терминале:

```bash
$ curl --data "client_id=oauth-backend&client_secret=$SECRET&token=$TOKEN" \
http://localhost:8080/realms/myrealm/protocol/openid-connect/token/introspect | jq
```

Конечная точка вернет JSON-документ с состоянием токена и связанной информацией.

Другой подход к проверке токенов доступа, выданных Keycloak, — это их прямая проверка в приложении.

Поскольку Keycloak использует JWT в качестве формата токена доступа, это означает, что вы можете анализировать и читать содержимое напрямую из вашего приложения, а также проверять, что токен был выдан Keycloak, поскольку он подписан Keycloak с использованием его закрытых ключей подписи. Одна важная вещь, которую следует отметить об этом подходе, заключается в том, что сессия не проверяется, что означает, что пользователь мог выйти из системы, но приложение все равно будет считать токен действительным.

Все клиентские библиотеки Keycloak проверяют токены напрямую без использования конечной точки интроспекции токенов. Также доступны несколько хороших библиотек для разных языков программирования. Чтобы проверить токен доступа, вам нужно выполнить следующие шаги:

- Получить открытые ключи подписи из JWKS-конечной точки, предоставленной Keycloak.
- Проверить подпись токена доступа.
- Проверить, что токен доступа не истек.
- Проверить издателя, аудиторию и тип токена.
- Проверить любые другие утверждения, которые важны для вашего приложения.