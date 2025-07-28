
# Анонімне сканування та використання проксі через Tor

Цей README описує, як налаштувати Tor + Privoxy, використовувати `proxychains`, `torsocks` та запускати інструменти на зразок `nuclei`, `ffuf`, `curl`, `nmap` через Tor для забезпечення анонімності.

---

## 🔧 Встановлення Tor

### Варіант 1: через офіційний репозиторій Tor Project

```bash
sudo apt update
sudo apt install -y gnupg apt-transport-https curl
curl -fsSL https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | \
  sudo tee /usr/share/keyrings/tor-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] \
  https://deb.torproject.org/torproject.org $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/tor.list

sudo apt update
sudo apt install -y tor deb.torproject.org-keyring
```

### Перевірка

```bash
sudo systemctl status tor
tor --version
```

---

## 🔌 Налаштування Privoxy

```bash
sudo apt install tor privoxy
```

Відредагуйте файл конфігурації:

```bash
sudo vi /etc/privoxy/config
```

Додайте в кінець:

```
forward-socks5t   /   127.0.0.1:9050 .
```

Перезапустіть службу:

```bash
sudo systemctl restart privoxy
```

---

## 🛠️ Встановлення `torsocks` і `proxychains`

```bash
sudo apt install torsocks proxychains
```

---

## ✅ Приклади запуску

### Перевірка IP через Tor

```bash
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org
torsocks curl https://check.torproject.org
curl --proxy http://127.0.0.1:8118 https://check.torproject.org
```

---

### Сканування через Tor

```bash
torsocks nmap -sV -Pn -n <target-ip>
```

---

### nuclei через Tor

```bash
nuclei -u <target-url> -proxy socks5://127.0.0.1:9050
```

---

### ffuf через Privoxy (Tor)

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -x http://127.0.0.1:8118
```

---

### Будова проксі-ланцюга

```
[Tool: curl / ffuf / nuclei / etc] 
           ↓ HTTP
     http://127.0.0.1:8118 (Privoxy)
           ↓ SOCKS5
     socks5://127.0.0.1:9050 (Tor)
           ↓
      Вихідний Tor-вузол
```
### Приклад маршруту через Tor

```
[Твій браузер]
   ↓
[Entry Node (Guard)]          ← бачить твою IP, але не знає, куди йде трафік
   ↓
[Middle Relay]                ← просто передає зашифрований трафік далі
   ↓
[Exit Node]                   ← розшифровує останній шар і відправляє до сайту
   ↓
[Цільовий сайт]               ← бачить IP exit node, а не твій

```
---

## 📦 Запуск через `proxychains`

```bash
proxychains curl https://check.torproject.org
proxychains nmap -sT -Pn <target-ip>
```

> За замовчуванням `proxychains` використовує `/etc/proxychains.conf`, переконайтесь, що він містить:
```
socks5 127.0.0.1 9050
```

---

✅ Це дозволяє маршрутизувати весь трафік інструментів через Tor для забезпечення анонімності при проведенні пентестів.
