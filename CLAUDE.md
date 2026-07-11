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
- Aquest repositori conté una aplicació de gestió de cupons (BBDD de gestió
  de processos), en HTML/JavaScript pur.
- **Arquitectura multi-any (des de la campanya 2027):** tota la lògica
  JavaScript viu en un únic fitxer compartit **`app.js`**. Els fitxers
  `bbdd_cupons_TB_2026.html` i `bbdd_cupons_TB_2027.html` són fitxers prims
  que només contenen l'HTML/CSS i una petita constant de configuració abans
  de carregar `app.js`:
  ```html
  <script>const YEAR_CONFIG = { year: 2026, table: 'cupons', strictComplete: false };</script>
  <script src="app.js"></script>
  ```
  **Qualsevol canvi/millora de codi que facis s'ha d'aplicar a `app.js`**
  (mai duplicar-lo entre els dos HTML), perquè afecti automàticament totes
  dues campanyes. Si en el futur cal una diferència de comportament entre
  anys, afegeix-la com a nova propietat de `YEAR_CONFIG` i llegeix-la des
  d'`app.js` (com ja es fa amb `table` i `strictComplete`), en lloc de
  bifurcar el codi.
- `YEAR_CONFIG.table` indica la taula de Neon amb les dades de cupons
  d'aquell any (`cupons` per al 2026, `cupons_2027` per al 2027 — mateix
  esquema, taula pròpia i buida per a cada campanya nova).
- `YEAR_CONFIG.strictComplete` controla `checkAutoComplete()` a `app.js`:
  si és `true` (2027), un registre torna automàticament a PROCÉS si deixa de
  complir tots els requisits (dates de tramitació, ITA, Web) encara que
  hagués estat COMPLETAT. Si és `false` (2026), un cop COMPLETAT es queda
  així per sempre (només es pot canviar manualment des del desplegable
  Estat); l'auto-promoció a COMPLETAT quan tot està fet es manté en tots
  dos casos.
- La resta de configuració (contrasenya Master, Admins ajudants, PINs,
  llistes KAM/Equip/Web/Hora) es comparteix entre 2026 i 2027 — mateixes
  taules Neon (`admin_config`, `admins`, `app_lists`, `user_pins`), no per
  any.
- **Pestanya "Requeriments"** (seguiment post-presentació, només Master):
  `YEAR_CONFIG.reqTable` indica la taula de requeriments de l'any
  (`requeriments` per 2026, `requeriments_2027` per 2027 — mateix esquema,
  vinculada per `cupo_id` a `YEAR_CONFIG.table`; només es poden crear
  requeriments sobre registres COMPLETAT). El catàleg de tipus de
  requeriment (`requeriment_tipus`) és **compartit** entre anys, com la
  resta de configuració. Al seleccionar l'empresa en crear un requeriment
  nou, es mostra i es referencia pel número de Tiquet (més fàcil d'
  identificar), però el vincle real guardat és l'`id` intern de `cupons`
  — el Tiquet es recalcula sol i no és fiable com a clau permanent.
- La base de dades és **Neon (PostgreSQL)**, no Supabase. La connexió es fa
  amb el driver `@neondatabase/serverless` via `import()` dinàmic des de
  `esm.sh`. Cal HTTPS (GitHub Pages) per funcionar — no funciona obrint el
  fitxer amb `file://`.
- La funció `neonQuery(sql, params)` ha d'EMBOLICAR sempre el resultat com
  `{rows: res.rows || res}`, perquè el driver de Neon a vegades retorna
  directament un array de files (no un objecte `{rows: [...]}`). Si s'oblida
  aquest embolcall, les lectures (`sbGet`) tornen sempre buides encara que
  la BBDD tingui dades.
- Canvis a l'**estructura** de les taules de dades (`ALTER TABLE`, columnes
  noves a `cupons`/`cupons_2027`, etc.) s'han de fer manualment a Neon (SQL
  Editor) — no hi ha sincronització automàtica entre el codi i l'esquema de
  la base de dades. Quan afegeixis un camp nou al formulari/taula, genera
  també l'SQL `ALTER TABLE` (per a totes dues taules `cupons`/`cupons_2027`
  si el camp aplica a totes dues) perquè l'usuari el pugui executar a Neon,
  i avisa'l explícitament que cal fer-ho.
- GitHub Pages es publica des de la branca `main`, arrel del repositori.
  URLs públiques:
  `https://acrestruch.github.io/TopBrands/bbdd_cupons_TB_2026.html`
  `https://acrestruch.github.io/TopBrands/bbdd_cupons_TB_2027.html`

## Estil de treball
- L'usuari (Jaume) treballa en català. Respon sempre en català.
- Prefereix explicacions curtes i pas a pas quan calgui acció manual seva
  (per exemple, executar SQL a Neon).
- No cal demanar confirmació abans de fer `git push`; sí cal avisar-lo si
  un canvi requereix una acció manual a Neon.
