# Y-API-Preistabelle — nachgerechnet, nicht abgeschrieben

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · **Deutsch** · [Português](README.pt.md)

> **Offenlegung:** Ich arbeite an Y-API, das hier ist also ein Erstanbieter-Werkzeug, das eine Erstanbieter-Datei liest. Alles, was es ausgibt, stammt aus <https://y-api.bestvirtualgoods.com/pricing.json> — öffentlich und ohne API-Schlüssel. Führe das Skript aus und prüfe die Ausgabe selbst gegen die Quelle.

Ein Skript, keine Abhängigkeiten, kein API-Schlüssel. Es liest die veröffentlichte Preisdatei von Y-API und erzeugt [table.md](table.md) mit **Barpreisen** — dem Betrag, der tatsächlich von deiner Karte abgebucht wird, und dem einzigen, der sich mit dem Listenpreis eines anderen Anbieters vergleichen lässt.

## Warum das nötig ist

Y-API gibt Preise in **Guthaben** an, und aktuell ergibt $1 Einzahlung $20 Guthaben. Guthaben ist, was vom Kontostand abgezogen wird — nicht, was deiner Karte berechnet wird. Eine Tabelle, die beim Guthaben stehen bleibt, lässt jedes Modell 20× teurer aussehen, und eine Tabelle mit hartcodiertem „durch 20 teilen" wird an dem Tag stillschweigend falsch, an dem der Aktionskurs endet. Dieses Skript liest den Kurs aus **derselben Datei**, aus der es die Preise liest.

## Ausführen

```bash
node price-table.mjs > table.md                            # veröffentlichte Datei holen
PRICE_JSON=./pricing.json node price-table.mjs             # gespeicherte Kopie offline rendern
```

Erfordert Node 18+ (globales `fetch`). Die `table.md` in diesem Repo wird jeden Montag von einer GitHub Action neu erzeugt und **nur bei Änderung committet** — die Commit-Historie *ist* also das Protokoll der Preisänderungen.

## Zwei Regeln, die das Skript einhält

Wird eine von beiden gebrochen, entsteht eine Tabelle, die der Seite, von der sie stammt, stillschweigend widerspricht.

1. **Der Barpreis wird nachgerechnet, nicht kopiert.** Die Datei veröffentlicht `cash_price`, aber das Skript rechnet `credit_price / top_up.quota_rate` neu und **scheitert laut**, wenn beides abweicht. Weder ein veralteter Kurs noch eine handeditierte Zahl kann in die Tabelle rutschen.
2. **`vendor_cheaper_on_cached_input` wird nie allein ausgegeben.** Dieses Flag bedeutet: Der Cache-Eingabepreis des Anbieters liegt unter unserem Barpreis — für cache-lastige Workloads sind wir die teurere Option. Manche Zeilen tragen es jedoch zusammen mit `historical-price` (eine alte Preisliste des Anbieters), und die Y-API-Seite schließt sie bewusst aus ihrer Zählung „wir verlieren beim Cache" aus. **Gibt man das Flag ohne den Vorbehalt aus, geht der Leser mit einer falschen Zahl weg.** Deshalb liefert die Tabelle beide Spalten.

## Die Ausgabe lesen

- **Der Katalog** — alle Modelle, Guthaben und Barpreis, pro 1M Tokens, nach Preis sortiert.
- **Gegen den Listenpreis des Anbieters** — die 11 Modelle mit einem von uns verifizierten und zitierten Anbieterpreis, mit Vielfachem, Cache-Flag, Vorbehalten, Prüfdatum und Link zur Anbieterseite. Die restlichen 4 stehen als „kein verifizierbarer Anbieterpreis" da; die Datei sagt das explizit, statt zu schätzen.

`× ours` liest sich als „der Anbieter nimmt das Vielfache unseres Barpreises" — ein großes Vielfaches heißt, dass **Y-API billiger** ist, nicht teurer.

## Reichweite, ehrlich gesagt

- Die Tabelle ist so aktuell wie die Quelldatei. In der Kopfzeile stehen das `synced_at` der Quelle und das Renderdatum; sind beide alt, gibt es hier keine Neuigkeit.
- Sie rendert die Preize von **einem** Gateway. Sie vergleicht nicht über Anbieter hinweg und verfolgt keine Preisgeschichte anderer.
- Preise bewegen sich, und der Aufladekurs ist ein Aktionskurs ohne angekündigtes Enddatum. Wer mit dieser Tabelle entscheidet, sollte aus der Quelle neu ableiten, nicht aus einer Kopie von `table.md`.

---

*Teil des [Y-API-Profils](https://github.com/zhuo01-l2ab).*
