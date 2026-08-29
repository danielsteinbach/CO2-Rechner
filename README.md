# Werkstoffbilanz

Materialvergleichsrechner für Schilder und Plattenzuschnitte. Stellt 20 Plattenwerkstoffe
nebeneinander und zeigt für ein konkretes Projekt zwei Zahlen je Material: **Kilogramm CO₂**
und **Euro**.

Eine einzige HTML-Datei. Kein Build, kein Framework, kein Server, keine Datenbank.

**Live: https://danielsteinbach.github.io/CO2-Rechner/**

## Veröffentlichung

Die Seite wird über GitHub Pages aus dem Branch `main`, Ordner `/ (root)` ausgeliefert
(*Settings → Pages → Build and deployment*). Eine geänderte `index.html` ist nach dem Commit
in ein bis zwei Minuten online — es gibt keinen Build-Schritt dazwischen.

Für eine eigene Domain im selben Menü unter *Custom domain* eintragen und beim Domain-Anbieter
einen CNAME auf `danielsteinbach.github.io` setzen.

## Was die Seite kann

- **Vier Grundangaben** für alle Materialien: Fläche, Verschnitt, Antriebsart des
  Transportfahrzeugs, Verbrauch und Preis.
- **Diesel oder Elektro.** Bei Elektro ist der Strom-Emissionsfaktor frei einstellbar;
  voreingestellt sind 152 g CO₂/kWh, die österreichische Stromaufbringung.
- **Sechs Eingaben je Material**: Stärke, Preis, Rezyklatanteil, Recyclingquote am Lebensende,
  Transportweg, Anzahl Nutzungen. Jedes Feld hat einen erklärenden Tooltip.
- **Materialien abwählen**: Was für ein Projekt nicht infrage kommt, wird über das Kästchen vor dem
  Namen abgewählt — es bleibt hellgrau sichtbar, fällt aber aus dem Maßstab des Diagramms heraus,
  damit die verbleibenden Balken die volle Breite nutzen.
- **Preisvorschlag**, der sich aus Richtpreis je dm³ mal Materialstärke ergibt und daher
  automatisch mitwächst, wenn die Stärke geändert wird. Jederzeit überschreibbar.
- **Alle Kennwerte korrigierbar**: Dichte, Treibhauspotenzial für Neuware und Rezyklat,
  Verbrennungsfaktor, Richtpreis.
- **Getrennte Ergebnisspalten**: CO₂ vor Gutschrift, Recycling-Gutschrift, CO₂-Bilanz, kg CO₂ je
  Nutzung und Kosten — die Gutschrift verschwindet nicht in einer Gesamtzahl.
- **Diagramm** mit zwei Balken je Material, sortierbar nach CO₂, CO₂ je Nutzung oder Preis.
- **PDF-Export** auf eine A4-Seite quer über den Druckdialog. Ignoriert ein Druckdialog die
  Querformat-Angabe — auf Mobilgeräten kommt das vor —, fällt die Ausgabe auf Hochformat
  zurück und bleibt trotzdem vollständig: die Spalten sind in Prozent bemessen und die Zellen
  dürfen umbrechen, die Tabelle passt sich also jeder Seitenbreite an.
  Die Balken sind über `border-top` gezeichnet statt über `background`, damit sie auch dort
  gefüllt erscheinen, wo der Hintergrunddruck abgeschaltet ist — in Firefox ist das die
  Voreinstellung.
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
| `q`  | voreingestellte Recyclingquote am Lebensende in Prozent |
| `rz` | optional: voreingestellter Rezyklatanteil in Prozent, fehlt er, gilt 0 |

Ein neues Material ist eine weitere Zeile in dieser Liste — sonst ist nichts anzupassen,
Tabelle, Diagramm und PDF wachsen automatisch mit.

### Warum `rz` nur beim Karton gesetzt ist

Der Rezyklatanteil ist bewusst fast überall 0. Vorbelegt ist er nur bei den beiden Kartonzeilen,
weil dort die *Sorte* den Faserstoff festlegt: Chromoduplex und Triplex (GD/GT) bestehen zu rund
80 % aus Altpapier, Chromo- und Zellstoffkarton (GC/GZ) aus Frischfaser. Bei allen anderen
Werkstoffen wäre eine Vorbelegung aus einem von zwei Gründen falsch:

- Bei **Stahl, Glas, Span-, MDF- und HDF-Platten** enthält der Neuware-Faktor der Datenquelle
  den branchenüblichen Schrott-, Scherben- und Altholzanteil bereits. Ein zusätzlicher Eintrag
  hier würde denselben Vorteil doppelt zählen. Bei Stahl und Glas legt die Datenquelle den Anteil
  nicht offen, er lässt sich also nicht herausrechnen — Nebenwirkung: die Gutschrift fällt für diese
  beiden etwas zu großzügig aus. Bewusste Wahl: lieber die Herstellung richtig als die Gutschrift.

  Bei **Spanplatte** geht es besser, weil die Deklaration ihre Mischung offenlegt (67 % Nadel-Frischholz,
  13 % Laub-Frischholz, **20 % Altholz**; einzelne Werke bis 50 %). Der Anteil ist deshalb
  herausgerechnet: `p` ist mit **0,30** der Frischholzwert, `rz` steht auf 20, und
  `0,30 × 0,80 + 0,15 × 0,20 = 0,27` ergibt wieder genau den ÖKOBAUDAT-Marktwert. Angezeigt wird
  dasselbe wie zuvor, aber die Gutschrift zählt ab 20 % statt ab null.

  **MDF und HDF** deklarieren **„kein Altholz enthalten"** — Altholzfasern lassen sich bisher nicht zu
  neuen Faserplatten verarbeiten. Dort ist die 0 keine Behelfslösung, sondern die Wirklichkeit. Genau
  darum liegt Spanplatte im GWP deutlich unter MDF.
- Bei **Aluminium und den Kunststoffen** gibt es keinen belastbaren Durchschnitt. Der
  Environmental Profile Report von European Aluminium nennt für Walzprodukte bewusst keinen
  Rezyklatanteil, weil er je Werk und Charge zwischen praktisch 0 und über 75 % liegt.

Die Transportkonstanten stehen direkt darunter: `DIESEL` (3,2 kg CO₂ je Liter) und `KM0`
(voreingestellter Transportweg, 10 km).

## Wie gerechnet wird

```
Masse        = Fläche × (1 + Verschnitt) × Stärke × Dichte
GWP-Faktor   = Neuware × (1 − Rezyklatanteil) + Rezyklat × Rezyklatanteil
Herstellung  = Masse × GWP-Faktor
Transport    = km × Verbrauch/100 × Emissionsfaktor
Lebensende   = Masse × (1 − Recyclingquote) × Verbrennungsfaktor
Gutschrift   = − Masse × max(0, Recyclingquote − Rezyklatanteil) × (Neuware − Rezyklat)
CO₂-Bilanz   = Herstellung + Transport + Lebensende + Gutschrift
CO₂ je Nutz. = CO₂-Bilanz / max(1, Nutzungen)
Kosten       = Fläche × (1 + Verschnitt) × Preis je m²  +  Transportkosten
```

Die Gutschrift ist ein **vereinfachter Ansatz in Anlehnung an die Nettofluss-Logik von Modul D
nach EN 15804**: gutgeschrieben wird nur die Recyclingquote abzüglich des bereits eingekauften
Rezyklatanteils, sonst zählte man denselben Kreislauf doppelt. Recyclingverluste,
Aufbereitungsaufwand und materialspezifische Substitutionsfaktoren werden nicht eigens
modelliert.

Zwei weitere bewusste Vereinfachungen: Der nicht stofflich recycelte Anteil wird als
**thermisch verwertet** gerechnet — Deponierung, Export oder Zwischenlagerung bildet die Seite
nicht ab. Und der Transport gilt je Material als **eigene volle Fahrt**, unabhängig vom Gewicht.

Im Diagramm wird die Gutschrift von der Herstellung abgezogen, damit die Balkenlänge exakt dem
danebenstehenden Wert entspricht.

## Datenquellen

- Kunststoffe: [PlasticsEurope Eco-Profiles](https://plasticseurope.org/), europäischer Branchendurchschnitt
- PVC: [American Chemistry Council, Cradle-to-Gate LCA of PVC Resin](https://www.americanchemistry.com/)
- Metalle, Glas, Holzwerkstoffe: [ÖKOBAUDAT](https://www.oekobaudat.de/) (BBSR) und [baubook / IBO](https://www.ibo.at/)
- MDF und HDF: rund 540 kg CO₂e/m³ für MDF laut [nachhaltiges-bauen.de](https://nachhaltiges-bauen.de/baustoffe/MDF-Platten),
  bei 750 kg/m³ Rohdichte also etwa 0,75 kg CO₂e/kg; HDF anteilig höher wegen Dichte und Bindemittel
- Strommix Österreich: [Umweltbundesamt Österreich](https://www.umweltbundesamt.at/), 152 g CO₂e/kWh
  (123 g direkt plus 29 g vorgelagert, Bezugsjahr 2023, inklusive Importanteil)
- Diesel: 3,2 kg CO₂ je Liter, Well-to-Wheel — Verbrennung (2,64 kg) plus Förderung und Raffinerie
- Preise: deutschsprachige Zuschnitt-Onlineshops, netto ohne Umsatzsteuer, Stand August 2026

Diesel- und Stromfaktor sind damit gleich abgegrenzt: beide enthalten die Energievorkette. Ein
Vergleich mit dem reinen Verbrennungswert des Diesels würde den Verbrenner systematisch
besserstellen. Bei den Materialdaten ist die Energievorkette ebenfalls enthalten — eine
identische Systemgrenze wie bei einem Energieträger ist das aber nicht, A1–A3 umfasst die
Herstellung eines Produkts, nicht die Bereitstellung eines Brennstoffs.

Die Emissionsfaktoren sind europäische Mittelwerte mit einer typischen Streuung von ±30 %.
Die Preise streuen je nach Menge und Lieferant deutlich stärker.

**Nicht alle Werte sind gleich gut belegt.** Die Neuware-Faktoren (`p`) stammen aus den genannten
Quellen. Die Rezyklat-Faktoren (`s`) sind bei mehreren Materialien Größenordnungs-Schätzungen; sie
gehen über die Gutschrift direkt in die Bilanz ein und sind der unsicherste Teil der CO₂-Rechnung.
Der Verbrennungsfaktor (`e`) ist rein stöchiometrisch aus dem fossilen Kohlenstoffanteil gerechnet,
ohne Gutschrift für Energierückgewinnung — also bewusst konservativ. Bei Holz, Papier und Karton
steht er auf 0, weil nur fossiles CO₂ bilanziert wird; die geringen Methan- und Lachgasmengen aus
der Verbrennung sind nicht modelliert (bei Holz rund 0,06 kg CO₂e/kg und damit innerhalb der
Unsicherheit des Herstellungswerts selbst).

**Holzwerkstoffe** stammen aus den generischen ÖKOBAUDAT-Datensätzen „Durchschnitt DE" nach EN 15804+A2,
bei denen der fossile Anteil getrennt ausgewiesen ist: Spanplatte 171,2 kg CO₂e/m³ bei 633 kg/m³
(0,270 kg/kg Marktmix, siehe Rückrechnung oben), MDF 323,1 bei 738 (0,438), HDF 345,8 bei 850 (0,407), Nadelschnittholz 57,2 bei 485
(0,118). Ältere Datensätze verrechnen die biogene CO₂-Aufnahme negativ in Modul A1 und ergeben in Summe
Minuswerte — die sind bewusst nicht verwendet. Die Spanne ist beträchtlich: zwei Hersteller-EPDs für
Spanplatte (Egger 0,224, Pfleiderer ≈0,196) liegen 20–25 % unter dem Branchenschnitt, baubook nennt für
MDF mit 0,641 kg/kg rund 46 % mehr als ÖKOBAUDAT.

**Karton** ist in zwei Zeilen aufgeteilt, weil sich die Sorten in zwei Kennwerten unterscheiden:
Recyclingkarton ist dichter (GD2: 400 g/m² bei 545 µm → 0,73 kg/dm³), Frischfaserkarton
voluminöser (GC1: 400 g/m² bei 605 µm → 0,66; GC2 sogar 700 µm → 0,57) und damit je Millimeter
leichter. Der Herstellungsfaktor stammt aus dem Carbon Footprint of Carton Packaging von
Pro Carton (929 kg/t im europäischen Mittel) und der IFEU-Studie zu Büropapieren
(Frischfaser 1.116 kg/t, Recyclingfaser 933 kg/t).

### Recyclingquoten am Lebensende (`q`)

Die Startwerte sind österreichische Durchschnitte der **stofflichen** Verwertung, nicht der Verwertung
insgesamt. Österreich hat ein Deponieverbot für unbehandelte Abfälle: Was nicht stofflich verwertet wird,
wird verbrannt — genau so rechnet die Seite.

| Gruppe | Startwert | Beleg |
|---|---|---|
| Papier, Karton | 80 % | ARA-Sammelbilanz 2024, Verpackungen |
| Holz, Spanplatte | 65 % | Altholzaufkommen Austria 2022: 989.000 t von 1,14 Mio. t an Recyclinganlagen |
| MDF, HDF | 50 % | dito, aber Faserplatten sind der schwierigste Altholzstrom |
| Stahlblech | 85 % | Näherung; Verpackungsquote 75 %, sauberes Blech liegt darüber |
| Aluminiumblech | 80 % | Näherung; Verpackungsquote 50 %, EU-End-of-Life 69 %, Bauprodukte höher |
| Flachglas | 10 % | Glass for Europe: rund 5 % werden wieder zu Flachglas |
| Kunststoffe | 10–20 % | Umweltbundesamt REP-0744: 26 % aller Kunststoffabfälle stofflich, Plattenware darunter |

Zwei Werte überraschen und sind trotzdem richtig. **Flachglas** steht niedrig, weil die bekannten 86 %
für *Behälterglas* gelten; Fensterglas und Glasplatten kommen beschichtet und gerahmt im Bauschutt an.
**Altholz** steht hoch, weil Österreich sein Altholz überwiegend der Span- und Faserplattenindustrie
zuführt statt dem Heizkraftwerk — anders als Deutschland. Für Bau- und Gewerbeschrott aus Stahl und
Aluminium gibt es keine belastbare österreichische Quote; diese beiden Werte sind Näherungen.

**Sprit- und Strompreis werden nicht automatisch abgerufen.** Eine Abfrage bei der E-Control
bräuchte einen Server oder Build-Job — die Seite ist reines statisches HTML und käme im Browser
nicht an CORS vorbei — und würde am Ergebnis fast nichts ändern: Der Spritpreis wirkt nur auf die
Transportkosten (bei 10 km ein bis zwei Euro neben 20 bis 150 Euro Material) und überhaupt nicht
auf die CO₂-Bilanz. Die Materialpreise, auf die es ankommt, gibt es ohnehin nirgends als
Schnittstelle. Beide Felder sind deshalb Startwerte, die überschrieben und gespeichert werden.

**Aktualitätshinweis:** PlasticsEurope hat die Eco-Profiles für Polyolefine und PVC im März 2026
an neuere Öl- und Gas-Vorkettendaten angeglichen; für die Polymere gehen die Werte dadurch
tendenziell nach oben. Das aktuelle PVC-Profil weist 2,08 kg CO₂e/kg Harz aus, was mit dem
Verarbeitungsaufschlag zu den hinterlegten 2,80 für die Hartplatte passt. Die Polyolefin-Werte
(PE-HD, PE-LD, PP) sind noch gegen die neuen Datensätze abzugleichen.

## Grenzen

Nicht enthalten sind Bedruckung, Kaschierung, Beschläge, Montage und Reinigung.

Die Spalte **Nutzungen** zählt, wie oft ein Stück seinen Zweck erfüllt, bevor es weg ist. Bei
fester Beschilderung sind das die Standjahre, bei Film und Bühne die Einsätze: eine Requisite,
die einmal acht Stunden im Bild steht, hat eine Nutzung — wird sie zwölfmal wiederverwendet,
zwölf. Voreingestellt ist 1, dann entspricht `CO₂ je Nutzung` genau der Bilanz.

Der Teiler ist bei **1 nach unten begrenzt**: ein Stück kann nicht weniger als einmal genutzt
werden, und ein achtstündiger Einsatz darf seinen Fußabdruck nicht auf ein fiktives Jahr
hochgerechnet vervielfachen. Damit bleibt ein Fall bewusst offen — ein Objekt, das dauerhaft
gebraucht wird, aber schneller kaputtgeht als der Bedarf endet (ein Aufsteller für zwei Jahre,
der nur ein halbes hält). Dafür die Projektfläche entsprechend vervielfachen.

Die Seite ist eine Entscheidungshilfe für die Materialwahl, **keine Ökobilanz**. Für eine
vollständige LCA fehlen detaillierte Entsorgungsszenarien, Recyclingprozesse sowie Qualitäts-
und Substitutionsfaktoren; außerdem zieht die Bilanz Herstellung, Lebensende und Modul-D-
Gutschrift zu einer Zahl zusammen, die eine EPD getrennt ausweisen würde.

## Haftungsausschluss

Für die Richtigkeit, Vollständigkeit und Aktualität der berechneten Werte wird keine
Gewährleistung und keine Garantie übernommen. Die Nutzung erfolgt auf eigene Verantwortung;
eine Haftung für Entscheidungen, die auf Grundlage dieser Ergebnisse getroffen werden, ist
ausgeschlossen.

## Korrekturen

Wer für ein Material eine belastbarere Quelle hat — ein Umweltdatenblatt, ein
Lieferantendatenblatt, einen realen Einkaufspreis — oder ein fehlendes Material vermisst:

**Daniel Steinbach** · danielsteinbach@gmx.net · +43 664 421 70 82
