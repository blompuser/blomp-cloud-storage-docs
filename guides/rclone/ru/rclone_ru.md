# rclone - вопросы и ответы

<!-- TOC -->

- [rclone - вопросы и ответы](#rclone---%D0%B2%D0%BE%D0%BF%D1%80%D0%BE%D1%81%D1%8B-%D0%B8-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D1%8B)
- [test](#test)
    - [Как установить и настроить rclone для Blomp.com](#%D0%BA%D0%B0%D0%BA-%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%B8%D1%82%D1%8C-%D0%B8-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D1%8C-rclone-%D0%B4%D0%BB%D1%8F-blompcom)
        - [Для пользователей Linux/macOS/BSD](#%D0%B4%D0%BB%D1%8F-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9-linuxmacosbsd)
    - [Как настроить rclone для Blomp.com](#%D0%BA%D0%B0%D0%BA-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D1%8C-rclone-%D0%B4%D0%BB%D1%8F-blompcom)

<!-- /TOC -->

## Как установить и настроить rclone для Blomp.com

Есть [несколько способов как вы можете установить rclone.org](https://rclone.org/downloads/)

- [Скачайте](https://rclone.org/downloads/) и установите по инструкции.

### Для пользователей Linux/macOS/BSD

Чтобы установить rclone в системах Linux / macOS / BSD, запустите:

```shell
curl https://rclone.org/install.sh | sudo bash
```

Для бета-установки запустите:

```shell
curl https://rclone.org/install.sh | sudo bash -s beta
```

Обратите внимание, что этот скрипт сначала проверяет установленную версию rclone и не будет загружать повторно, если в этом нет необходимости.

Если у вас curl не установлен, тогда его можно установить запустив:

```shell
sudo apt update && sudo apt install -y curl
```

## Как настроить rclone для Blomp.com

- Сохраните этот текст поменяв пользователя и пароль на ваши
  ```ini
  [blomp]
  type                  = swift
  env_auth              = false
  user                  = ВАШ@ПОЛЬЗОВАТЕЛЬ
  key                   = ВАШ-ПАРОЛЬ
  auth                  = https://authenticate.blomp.com
  tenant                = storage
  tenant_id             = 8b989f118e624ca6957e102775583f6f
  region                = RegionOne
  auth_version          = 2
  endpoint_type         = public
  leave_parts_on_error  = true
  no_chunk              = false
  ```

  Сохраните как `rclone.conf`:
  - Linux/Mac:  `~/.config/rclone/rclone.conf`
  - Windows:    `%CSIDL_PROFILE%\.config\rclone`
