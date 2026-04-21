# Methoden- und Tool-Dokumentation

## Inhaltsverzeichnis

1. [Einleitung](#einleitung)
2. [Methodenueberblick](#methodenueberblick)
3. [Theorieteil je Methode](#theorieteil-je-methode)
   - [HILOMOT](#hilomot)
   - [LOLIMOT](#lolimot)
   - [MLP](#mlp-multilayer-perceptron)
   - [SVR](#svr-support-vector-regression)
   - [GPR](#gpr-gaussian-process-regression)
4. [Vorteile und Nachteile im Vergleich](#vorteile-und-nachteile-im-vergleich)
5. [Tool-Beschreibung: LMNTool Multi GUI](#tool-beschreibung-lmntool-multi-gui)
   - [Ziel und Einsatzbereich](#ziel-und-einsatzbereich)
   - [Daten und Vorverarbeitung](#daten-und-vorverarbeitung)
   - [Methoden-Tabs und Parameter](#methoden-tabs-und-parameter)
   - [Training und Bewertungslogik](#training-und-bewertungslogik)
   - [Visualisierungen](#visualisierungen)
   - [Report- und Datenexport](#report--und-datenexport)
6. [Empfehlungen fuer die Methodenauswahl](#empfehlungen-fuer-die-methodenauswahl)
7. [Literaturhinweise](#literaturhinweise)

---

## Einleitung

Diese Dokumentation beschreibt die in LMNTool implementierten Regressionsmethoden und erklaert, wie sie im Tool eingesetzt und verglichen werden. Der Fokus liegt auf:

- theoretischen Grundlagen,
- Vorteilen und Nachteilen je Methode,
- praxisnaher Einordnung fuer die Auswahl,
- Bedienung und Exportfunktionen des Tools.

Die Anwendung arbeitet auf 3-spaltigen Datensaetzen mit Zielgroesse y sowie Eingangsvariablen x1 und x2.

---

## Methodenueberblick

Im Tool stehen derzeit diese Verfahren zur Verfuegung:

- HILOMOT (Hierarchical Local Model Tree)
- LOLIMOT (Local Linear Model Tree)
- MLP (Multilayer Perceptron)
- SVR (Support Vector Regression, RBF)
- GPR (Gaussian Process Regression)

Alle Methoden werden anhand einheitlicher Kennzahlen verglichen (u. a. MSE, MAE, Korrelation, R2, numerische Effizienz, Interpretierbarkeit und Extrapolations-Linearitaet).

---

## Theorieteil je Methode

### HILOMOT

HILOMOT ist ein hierarchisches lokales Modellnetz. Der Eingangsraum wird rekursiv in Regionen aufgeteilt; in jeder Region wird ein lokales lineares Modell geschaetzt. Die globale Vorhersage ergibt sich als gewichtete Summe lokaler Modelle.

Grundstruktur:

$$
\hat{y}(\mathbf{x}) = \sum_{i=1}^{M} \Phi_i(\mathbf{z})\,\hat{y}_i(\mathbf{x})
$$

mit lokaler linearen Form:

$$
\hat{y}_i(\mathbf{x}) = \boldsymbol{\theta}_i^T \tilde{\mathbf{x}}, \quad \tilde{\mathbf{x}}=[1,x_1,\dots,x_n]^T
$$

Charakteristisch fuer HILOMOT sind oblique Splits (schraege Trennflaechen). Dadurch kann die Methode auch diagonale Strukturen gut darstellen, oft mit weniger Regionen als achsenparallele Verfahren.

Typische Eigenschaften:

- lokale Modellguete plus globale Glattheit durch weiche Gueltigkeitsfunktionen,
- gute Interpretierbarkeit ueber Partitionen und lokale lineare Modelle,
- modellseitige Komplexitaetskontrolle ueber max_leafs, min_samples_per_leaf, smoothness.

### LOLIMOT

LOLIMOT ist konzeptionell eng mit HILOMOT verwandt, nutzt aber achsenparallele Splits. Das heisst, Trennungen erfolgen entlang einzelner Eingangsachsen.

Vereinfachte Split-Idee:

$$
\mathbf{w}=\mathbf{e}_d \Rightarrow \mathbf{w}^T\mathbf{z}+b=0
$$

Dadurch ist LOLIMOT meist einfacher und stabiler zu verstehen, kann aber fuer schraeg orientierte Zusammenhaenge mehr lokale Modelle benoetigen.

Typische Eigenschaften:

- hohe Transparenz der Partitionierung,
- gute Kontrolle ueber lokale Modellkomplexitaet,
- bei stark nichtachsenparallelen Daten ggf. geringere Effizienz als HILOMOT.

### MLP (Multilayer Perceptron)

Das MLP ist ein neuronales Netz mit mehreren vollvernetzten Schichten und nichtlinearen Aktivierungen. Es approximiert komplexe nichtlineare Abbildungen direkt als globale Funktion.

Schematisch:

$$
\mathbf{h}^{(l+1)} = \sigma\big(W^{(l)}\mathbf{h}^{(l)} + \mathbf{b}^{(l)}\big), \quad
\hat{y}=W^{(L)}\mathbf{h}^{(L)}+b^{(L)}
$$

Training erfolgt ueber Gradientenverfahren (Backpropagation, im Tool via scikit-learn MLPRegressor).

Typische Eigenschaften:

- hohe Flexibilitaet bei komplexen Zusammenhaengen,
- gute Fit-Qualitaet bei genug Daten und sinnvoller Regularisierung,
- geringere Interpretierbarkeit als lokale Modellnetze.

### SVR (Support Vector Regression)

SVR basiert auf dem Maximum-Margin-Prinzip. Statt jeden Fehler gleich zu bestrafen, arbeitet SVR mit einer epsilon-insensitiven Verlustfunktion und sucht eine robuste Funktion mit kontrollierter Komplexitaet.

Optimierungsprinzip (vereinfacht):

$$
\min_{\mathbf{w},b,\xi,\xi^*} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_i(\xi_i+\xi_i^*)
$$

unter Nebenbedingungen zur epsilon-Zone.

Mit RBF-Kernel lassen sich starke Nichtlinearitaeten abbilden:

$$
K(\mathbf{x},\mathbf{x}') = \exp(-\gamma\|\mathbf{x}-\mathbf{x}'\|^2)
$$

Typische Eigenschaften:

- oft sehr robust bei kleinen bis mittleren Datensaetzen,
- gute Generalisierung bei sauber gewaehlten Hyperparametern,
- weniger intuitiv interpretierbar, Hyperparameterabstimmung relevant.

### GPR (Gaussian Process Regression)

GPR ist ein probabilistisches, nichtparametrisches Verfahren. Statt nur einen Punktwert liefert es eine Verteilung ueber Funktionen. Kernidee: Werte, deren Eingaben aehnlich sind, sollen korrelierte Ausgaben haben.

Mit Kernelmatrix K und Rauschterm sigma_n^2:

$$
\hat{f}(\mathbf{x}_*) = k_*^T (K + \sigma_n^2 I)^{-1}\mathbf{y}
$$

GPR liefert zusaetzlich Unsicherheitsschaetzungen (im Tool liegt der Fokus aktuell auf der Praediktion).

Typische Eigenschaften:

- sehr gute Performance bei kleinen bis mittleren Datenmengen,
- mathematisch gut fundierte Unsicherheitsmodellierung,
- Rechenaufwand skaliert kubisch mit Stichprobenzahl im Standardfall.

---

## Vorteile und Nachteile im Vergleich

### HILOMOT

Vorteile:

- gute Balance aus Genauigkeit und Interpretierbarkeit,
- kann diagonale Strukturen mit obliquen Splits effizient erfassen,
- physikalisch oft gut plausibilisierbar ueber lokale lineare Teilmodelle.

Nachteile:

- mehr Hyperparameter als bei einfachen globalen Modellen,
- Baumwachstum ist greedy (lokal beste Entscheidung je Schritt),
- bei sehr verrauschten Daten kann Feintuning noetig sein.

### LOLIMOT

Vorteile:

- sehr gut interpretierbar durch achsenparallele Regionen,
- robuste und oft stabile Modellbildung,
- einfacher zu erklaeren als komplexe Black-Box-Modelle.

Nachteile:

- ggf. mehr Blaetter fuer schraeg orientierte Datenstrukturen,
- kann bei stark gekruemmten Zusammenhaengen schneller an Grenzen kommen,
- Extrapolation bleibt wie bei lokalen Netzen grundsaetzlich datenabhaengig.

### MLP

Vorteile:

- hohe Modellflexibilitaet,
- kann komplexe Nichtlinearitaeten gut approximieren,
- in vielen Szenarien starke Fit-Ergebnisse.

Nachteile:

- geringe Interpretierbarkeit,
- sensitiv gegen Hyperparameter und Datenvorbereitung,
- Gefahr von Overfitting bei zu hoher Modellkapazitaet.

### SVR

Vorteile:

- starke Generalisierung bei begrenzten Daten,
- robustes Lernprinzip durch Margin und epsilon-Zone,
- mit RBF-Kernel leistungsfaehig fuer nichtlineare Muster.

Nachteile:

- Hyperparameter C, gamma, epsilon beeinflussen Ergebnis stark,
- bei sehr grossen Datensaetzen rechenaufwaendig,
- direkte Modellinterpretation eingeschraenkt.

### GPR

Vorteile:

- probabilistische Grundlage,
- gute Genauigkeit bei kleinen bis mittleren Daten,
- grundsaetzlich Unsicherheitsabschaetzung moeglich.

Nachteile:

- hohe Laufzeit- und Speicherkosten fuer grosse Datenmengen,
- Kernelwahl beeinflusst Ergebnis deutlich,
- bei sehr grossen N oft nur mit Approximationen praktikabel.

---

## Tool-Beschreibung: LMNTool Multi GUI

### Ziel und Einsatzbereich

LMNTool Multi GUI ist ein Vergleichs- und Analysewerkzeug fuer mehrere Regressionsverfahren auf demselben Datensatz. Es unterstuetzt sowohl datengetriebene Modellwahl als auch dokumentierbare Entscheidungsprozesse.

### Daten und Vorverarbeitung

- CSV-Import mit 3 Spalten (y, x1, x2),
- numerische Typisierung der Daten,
- automatische Aktivierung der Trainings- und Exportfunktionen nach erfolgreichem Laden,
- initiale 3D-Datenansicht direkt nach Import.

### Methoden-Tabs und Parameter

Das Tool bietet je Methode einen eigenen Parameter-Tab:

- HILOMOT: z. B. max_leafs, min_samples_per_leaf, smoothness, demanded_membership_value, oblique,
- LOLIMOT: achsenparallele Variante mit analogen Kernparametern,
- MLP: hidden_layer_sizes, alpha, learning_rate_init, max_iter,
- SVR: C, gamma, epsilon,
- GPR: length_scale, noise_level, n_restarts_optimizer, max_fit_samples.

Zusaetzlich gibt es einen Decision-Weights-Tab zur interaktiven Gewichtung der Kriterien:

- Accuracy,
- Extrapolation Linearity,
- Numerical Efficiency,
- Interpretability.

Die Gewichte werden normalisiert und fuer die modelluebergreifende Empfehlung verwendet.

### Training und Bewertungslogik

- Einzeltraining je Methode oder Batch-Training aller Methoden,
- Cross-Validation (konfigurierbare Fold-Zahl),
- automatische Kennzahlenberechnung (MSE, MAE, Corr, R2),
- Zusatzmetriken fuer Extrapolationsverhalten, numerische Effizienz (geschaetzte FLOPs) und Interpretierbarkeit,
- automatische Anzeige des normalisierten Vergleichsplots nach Train-All-Abschluss.

### Visualisierungen

Verfuegbare Plottypen in der interaktiven Canvas:

- 3D Surface,
- Training Data 3D,
- Partition Map (fuer HILOMOT/LOLIMOT),
- Correlation Plot,
- Tilt Angle Cuts,
- Evaluation Bars,
- Evaluation Bars (Normalized).

Hinweis zur normierten Darstellung: Werte werden auf [0,1] normiert, wobei 1 jeweils den besten Methodenwert je Kriterium repraesentiert. Das Effizienz-Kriterium ist als Efficiency mit invertierter Richtung modelliert (niedrigere FLOPs sind besser).

### Report- und Datenexport

Unter Export stehen mehrere Formate bereit:

- Comparison Report (JSON + CSV + TXT),
- Comparison Report (LaTeX),
- Comparison Report (PDF via LaTeX),
- Comparison Report (JSON + CSV),
- Comparison Report (JSON),
- Comparison Report (CSV),
- Comparison Report (Text),
- HILOMOT Parameters (CSV).

Der dedizierte HILOMOT-Parameterexport schreibt eine tabellarische CSV mit mindestens:

- method,
- max_leafs,
- min_samples_per_leaf,
- smoothness,
- demanded_membership_value,
- oblique,
- trained_leaves,
- prediction_ops,
- interpretability_score.

---

## Empfehlungen fuer die Methodenauswahl

- Wenn Interpretierbarkeit und lokale Struktur wichtig sind: HILOMOT oder LOLIMOT.
- Wenn diagonale/gekoppelte Strukturen dominant sind: HILOMOT mit oblique Splits bevorzugen.
- Wenn datengetriebene Genauigkeit im Vordergrund steht und Interpretierbarkeit zweitrangig ist: MLP oder SVR pruefen.
- Bei kleineren Datensaetzen mit Bedarf an probabilistischer Modellierung: GPR ist oft stark.
- Fuer robuste Entscheidung immer mehrere Kriterien gemeinsam betrachten (Accuracy, Linearity, Efficiency, Interpretability) und Cross-Validation mit einbeziehen.

---

## Literaturhinweise

- Nelles, O. Nonlinear System Identification: From Classical Approaches to Neural Networks and Fuzzy Models.
- Vapnik, V. The Nature of Statistical Learning Theory.
- Rasmussen, C. E., Williams, C. K. I. Gaussian Processes for Machine Learning.
- Goodfellow, I., Bengio, Y., Courville, A. Deep Learning.

---

Dieses Dokument ist als methodenuebergreifende Ergaenzung zur bestehenden HILOMOT-Dokumentation gedacht.