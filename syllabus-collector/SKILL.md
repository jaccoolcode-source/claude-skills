---
description: Zbiera informacje z internetu o programie nauczania dla polskich szkół (klasy 1-8 szkoła podstawowa, 1-4 liceum) dla danego przedmiotu, buduje konspekt i CLAUDE.md gotowe do użycia z /vitepress-page. Usage: /syllabus-collector "klasa 5 matematyka" or /syllabus-collector "liceum 2 fizyka"
argument-hint: "[klasa i poziom] [przedmiot] — np. '5 matematyka' albo 'liceum 1 chemia'"
allowed-tools: WebSearch, WebFetch, Read, Write, Glob, Bash
---

# Syllabus Collector — Polski Program Nauczania

Twoje zadanie: zebrać informacje o programie nauczania dla polskich szkół, zbudować szczegółowy konspekt oraz plik CLAUDE.md dla projektu VitePress.

Żądanie użytkownika: **$ARGUMENTS**

---

## Krok 1 — Rozpoznaj dane wejściowe

Parsuj $ARGUMENTS i wyodrębnij:
- **Poziom szkoły**: szkoła podstawowa (klasy 1–8) lub liceum/technikum (klasy 1–4)
- **Numer klasy**: liczba podana przez użytkownika
- **Przedmiot**: matematyka, fizyka, chemia, biologia, historia, geografia, język polski, język angielski, informatyka, itp.

Jeśli dane są niejednoznaczne, zapytaj użytkownika przed kontynuowaniem.

Utwórz wewnętrzne zmienne:
- `KLASA` = np. "klasa 5" lub "liceum klasa 2"
- `PRZEDMIOT` = np. "matematyka"
- `POZIOM` = "podstawowa" lub "liceum"
- `SLUG` = slug w snake_case, np. "matematyka-klasa-5" lub "fizyka-liceum-2"

---

## Krok 2 — Przedstaw plan implementacji i poczekaj na akceptację

**Zanim zaczniesz zbierać dane**, wyświetl użytkownikowi plan w następującym formacie i poczekaj na jego zatwierdzenie:

```
## Plan implementacji: {PRZEDMIOT} — {KLASA}

**Faza 1 — Zbieranie danych (WebSearch + WebFetch)**
- [ ] 1a. Oficjalna podstawa programowa MEN (gov.pl, podstawaprogramowa.pl)
- [ ] 1b. Portale edukacyjne (epodreczniki.pl, scholaris.pl, wsip.pl, pistacja.tv)
- [ ] 1c. Spisy treści podręczników (Nowa Era, WSiP, GWO)
- [ ] 1d. Zbiory zadań i karty pracy
- [ ] 1e. Materiały dodatkowe (multimedia, egzaminy)

**Faza 2 — Analiza i synteza**
- [ ] 2a. Wyodrębnij działy i tematy z podstawy programowej
- [ ] 2b. Uzupełnij o szczegóły z podręczników
- [ ] 2c. Zbierz przykładowe zadania (po 2 na dział)
- [ ] 2d. Zidentyfikuj kluczowe umiejętności

**Faza 3 — Budowanie konspektu**
- [ ] 3a. Napisz konspekt Markdown z działami, zadaniami, linkami
- [ ] 3b. Zastosuj ton pedagogiczny dla wrażliwych dzieci

**Faza 4 — Generowanie CLAUDE.md**
- [ ] 4a. Utwórz CLAUDE.md z instrukcjami dla projektu VitePress
- [ ] 4b. Zawrzyj mapę sekcji i stron do wygenerowania
- [ ] 4c. Dodaj wytyczne pedagogiczne jako stałe reguły projektu

**Szacowana liczba stron VitePress:** [N działów → N stron + 1 index]
**Gotowe komendy /vitepress-page:** [lista komend]

Czy zatwierdzasz ten plan? Mogę też dostosować zakres przed rozpoczęciem.
```

Po akceptacji użytkownika przejdź do Kroku 3. Aktualizuj status kroków (`[x]`) w miarę postępu i informuj użytkownika po każdej fazie.

---

## Krok 3 — Przeszukaj polskie zasoby edukacyjne

Przeszukaj **co najmniej 8 różnych źródeł**. Użyj WebSearch z poniższymi zapytaniami, a następnie WebFetch dla najbardziej obiecujących linków. Informuj użytkownika o postępie po każdej podkategorii.

### 3a. Oficjalna podstawa programowa (obowiązkowe)

Wykonaj WebSearch dla:
- `"podstawa programowa" {PRZEDMIOT} {KLASA} site:gov.pl`
- `"podstawa programowa" {PRZEDMIOT} {POZIOM} szkoła filetype:pdf`
- `podstawa programowa {PRZEDMIOT} {KLASA} MEN 2017`

Pobierz treść z:
- `https://podstawaprogramowa.pl/` — strona MEN z oficjalną podstawą
- `https://www.gov.pl/web/edukacja/podstawa-programowa` — oficjalny serwis rządowy

### 3b. Portale edukacyjne (minimum 3)

Wykonaj WebSearch i pobierz treści z:
- `epodreczniki.pl {PRZEDMIOT} {KLASA}` → `https://epodreczniki.pl`
- `{PRZEDMIOT} {KLASA} "program nauczania" site:scholaris.pl`
- `{PRZEDMIOT} {KLASA} materiały wsip.pl`
- `{PRZEDMIOT} {KLASA} "co uczeń powinien" portal edukacyjny`
- `{PRZEDMIOT} {KLASA} lekcje superbelka.pl OR pistacja.tv OR szkolnictwo.pl`

### 3c. Podręczniki i zbiory zadań (minimum 2 źródła)

Wykonaj WebSearch dla:
- `{PRZEDMIOT} {KLASA} "zbiór zadań" "przykładowe zadania" PDF`
- `{PRZEDMIOT} {KLASA} podręcznik "Nowa Era" OR "WSiP" OR "GWO" spis treści`
- `{PRZEDMIOT} {KLASA} karty pracy ćwiczenia PDF`
- `{PRZEDMIOT} {KLASA} zadania przykładowe sprawdzian`

Interesują Cię spisy treści, listy rozdziałów i przykładowe zadania — nie musisz pobierać pełnych podręczników.

### 3d. Dodatkowe źródła (opcjonalne, ale wartościowe)

- `{PRZEDMIOT} {KLASA} "umiejętności ucznia" egzamin ósmoklasisty OR matura` (jeśli dotyczy)
- `matematyka.pl {KLASA}` (dla matematyki)
- `{PRZEDMIOT} {KLASA} youtube.com kanał edukacyjny polecany`
- `{PRZEDMIOT} {KLASA} repetytoria filmy edukacyjne`

---

## Krok 4 — Analiza i synteza

Po zebraniu materiałów poinformuj użytkownika: "Faza 1 zakończona — analizuję zebrane dane..."

Dokonaj analizy:
1. **Wyodrębnij wszystkie tematy i zagadnienia** wymienione w podstawie programowej
2. **Porównaj z tym, co faktycznie pojawia się w podręcznikach** — często podstawa jest lakoniczna, a podręczniki są konkretniejsze
3. **Zbierz przykładowe zadania i ćwiczenia** z każdego działu — minimum 2-3 przykłady na dział
4. **Zidentyfikuj kluczowe umiejętności** jakie uczeń powinien zdobyć (co potrafi zrobić, a nie tylko co wie)
5. **Zanotuj typowe trudności** — gdzie uczniowie najczęściej mają problemy

---

## Krok 5 — Zbuduj konspekt

Poinformuj użytkownika: "Faza 2 zakończona — buduję konspekt..."

Stwórz konspekt w formacie Markdown. Pamiętaj: **konspekt jest przeznaczony dla wrażliwych dzieci, które potrzebują budowania poczucia własnej wartości**. Używaj języka zachęcającego, pozytywnego, który celebruje postęp.

```markdown
# [Przedmiot] — [Klasa] | Konspekt Programu Nauczania

## Informacje o programie

- **Poziom**: [Szkoła podstawowa / Liceum]
- **Klasa**: [numer]
- **Przedmiot**: [nazwa]
- **Podstawa prawna**: [link do podstawy programowej MEN]
- **Główne źródła**: [lista źródeł z linkami]

---

## Cel ogólny

[Jednoakapitowy opis tego, co dziecko będzie umiało po przerobeniu całego materiału — napisany pozytywnie, motywująco, w stylu "Poznasz...", "Będziesz potrafić...", "Odkryjesz..."]

---

## Mapa tematyczna — Czego się nauczysz?

[Wizualna mapa lub lista głównych działów z krótkim opisem każdego — napisana zachęcająco]

---

## Działy i tematy

### Dział 1: [Nazwa działu]

**Co to jest i dlaczego jest ważne:**
[Krótkie, przyjazne wyjaśnienie — powiązanie z codziennym życiem dziecka]

**Tematy:**
- [Temat 1.1]
- [Temat 1.2]
- [Temat 1.3]
...

**Kluczowe umiejętności (co uczeń potrafi po tym dziale):**
- Potrafię... [konkretna umiejętność]
- Rozumiem... [konkretna wiedza]
- Potrafię zastosować... [zastosowanie praktyczne]

**Przykładowe zadania:**

*Zadanie podstawowe (każdy to potrafi!):*
> [Treść zadania — proste, budujące pewność siebie]
> Wskazówka: [jak podejść do zadania]

*Zadanie rozwinięte (super wyzwanie!):*
> [Treść trudniejszego zadania]

**Zasoby edukacyjne:**
- 📖 [Nazwa zasobu](link) — [co tam znajdziesz]
- 🎥 [Nazwa zasobu](link) — [opis]
- ✏️ [Nazwa zasobu](link) — [ćwiczenia]

---

### Dział 2: [Nazwa działu]

[...powtórz strukturę...]

---

## Umiejętności przekrojowe

[Lista umiejętności, które rozwijają się przez cały rok — myślenie logiczne, praca z tekstem, współpraca itp.]

---

## Powiązania z innymi przedmiotami

| Temat | Powiązanie z |
|-------|-------------|
| [Temat] | [Przedmiot, klasa] |

---

## Sprawdziany i egzaminy

[Informacje o tym, co może pojawić się na sprawdzianach, egzaminie ósmoklasisty lub maturze — jeśli dotyczy. Ton: pomocny, nie straszący.]

---

## Słowniczek kluczowych pojęć

| Pojęcie | Definicja przyjazna dla ucznia |
|---------|-------------------------------|
| [Pojęcie] | [Proste wyjaśnienie] |

---

## Źródła i zasoby

### Strony rządowe i oficjalne
- [link](url) — opis

### Portale edukacyjne
- [link](url) — opis

### Zbiory zadań i ćwiczenia
- [link](url) — opis

### Multimedia
- [link](url) — opis

---

## Wskazówki dla nauczyciela / rodzica

[Krótkie sugestie dotyczące podejścia do nauki z wrażliwym dzieckiem — jak celebrować małe sukcesy, jak radzić sobie z trudnościami, jak budować motywację]
```

---

## Krok 6 — Wytyczne dotyczące tonu i podejścia pedagogicznego

**Konspekt jest przeznaczony dla wrażliwych dzieci.** Stosuj konsekwentnie:

### Język budujący pewność siebie
- Używaj "Odkryjesz...", "Nauczysz się...", "Będziesz potrafić..." zamiast "Musisz umieć..."
- Każdy dział zaczynaj od połączenia z codziennym życiem — "Czy wiesz, że matematyka jest w..."
- Unikaj słów: "musisz", "obowiązkowo", "trudne", "skomplikowane"
- Używaj słów: "krok po kroku", "powoli", "to normalne że...", "każdy tak zaczynał"

### Struktura zadań
- Zawsze zacznij od zadania, które każdy może rozwiązać (budowanie sukcesu)
- Stopniuj trudność wyraźnie
- Dodawaj wskazówki do każdego zadania
- Każde zadanie kończ pozytywnym wzmocnieniem

### Komunikaty o błędach i trudnościach
- Normalizuj trudności: "Ten temat bywa trudny — to zupełnie normalne"
- Proponuj strategie: "Jeśli coś nie wychodzi, spróbuj..."
- Celebruj próby, nie tylko sukcesy

---

## Krok 7 — Weryfikacja jakości konspektu

Przed przejściem do Kroku 8 sprawdź:
- [ ] Konspekt pokrywa **wszystkie tematy** z oficjalnej podstawy programowej
- [ ] Każdy dział ma **minimum 2 przykładowe zadania** (podstawowe i rozwinięte)
- [ ] Każdy dział ma **listę konkretnych umiejętności** (co uczeń potrafi, nie tylko wie)
- [ ] Wszystkie linki są **prawdziwe i konkretne** (nie wymyślone — jeśli nie masz pewności, zaznacz jako "do weryfikacji")
- [ ] Ton jest **zachęcający i budujący** przez cały dokument
- [ ] Konspekt zawiera **co najmniej 5 różnych źródeł** z linkami

---

## Krok 8 — Wygeneruj plik CLAUDE.md

Poinformuj użytkownika: "Faza 3 zakończona — generuję CLAUDE.md dla projektu VitePress..."

Utwórz plik `CLAUDE.md` w bieżącym katalogu roboczym. Ten plik będzie instrukcją dla Claude podczas pracy z projektem VitePress opartym na tym konspekcie.

Szablon CLAUDE.md:

```markdown
# CLAUDE.md — [Przedmiot] klasa [X] | Portal Edukacyjny VitePress

## O projekcie

Portal edukacyjny dla uczniów klasy [X] ([szkoła podstawowa/liceum]), przedmiot: **[Przedmiot]**.
Oparty na VitePress. Każda strona to lekcja z teorią, przykładami, zadaniami i quizem.

**Plik konspektu:** `konspekt-{SLUG}.md` — główne źródło treści dla wszystkich stron.

---

## Zasady pedagogiczne (OBOWIĄZKOWE — stosuj w każdej generowanej treści)

### Dla wrażliwych dzieci — zawsze:
- Zaczynaj każdy temat od połączenia z codziennym życiem ucznia
- Pierwsze zadanie w każdej sekcji musi być **bardzo proste** — buduj sukces przed trudnością
- Gradacja: podstawowe → średnie → wyzwanie (nigdy odwrotnie)
- Unikaj słów: "musisz", "obowiązkowo", "trudne", "skomplikowane", "każdy to wie"
- Używaj: "krok po kroku", "to normalne że...", "każdy tak zaczynał", "świetnie!"
- Każde wytłumaczenie błędu zaczynaj od "To zupełnie normalne..." lub "Wielu uczniów..."
- Zawsze proponuj wskazówkę przed odpowiedzią, nigdy nie dawaj odpowiedzi bez drogi do niej

### Styl języka:
- Zwracaj się bezpośrednio do ucznia: "Ty", "Twój", "Wyobraź sobie..."
- Używaj metafor z życia codziennego dziecka (gry, sport, jedzenie, przyjaciele)
- Krótkie zdania. Jeden pomysł na raz.

---

## Mapa sekcji VitePress

Na podstawie konspektu `konspekt-{SLUG}.md` należy wygenerować:

### Sekcja główna: `/{SLUG}/`

| Strona | Plik | Zawartość |
|--------|------|-----------|
| Strona główna | `index.md` | Przegląd sekcji, mapa tematyczna, ścieżka nauki |
[WYPEŁNIJ na podstawie działów z konspektu — jeden wiersz na dział]

### Komendy do wygenerowania:

```bash
# Cała sekcja naraz:
/vitepress-page "new section: [Przedmiot klasa X] with pages: [slug-dzialu-1, slug-dzialu-2, ...]"

# Lub strona po stronie:
/vitepress-page "[Dział 1: Nazwa] dla klasy [X] — [Przedmiot]"
/vitepress-page "[Dział 2: Nazwa] dla klasy [X] — [Przedmiot]"
[...]
```

---

## Konwencje projektu

### Frontmatter każdej strony
Każda strona musi mieć:
```yaml
---
title: [Tytuł działu]
description: [Jedno zdanie co uczeń odkryje]
category: {SLUG}
pageClass: layout-{SLUG}
difficulty: beginner | intermediate | advanced
tags: [{PRZEDMIOT}, klasa-{NR}, ...]
estimatedMinutes: [10-20]
---
```

### Struktura każdej strony lekcji
1. Tytuł + `<DifficultyBadge />`
2. Intro — "W tej lekcji odkryjesz..." (max 2 zdania)
3. Teoria z przykładem z życia
4. Zadanie podstawowe (każdy to potrafi) + wskazówka
5. Teoria rozszerzona (jeśli potrzeba)
6. Zadanie rozwinięte + wskazówka
7. Podsumowanie — "Nauczyłeś się..."
8. `## Quiz` z linkiem

### VitePress kontenery — kiedy używać
- `::: tip` — ciekawostka, skrót, sztuczka
- `::: info` — dodatkowa informacja, powiązanie z życiem
- `::: warning` — częsty błąd uczniów (pisz: "Uwaga — wiele osób tu się myli, ale Ty już wiesz!")
- `::: details Rozwiązanie` — odpowiedź do zadania (zawsze ukryta domyślnie)

---

## Zasoby źródłowe

Linki zebrane podczas tworzenia konspektu — używaj jako źródła przy generowaniu treści:

[WYPEŁNIJ z sekcji "Źródła i zasoby" konspektu]

---

## Checklist przed publikacją strony

- [ ] Pierwsze zadanie jest naprawdę proste (uczeń NIE może się pomylić)
- [ ] Każde zadanie ma wskazówkę i ukryte rozwiązanie w `::: details`
- [ ] Brak słów: "musisz", "trudne", "skomplikowane"
- [ ] Quiz link na końcu
- [ ] DifficultyBadge na początku
- [ ] Frontmatter kompletny
```

Zapisz ten plik jako `CLAUDE.md` w bieżącym katalogu.

---

## Krok 9 — Output końcowy

Wyświetl:
1. Gotowy konspekt w formacie Markdown
2. Potwierdzenie: "Plik `CLAUDE.md` został zapisany."
3. Blok z gotowymi komendami:

```
---
## Gotowe do użycia z /vitepress-page

CLAUDE.md zapisany — Claude będzie teraz stosować zasady pedagogiczne automatycznie.

Aby wygenerować cały portal jedną komendą:
/vitepress-page "new section: [Przedmiot klasa X] with pages: [slug1, slug2, ...]"

Aby generować stronę po stronie:
/vitepress-page "[Dział 1]"
/vitepress-page "[Dział 2]"
[...]
```
