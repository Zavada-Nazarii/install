# 📘 Unbound — встановлення та використання локального DNS для сабдомен-рекону

---

## 🧩 Що таке Unbound?

**Unbound** — це повноцінний, рекурсивний DNS-релевант, який дозволяє:
- напряму звертатись до root, TLD та авторитативних DNS
- уникати throttling/block від Google DNS, Cloudflare DNS тощо
- кешувати відповіді та працювати автономно

> Він є ідеальним для масових DNS-резолвів у сабдомен-реконі

---

## 🔍 Для чого використовувати локальний DNS замість публічних резолверів?

| Проблема з публічними DNS (1.1.1.1 / 8.8.8.8) | Рішення з локальним DNS (Unbound) |
|----------------------------------------------|-----------------------------------|
| Rate-limiting (після кількох тисяч запитів)  | Відсутній                         |
| Wildcard-результати або редиректи CDN        | Завжди точні authoritative-відповіді |
| Блокування VPS                                | Байпас через 127.0.0.1             |
| Неможливість перевірити NXDOMAIN              | Повна підтримка NXDOMAIN           |
| Трафік видно третім сторонам                  | Повна локалізація                  |

---

## ⚙️ Принцип роботи локального DNS (Unbound)

Unbound працює як повноцінний recursive resolver. Ось послідовність, як він перевіряє домен:

```text
  Ти перевіряєш: sub.example.com

  [1] Звертається до root-сервера (a.root-servers.net):
      → "Хто обслуговує .com?"

  [2] Отримує NS-сервери для .com-зони

  [3] Звертається до NS-серверів .com:
      → "Хто обслуговує example.com?"

  [4] Отримує авторитативні NS-сервери (наприклад, ns1.example.com)

  [5] Звертається до ns1.example.com:
      → "Чи існує sub.example.com?"

  [6] Отримує:
      → A-запис або → NXDOMAIN
```

✅ Уся ця ієрархія виконується **локально**, без сторонніх DNS.

---

## 📦 Встановлення Unbound (Ubuntu/Debian)

```bash
sudo apt update && sudo apt install unbound -y
```

---

## 📁 Завантаження root hints (обов'язково)

```bash
curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
```

---

## 🛠️ Конфігурація `/etc/unbound/unbound.conf`

```conf
server:
  verbosity: 1
  interface: 127.0.0.1
  access-control: 127.0.0.1/8 allow

  # Продуктивність
  num-threads: 4
  msg-cache-size: 100m
  rrset-cache-size: 100m
  cache-min-ttl: 300
  cache-max-ttl: 86400
  prefetch: yes
  prefetch-key: yes

  # IPv6 можна вимкнути, якщо не потрібен
  do-ip6: no

  # Обмеження
  do-not-query-localhost: no

  # Збільшення лімітів
  outgoing-range: 8192
  num-queries-per-thread: 4096
  so-rcvbuf: 4m
  so-sndbuf: 4m

  # Рекурсія напряму до root DNS
  harden-glue: yes
  harden-dnssec-stripped: yes
  unwanted-reply-threshold: 10000000
  module-config: "iterator"
# Root hints (авторитетні root-сервери)
root-hints: "/var/lib/unbound/root.hints"
```

> Перевір конфігурацію
```bash
sudo unbound-checkconf
```

> `module-config: "iterator"` вимикає DNSSEC, який може створювати конфлікти

> Помилки ? шукаємо і дебажимо
```bash
sudo journalctl -u unbound --no-pager | tail -n 50
```

---

## 🚀 Перевірка та запуск

```bash
sudo unbound-checkconf
sudo systemctl restart unbound
sudo systemctl enable unbound
```

---

## 🧪 Перевірка DNS

```bash
dig example.com @127.0.0.1 +short
```

> Має повернути IP, напр.: `93.184.216.34`

---

## 🧰 Використання з puredns

```bash
puredns resolve domains.txt \
  --resolvers <(echo 127.0.0.1) \
  --write verified.txt
```

або через Python-скрипт:
```bash
python3 puredns_verify_recursive.py --local
```

---

## ✅ Висновок

> Локальний DNS через Unbound — це стабільний, точний і швидкий спосіб перевірки мільйонів сабдоменів без залежності від публічних DNS.

Ідеально підходить для:
- recon-автоматизації
- точного видалення wildcard'ів
- безпечної роботи в хмарах

---

Готовий до інтеграції в будь-який recon-пайплайн.

