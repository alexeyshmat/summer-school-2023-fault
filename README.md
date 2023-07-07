# Практическая работа "Отказоустойчивость"

В рамках практической работы, для развернутого ранее web-сервера Apache2 - будет настроен отказоустойчивый обратный прокси-сервер с использованием nginx и Keepalived. Подобный сценарий может быть применим для публикации локального ресурса в глобальную сеть.

## Выгрузка формул Saltstack

Для выгрузки формул и шаблонов установки необходимо склонировать репозиторий и переместить файлы в директорию `/srv/salt/`:

```bash
git clone https://github.com/alexeyshmat/summer-school-fault-2023.git
```

## Настройка подключения к управляемым хостам

Для настройки подключения к управляемым хостам по SSH необходимо для каждой ноды добавить в файл `/etc/salt/roster` запись следующего вида:

```yaml
<hostname>:
  host: <ip_adress>
  user: <sudouser>
  passwd: '<pass>'
  sudo: True
```

*В качестве hostname для управляемых хостов далее будут использоваться имена вида* `<username-rp01>` и `<username-rp02>`

Для проверки подключения к управляемым хостам можно использовать команду:

```bash
salt-ssh '<username>-rp0[1-2]' -i test.ping
```

## Добавить сопоставление стейта и миньонов

Добавить в `/srv/salt/top.sls`:

```yaml
base:
  '<username>-rp0[1-2]':
    - keepalived
```

## Настройка конфигурационных файлов Keepalived и NGINX

Для настройки конфигурационного файла NGINX открыть файл шаблона конфигурации `nginx.conf.tmpl`.

Для параметра `proxy_pass` указать IP-адрес Apache сервера, установленного ранее.

Для настройки конфигурационного файла Keeaplived открыть файл шаблона конфигурации `keepalived.conf.tmpl`.

Для параметра virtual_ipadress указать VIP, по которому будет отвечать мастер сервер Keepalived:

```j2
      virtual_ipaddress {
         172.26.74.225/24
      }
```

## Запуск установки

Для настройки обратных прокси после настройки конфигурационных файлов запустить команду:

```bash
salt-ssh '*' state.apply
```
