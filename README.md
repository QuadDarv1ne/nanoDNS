# nanoDNS для PS4/PS5

**Репозиторий:** https://github.com/drakmor/nanoDNS

**Discord:** https://discord.gg/x2Ppvzwjhm

Минимальный DNS-прокси payload для PS4/PS5, который:

- слушает на настраиваемых локальных IPv4- и IPv6-адресах на порту `53`
- применяет локальные переопределения IPv4 для доменов, соответствующих shell-маскам
- перенаправляет все остальные DNS-запросы к вышестоящим резолверам из `/data/nanodns/nanodns.ini`
- хранит runtime-файлы в `/data/nanodns`
- записывает DNS-запросы и ответы в лог-файл
- может дополнительно дублировать логи в `stdout`/`klog`, когда `debug=1`
- поддерживает блок исключений для обхода локальных переопределений для выбранных доменов
- может привязывать слушающие сокеты к конкретным локальным IPv4- и IPv6-адресам

Вышестоящие резолверы пробуются в порядке, указанном в конфиге. Payload останавливается на первом валидном ответе в пределах настроенного бюджета времени.

## 💜 Поддержка разработки

 Если вы хотите поддержать проект, вы можете сделать пожертвование
 - USDT (TRC-20):  **`TKaUGEwMm9KBXzEoiaaKYBX2yCHAKASW3p`**
 - USDT (ERC-20):  **`0x313dD245dBA957A5560618eA882d08e66aaFb430`**
 - USDC (Solana):  **`5kv7j2RbUGaSP1kU1cZWj9jHH7d6rfvxmK6YXTYbH4um`**

## Сборка

```sh
export PS5_PAYLOAD_SDK=/opt/ps5-payload-sdk
make
```

Сборка для PS4:

```sh
export PS4_PAYLOAD_SDK=/opt/ps4-payload-sdk
make -f Makefile.ps4
```

## Развертывание

```sh
make test PS5_HOST=<ps5-ip-or-hostname>
```

Развертывание на PS4:

```sh
make -f Makefile.ps4 test PS4_HOST=<ps4-ip-or-hostname>
```

## Конфигурация

При запуске payload использует каталог `/data/nanodns`.
Путь к конфигурационному файлу — `/data/nanodns/nanodns.ini`.
Если файл не существует, он создаёт его со значениями по умолчанию:

```ini
[general]
log=/data/nanodns/nanodns.log
debug=0
quiet=0
bind=127.0.0.1
bind6=::1

[upstream]
server=1.1.1.1
server=8.8.8.8
server=77.88.8.8
timeout_ms=1500

[overrides]
*.playstation.com=0.0.0.0
*.playstation.com.*=0.0.0.0
playstation.com=0.0.0.0
*.playstation.net=0.0.0.0
*.playstation.net.*=0.0.0.0
*.psndl.net=0.0.0.0
playstation.net=0.0.0.0
psndl.net=0.0.0.0
# *.example.com=192.168.0.10
# exact.host.local=10.0.0.42

[exceptions]
feature.api.playstation.com
*.stun.playstation.net
stun.*.playstation.net
ena.net.playstation.net
post.net.playstation.net
gst.prod.dl.playstation.net
# auth.api.playstation.net
# *.allowed.playstation.net
```

Маски переопределений используют shell-подобное сопоставление с подстановочными знаками: `*`, `?`, классы в скобках, такие как `[abc]`, диапазоны, такие как `[a-z]`, и отрицательные классы, такие как `[!0-9]`, например:

- `*.example.com`
- `api??.test.local`
- `exact.host.local`

`debug=0` отключает дублирование вывода в консоль и `klog`, но файл, указанный в `log=`, всё равно получает все запросы и ответы. Лог-файл перезаписывается при каждом запуске.

`quiet=1` отключает всплывающие уведомления. По умолчанию `quiet=0`, поэтому уведомления о запуске показываются.

`bind=` задаёт локальный IPv4-адрес, используемый IPv4-слушающим сокетом. По умолчанию `127.0.0.1`.
Используйте `bind=0.0.0.0`, чтобы слушать на всех локальных IPv4-интерфейсах.

`bind6=` задаёт локальный IPv6-адрес, используемый IPv6-слушающим сокетом. По умолчанию `::1`.
Используйте `bind6=::`, чтобы слушать на всех локальных IPv6-интерфейсах.
Используйте `bind6=off`, чтобы полностью отключить IPv6-слушатель.

Записи в `[upstream]` пробуются по порядку. `timeout_ms` — это общий бюджет времени на попытки использования настроенных вышестоящих серверов для одного запроса.

Записи в `[exceptions]` также являются shell-масками, по одной на строку. Если запрос соответствует исключению, он перенаправляется к вышестоящему DNS и обходит все локальные правила переопределения.
