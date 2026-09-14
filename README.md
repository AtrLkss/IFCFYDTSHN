Вот нормальная инструкция под Windows. `rclone` будет читать файлы прямо из Яндекса и отправлять прямо в Nextcloud. На `C:` полная копия не создаётся — используются только небольшие буферы в памяти.

## 1. Установи rclone

Открой PowerShell:

```powershell
winget install Rclone.Rclone
```

Закрой PowerShell, открой заново и проверь:

```powershell
rclone version
```

Это официальный способ установки через Winget. [Инструкция rclone](https://rclone.org/install/).

## 2. Подключи Яндекс.Диск

Запусти:

```powershell
rclone config
```

Далее отвечай:

```text
n
```

Имя подключения:

```text
yandex-src
```

В списке хранилищ найди `Yandex Disk` или введи:

```text
yandex
```

Дальше:

- `client_id` — нажми Enter;
- `client_secret` — нажми Enter;
- `Edit advanced config?` — `n`;
- `Use web browser?` — `y`.

Откроется браузер. Войди в Яндекс и разреши доступ. Затем подтверди сохранение подключения клавишей `y`. Это штатная OAuth-авторизация без передачи пароля Яндекса rclone. [Настройка Яндекс.Диска](https://rclone.org/yandex/).

Проверь:

```powershell
rclone lsd yandex-src:
```

Должен появиться список папок Яндекс.Диска.

## 3. Создай пароль приложения Nextcloud

В веб-интерфейсе Nextcloud:

1. Нажми на аватар.
2. Открой личные настройки.
3. Перейди в раздел «Безопасность».
4. Найди «Устройства и сеансы».
5. Создай пароль приложения с названием `rclone`.
6. Скопируй полученный пароль.

Обычный пароль от Nextcloud лучше не использовать.

## 4. Подключи Nextcloud

Снова:

```powershell
rclone config
```

Создай ещё одно подключение:

```text
n
```

Имя:

```text
nextcloud-dst
```

Тип:

```text
webdav
```

В поле URL вставь адрес WebDAV. Он выглядит так:

```text
https://ТВОЙ-ДОМЕН/remote.php/dav/files/ТВОЙ_ЛОГИН/
```

Точный адрес можно найти в веб-интерфейсе Nextcloud: «Файлы» → «Настройки файлов» в нижней левой части страницы → WebDAV.

Далее:

- vendor: `nextcloud`;
- user: твой логин Nextcloud;
- password: выбери ввод собственного пароля и вставь созданный пароль приложения;
- bearer token: Enter;
- advanced config: `n`;
- сохранить: `y`.

Проверь:

```powershell
rclone lsd nextcloud-dst:
```

Должен появиться список папок Nextcloud.

## 5. Пробный запуск

Сначала ничего не копируем, только смотрим предполагаемые действия:

```powershell
rclone copy yandex-src: nextcloud-dst:"Yandex disk" --dry-run --progress --create-empty-src-dirs --exclude "/.sync/**" --exclude "desktop.ini"
```

В выводе не должно быть `401 Unauthorized`, `403 Forbidden` или других ошибок авторизации.

## 6. Настоящий перенос

Перед запуском поставь настольный клиент Nextcloud на паузу, чтобы он не мешался. Затем:

```powershell
rclone copy yandex-src: nextcloud-dst:"Yandex disk" --progress --stats 10s --transfers 2 --checkers 8 --buffer-size 8M --create-empty-src-dirs --exclude "/.sync/**" --exclude "desktop.ini" --log-file="C:\rclone-yandex-nextcloud.log" --log-level INFO
```

Важно:

- используй `copy`, не `sync` и не `move`;
- окно PowerShell должно оставаться открытым;
- отключи сон компьютера на время переноса;
- если соединение оборвётся, просто запусти эту же команду снова — завершённые файлы будут проверены и пропущены;
- трафик проходит через компьютер, но файлы целиком на `C:` не сохраняются.

После завершения проверь:

```powershell
rclone check yandex-src: nextcloud-dst:"Yandex disk" --one-way --progress --exclude "/.sync/**" --exclude "desktop.ini"
```

Если ошибок `ERROR` и различий нет — возобновляй настольный Nextcloud. Благодаря включённым Virtual Files он должен показать новые данные как облачные файлы, не скачивая всё обратно на системный диск.
