# Gerald

Красивый десктопный VPN-клиент для Windows в духе **Happ** — с реальным подключением через **Xray-core**, импортом подписок и share-ссылок, системным прокси, пингом серверов и живой статистикой.

![Gerald](docs/preview.png)

## Возможности

- **Реальное подключение** через Xray-core: VLESS, VMess, Trojan, Shadowsocks, Hysteria2 (включая Reality, TLS, WS, gRPC, XHTTP, HTTPUpgrade).
- **Подписки** Happ / Remnawave / Marzban / 3x-ui / SyncVPN: Base64-списки, обычные списки ссылок и полные Xray-конфиги (JSON), обновление в один клик, остаток трафика и срок действия.
- **HWID как в Happ**: Gerald генерирует идентификатор устройства и отправляет заголовки `x-hwid`, `x-device-os`, `x-ver-os`, `x-device-model` — провайдеры вроде SyncVPN отдают настоящие серверы только так.
- **Иконки стран**: Gerald берёт флаги из emoji в названии сервера, из `countryCode`/`icon` в конфигах провайдера и показывает изображения флагов (с запасным вариантом в виде цветных инициалов).
- **Ссылки вида `happ://add/…`** и страницы подписки `https://sub.провайдер/ключ` — Gerald сам разворачивает их в список серверов.
- **Импорт share-ссылок** `vless://`, `vmess://`, `trojan://`, `ss://` — по одной или пачкой.
- **Системный прокси Windows** включается и выключается автоматически, исходные настройки пользователя восстанавливаются.
- **Пинг серверов** (TCP) и авто-проверка списка.
- **Статистика** скорости и трафика из Stats API Xray, график активности.
- **Тёмный неоновый интерфейс** с анимированной кнопкой подключения, кастомным заголовком окна, треем и аккуратными состояниями.
- **Демо-профиль** «Прямое подключение» — позволяет проверить работу движка и UI без VPN-сервера.
- Логи Xray в реальном времени, настройки портов, блокировка рекламы, обход ресурсов РФ, автозапуск.

## Скриншоты

| Подключено | Отключено |
| --- | --- |
| ![](docs/preview.png) | ![](docs/preview-disconnected.png) |

> На скриншотах интерфейс с демонстрационными данными.

## Запуск из исходников

```bash
npm install
npm start
```

Требуется Node.js 20+ и Windows 10/11. Xray-core уже лежит в `resources/bin/xray.exe`.

## Готовая сборка

```bash
npm run pack
```

Появится портативная папка `dist/Gerald-win32-x64/` с `Gerald.exe` — устанавливать ничего не нужно, можно запускать напрямую.

Для NSIS-инсталлятора:

```bash
npm i -D electron-builder
npm run dist
```

## Linux

```bash
npm install
npm run pack:linux        # dist/Gerald-linux-x64/ + dist/Gerald-linux-x64.tar.gz
```

В сборку уже входят Xray-core для x64 и arm64 (`resources/bin/linux-x64`, `resources/bin/linux-arm64`). В готовом архиве:

```bash
tar -xzf Gerald-linux-x64.tar.gz
chmod +x Gerald-linux-x64/Gerald Gerald-linux-x64/resources/bin/xray
sudo chown root:root Gerald-linux-x64/chrome-sandbox
sudo chmod 4755 Gerald-linux-x64/chrome-sandbox
./Gerald-linux-x64/Gerald
```

Если `chrome-sandbox` настроить нельзя (например, запуск от пользователя без root), запускайте `./Gerald --no-sandbox`.

Системный прокси на Linux включается через `gsettings` (GNOME/Unity). На остальных окружениях Gerald поднимает локальные SOCKS5/HTTP-порты, а прокси нужно указать вручную. Автозапуск создаёт `~/.config/autostart/gerald.desktop`.

## Android

```bash
cd android
node ../scripts/build-android-assets.js   # renderer → assets/renderer
gradle assembleDebug                      # или ./gradlew assembleDebug
```

Требуется JDK 17+, Android SDK (platform 36, build-tools 36.0.0+) и Gradle 8.11+. Готовые APK появляются в `android/app/build/outputs/apk/debug/`:

- `app-arm64-v8a-debug.apk` — телефоны и планшеты (рекомендуется);
- `app-armeabi-v7a-debug.apk` — старые 32-битные устройства;
- `app-x86_64-debug.apk` — эмуляторы;
- `app-universal-debug.apk` — все ABI сразу (крупный).

Установка:

```bash
adb install -r android/app/build/outputs/apk/debug/app-arm64-v8a-debug.apk
```

Готовые сборки также лежат в `dist/`:

- `dist/Gerald-android-arm64.apk` — телефоны и планшеты;
- `dist/Gerald-android-arm32.apk` — старые 32-битные устройства;
- `dist/Gerald-android-x86_64.apk` — эмуляторы;
- `dist/Gerald-android-universal.apk` — все ABI сразу.

```bash
adb devices                                     # проверить, что телефон виден
adb install -r dist/Gerald-android-arm64.apk    # обновить/установить Gerald
adb shell am start -n su.gerald.client/.MainActivity
```

Если сеть блокирует `dl.google.com` и `maven.google.com`, в `android/settings.gradle` уже прописаны зеркала Tencent и Huawei Cloud. Проект с кириллицей в пути требует `android.overridePathCheck=true` — он уже включён в `android/gradle.properties`.

Внутри — тот же интерфейс Gerald, `libXray.aar` (Xray-core в процессе приложения) и `hev-socks5-tunnel` для tun2socks. При первом подключении Android попросит разрешение на VPN. Приложение исключено из туннеля, чтобы собственный трафик Xray не заворачивался сам в себя. Поддерживаются deep links `happ://`, `incy://`, `v2raytun://`.

## Как добавить серверы

1. Откройте **Серверы → Добавить** или **Подписки → Добавить подписку**.
2. Вставьте ссылку на подписку (например, из личного кабинета провайдера) или share-ссылку. Подойдут обычные `https://…`, ссылки `happ://add/…` и страницы подписки `https://sub.провайдер/ключ`.
3. Выберите сервер и нажмите большую кнопку питания.

Если сервер отвечает «Подписка не найдена» или присылает веб-страницу вместо списка серверов, ссылка устарела. Откройте личный кабинет провайдера или его приложение и скопируйте актуальную ссылку заново — часто это кнопка «Добавить в приложение».

Некоторые провайдеры (например, SyncVPN) отдают серверы только при наличии HWID. Gerald делает это автоматически: при первом запуске создаётся идентификатор устройства, он хранится в `state.json` и отправляется при каждом обновлении подписки. Сброс настроек идентификатор не меняет.

При первом запуске доступен только демо-профиль «Прямое подключение» — он поднимает локальный SOCKS/HTTP-прокси без удалённого сервера. Для настоящего VPN нужна ваша подписка или ссылка на сервер.

## Как это работает

```
Gerald (Electron)
 ├─ Renderer  — интерфейс, анимации, статистика
 ├─ Main      — подписки, состояние, настройки, трей
 ├─ Xray-core — VLESS/VMess/Trojan/SS, Reality/TLS, маршрутизация
 └─ Windows   — системный прокси 127.0.0.1:10809 (HTTP), :10808 (SOCKS5)
```

- Локальные порты по умолчанию: SOCKS5 `10808`, HTTP `10809` (меняются в настройках).
- API статистики Xray слушает `127.0.0.1` на свободном порту.
- Системный прокси включается только при активном подключении и восстанавливается при выходе.
- Данные и `state.json` хранятся в `%APPDATA%\Gerald`.

## Ограничения

- Пока только Windows (системный прокси через WinINET + PowerShell).
- Нет TUN-режима: трафик приложений, которые игнорируют системный прокси, не перехватывается.
- Не поддерживаются TUIC и WireGuard-серверы без соответствующего outbound: Gerald запускает то, что умеет Xray-core. Hysteria2 поддерживается начиная с Xray 26.x.
- Gerald не продаёт и не раздаёт VPN-доступ: приложение работает с вашими собственными серверами и подписками.

## Структура

```
src/
  main/
    main.js          IPC, окно, трей, состояние, подключение
    xray.js          запуск Xray, конфиг, статистика, TCP-пинг
    links.js         парсер vless/vmess/trojan/ss и генератор outbound
    subscription.js  загрузка и разбор подписок
    sysproxy.js      системный прокси Windows
    store.js         JSON-хранилище
  preload.js         безопасный мост contextBridge
  renderer/          HTML/CSS/JS интерфейса
resources/
  bin/               Xray-core для Windows x64
```

## Лицензия

MIT. Xray-core распространяется под собственной лицензией MPL-2.0 — см. `resources/bin/LICENSE`.
