---
name: rezepte-import
description: Holt neue Rezept-Links aus Martins Google-Formular-Antworten (Drive-Tabelle "GLUT - Rezpete Links (Antworten)"), liest die Instagram-Beschreibungen und trägt sie als Seeds in index.html (Projekt Pizza/GLUT, Bereich Kochen) ein. Verwenden wenn Martin "Rezepte aktualisieren", "neue Rezepte", "Links einlesen", "wartende Links", "Rezepte importieren" oder /rezepte-import sagt, auch bei einem einzelnen eingetippten Wort wie "rezepte".
---

# Rezepte-Import (Projekt Pizza)

Ziel: alle Links, die noch nicht als Rezept in `index.html` stehen, als fertige Rezepte eintragen. Martin tippt nur "Rezepte aktualisieren", der Rest läuft hier durch.

## Ablauf

1. **Links holen.** Drive-Tabelle `1QoMp53nP3KgpzNX11cMyT75TkqVbFpi9B7wn_vYIb58` mit `read_file_content` lesen (Drive-MCP per ToolSearch laden: `search_files`, `read_file_content`). Jede Zeile = ein Link, die Spalte ist die Kategorie. Spalte "Beilagen" = Kategorie `Beilage`. Spalten: Vorspeise, Hauptspeise, Beilagen, Soßen, Dessert, Fitness-Meal, Fitness-Süßes, Brot & Gebäck, Suppe.
2. **Neue herausfiltern.** Instagram-Shortcode (`/reel/XXXX/` oder `/p/XXXX/`) je Link mit `grep -c "<code>" index.html` prüfen. Treffer 0 = neu. Treffer ≥1 = schon drin, überspringen.
3. **Inhalt lesen.** Eingebauter Browser (`mcp__Claude_Browser__*`), kein Login nötig:
   - `navigate` auf `https://www.instagram.com/reel/<code>/`, 2 s warten, `get_page_text`.
   - Beschreibung steht meist vollständig im Text (Zutaten, Schritte, Nährwerte). Ist sie mit "mehr" abgeschnitten: `find "mehr"` und anklicken, dann nochmal lesen.
   - Mehrere Links in einem `browser_batch` abarbeiten.
   - Claude-in-Chrome ist nicht nötig. Ist es nicht verbunden, ignorieren.
   - Nur Teaser ("Kommentiere X und ich schicke dir das Rezept", Karussell ohne Text): trotzdem anlegen, mit den bekannten Fakten und Hinweis "Mengen fehlen". Nichts erfinden.
4. **Seeds schreiben.** Siehe Format unten. Backup vorher: `cp index.html "Delete Files/index.html.bak-JJJJMMTT"`.
5. **Prüfen** (Preview `pizza`, Port 8123): alte Speicherstand-Simulation, siehe unten. Danach `localStorage.removeItem('glut.koch.v1')`.
6. **Melden:** welche Rezepte neu, in welcher Kategorie, was geschätzt oder lückenhaft ist. Nicht committen, bevor Martin es sagt.

## Format in index.html

- Neue Konstante `NEU<KV>=[sn(...),...]` direkt vor `const FKATS=`, `KV` = neue Speicherversion (`KV=` in der Zeile `const KATS=...,KKEY=...,KV=..`).
- `sn(id,name,kat,zutatenArray,textArray,videoUrl)`. Text nummerierte Schritte, Leerzeile, dann Nährwerte laut Beschreibung, Hinweise, `Quelle: <account> (Instagram)`.
- Zutaten zuerst Menge ("Für 4 Bagels:" als Kopfzeile), deutsche Wörter, Beschreibung übersetzen wenn englisch.
- `NW`-Map: `id:[portionen,kcal pro Portion,Eiweiß,KH,Fett]`. Fehlen Nährwerte, schätzen und im Text "Nährwerte sind geschätzt" schreiben. Bei Teasern weglassen.
- Registrieren an 3 Stellen: Default-Liste in `ladenK` (`...NEU<KV>`), Migrationsliste (`...NEU<KV>.map(n=>[<KV>,n])`), `KV` hochzählen.
- Kategorie muss in `KATS` stehen. Fitness-Kategorien (`FKATS`) erscheinen im Fitness-Bereich, nicht unter Kochen.
- Wartende Links verschwinden automatisch, sobald ein Rezept dasselbe Video hat.
- Beim Einfügen per Python-Skript arbeiten (Scratchpad), Zeilen in index.html sind sehr lang.

## Prüfen

Im Preview per `javascript_tool`:
```js
localStorage.setItem('glut.koch.v1',JSON.stringify({v:<KV-1>,rezepte:[{id:'mayo',name:'Light-Mayonnaise',kat:'Soßen'}],kats:[],links:[]}));location.reload()
```
Dann Kochen öffnen, Kategorien per `get_page_text` prüfen: neue Rezepte da, keine Konsolenfehler. Fitness-Rezepte im Fitness-Bereich prüfen.

## Fallen

- Formular-Tabelle enthält auch bereits importierte Links, immer gegen index.html abgleichen.
- Beschreibungen widersprechen sich manchmal (z. B. Video zeigt 2 Eigelb, Text 1 Ei): Text nehmen, Hinweis in Rezept.
- Instagram-Cookie-Banner blockiert Screenshots, `get_page_text` geht trotzdem.
- Keine Affiliate-/Rabattcodes oder Kommentare der Nutzer übernehmen.
