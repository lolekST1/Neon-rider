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

Gra to pojedynczy plik `index.html` w katalogu głównym — to on jest publikowany pod adresem
`https://lolekst1.github.io/Neon-rider/`. Cała logika gry (fizyka, grafika, dźwięk) mieści się
w tym jednym pliku.

## PWA
Gra jest instalowalna i działa offline. Pliki towarzyszące:
- `manifest.webmanifest` — nazwa, ikony, kolory, tryb fullscreen
- `sw.js` — service worker (offline)
- `icons/` — ikony aplikacji

**Ważne przy zmianie strategii cache:** dokument HTML jest pobierany w trybie
*network-first*, żeby nowa wersja była widoczna od razu po deployu. Nie zmieniaj tego na
cache-first, bo gracz zobaczyłby starą wersję. Przy większych zmianach plików
pomocniczych podnieś stałą `CACHE` w `sw.js`.

## Numer wersji
Przy każdej nowej wersji zwiększ stałą `VERSION` w `index.html`. Jest ona wyświetlana
w prawym dolnym rogu ekranu (`v<VERSION>`) — dzięki temu od razu widać, która wersja
jest opublikowana (pomaga potwierdzić, że cache się odświeżył).
