---
name: kalorie-odhad
description: Use when the user reports (in Czech or any language) what they ate or drank, to estimate its calories/macros from the model's own nutrition knowledge and log it into data/denik.json. Trigger on phrases like "snědl jsem", "dal jsem si", "vypil jsem", "k snídani/obědu/svačině/večeři jsem měl", "zapiš mi", or when the user simply names food/drink items they consumed. Do not use this for questions about what to eat next — that is kalorie-doporuceni.
---

# kalorie-odhad

Zapisuje zkonzumované jídlo/pití do deníku a odhaduje jeho kalorickou hodnotu.
Žádná kalorická databáze ani vzorec v kódu — odhad počítáš vždy sám jako
model, ze svých znalostí výživy.

## Postup

1. **Načti kontext.** Přečti `data/denik.json` (pokud neexistuje, pracuj s
   prázdným objektem `{}`) a `data/profil.json` (pokud existuje — potřebuješ
   ho pro dopočet zbývajícího kalorického rozpočtu v potvrzení).

2. **Rozpoznej položky.** Z uživatelovy zprávy vytáhni jednu nebo víc
   položek jídla/pití. Pokud množství/porce není uvedena, zvol rozumný
   běžný odhad (např. "banán" → ~120 g, "sklenice mléka" → ~250 ml) a tento
   předpoklad viditelně napiš uživateli, ať ho může opravit.

3. **Odhadni kalorie.** Pro každou položku odhadni kcal (a pokud to jde
   rozumně odhadnout, i bílkoviny/sacharidy/tuky v gramech) na základě
   svých znalostí složení potravin a typické porce. U složených/domácích
   jídel (např. "guláš s knedlíkem") odhadni podle typického receptu a
   uveď to jako hrubý odhad.

4. **Zapiš do deníku.** V `data/denik.json` najdi/vytvoř klíč pro dnešní
   datum (formát `YYYY-MM-DD`, dnešní datum) a přidej položku do pole
   `polozky`:
   ```json
   {
     "cas": "HH:MM",
     "popis": "text tak, jak to uživatel popsal",
     "kcal": 000,
     "bilkoviny_g": 00,
     "sacharidy_g": 00,
     "tuky_g": 00,
     "poznamka": "případný předpoklad o porci"
   }
   ```
   Přepočti `celkem_kcal` daného dne jako součet `kcal` všech položek.
   Pokud `data/denik.json` ještě neexistuje, vytvoř ho.

5. **Aktualizuj přehled.** Přegeneruj `prehled.html` (viz sdílený formát
   popsaný v `kalorie-doporuceni/SKILL.md`) tak, aby odpovídal novému stavu
   deníku — čistě jako statický HTML výstup, žádný klientský výpočet.

6. **Potvrď uživateli** krátce a přehledně:
   - co bylo zapsáno a odhadnuté kcal (+ makra, pokud jsou)
   - použitý předpoklad o porci, pokud nějaký byl
   - dnešní součet kcal
   - pokud existuje `data/profil.json` s denním cílem: kolik kalorií dnes
     ještě zbývá (cíl − součet)
   - pokud `data/profil.json` neexistuje, jen krátce podotkni, že pro
     zobrazení zbývajícího rozpočtu je potřeba nastavit profil (stačí říct
     pohlaví, věk, výšku, váhu a úroveň pohybu)

Buď stručný — jde o rychlý průběžný zápis, ne o dlouhý text.
