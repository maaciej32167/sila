# CLAUDE.md — Siła App

## Reguły pracy

- **Zmiany testuj lokalnie** — nie wdrażaj na Netlify bez wyraźnego polecenia użytkownika
- **Przed nietrywialną zmianą** — przedstaw plan i zapytaj o zgodę
- **Przy prostych, jednoznacznych poleceniach** — działaj od razu bez pytania
- **Przed deploy** — zawsze sprawdź składnię JS: `node -e "new Function(require('fs').readFileSync('index.html','utf8').match(/<script>([\s\S]*?)<\/script>/)[1])"`

---

## Opis projektu

**Siła** — PWA do śledzenia rekordów siłowych (1RM, objętość, historia treningów).

- Single-file app: cały kod w `index.html` (HTML + CSS + JS)
- Dane przechowywane lokalnie w `localStorage` (brak backendu)
- Netlify: `sila32167.netlify.app`
- Język: polski

---

## Serwer lokalny

```bash
cd /Users/maciejgierlik/Documents/Aplikacje/sila
python3 -m http.server 8080
```

- Mac: `http://localhost:8080`
- Telefon (ta sama sieć Wi-Fi): `http://192.168.5.171:8080`
- Po zmianie w `index.html` — odśwież przeglądarkę (Cmd+R)
- Service Worker nie działa lokalnie — ignoruj błędy SW
- IP Maca może się zmienić: `ipconfig getifaddr en0`

## Deploy na Netlify

> ⚠️ Tylko na wyraźne polecenie użytkownika.

**Automatyczny deploy (GitHub Actions):**
- Każdy `git push origin main` → automatyczny deploy na `sila32167.netlify.app`
- GitHub repo: `https://github.com/maaciej32167/sila`
- Workflow: `.github/workflows/deploy.yml`

```bash
git add -p          # staging zmian
git commit -m "..."
git push            # deploy startuje automatycznie
```

**Ręczny deploy (awaryjny):**
```bash
~/.npm-global/bin/netlify deploy --prod --dir=.
```

---

## Architektura JS (index.html)

### Kluczowe funkcje
| Funkcja | Opis |
|---|---|
| `load()` | Wczytuje dane z localStorage przy starcie |
| `persist()` | Zapisuje dane; przy braku miejsca przycina stare wpisy (chroni PR) |
| `renderAll()` | Renderuje aktualną stronę |
| `setPage(name)` | Przełącza zakładki |
| `addRecord()` | Dodaje nowy rekord treningowy |
| `estimate1RM(weight, reps)` | Wzór Brzycki: `weight × (36 / (37 - reps))` |
| `effectiveWeight(record)` | Rzeczywisty ciężar (BW + dodatkowe dla ćwiczeń BW) |
| `getProtectedIds()` | Zwraca Set z ID top-2 rekordów PR każdego ćwiczenia |
| `deleteRecord(id)` | Usuwa wpis; dla PR wymaga podwójnego kliknięcia |
| `daily1RMSeries()` | Seria danych 1RM w czasie dla wybranego ćwiczenia |

### localStorage — klucze
| Klucz | Zawartość |
|---|---|
| `sila_exercises` | Lista ćwiczeń (JSON array) |
| `sila_records` | Wszystkie rekordy (JSON array) |
| `sila_last_ex` | Ostatnio wybrane ćwiczenie |
| `sila_body_weight_kg` | Aktualna waga ciała |
| `sila_body_weight_history` | Historia pomiarów wagi (JSON array) |
| `sila_bw_exercises` | Ćwiczenia z masą ciała (JSON array) |
| `sila_onboarded` | Flaga pierwszego uruchomienia |

### Ochrona PR
- `getProtectedIds()` — top-2 rekordy per ćwiczenie są nienaruszalne
- `persist()` — przy braku miejsca usuwa najstarsze NIE-PR wpisy
- `deleteRecord()` — wymaga dwukrotnego kliknięcia dla rekordów PR

### Nawigacja (zakładki)
- `main` — dodaj rekord + PR
- `history` → `menu` / `ex` / `all` / `day`
- `manage` — zarządzanie ćwiczeniami
- `info` — waga ciała + wykres wagi
- `backup` — eksport/import JSON

---

## Design System — Obsidian Glass

### Zmienne CSS
```css
--bg:      #080808
--card:    rgba(255,255,255,0.05)   /* glass */
--card2:   rgba(255,255,255,0.03)
--border:  rgba(255,255,255,0.09)
--txt:     #f1f5f9
--muted:   #64748b
--accent:  #818cf8                  /* indigo */
--accent2: #a78bfa                  /* violet */
--danger:  #f87171
--radius:  16px
```

### Efekty
- Karty: `backdrop-filter: blur(20px)`
- Tło: radial-gradient indigo u góry ekranu
- Przycisk główny: `linear-gradient(135deg, #818cf8, #a78bfa)`
- Aktywna ikona nav: `filter: drop-shadow` w kolorze akcentu
- Toast: floating pill z blur, `bottom: 90px` (nad nav barem)

---

## Znane ograniczenia / quirks

- **iOS Safari + position:fixed** — nie używać dla elementów stale widocznych; zamiast tego layout flex na `body` (obecne rozwiązanie)
- **localStorage na iOS** — może milczeć przy błędzie zapisu; `sSet()` zwraca boolean
- **Service Worker** — działa tylko na HTTPS lub localhost; lokalnie ignorowany
- **Wzór 1RM** — Brzycki nie działa dla ≥37 powtórzeń (zwraca `null`)
