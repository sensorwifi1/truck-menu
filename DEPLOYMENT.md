# Truck Menu — Wdrożenie na VPS

Instrukcja wdrożenia aplikacji Truck Menu (Food Truck POS) na serwerze VPS z Debian/Ubuntu.

## Wymagania

- VPS z Debian 12 lub Ubuntu 22.04+
- Konto z uprawnieniami `sudo`
- Publiczny adres IP (lub domena)

## 1. Instalacja Dockera

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg

# Klucz GPG Dockera
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Repozytorium Dockera
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalacja
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Dodaj użytkownika do grupy docker
sudo usermod -aG docker $USER
newgrp docker
```

## 2. Klonowanie repozytorium

```bash
cd ~
git clone https://github.com/zjedzit/truck-menu.git
cd truck-menu
```

## 3. Konfiguracja środowiska

```bash
cp .env.example .env
nano .env
```

Uzupełnij zmienne w pliku `.env`:

```env
POSTGRES_USER=truckmenu
POSTGRES_PASSWORD=MojeHaslo2026!
POSTGRES_DB=truckmenu_db
OVH_AI_ENDPOINTS_ACCESS_TOKEN=eyJhbGciOiJFZERTQSIsImtpZCI6IjgzMkFGNUE5ODg3MzFCMDNGM0EzMTRFMDJFRUJFRjBGNDE5MUY0Q0YiLCJraW5kIjoicGF0IiwidHlwIjoiSldUIn0.eyJ0b2tlbiI6IjF6WThGejZnQjUvQ2VPbUt0b3NIZ0o0RnRuNDA3cmJqVkFEZW5lQnhyRGM9In0.ZBYO2_TvLvd-hCxFfMWJPVZJW7xuJSUMcZLGKkvZDB4MeYOLHeyeJGjwU3wbT76FWMQ_fe8_aL9gMVMq72OJAQ
AI_MODEL_NAME=gpt-oss-120b
```

## 4. Uruchomienie

```bash
docker compose up -d --build
```

Sprawdzenie statusu:

```bash
docker compose ps
docker compose logs app -f
```

Aplikacja będzie dostępna na `http://<IP_SERWERA>:8080`.

## 5. Aktualizacja

```bash
cd ~/truck-menu
git pull origin main
docker compose down
docker compose up -d --build
```

## 6. Zarządzanie

```bash
# Restart aplikacji
docker compose restart app

# Logi
docker compose logs app -f
docker compose logs db -f

# Pełny reset (UWAGA: usuwa dane!)
docker compose down -v
docker compose up -d --build
```

## 7. Backup bazy danych

```bash
# Eksport
docker exec truckmenu_db pg_dump -U truckmenu truckmenu_db > backup_$(date +%Y%m%d).sql

# Import
cat backup_20260422.sql | docker exec -i truckmenu_db psql -U truckmenu truckmenu_db
```

## Rozwiązywanie problemów

| Problem                            | Rozwiązanie                                                                              |
| ---------------------------------- | ----------------------------------------------------------------------------------------- |
| `no configuration file provided` | Upewnij się, że jesteś w katalogu `~/truck-menu` (gdzie jest `docker-compose.yml`) |
| `docker: command not found`      | Zainstaluj Dockera (krok 1)                                                               |
| Aplikacja nie startuje             | Sprawdź logi:`docker compose logs app`                                                 |
| Baza nie działa                   | Sprawdź:`docker compose logs db`                                                       |
