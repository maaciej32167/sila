# Lokalny serwer deweloperski

## Uruchomienie

```bash
cd /Users/maciejgierlik/Documents/Aplikacje/sila
python3 -m http.server 8080
```

## Adresy

- **Mac (przeglądarka):** http://localhost:8080
- **Telefon (ta sama sieć Wi-Fi):** http://192.168.5.73:8080

## Uwagi

- Po każdej zmianie w `index.html` wystarczy odświeżyć stronę w przeglądarce (Cmd+R)
- Service Worker nie działa lokalnie (tylko na localhost/HTTPS) — ignoruj błędy SW
- Telefon musi być w tej samej sieci Wi-Fi co Mac
- IP Maca może się zmienić po zmianie sieci — sprawdź: `ipconfig getifaddr en0`

## Deploy na Netlify (produkcja)

> ⚠️ **Deploy tylko na wyraźne polecenie użytkownika.**
> Wszystkie zmiany w kodzie przeprowadzać lokalnie i testować pod `http://localhost:8080`.
> Netlify uruchamiać wyłącznie gdy użytkownik jawnie poprosi o wdrożenie.

```bash
cd /Users/maciejgierlik/Documents/Aplikacje/sila
~/.npm-global/bin/netlify deploy --prod --dir=.
```
