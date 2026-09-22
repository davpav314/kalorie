# fit-poradce

Osobní kalorický poradce, který běží přímo v [Claude Code](https://claude.com/claude-code) —
žádná appka, žádná kalorická databáze v kódu. Odhad kalorií i doporučení
jídelníčku dělá vždy model sám na základě dvou skillů.

Marketingová stránka produktu: [`index.html`](index.html).

## Co umí

- **Personalizace** — pohlaví, věk, výška, váha, úroveň pohybové aktivity →
  dopočítá denní kalorický cíl (Mifflin-St Jeor).
- **Průběžný zápis jídla** — napíšeš, co jsi snědl/vypil, model odhadne
  kalorie (a makra) a zapíše je do deníku.
- **Přehled** — kalorie po jednotlivých položkách i po dnech
  (`prehled.html`, ukázka v [`prehled.example.html`](prehled.example.html)).
- **Doporučení na zbytek dne** — co jíst, aby ses trefil do denního cíle.

## Jak to spustit

1. Otevři tuto složku v Claude Code.
2. Napiš Claude, co jsi snědl/vypil, nebo požádej o nastavení profilu
   ("nastav mi profil"). Pravidla chování jsou v [`CLAUDE.md`](CLAUDE.md) a
   ve skillech v [`.claude/skills`](.claude/skills).
3. Ptej se kdykoliv na dnešní přehled nebo na to, co jíst do konce dne.

## Struktura

```
CLAUDE.md                              pravidla a přehled projektu pro Claude
.claude/skills/kalorie-odhad/          skill: odhad kalorií + zápis do deníku
.claude/skills/kalorie-doporuceni/     skill: profil, cíl, přehled, doporučení
data/profil.example.json               ukázka formátu profilu
data/denik.example.json                ukázka formátu deníku
prehled.example.html                   ukázka generovaného přehledu
index.html                             marketingová stránka produktu
```

`data/profil.json`, `data/denik.json` a `prehled.html` obsahují osobní údaje
a jsou v `.gitignore` — do repozitáře se necommitují, generují se lokálně
při používání.
