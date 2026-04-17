# HILOMOT – Einführung, Motivation und Ansatz

## Eine Lernunterlage für die Hochschule

---

## 1  Ausgangsproblem: Warum brauchen wir lokale Modelle?

In der Praxis begegnen uns ständig **nichtlineare Systeme** – ob Verbrennungsmotoren, chemische Reaktoren oder Roboterkinematiken. Ein einfaches lineares Modell $y = a \cdot x + b$ kann solche Zusammenhänge nicht beschreiben.

**Idee:** Statt ein einziges kompliziertes nichtlineares Modell zu bauen, zerlegen wir das Problem in **viele kleine Teilprobleme**, die jeweils einfach (linear) lösbar sind.

> **Analogie:** Stell dir eine Landkarte vor. Statt die gesamte Erdoberfläche mit einer einzigen Formel zu beschreiben, verwenden wir viele kleine, flache Kartenblätter. Jedes Blatt ist für sich genommen einfach (flach/linear), aber zusammen beschreiben sie die gekrümmte Oberfläche beliebig genau.

Das führt uns zum Konzept der **lokalen Modellnetze:**

$$\hat{y}(x) = \sum_{i=1}^{M} \Phi_i(x) \cdot \hat{y}_i(x)$$

- $\hat{y}_i(x)$: ein einfaches lineares Modell, das in einer bestimmten Region „zuständig" ist
- $\Phi_i(x)$: eine Gewichtungsfunktion, die angibt, **wie stark** Modell $i$ an der Stelle $x$ beiträgt
- Es gilt immer: $\sum_i \Phi_i(x) = 1$ – die Gewichte summieren sich zu 1 (Partition der Eins)

Die zentrale Frage ist nun: **Wie teilen wir den Eingangsraum sinnvoll auf?**

---

## 2  LOLIMOT – Der bewährte Ansatz

LOLIMOT (*Local Linear Model Tree*) beantwortet diese Frage mit einer einfachen Strategie:

**Schneide den Raum immer entlang einer Achse durch.**

Das bedeutet: Jeder Split teilt eine Region in zwei Hälften – immer parallel zu einer Koordinatenachse. Wie beim Schneiden eines Kuchens, aber nur gerade Schnitte, immer horizontal oder vertikal.

**Vorgehen (vereinfacht):**
1. Starte mit einem einzigen Modell für den ganzen Raum
2. Finde die Region mit dem größten Fehler
3. Teile sie in der Mitte – probiere jede Achse durch, nimm den besten Schnitt
4. Trainiere die lokalen Modelle neu
5. Wiederhole, bis der Fehler klein genug ist

**Stärken:** Einfach, schnell, robust, gut verstanden.

**Aber:** Was passiert, wenn die „wahre" Grenze zwischen zwei Verhaltensbereichen **schräg** verläuft?

> **Beispiel:** Ein Motor verhält sich unterschiedlich je nach Kombination von Drehzahl und Last. Die Grenze zwischen „magerem" und „fettem" Betrieb verläuft diagonal im Drehzahl-Last-Kennfeld. LOLIMOT muss diese diagonale Grenze durch viele kleine Treppenstufen (achsenparallele Schnitte) annähern – das kostet **unnötig viele Teilmodelle**.

---

## 3  HILOMOT – Die Weiterentwicklung

### 3.1  Kernidee: Schräg schneiden dürfen

**HILOMOT** (*Hierarchical Local Model Tree*) löst genau dieses Problem. Statt nur achsenparallel zu schneiden, darf die Trennlinie **beliebig orientiert** sein:

| | LOLIMOT | HILOMOT |
|---|---|---|
| **Schnittrichtung** | Nur entlang der Achsen | Beliebig orientiert (schräg) |
| **Trennfläche** | $x_j = \text{const}$ | $w^\top x + b = 0$ (eine Hyperebene) |

> **Analogie:** LOLIMOT schneidet den Kuchen nur waagrecht oder senkrecht. HILOMOT darf auch diagonal schneiden – und trifft damit die natürliche Form des Kuchens oft viel besser.

### 3.2  Der Baum als Organisationsstruktur

HILOMOT organisiert die Aufteilung als **Binärbaum:**

```
            [Wurzel-Gate]
           /             \
     [Gate]              Modell 3
     /    \
Modell 1   Modell 2
```

- **Innere Knoten** = Entscheidungen: „Gehört der Punkt eher links oder rechts?"
- **Blätter** = Lokale lineare Modelle

An jedem inneren Knoten sitzt eine **weiche Entscheidungsfunktion** (Sigmoid/Logistik):

$$g(x) = \frac{1}{1 + e^{-(w^\top x + b)}}$$

Diese Funktion gibt einen Wert zwischen 0 und 1 aus:
- Nahe 1 → der Punkt gehört „eher links"
- Nahe 0 → der Punkt gehört „eher rechts"
- Genau 0,5 → der Punkt liegt auf der Trennfläche

**Wichtig:** Der Übergang ist **weich/glatt**, nicht hart. Es gibt keinen abrupten Sprung an der Grenze – das sorgt für ein glattes Gesamtmodell.

### 3.3  Wie entsteht das Gesamtgewicht eines Blattes?

Das Gewicht eines Blattes ergibt sich aus dem **Produkt aller Entscheidungen auf dem Weg von der Wurzel zum Blatt:**

> Für Modell 1 im Beispielbaum oben:
> - An der Wurzel: „nach links" → Gewicht $g_{\text{Wurzel}}(x)$
> - Am zweiten Knoten: „nach links" → Gewicht $g_{\text{Knoten}}(x)$
> - Gesamt: $\Phi_1(x) = g_{\text{Wurzel}}(x) \cdot g_{\text{Knoten}}(x)$

Dieses Pfadprodukt garantiert automatisch, dass sich alle Blattgewichte zu 1 summieren.

### 3.4  Wie wird HILOMOT trainiert?

Das Training folgt dem gleichen **Wachstumsprinzip** wie LOLIMOT:

1. **Starte einfach:** Ein einziges lineares Modell für alle Daten
2. **Finde das schwächste Blatt:** Wo ist der Fehler am größten?
3. **Spalte es auf:** Ersetze das Blatt durch einen neuen Knoten mit zwei Kindern
4. **Optimiere die Trennfläche:** Finde die beste Orientierung $(w, b)$ des Schnitts – hier liegt der Unterschied zu LOLIMOT:
   - LOLIMOT: Probiere $p$ Achsen durch (diskret, einfach)
   - HILOMOT: Optimiere Richtung und Position kontinuierlich (z. B. Gradientenabstieg)
5. **Trainiere die lokalen Modelle** (gewichtete kleinste Quadrate)
6. **Wiederhole** bis der Fehler akzeptabel ist

---

## 4  Gegenüberstellung: Wann nehme ich was?

| Aspekt | LOLIMOT | HILOMOT |
|---|---|---|
| **Schnitte** | Achsenparallel | Beliebig orientiert |
| **Anzahl Modelle** | Tendenziell mehr (bei schrägen Grenzen) | Oft weniger nötig |
| **Training** | Schnell, einfach | Aufwendiger (nichtlineare Optimierung) |
| **Robustheit** | Sehr robust | Empfindlicher (lokale Optima möglich) |
| **Interpretierbarkeit** | Regionen als Rechtecke leicht vorstellbar | Schräge Regionen etwas abstrakter |
| **Gut geeignet für** | Probleme mit achsennahen Trennungen, schnelles Prototyping | Stark gekoppelte Eingänge, kompakte Modelle |

> **Merksatz:** HILOMOT enthält LOLIMOT als Spezialfall. Wenn man die Trennrichtung $w$ auf Einheitsvektoren $e_j$ einschränkt, wird jeder HILOMOT-Split zu einem LOLIMOT-Split.

---

## 5  Zusammenfassung

```
Nichtlineares System
        │
        ▼
Idee: Zerlege in lokale lineare Modelle
        │
        ├──► LOLIMOT: Achsenparallele Schnitte
        │       ✔ Einfach, schnell, robust
        │       ✘ Ineffizient bei schrägen Grenzen
        │
        └──► HILOMOT: Schräge Schnitte (Hyperebenen)
                ✔ Kompaktere Modelle, flexibler
                ✘ Aufwendigeres Training
```

**Beide Verfahren** erzeugen glatte, interpretierbare Modelle aus lokalen linearen Bausteinen. Die Wahl hängt vom konkreten Problem ab: Wenn die Daten nahelegen, dass die Grenzen zwischen Betriebsbereichen schräg verlaufen, lohnt sich der Mehraufwand von HILOMOT. Für einen schnellen ersten Ansatz ist LOLIMOT oft die bessere Wahl.
