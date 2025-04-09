# Интеграция приложений с Keycloak

В этом разделе вы узнаете, как применить основные концепции и параметры конфигурации в Keycloak для настройки ваших приложений и их интеграции с Keycloak.

## Основные темы:

- Выбор архитектуры интеграции
- Выбор варианта интеграции
- Интеграция с приложениями на Golang
- Интеграция с приложениями на Java
- Интеграция с приложениями на JavaScript
- Интеграция с приложениями на Node.js
- Использование обратного прокси
- Не реализовывайте собственную интеграцию

Для запуска сервера выполните следующую команду:

```
$ docker run -it -p 8180:8180 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:22.0.0 \
  start-dev --http-port=8180
```

Создайте новую область "myrealm".

## Конфигурация клиентов

### mybrowserapp
- Client ID: mybrowserapp
- Root URL: http://localhost:8080
- Valid Redirect URI: /*
- Web origins: +

### mywebapp
- Client ID: mywebapp
- Client authentication: ON
- Root URL: http://localhost:8080
- Valid Redirect URI: /*

### mybackend
- Client ID: mybackend
- Client authentication: ON
- Direct Access Grants: ON
- Root URL: http://localhost:8080

### proxy-client
- Client ID: proxy-client
- Client authentication: ON
- Root URL: http://localhost
- Valid Redirect URI: /*

Создайте пользователя:
- Username: alice
- Password: alice

## Выбор архитектуры интеграции

Существует два основных стиля интеграции:
1. Встроенная (Embedded)
2. Через прокси (Proxied)

### Встроенная интеграция
Приложение напрямую взаимодействует с Keycloak и отвечает за обработку запросов и ответов OpenID Connect.

### Интеграция через прокси
Между приложением и Keycloak существует прослойка. Интеграция управляется службой, работающей перед приложением.

## Интеграция с приложениями на Golang

Используйте пакет `go-oidc`:

```go
func createConfig(provider oidc.Provider) (oidc.Config, oauth2.Config) {
    oidcConfig := &oidc.Config{
        ClientID: "mywebapp",
    }
    config := oauth2.Config{
        ClientID:     oidcConfig.ClientID,
        ClientSecret: "CLIENT_SECRET",
        Endpoint:     provider.Endpoint(),
        RedirectURL:  "http://localhost:8080/auth/callback",
        Scopes:       []string{oidc.ScopeOpenID, "profile", "email"},
    }
    return *oidcConfig, config
}
```

## Интеграция с приложениями на Java

### Quarkus
Используйте расширение `quarkus-oidc`:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-oidc</artifactId>
</dependency>
```

Конфигурация для frontend:
```properties
quarkus.oidc.auth-server-url=http://localhost:8180/realms/myrealm
quarkus.oidc.client-id=mywebapp
quarkus.oidc.client-secret=CLIENT_SECRET
quarkus.oidc.application-type=web-app
```

Конфигурация для backend:
```properties
quarkus.oidc.auth-server-url=http://localhost:8180/realms/myrealm
quarkus.oidc.client-id=mybackend
quarkus.oidc.application-type=service
```

### Spring Boot

Конфигурация для frontend (`application.yaml`):
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          myfrontend:
            provider: keycloak
            client-id: mywebapp
            client-secret: CLIENT_SECRET
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/"
            scope: openid
        provider:
          keycloak:
            issuer-uri: http://localhost:8180/realms/myrealm
```

Конфигурация для backend:
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8180/realms/myrealm
```

## Интеграция с приложениями на JavaScript

Используйте Keycloak JavaScript адаптер:

```html
<script type="text/javascript" src="KC_URL/js/keycloak.js"></script>
<script>
const keycloak = new Keycloak();
await keycloak.init({ onLoad: 'login-required' });
</script>
```

## Интеграция с приложениями на Node.js

Используйте пакет `keycloak-connect`:

```javascript
var keycloak = new Keycloak({ store: memoryStore });

app.use(keycloak.middleware());

app.get('/', keycloak.protect(), function(req, res) {
    res.send('Welcome!');
});
```

## Использование обратного прокси

Настройте Apache HTTP Server с модулем `mod_auth_openidc`:

```apache
LoadModule auth_openidc_module modules/mod_auth_openidc.so
<VirtualHost *:80>
    ProxyPass / http://localhost:8000/
    ProxyPassReverse / http://localhost:8000/
    OIDCCryptoPassphrase CHANGE_ME
    OIDCProviderMetadataURL http://localhost:8180/realms/myrealm/.well-known/openid-configuration
    OIDCClientID proxy-client
    OIDCClientSecret CLIENT_SECRET
    OIDCRedirectURI http://localhost/callback
    <Location />
        AuthType openid-connect
        Require valid-user
    </Location>
</VirtualHost>
```

## Рекомендации

Не реализовывайте собственную интеграцию. Используйте проверенные библиотеки и фреймворки.

## Заключение

Используйте известные и установленные открытые стандарты для обеспечения совместимости и безопасности ваших приложений. Избегайте реализации собственных решений, полагайтесь на специализированные и обновляемые реализации.

## Вопросы

1. Какой лучший способ интеграции с Keycloak?
2. Стоит ли всегда рассматривать использование адаптеров Keycloak, если они подходят для вашего технологического стека?
3. Как защитить нативное или мобильное приложение с помощью Keycloak?
4. Какой лучший вариант интеграции для облачных приложений?


---

```
curl    -X 'GET'   'http://localhost:5272/api/TodoItems'    -H 'accept: text/plain' -H "Authorization: Bearer $access_token"
```