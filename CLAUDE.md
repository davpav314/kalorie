# fit-poradce

Osobní kalorický poradce běžící přímo v Claude Code. Žádná kalorická databáze,
žádné výpočetní skripty — odhad kalorií i doporučení jídelníčku dělá vždy model
sám (Sonnet, max effort) na základě svých znalostí výživy, uložený jen do
jednoduchých JSON souborů.

## Jak to funguje

Uživatel s aplikací komunikuje volně v chatu (česky). Podle obsahu zprávy se
aktivuje jeden ze dvou skillů:

- **kalorie-odhad** – uživatel napíše, co snědl/vypil → model odhadne kalorie
  (a makra) a zapíše položku do deníku.
- **kalorie-doporuceni** – uživatel chce nastavit/aktualizovat profil, vidět
  dnešní přehled, nebo chce poradit, co jíst do konce dne → model spočítá
  kalorický cíl, zkontroluje deník a navrhne konkrétní jídla.

Podrobná pravidla jsou v `.claude/skills/kalorie-odhad/SKILL.md` a
`.claude/skills/kalorie-doporuceni/SKILL.md` — Claude je najde a použije
automaticky, není potřeba je volat ručně.

## Datový model

- `data/profil.json` – osobní údaje uživatele (pohlaví, věk, výška, váha,
  úroveň pohybové aktivity, cíl, dopočtený denní kalorický cíl). Vytváří a
  aktualizuje skill `kalorie-doporuceni`.
- `data/denik.json` – deník zkonzumovaných položek, klíčovaný datem
  (`YYYY-MM-DD`), pro každý den seznam položek (čas, popis, kcal, makra) a
  denní součet.
- Oba soubory jsou **osobní údaje a jsou v `.gitignore`** — nikdy se
  necommitují do repozitáře. V repu jsou jen vzorové soubory
  `data/profil.example.json` a `data/denik.example.json`, které ukazují
  formát.
- `prehled.html` – statický přehled zkonzumovaných kalorií po položkách a po
  dnech, který model po každé změně deníku znovu vygeneruje (čistý
  HTML/CSS výstup s daty, žádná klientská logika). Také je v `.gitignore`
  (obsahuje osobní data), v repu je vzor `prehled.example.html`.

## Pravidla pro model

1. Nikdy nevytvářej samostatný skript/knihovnu s kalorickou databází ani
   vzorcem napevno v kódu mimo skilly — výpočet BMR/TDEE i odhad kalorií jídla
   dělej vždy přímo jako model při zpracování požadavku.
2. Odhady kalorií jsou vždy jen odhad — u nejasného množství uveď předpoklad
   (např. "předpokládám střední porci, ~150 g") přímo u položky, ať to
   uživatel může opravit.
3. Když profil (`data/profil.json`) chybí, při první žádosti o doporučení či
   přehled si vyžádej: pohlaví, věk, výšku (cm), váhu (kg), úroveň pohybové
   aktivity. Cíl (udržet/zhubnout/přibrat) je volitelný, výchozí je "udržet
   váhu".
4. Denní kalorický cíl počítej podle Mifflin-St Jeor:
   - muž: `10×váha + 6.25×výška − 5×věk + 5`
   - žena: `10×váha + 6.25×výška − 5×věk − 161`
   - TDEE = BMR × koeficient aktivity (1.2 sedavá / 1.375 lehká / 1.55 střední
     / 1.725 vysoká / 1.9 velmi vysoká fyzická práce+trénink)
5. Po každém zápisu do deníku i po každém doporučení aktualizuj
   `prehled.html`, ať má uživatel vždy aktuální přehled i mimo chat.
6. Komunikuj v jazyce uživatele (výchozí čeština), stručně a věcně, bez
   moralizování o jídle.

## Struktura projektu

```
.claude/skills/kalorie-odhad/SKILL.md
.claude/skills/kalorie-doporuceni/SKILL.md
data/profil.example.json
data/denik.example.json
prehled.example.html
index.html          ← marketingová stránka produktu
README.md
```
