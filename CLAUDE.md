# Instruccions per a Claude Code — projecte TopBrands

## Git
Sempre que facis un canvi de codi en aquest projecte, fes automàticament,
sense preguntar confirmació:
```
git add .
git commit -m "<missatge descriptiu del canvi>"
git push
```
Confirma sempre a l'usuari, al final de la resposta, que els canvis s'han
pujat correctament a GitHub (o avisa clarament si el push ha fallat).

## Context del projecte
- Aquest repositori conté `bbdd_cupons_TB_2026.html`, una aplicació de gestió
  de cupons (BBDD de gestió de processos), en HTML/JavaScript pur.
- La base de dades és **Neon (PostgreSQL)**, no Supabase. La connexió es fa
  amb el driver `@neondatabase/serverless` via `import()` dinàmic des de
  `esm.sh`. Cal HTTPS (GitHub Pages) per funcionar — no funciona obrint el
  fitxer amb `file://`.
- La funció `neonQuery(sql, params)` ha d'EMBOLICAR sempre el resultat com
  `{rows: res.rows || res}`, perquè el driver de Neon a vegades retorna
  directament un array de files (no un objecte `{rows: [...]}`). Si s'oblida
  aquest embolcall, les lectures (`sbGet`) tornen sempre buides encara que
  la BBDD tingui dades.
- Canvis a l'**estructura** de la taula (`ALTER TABLE`, columnes noves, etc.)
  s'han de fer manualment a Neon (SQL Editor) — no hi ha sincronització
  automàtica entre el codi i l'esquema de la base de dades. Quan afegeixis
  un camp nou al formulari/taula, genera també l'SQL `ALTER TABLE` perquè
  l'usuari el pugui executar a Neon, i avisa'l explícitament que cal fer-ho.
- GitHub Pages es publica des de la branca `main`, arrel del repositori.
  URL pública: `https://acrestruch.github.io/TopBrands/bbdd_cupons_TB_2026.html`

## Estil de treball
- L'usuari (Jaume) treballa en català. Respon sempre en català.
- Prefereix explicacions curtes i pas a pas quan calgui acció manual seva
  (per exemple, executar SQL a Neon).
- No cal demanar confirmació abans de fer `git push`; sí cal avisar-lo si
  un canvi requereix una acció manual a Neon.
