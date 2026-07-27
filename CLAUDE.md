# Zasady projektu Slider

## Najważniejsza zasada
**Każdą wersję wrzucaj na gałąź `main` i publikuj ją przez GitHub Pages.**

Za każdym razem, gdy powstaje nowa wersja gry:
1. Zmiany trafiają na gałąź `main`.
2. Wersja zostaje automatycznie opublikowana na GitHub Pages.

## Publikacja (GitHub Pages)
GitHub Pages jest ustawione w trybie **„Deploy from a branch"**:
- **Source:** Deploy from a branch
- **Branch:** `main` / katalog `/ (root)`

Dzięki temu każdy push na `main` publikuje się sam — GitHub przebudowuje stronę
automatycznie, bez żadnego workflow. Plik `.nojekyll` wyłącza przetwarzanie Jekyll.

Gra jest dostępna pod adresem **https://lolekst1.github.io/Neon-rider/**

## Numer wersji
Przy każdej nowej wersji zwiększ stałą `VERSION` w `index.html`. Jest ona wyświetlana
w prawym dolnym rogu ekranu (`v<VERSION>`) — dzięki temu od razu widać, która wersja
jest opublikowana (pomaga potwierdzić, że cache się odświeżył).

**Aktualna wersja: 2.2**

## PWA
Gra jest instalowalna i działa offline. Pliki towarzyszące:
- `manifest.webmanifest` — nazwa, ikony, kolory, tryb fullscreen, orientacja pozioma
- `sw.js` — service worker (offline)
- `icons/` — ikony aplikacji (192, 512, maskable 512, apple-touch 180, favicon 32)

**Ważne przy zmianie strategii cache:** dokument HTML jest pobierany w trybie
*network-first*, żeby nowa wersja była widoczna od razu po deployu. Nie zmieniaj tego na
cache-first, bo gracz zobaczyłby starą wersję. Przy większych zmianach plików
pomocniczych podnieś stałą `CACHE` w `sw.js`.

Ikony generowane są skryptem rysującym je na Canvasie w stylu gry (neonowy tor +
ścigacz). Skrypt nie jest w repo — w razie potrzeby napisz go od nowa.

---

# Stan projektu i architektura

## Struktura
Cała gra to **jeden plik `index.html`** (~1900 linii): fizyka, grafika, dźwięk, UI.
Bez bibliotek, bez builda — czysty Canvas 2D + Web Audio API.

Sekcje w pliku (oznaczone komentarzami `/* ---- nazwa ---- */`):
`palettes` → `terrain` → `audio` → `game state` → `helpers` → `lifecycle` →
`input` → `update` → `render` → `utils` → `loop` → `PWA`

## Fizyka (najważniejsza atrakcja gry — traktuj z szacunkiem)
Motocykl to **ciało sztywne 2D z dwoma kołami**, symulowane w **6 podkrokach** na klatkę.
Każde koło koliduje z torem osobno (impulsy w punkcie kontaktu), dzięki czemu lądowanie
na tylnym kole **płynnie** przenosi motor na przednie — bez przeskakiwania kąta.

Kluczowe zasady, których nie łam:
- **Rysowane koła leżą dokładnie na ciałach fizycznych** (`WHEELS`, `WHEEL_R`) — co widać,
  to koliduje.
- **Crash tylko gdy karoseria lub kierowca dotknie linii** (`BODY_PTS`). Koła nigdy nie
  powodują rozbicia — lądowanie na kołach pod dowolnym kątem jest bezpieczne.
- **Pojazd nie rusza sam** — jedzie wyłącznie gdy trzymasz przycisk; po puszczeniu toczy
  się (opór toczenia + opór powietrza), a przy prawie zerowej prędkości stoi (hamulec
  postojowy), nawet na zboczu.
- **Salta lecą do tyłu** (backflip, `p.angle` maleje).
- Puszczenie przycisku w locie stabilizuje obrót i delikatnie ustawia motor do
  nachylenia lądowania (`ALIGN_K`) — wczesne wycofanie ratuje, lądowanie w połowie
  obrotu zabija.

Główne stałe (wszystkie na początku sekcji fizyki): `GRAVITY 1200`, `ENGINE_F 1700`,
`ROLL_F 260`, `DRAG 0.35`, `PARK_SPEED 40`, `MAX_DRIVE_SPEED 780`, `MAX_SPEED 1200`,
`SPIN_RATE 8.0`, `SPIN_EASE 4`, `AIR_DAMP 2.6`, `ALIGN_K 12`, `INV_I 1/420`,
`SUSPENSION 0.75`, `CONTACT_VA_MAX 2.2`, `WHEEL_R 6.4`.

## Zaimplementowane mechaniki
- **Teren proceduralny** — warstwowy szum wartościowy, `terrainY(x)`; trudność rośnie
  z dystansem.
- **Przepaście** (`inGap`, `gapShape`) — linia się urywa co ~2600 px (~45% segmentów).
  Teren wokół dziury jest ukształtowany: rampa przed krawędzią i obniżone lądowanie,
  więc **każda przepaść jest do przeskoczenia z rozpędu**; wolny wjazd = śmierć.
- **Power-upy** (co ~2,4–4,6 tys. px): TURBO, SHIELD (ratuje przed jednym crashem),
  MAGNET, LOW-G. Aktywne efekty jako plakietki z czasem pod wynikiem.
- **Combo** (do ×9) — każdy trik podbija mnożnik, wygasa po 6 s.
- **Obręcze** (hoops) — złote pierścienie w powietrzu, punkty × combo.
- **Kolce** — czerwone, **cyklicznie chowają się** (`obsExt`: ~2 s wysunięte, ~1,1 s
  schowane). Zatrzymanie się przed kolcem nie jest ślepym zaułkiem. Znacznik u podstawy
  zawsze widoczny.
- **Strefy zdarzeń** (`zoneAt`) — STORM (boczny wiatr, błyskawice) i NIGHT (ciemność
  wokół motoru).
- **Duch rekordu** — najlepszy przejazd nagrywany co 60 ms, odtwarzany w kolejnych biegach.
- **Misje dzienne** — 3 losowane z daty, po 15 ◆ za ukończenie, postęp w menu.
- **Portfel + garaż** — diamenty kumulują się; skiny: NEON (darmowy, zmienia kolor
  z paletą poziomu), PINK 60, VENOM 120, ICE 180, GHOST 250, GOLD 400.
  GOLD to **metaliczne wykończenie** (pola `metal`/`edge`/`sparkle` w definicji skina) —
  gradient z wędrującym połyskiem, polerowana krawędź, iskrzący ślad.

## Grafika motoru
`drawBike()` rysuje ścigacza z boku: felgi trójramienne obracające się z prędkością,
pochylony widelec, wahacz, wydech, silnik, klinowa owiewka z zadartym tyłem, szyba,
reflektor, światło stopu, kierowca w pozycji wyścigowej z kaskiem.
`bodyPaint()` zwraca płaski kolor albo gradient metaliczny dla skinów `metal`.

## UI / responsywność
Panele (menu, garaż, game over) centrowane przez `margin:auto` w przewijalnym `.screen`
— **nie używaj `align-items:center`**, bo przy niskim viewportcie obcina treść z obu stron
i nie da się doscrollować. Dla ekranów `max-height: 560px` jest kompaktowy layout
(mniejsza typografia, skiny w jednym rzędzie). HUD respektuje safe-area.

---

# Jak pracować (sprawdzony workflow)

## Testowanie
Chromium jest preinstalowany. Instalacja sterownika i test:
```bash
npm init -y >/dev/null 2>&1 && npm install playwright-core >/dev/null 2>&1
# skrypt testowy MUSI leżeć w katalogu projektu (inaczej nie znajdzie modułu)
node _t.js
rm -rf node_modules package.json package-lock.json _t.js   # zawsze sprzątaj przed commitem
```
Ścieżka do przeglądarki: `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`,
uruchamiaj z `args:['--no-sandbox']`.

Szybki test składni bez przeglądarki:
```bash
node -e "const fs=require('fs');const c=fs.readFileSync('index.html','utf8').match(/<script>([\s\S]*)<\/script>/)[1];require('vm').compileFunction(c,[],{});console.log('JS OK')"
```

W testach można sięgać do stanu gry przez `p.evaluate(()=>({s:state, ...player}))`,
wywoływać `startGame()`, `applyPower('turbo')`, podmieniać `update=function(){}` żeby
zamrozić fizykę do zrzutów ekranu. `state`: 2 = gra, 3 = game over.

**Pomiar FPS:** licz klatki wewnątrz strony (`p.evaluate` z pętlą `requestAnimationFrame`),
nie przez odpytywanie z zewnątrz — to zaniża wynik. W kontenerze bez GPU ~30 FPS to norma,
nie regresja.

## Weryfikacja deployu
Ruch wychodzący do `github.io` jest **zablokowany przez proxy**, więc nie da się pobrać
opublikowanej strony. Zamiast tego sprawdzaj build przez API:
```bash
curl -sS -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/lolekST1/Neon-rider/actions/runs?event=dynamic&per_page=1"
```
Szukaj `"conclusion":"success"` przy właściwym `head_sha`. Build „pages build and
deployment" trwa ~20–40 s. **Uwaga:** zdarzyło się, że build utknął w kolejce na godzinę
i został anulowany przez kolejny push — jeśli zmiana „nie działa u użytkownika",
najpierw sprawdź, czy jej build faktycznie się wdrożył.

Testy PWA (service worker) wymagają HTTP — `python3 -m http.server`. Testuj też
**pod podkatalogiem** `/Neon-rider/`, bo tak serwuje Pages.

## Commity
Nie używaj backticków w komunikacie commita przez `git commit -m` — powłoka je wykona.
Dla dłuższych opisów: `git commit -F - <<'MSG' ... MSG`.

---

# Pomysły na dalej (niezrealizowane)
- Wyróżniki dla pozostałych płatnych skinów (ICE — mroźna poświata, VENOM — pulsujący
  ślad), żeby każdy miał charakter niezależnie od palety poziomu.
- Podwójne kolce / trudniejsze warianty przeszkód przy wyższych wynikach.
- Tabela wyników, więcej typów stref, dźwiękowe warianty muzyki per poziom.
