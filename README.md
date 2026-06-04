# Universal Links через GitHub Pages (бесплатный HTTPS для теста)

Цель: тапаешь `https://<username>.github.io/open?...` из Telegram → iOS открывает апп напрямую,
без браузера и без консоли.

## Содержимое этой папки

```
ghpages/
  .nojekyll                                  ← КРИТично: иначе Pages выкидывает .well-known
  .well-known/apple-app-site-association     ← AASA (appID = TeamID.bundleId)
  index.html                                 ← страница со ссылкой (скопировать → в Telegram)
  open/index.html                            ← web-fallback, если апп не открылся
```

## Шаги

### 1. Залить на GitHub Pages
```bash
# создай публичный репозиторий, напр. deeplink-poc, и запушь СОДЕРЖИМОЕ ghpages/ в корень:
cd ghpages
git init && git add -A && git commit -m "ul test"
git branch -M main
git remote add origin https://github.com/<USERNAME>/<REPO>.git
git push -u origin main
```
GitHub → репозиторий → **Settings → Pages** → Source: `Deploy from a branch`, branch `main` / `/root`.
Через ~1 мин сайт живёт на `https://<USERNAME>.github.io/<REPO>/`.

> ⚠️ Если репозиторий называется НЕ `<USERNAME>.github.io`, сайт будет в подпапке
> `https://<USERNAME>.github.io/<REPO>/`, а Universal Links **требуют AASA в КОРНЕ домена**
> (`https://<USERNAME>.github.io/.well-known/...`). Поэтому для теста проще создать репозиторий
> с именем **`<USERNAME>.github.io`** — тогда сайт в корне, и всё совпадает.

### 2. Проверить, что AASA отдаётся
```bash
curl -i https://<USERNAME>.github.io/.well-known/apple-app-site-association
# должен прийти JSON с "appIDs": ["AJUA778725.com.test.poc-deeplink123-321111"]
```
Если 404 — не залит `.nojekyll` или Pages ещё не задеплоил.

### 3. Прописать домен в апп
В `poc-deeplink/poc-deeplink.entitlements` заменить `USERNAME`:
```xml
<string>applinks:<USERNAME>.github.io?mode=developer</string>
```
На iPhone включить **Developer Mode** (Settings → Privacy & Security → Developer Mode).
Пересобрать и переустановить апп на устройство.

### 4. Проверить Team ID в AASA
`appID = <TeamID>.<bundleId>`. Сейчас в файле `AJUA778725.com.test.poc-deeplink123-321111`.
Сверь TeamID (он же App ID Prefix) в Apple Developer → Identifiers. Если отличается — поправь
`.well-known/apple-app-site-association` и перезалей.

### 5. Тест
1. Открой на телефоне `https://<USERNAME>.github.io/` → кнопка **«Скопировать ссылку»**.
2. Вставь ссылку в Telegram (Saved Messages) и **тапни её ТАМ** (не на самой странице —
   внутри того же домена UL не срабатывает).
3. Апп откроется напрямую. В нём: `entrypoint = universal_link`, RAW URL = твоя ссылка,
   `sourceApplication` = **отсутствует** (для UL источник iOS не сообщает — это ожидаемо).

## Если не открывается
- `curl` AASA вернул не JSON / 404 → `.nojekyll` / Pages / путь.
- TeamID в AASA ≠ реальный App ID Prefix.
- Домен в entitlement ≠ домен Pages, или не пересобрали апп.
- Не включён Developer Mode (нужен для `?mode=developer`).
- Тапнули по ссылке на самой github.io-странице (тот же домен) — надо из Telegram.
- iOS кэширует AASA: переустанови апп (`?mode=developer` обходит CDN-кэш).
