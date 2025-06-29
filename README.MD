# VLESS + XKeen на роутере Keenetic

## Другие инструкции

- [XKeen](https://github.com/Corvus-Malus/XKeen)
- [Видео от Антенки](https://www.youtube.com/watch?v=CtiILKiAg5Y)
- [3x-ui от Давайте про IT](https://www.youtube.com/watch?v=zt4oxHeUbdk&t=1225s)
- [X-Ray Core (без web-панели)](https://www.youtube.com/watch?v=PHn5JE9rXgg)

## Требования

Без чего не получится выполнить инструкции

- Роутер Keenetic с USB разъёмом (проверено на Keenetic Giga (KN-1012))
- USB-стик

## Покупаем сервер

- https://adminvps.ru
- https://ishosting.com
- https://ru.tophosts.net/rating-vps-for-vpn

Покупаем сервер в каких нибудь Нидерландах или Германии. На почту придут креды

Если нужно редактировать файлы на сервере, удобно подключиться по `ssh` из `VSCode`

## XRay Core

Устанавливаем по [инструкции](https://github.com/XTLS/Xray-core?tab=readme-ov-file#installation)

Лог установки на момент написания

```
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
Description=Xray Service
Documentation=https://github.com/xtls
After=network.target nss-lookup.target
[Service]
User=nobody
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
# /etc/systemd/system/xray.service.d/10-donot_touch_single_conf.conf
# In case you have a good reason to do so, duplicate this file in the same directory and make your customizes there.
# Or all changes you made will be lost!  # Refer: https://www.freedesktop.org/software/systemd/man/systemd.unit.html
[Service]
ExecStart=
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json
installed: /usr/local/bin/xray
installed: /usr/local/share/xray/geoip.dat
installed: /usr/local/share/xray/geosite.dat
installed: /usr/local/etc/xray/config.json
installed: /var/log/xray/
installed: /var/log/xray/access.log
installed: /var/log/xray/error.log
installed: /etc/systemd/system/xray.service
installed: /etc/systemd/system/xray@.service
removed: /tmp/tmp.0AbI5WboKE
info: Xray v25.6.8 is installed.
You may need to execute a command to remove dependent software: apt purge curl unzip
Created symlink /etc/systemd/system/multi-user.target.wants/xray.service → /etc/systemd/system/xray.service.
info: Enable and start the Xray service
```

## 3x-ui

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

## XKeen

Установим на роутер пакетный менеджер `opkg` и пакет `XKeen`

### opkg

Устанавливаем `opkg` по [инструкции](https://help.keenetic.com/hc/ru/articles/360021214160-Установка-системы-пакетов-репозитория-Entware-на-USB-накопитель)

После установк можно зайти на роутер

```
ssh root@192.168.1.1
passwor: keenetic
```

#### Проблемы

Замеченные проблемы

- При скачаивании инсталятора через Safari, браузер распаковывает архив после скачивания, поэтому надо скачивать через Chrome например и копировать в папку install
- Может не стартовать `ssh`-сервер, тогда [читаем как чинить](https://forum.keenetic.ru/topic/6012-работа-с-entware-при-обновлениях-прошивки-keenetic/). В инструкции старый путь до файла, актуальный путь на момент написания документа такой `/etc/config/dropbear.conf`

### XKeen

Есть [оригинальный репозиторий](https://github.com/Skrill0/XKeen) и [форк](https://github.com/jameszeroX/XKeen), вроде форк нормальный и можно пользоваться

#### Политика

Делаем политику с именем `XKeen` и включаем в ней активное интернет подключение. Вроде как она нужна `XKeen`-у чтобы чо-то там делать

#### Установка

Устанавливаем по [инструкции](https://github.com/jameszeroX/XKeen)

После установки в папке `/etc/xray/configs` заменяем файлы `03_inbounds.json`, `04_outbounds.json` и `05_routing.json` по [инструкции](https://github.com/Corvus-Malus/XKeen?tab=readme-ov-file#настройка-xray)

#### Применение политика

Добавляем устройства, которым нужен VPN в политику `XKeen`

