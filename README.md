# Nachkomma

Was in einer Fließkommazahl wirklich steht — **jedes Bit anklickbar**, daneben der exakte
Wert. Nicht gerundet angezeigt, sondern mit BigInt vollständig ausgerechnet:
`0.1` ist in Wahrheit

```
0,1000000000000000055511151231257827021181583404541015625
```

### → [Öffnen](https://ssims437.github.io/nachkomma/)

Eine einzelne HTML-Datei. Kein Build, keine Bibliothek, nichts verlässt den Browser.

---

## Was drin ist

| | |
|---|---|
| **Bitfeld** | Vorzeichen, Exponent, Mantisse einzeln umschaltbar — der Wert folgt sofort |
| **Exakter Wert** | die vollständige Dezimalentwicklung, blau der Teil, den der Browser zeigt, rot der Rest |
| **Nachbarn** | eine Zahl höher, eine tiefer — und der Abstand dazwischen (ULP) |
| **Wo die Zahlen sitzen** | der Abstand über 90 Größenordnungen, und der Punkt, ab dem ganze Zahlen aus dem Raster fallen |
| **Summieren** | vier Verfahren gegen die exakt gerechnete Summe |
| **Selbstprüfung** | 40 404 Fälle, keine Stichprobe |

## Das Stück, auf das es mir ankommt

Dass `0.1 + 0.2 ≠ 0.3` ist, weiß jeder. Interessanter ist, was beim **Aufaddieren vieler
Zahlen** passiert — und dass die verbreitete Faustregel dagegen nicht immer hilft.

**Eine Million Mal 0.1**, erwartet wird 100 000:

| Verfahren | Ergebnis | Fehler in ULP |
|---|---|---|
| naiv, der Reihe nach | 100000.00000133288 | 91 595 |
| paarweise | 99999.99999999977 | 16 |
| Kahan | **100000** | **exakt** |
| sortiert, klein zuerst | 100000.00000133288 | 91 595 |

**10<sup>16</sup> plus eine Million Einsen**, erwartet wird 10 000 000 001 000 000:

| Verfahren | Ergebnis | Fehler in ULP |
|---|---|---|
| naiv, der Reihe nach | 10000000000000000 | 500 000 |
| paarweise | 10000000000999880 | 60 |
| Kahan | **10000000001000000** | **exakt** |
| sortiert, klein zuerst | **10000000001000000** | **exakt** |

Die zweite Zeile ist der Fall, den man gesehen haben sollte: Der Abstand zum Nachbarn
beträgt bei 10<sup>16</sup> genau 2. Jede einzelne Eins ist kleiner als die Hälfte davon
und wird deshalb **weggerundet, eine Million Mal hintereinander**. Am Ende fehlt die
gesamte Million, und kein Zwischenergebnis sah je verdächtig aus.

**Und die Faustregel?** „Klein zuerst sortieren" rettet den zweiten Fall vollständig — und
bringt im ersten **exakt null**. Alle Summanden sind dort gleich groß, es gibt nichts zu
sortieren; der Fehler entsteht daran, dass die *Summe* wächst, nicht die Summanden. Die
Regel hilft gegen ungleiche Größenordnungen, nicht gegen lange Ketten.

Die harmonische Reihe bis 10<sup>6</sup> zeigt beides gemischt: naiv 414 ULP daneben,
sortiert 27, paarweise 1, Kahan 0.

## Beweis statt Behauptung

Eine „exakte" Anzeige ist nur so viel wert wie ihre Prüfung. Auf der Seite steckt ein
Knopf, der das Verfahren gegen sich selbst prüft:

```
40 404 Fälle geprüft, kein einziger falsch · 6,1 s
```

| Was | Umfang | Kriterium |
|---|---|---|
| Exakte Darstellung | 20 000 zufällige Bitmuster | zurückgelesen muss **dasselbe Bitmuster** entstehen |
| Nachbarn | 20 000 Bitmuster | genau ein Muster entfernt, Abstand passt zum ULP |
| Exakte Summe | 400 Reihen | stimmt mit unabhängig gerechneter Bruchsumme überein |
| Kahan gegen naiv | 4 Reihen | darf nie schlechter sein |

Die erste Zeile ist die wichtigste: Wenn die ausgerechnete Dezimalzahl beim Zurücklesen
dasselbe Bitmuster ergibt, kann sie nicht daneben liegen — sonst wäre eine andere Zahl
näher gewesen.

## Wie das exakt geht

Jede binäre Bruchzahl hat eine **endliche** Dezimaldarstellung. Der Wert ist
`m · 2^e` mit ganzzahligem `m`; für negatives `e` gilt

```
m / 2^k  =  m · 5^k / 10^k
```

Zähler mit BigInt ausrechnen, Komma um `k` Stellen schieben, fertig. Kein Runden, keine
Näherung — deshalb sind bei der kleinsten subnormalen Zahl auch alle **1075 Ziffern** da.

## Was mich das gekostet hat

**Der Abstand am oberen Rand war „—".** `ulp(x)` fragt den nächstgrößeren Nachbarn und
zieht ab. Bei der größten endlichen Zahl ist dieser Nachbar unendlich, die Differenz damit
unendlich, und die Anzeige gab auf. Der Abstand ist dort trotzdem definiert — nur eben nach
unten. Ein Sonderfall, der genau an einer Stelle auftritt und den man ohne die
Vorlagen-Liste nie zu Gesicht bekommt.

**Die Faustregel stimmte nicht.** Ich hatte „sortiert, klein zuerst" als Verfahren
aufgenommen in der Erwartung, dass es überall hilft. Bei einer Million gleicher Summanden
liefert es Ziffer für Ziffer dasselbe wie die naive Schleife. Das ist kein Fehler im
Programm, sondern eine falsche Erwartung — und der Grund, warum die Tabelle jetzt vier
Verfahren nebeneinander zeigt statt einer Empfehlung.

**ULP ist das einzige brauchbare Maß.** Ein absoluter Fehler von 0,0000013 klingt winzig
und ist bei 100 000 trotzdem 91 595 Schritte im Zahlenraster. Ein Fehler von 1 000 000
klingt riesig und ist bei 10<sup>16</sup> nur 500 000 Schritte. Erst in ULP werden die
Fälle vergleichbar.

## Benutzen

Datei öffnen genügt — über `file://` funktioniert alles. Wer lieber einen Server mag:

```bash
python -m http.server 8000
```

Jedes Bit ist anklickbar; die Vorlagen-Liste enthält die interessanten Sonderfälle
(subnormal, kleinste und größte Zahl, ±0, Unendlich, NaN, 2<sup>53</sup>+1).

## Lizenz

[MIT](LICENSE) — nimm es, zerleg es, bau was Besseres.

Verwandt: [Plotterblätter](https://github.com/ssims437/plotterblaetter) ·
[Redundanz](https://github.com/ssims437/redundanz) ·
[Reparatur](https://github.com/ssims437/reparatur) ·
[Würfel](https://github.com/ssims437/wuerfel) ·
[Rechenwerk](https://github.com/ssims437/rechenwerk) ·
[Zeitsprung](https://github.com/ssims437/zeitsprung) ·
[Gradtage](https://github.com/ssims437/gradtage)
