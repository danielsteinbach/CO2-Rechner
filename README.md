# Werkstoffbilanz

Materialvergleichsrechner für Schilder und Plattenzuschnitte. Stellt 17 Plattenwerkstoffe
nebeneinander und zeigt für ein konkretes Projekt zwei Zahlen je Material: **Kilogramm CO₂**
und **Euro**.

Eine einzige HTML-Datei. Kein Build, kein Framework, kein Server, keine Datenbank.

## Veröffentlichen auf GitHub Pages

1. Neues Repository anlegen, zum Beispiel `werkstoffbilanz`.
2. `index.html` hochladen (Web-Oberfläche: *Add file → Upload files*).
3. *Settings → Pages → Build and deployment*: als Source **Deploy from a branch** wählen,
   Branch `main`, Ordner `/ (root)`, speichern.
4. Nach ein bis zwei Minuten liegt die Seite unter
   `https://<dein-benutzername>.github.io/werkstoffbilanz/`

Für eine eigene Domain im selben Menü unter *Custom domain* eintragen und beim Domain-Anbieter
einen CNAME auf `<dein-benutzername>.github.io` setzen.

## Was die Seite kann

- **Vier Grundangaben** für alle Materialien: Fläche, Verschnitt, Antriebsart des
  Transportfahrzeugs, Verbrauch und Preis.
- **Diesel oder Elektro.** Bei Elektro ist der Strom-Emissionsfaktor frei einstellbar;
  voreingestellt sind 152 g CO₂/kWh, die österreichische Stromaufbringung.
- **Fünf Eingaben je Material**: Stärke, Preis, Rezyklatanteil, Verwertungsquote, Transportweg.
- **Preisvorschlag**, der sich aus Richtpreis je dm³ mal Materialstärke ergibt und daher
  automatisch mitwächst, wenn die Stärke geändert wird. Jederzeit überschreibbar.
- **Alle Kennwerte korrigierbar**: Dichte, Treibhauspotenzial für Neuware und Rezyklat,
  Verbrennungsfaktor, Richtpreis.
- **Diagramm** mit zwei Balken je Material, sortierbar nach CO₂ oder Preis.
- **PDF-Export** auf eine A4-Seite über den Druckdialog.
- Alle Eingaben bleiben im Browser des Besuchers gespeichert (`localStorage`), nichts wird
  irgendwohin übertragen. Die Seite lädt keinerlei Daten nach — einzige externe Ressource ist
  die Schrift *Schibsted Grotesk* von Google Fonts.

## Werte ändern

Alle Materialdaten stehen im `<script>`-Block ganz unten in der Konstante `BASE`. Ein Eintrag:

```js
{id:"pvc", n:"PVC (massiv)", d:1.38, p:2.80, s:1.20, e:1.41, mm:2.00, pr:6.50, q:30}
```

| Feld | Bedeutung |
|------|-----------|
| `id` | interner Schlüssel, muss eindeutig sein |
| `n`  | angezeigter Name |
| `d`  | Dichte in kg/dm³ |
| `p`  | Treibhauspotenzial Neuware in kg CO₂e/kg (Module A1–A3) |
| `s`  | Treibhauspotenzial Rezyklat in kg CO₂e/kg |
| `e`  | freigesetztes CO₂ bei Verbrennung in kg CO₂/kg (bei Holz, Papier, Karton `0`, weil biogen) |
| `mm` | voreingestellte Materialstärke |
| `pr` | Richtpreis netto in €/dm³ |
| `q`  | voreingestellte Verwertungsquote in Prozent |

Ein neues Material ist eine weitere Zeile in dieser Liste — sonst ist nichts anzupassen,
Tabelle, Diagramm und PDF wachsen automatisch mit.

Die Transportkonstanten stehen direkt darunter: `DIESEL` (2,64 kg CO₂ je Liter) und `KM0`
(voreingestellter Transportweg, 10 km).

## Wie gerechnet wird

```
Masse        = Fläche × (1 + Verschnitt) × Stärke × Dichte
GWP-Faktor   = Neuware × (1 − Rezyklatanteil) + Rezyklat × Rezyklatanteil
Herstellung  = Masse × GWP-Faktor
Transport    = km × Verbrauch/100 × Emissionsfaktor
Lebensende   = Masse × (1 − Verwertungsquote) × Verbrennungsfaktor
Gutschrift   = − Masse × max(0, Verwertungsquote − Rezyklatanteil) × (Neuware − Rezyklat)
CO₂ gesamt   = Herstellung + Transport + Lebensende + Gutschrift
Kosten       = Fläche × (1 + Verschnitt) × Preis je m²  +  Transportkosten
```

Die Gutschrift folgt dem Ansatz der vermiedenen Primärproduktion (Modul D nach EN 15804) und
wird nur auf den Netto-Schrott gerechnet — also auf die Verwertungsquote abzüglich des bereits
eingekauften Rezyklatanteils. Sonst würde derselbe Kreislauf zweimal gutgeschrieben.

Im Diagramm wird die Gutschrift von der Herstellung abgezogen, damit die Balkenlänge exakt dem
danebenstehenden Wert entspricht.

## Datenquellen

- Kunststoffe: [PlasticsEurope Eco-Profiles](https://plasticseurope.org/), europäischer Branchendurchschnitt
- PVC: [American Chemistry Council, Cradle-to-Gate LCA of PVC Resin](https://www.americanchemistry.com/)
- Metalle, Glas, Holzwerkstoffe: [ÖKOBAUDAT](https://www.oekobaudat.de/) (BBSR) und [baubook / IBO](https://www.ibo.at/)
- Strommix Österreich: [Umweltbundesamt Österreich](https://www.umweltbundesamt.at/), 152 g CO₂e/kWh
  (123 g direkt plus 29 g vorgelagert, Bezugsjahr 2023, inklusive Importanteil)
- Preise: deutschsprachige Zuschnitt-Onlineshops, netto ohne Umsatzsteuer, Stand August 2026

Die Emissionsfaktoren sind europäische Mittelwerte mit einer typischen Streuung von ±30 %.
Die Preise streuen je nach Menge und Lieferant deutlich stärker.

## Grenzen

Nicht enthalten sind Bedruckung, Kaschierung, Beschläge, Montage, Reinigung — und vor allem
die **Nutzungsdauer**. Die Seite vergleicht Materialmengen, keine Standzeiten. Ein Blechschild,
das fünfzehn Jahre hält, ist pro Nutzungsjahr besser als ein dreimal ersetztes Kartonschild,
auch wenn die Herstellungszahl das Gegenteil nahelegt.

## Haftungsausschluss

Für die Richtigkeit, Vollständigkeit und Aktualität der berechneten Werte wird keine
Gewährleistung und keine Garantie übernommen. Die Nutzung erfolgt auf eigene Verantwortung;
eine Haftung für Entscheidungen, die auf Grundlage dieser Ergebnisse getroffen werden, ist
ausgeschlossen.

## Korrekturen

Wer für ein Material eine belastbarere Quelle hat — ein Umweltdatenblatt, ein
Lieferantendatenblatt, einen realen Einkaufspreis — oder ein fehlendes Material vermisst:

**Daniel Steinbach** · danielsteinbach@gmx.net · +43 664 421 70 82
