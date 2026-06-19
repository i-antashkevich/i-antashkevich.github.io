# Universal Links для DemoBank через GitHub Pages (бесплатный HTTPS)

Цель: тапаешь `https://i-antashkevich.github.io/open?...` из Telegram → iOS открывает DemoBank
напрямую, без браузера. SDK получает URL через `handleDeeplinkUserActivity:isColdStart:`.

## Содержимое репозитория (корень домена)

```
.nojekyll                                  ← КРИТично: иначе Pages выкидывает .well-known
.well-known/apple-app-site-association     ← AASA (appID = AJUA778725.ru.facct.DemoBank-legacy)
index.html                                 ← страница со ссылкой (скопировать → в Telegram)
open/index.html                            ← web-fallback, если апп не открылся
```

Репозиторий называется `i-antashkevich.github.io`, поэтому сайт живёт в КОРНЕ домена
(`https://i-antashkevich.github.io/`), а AASA доступен по
`https://i-antashkevich.github.io/.well-known/apple-app-site-association` — это и требуется
для Universal Links. Домен публичный с валидным HTTPS, поэтому iOS тянет AASA через CDN Apple:
`?mode=developer` и Developer Mode на устройстве НЕ нужны.

## Деплой

GitHub Pages обслуживает ветку **`main`**. После пуша в `main` сайт обновляется за ~1 мин.

## Проверка, что AASA отдаётся

```bash
curl -i https://i-antashkevich.github.io/.well-known/apple-app-site-association
# JSON с "appIDs": ["AJUA778725.ru.facct.DemoBank-legacy"], Content-Type: application/json
```
Если 404 — не залит `.nojekyll` или Pages ещё не задеплоил.

## Связка с приложением

1. В `DemoBank/DemoBank.entitlements` прописан домен:
   ```xml
   <key>com.apple.developer.associated-domains</key>
   <array>
     <string>applinks:i-antashkevich.github.io</string>
   </array>
   ```
   `appID = <TeamID>.<bundleId> = AJUA778725.ru.facct.DemoBank-legacy` — должен совпадать с AASA.
2. Пересобрать и переустановить DemoBank на устройство (iOS кэширует AASA — после правки AASA
   переустановить апп).

## Тест

1. Открой на телефоне `https://i-antashkevich.github.io/` → кнопка **«Скопировать ссылку»**.
2. Вставь ссылку в Telegram (Saved Messages) и **тапни её ТАМ** (не на самой странице —
   внутри того же домена UL не срабатывает).
3. DemoBank откроется напрямую. `sourceApplication` для Universal Link **отсутствует**
   (источник iOS не сообщает — это ожидаемо).

## Если не открывается
- `curl` AASA вернул не JSON / 404 → `.nojekyll` / Pages ещё деплоит.
- TeamID/bundleId в AASA ≠ реальные (`AJUA778725.ru.facct.DemoBank-legacy`).
- Домен в entitlement ≠ `i-antashkevich.github.io`, или апп не пересобрали.
- Тапнули по ссылке на самой github.io-странице (тот же домен) — надо из Telegram.
- iOS кэширует AASA: переустанови апп.
