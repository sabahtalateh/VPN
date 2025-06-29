# VLESS + XKeen на роутере Keenetic

## Другие инструкции

- [3x-ui от aeza](https://wiki.aeza.net/razvertyvanie-proksi-protokola-vless-s-pomoshyu-3x-ui)
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

## XRay Core

Устанавливаем по [инструкции](https://github.com/XTLS/Xray-core?tab=readme-ov-file#installation)

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

