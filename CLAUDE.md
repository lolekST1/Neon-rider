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
`https://lolekst1.github.io/Neon-rider/`.
