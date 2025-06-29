# XRay + VLESS + XKeen на роутере Keenetic

## Другие инструкции

- [XKeen](https://github.com/Corvus-Malus/XKeen)
- [Видео от Антенки](https://www.youtube.com/watch?v=CtiILKiAg5Y)
- [3x-ui от Давайте про IT](https://www.youtube.com/watch?v=zt4oxHeUbdk&t=1225s)
- [X-Ray Core (без web-панели) от ServerTechnologies](https://www.youtube.com/watch?v=PHn5JE9rXgg) + [Инструкция](https://github.com/ServerTechnologies/simple-xray-core)

## Требования

Без чего не получится выполнить инструкции

- Роутер Keenetic с USB разъёмом (проверено на Keenetic Giga (KN-1012))
- USB-стик

## Покупаем сервер

- https://adminvps.ru
- https://ishosting.com
- https://ru.tophosts.net/rating-vps-for-vpn

Покупаем сервер в каких нибудь Нидерландах или Германии. На почту придут креды

## [XRay Core](https://github.com/XTLS/Xray-core)

Для установки и настройки удобно подключиться по `ssh` из `VSCode`

### Зависимости

```bash
apt update
```

```bash
apt install qrencode curl jq -y
```

### XRay Core

Устанавливаем по [инструкции](https://github.com/XTLS/Xray-install?tab=readme-ov-file#basic-usage)

<details>

<summary>Лог установки (на момент написания)</summary>

```bash
# Скачивание файла проверки целостности архива (хеш-сумма для верификации)
Downloading verification file for Xray archive: https://github.com/XTLS/Xray-core/releases/download/v25.6.8/Xray-linux-64.zip.dgst
ok.

# Распаковка скачанного архива во временную папку для подготовки к установке
info: Extract the Xray package to /tmp/tmp.0AbI5WboKE and prepare it for installation.

# Попытка удалить старые конфигурационные файлы сервиса (они отсутствуют - это нормально для первой установки)
rm: cannot remove '/etc/systemd/system/xray.service.d/10-donot_touch_multi_conf.conf': No such file or directory
rm: cannot remove '/etc/systemd/system/xray@.service.d/10-donot_touch_multi_conf.conf': No such file or directory

# Системные файлы для управления Xray как сервисом операционной системы успешно созданы и настроены
# Это означает что теперь можно управлять Xray через команды systemctl (start/stop/restart/enable)
# и он будет автоматически запускаться при загрузке системы
info: Systemd service files have been installed successfully!

# Предупреждение о том, что сейчас будут показаны параметры запуска сервиса
warning: The following are the actual parameters for the xray service startup.

# Напоминание проверить правильность пути к конфигурационному файлу
warning: Please make sure the configuration file path is correctly set.

# Содержимое основного сервисного файла
# /etc/systemd/system/xray.service
[Unit]
Description=Xray Service                                                    # Описание сервиса
Documentation=https://github.com/xtls                                       # Ссылка на документацию
After=network.target nss-lookup.target                                      # Запускать после сети и DNS

[Service]
User=nobody                                                                 # Запуск от пользователя nobody (безопасность)
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE                    # Ограничение прав: сеть + привязка к портам
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE                      # Разрешенные возможности процесса
NoNewPrivileges=true                                                        # Запрет на повышение привилегий
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json   # Команда запуска с явным путем к конфигу
Restart=on-failure                                                          # Перезапуск при сбое
RestartPreventExitStatus=23                                                 # Не перезапускать при коде выхода 23
LimitNPROC=10000                                                            # Лимит количества процессов
LimitNOFILE=1000000                                                         # Лимит открытых файлов

[Install]
WantedBy=multi-user.target                                                  # Автозапуск в многопользовательском режиме

# Дополнительный конфигурационный файл сервиса
# /etc/systemd/system/xray.service.d/10-donot_touch_single_conf.conf

# Комментарий о том, что для кастомизации нужно создать копию файла
# In case you have a good reason to do so, duplicate this file in the same directory and make your customizes there.
# Or all changes you made will be lost!  # Refer: https://www.freedesktop.org/software/systemd/man/systemd.unit.html

[Service]
ExecStart=                                                                  # Сброс предыдущей команды ExecStart
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json   # Переопределение команды запуска

# Список установленных файлов:
installed: /usr/local/bin/xray
installed: /usr/local/share/xray/geoip.dat
installed: /usr/local/share/xray/geosite.dat
installed: /usr/local/etc/xray/config.json
installed: /var/log/xray/
installed: /var/log/xray/access.log
installed: /var/log/xray/error.log
installed: /etc/systemd/system/xray.service
installed: /etc/systemd/system/xray@.service

# Очистка временной папки после установки
removed: /tmp/tmp.0AbI5WboKE

# Подтверждение успешной установки
info: Xray v25.6.8 is installed.

# Напоминание о возможности удаления вспомогательных пакетов
You may need to execute a command to remove dependent software: apt purge curl unzip

# Создание символической ссылки для автозапуска сервиса
Created symlink /etc/systemd/system/multi-user.target.wants/xray.service → /etc/systemd/system/xray.service.

# Включение и запуск сервиса
info: Enable and start the Xray service
```
</details>

После установки можно проверить статус `xray`

```
systemctl status xray
```

### Конфигурация

В файле `/etc/systemd/system/xray.service` лежат параметры запуска `xray`, в частности путь до конфигурационного файла: `/usr/local/etc/xray/config.json`

<details>

<summary>Заменяем содержимое этого файла на такой шаблон</summary>

```json
{
    "log": {
        "loglevel": "warning"
    },
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
            {
                "type": "field",
                "domain": [
                    "geosite:category-ads-all"
                ],
                "outboundTag": "block"
            },
            {
                "type": "field",
                "ip": [
                    "geoip:cn"
                ],
                "outboundTag": "block"
            }
        ]
    },
    "inbounds": [
        {
            "listen": "0.0.0.0",
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "email": "main",
                        "id": "<UUID>",
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false,
                    "dest": "github.com:443",
                    "xver": 0,
                    "serverNames": [
                        "github.com",
                        "www.github.com"
                    ],
                    "privateKey": "<PRIVATE_KEY>",
                    "minClientVer": "",
                    "maxClientVer": "",
                    "maxTimeDiff": 0,
                    "shortIds": [
                        "<SHORTSID>"
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ],
    "policy": {
        "levels": {
            "0": {
                "handshake": 3,
                "connIdle": 180
            }
        }
    }
}
```

</details>

Заполним файл с ключами

```
echo "uuid: $(xray uuid)" >> /usr/local/etc/xray/.keys
```

```
xray x25519 >> /usr/local/etc/xray/.keys
```

```
echo "shortsid: $(openssl rand -hex 8)" >> /usr/local/etc/xray/.keys
```

После чего заменяем `<ПЕРЕМЕННЫЕ>` на значения из файла с ключами `/usr/local/etc/xray/.keys`

### Перезагружаем xray

```bash
systemctl restart xray
```

### Команды управления

Скопируем скрипты для управления

####

<details>

<summary><code>/usr/local/bin/mainuser</code></summary>

```bash
#!/bin/bash
protocol=$(jq -r '.inbounds[0].protocol' /usr/local/etc/xray/config.json)
port=$(jq -r '.inbounds[0].port' /usr/local/etc/xray/config.json)
uuid=$(cat /usr/local/etc/xray/.keys | awk -F': ' '/uuid/ {print $2}')
pbk=$(cat /usr/local/etc/xray/.keys | awk -F': ' '/Public key/ {print $2}')
sid=$(cat /usr/local/etc/xray/.keys | awk -F': ' '/shortsid/ {print $2}')
sni=$(jq -r '.inbounds[0].streamSettings.realitySettings.serverNames[0]' /usr/local/etc/xray/config.json)
ip=$(timeout 3 curl -s icanhazip.com)
link="$protocol://$uuid@$ip:$port?security=reality&sni=$sni&fp=firefox&pbk=$pbk&sid=$sid&spx=/&type=tcp&flow=xtls-rprx-vision&encryption=none#vless-$ip"
echo ""
echo "Link":
echo "$link"
echo ""
echo "QR:"
echo ${link} | qrencode -t ansiutf8
```

</details>

## opkg

Устанавливаем `opkg` по [инструкции](https://help.keenetic.com/hc/ru/articles/360021214160-Установка-системы-пакетов-репозитория-Entware-на-USB-накопитель)

<details>

<summary>Как отформатировать <code>USB</code> в <code>Ext4</code> на <code>MacOS</code></summary>

```bash
# Установить Homebrew, если не установлен
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Установить e2fsprogs
brew install e2fsprogs

# Найти USB устройство
diskutil list

# Размонтировать USB (замените diskX на ваше устройство)
diskutil unmountDisk /dev/diskX

# Форматировать в ext4
sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext4 -L "OPKG" /dev/diskX
```

</details>

После установки можно зайти на роутер по `ssh`

```
ssh root@192.168.1.1
passwor: keenetic
```

### Проблемы

Замеченные проблемы

- При скачаивании инсталятора через Safari, браузер распаковывает архив после скачивания, поэтому надо скачивать через Chrome например и копировать в папку install
- Может не стартовать `ssh`-сервер, тогда [читаем как чинить](https://forum.keenetic.ru/topic/6012-работа-с-entware-при-обновлениях-прошивки-keenetic/). В инструкции старый путь до файла, актуальный путь на момент написания документа такой `/etc/config/dropbear.conf`

## XKeen

Установим на роутер пакетный менеджер `opkg` и пакет `XKeen`

Есть [оригинальный репозиторий](https://github.com/Skrill0/XKeen) и [форк](https://github.com/jameszeroX/XKeen), вроде форк нормальный и можно пользоваться

### Политика

Делаем политику с именем `XKeen` и включаем в ней активное интернет подключение. Вроде как она нужна `XKeen`-у чтобы чо-то там делать

### Установка

Устанавливаем по [инструкции](https://github.com/jameszeroX/XKeen)

### Настройка

После установки в папке `/etc/xray/configs` заменяем файлы 

- `03_inbounds.json`: Вроде можно не заменять
- `04_outbounds.json`: Генерируем [генератором](https://rockblack.pro/xkeen_generator). Копируем туда ссылку из вывод команды `mainuser`
- `05_routing.json`: Генерируем [другим генератором](https://xray-routing-generator.netlify.app) (вроде работает на пункте **VPN кроме РФ**, в этом случае Российские сайты будут открываться напрямую а остальные через **VPN**)

### Перезапускаем XKeen

```bash
xkeen -restart
```

### Применение политики

Добавляем устройства, которым нужен VPN в политику `XKeen`


## Обновление XRay Core

[Инструкция](https://github.com/XTLS/Xray-install?tab=readme-ov-file#basic-usage)

## [DEPRECATED] 3x-ui

Можно вместо `xray core` установить `3x-ui` но там надо заморачиваться с сертификатами, и для простого сценария использования это оверкил

### Установка

Устанавливаем по [инструкции](https://github.com/MHSanaei/3x-ui)

### Настройка подключения

Создаём `inbound` (что бы это ни значило)
- Remark: **Vless** (может быть что угодно, просто имя)
- Protocol: **vless**
- Port: **443** (в инструкции был 443 но вроде работает и с любым другим)
- Security: **Reality**
- uTLS: **ios** (можно попробовать другие, вроде на chrome не работает)
- Client > Flow: **xtls-rprx-vision**
- Нажимаем **Get New Cert**
- Sniffing: **Включен** (вроде можно выключить)

### Сертификат

Так-же можно сгенерировать сертификаты на 10 лет, чтобы панель не ругалась. Можно даже купить доменное имя и выпустить сертификат на него, тогда браузер тоже не будет ругаться

```
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
  -keyout /etc/ssl/self_signed_cert/self_signed.key \
  -out /etc/ssl/self_signed_cert/self_signed.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

После генерации нужно в `Panel Settings > Certificates` настроить
- Public Key Path: `/etc/ssl/self_signed_cert/self_signed.crt`
- Private Key Path: `/etc/ssl/self_signed_cert/self_signed.key`

