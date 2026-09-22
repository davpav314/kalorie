---
name: kalorie-doporuceni
description: Use when the user wants to set up or update their personal profile (gender, age, height, weight, activity level), wants a summary/overview of today's (or another day's) calorie intake, or asks what to eat for the rest of the day to hit their calorie goal. Trigger on phrases like "nastav můj profil", "kolik mi zbývá kalorií", "shrň mi dnešek", "co mám jíst do konce dne", "doporuč mi jídlo". Do not use this to log food that was already eaten — that is kalorie-odhad.
---

# kalorie-doporuceni

Spravuje osobní profil, počítá denní kalorický cíl a navrhuje jídelníček na
zbytek dne. Žádný vzorec ani databáze v kódu mimo tento soubor — BMR/TDEE i
volbu konkrétních jídel počítáš a navrhuješ vždy sám jako model.

## 1. Profil (`data/profil.json`)

Pokud soubor neexistuje, nebo uživatel chce údaje změnit, vyžádej si (co
ještě nemáš) a ulož:

```json
{
  "pohlavi": "muž | žena",
  "vek": 0,
  "vyska_cm": 0,
  "vaha_kg": 0.0,
  "pohybova_aktivita": "sedavá | lehká | střední | vysoká | velmi vysoká",
  "cil": "udržet váhu | zhubnout | přibrat",
  "denni_kalorický_cil": 0,
  "aktualizovano": "YYYY-MM-DD"
}
```

- `cil` je volitelný, výchozí "udržet váhu".
- Pokud uživatel neuvede úroveň pohybu jednoznačně, nabídni stručně 5
  možností: sedavá (málo/žádný pohyb), lehká (sport 1–3×/týden), střední
  (sport 3–5×/týden), vysoká (sport 6–7×/týden), velmi vysoká (fyzicky
  náročná práce + denní trénink).

**Výpočet cíle (Mifflin-St Jeor):**

- BMR muž = `10×váha_kg + 6.25×výška_cm − 5×věk + 5`
- BMR žena = `10×váha_kg + 6.25×výška_cm − 5×věk − 161`
- koeficient aktivity: sedavá 1.2 / lehká 1.375 / střední 1.55 / vysoká
  1.725 / velmi vysoká 1.9
- TDEE = BMR × koeficient
- `denni_kalorický_cil` = TDEE zaokrouhlené na desítky; pokud `cil` je
  "zhubnout", odečti rozumný deficit (~15 %, ne víc než ~500 kcal); pokud
  "přibrat", přičti rozumný přebytek (~10–15 %). Vždy nastav rozumnou dolní
  mez (nikdy nedoporučuj cíl nižší než cca 1200 kcal pro ženy / 1500 kcal pro
  muže, u nižších hodnot uživatele na to uprav a upozorni).

Ulož výsledek do `denni_kalorický_cil` a aktualizuj `aktualizovano`.

## 2. Přehled dne

1. Přečti `data/denik.json`, najdi záznam pro požadovaný den (výchozí
   dnešek). Pokud chybí, součet je 0 a žádné položky.
2. Spočti `zbyva = denni_kalorický_cil − celkem_kcal`.
3. Ukaž přehledně: tabulku položek dne (čas, popis, kcal), denní součet,
   denní cíl, kolik zbývá (nebo o kolik je uživatel nad cílem — bez
   moralizování, věcně).

## 3. Doporučení jídelníčku na zbytek dne

Když uživatel chce poradit, co jíst:

1. Zjisti `zbyva` (viz výše). Pokud je profil neúplný, nejdřív ho dořeš (bod 1).
2. Zohledni, kolik je hodin / kolik jídel běžně ještě přijde (např. odpoledne
   → svačina + večeře, dopoledne → oběd + svačina + večeře) — rozděl
   `zbyva` rozumně mezi zbývající jídla, nesnaž se vše nacpat do jednoho.
3. Navrhni 2–3 konkrétní, realistické varianty jídel/svačin (ne obecné rady
   typu "jez zdravě"), s přibližnými kcal a hrubým poměrem
   bílkoviny/sacharidy/tuky tak, aby součet s dosavadní spotřebou seděl na
   denní cíl. Přihlédni k tomu, co už dnes jedl (např. pokud měl málo
   bílkovin, navrhni bílkovinnější variantu).
4. Pokud je `zbyva` záporné (uživatel je už nad cílem), řekni to věcně a
   klidně, navrhni lehčí zbytek dne nebo že dnešní cíl mírně přesáhne — bez
   kritiky.

## 4. Aktualizace `prehled.html`

Po každé změně profilu nebo na vyžádání přegeneruj (Write nástrojem) kořenový
soubor `prehled.html` jako samostatnou statickou stránku (inline CSS, žádný
JS s výpočty — všechna čísla už spočítaná modelem před zápisem):

- Nahoře: jméno/cíl a dnešní shrnutí (součet / cíl / zbývá) jako výrazné
  "karty".
- Tabulka dnešních položek (čas, popis, kcal).
- Pod tím přehled posledních cca 14 dní z `data/denik.json`: datum a denní
  součet (jednoduchá tabulka nebo řada pruhů/barů kreslených čistě CSS
  `width`/`background`, bez JS).
- Pokud `data/profil.json` nebo `data/denik.json` neexistují, vypiš vlídnou
  zprávu, že zatím nejsou žádná data.

Přehled piš česky, stručně a čitelně.
