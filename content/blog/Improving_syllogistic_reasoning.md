+++
Title = "Syllogistisches Denken in LLMs verbessern"
description = "Lerne, content effect in LLMs zu verringern um syllogistisches Denken mittels Activation Steering und neuro-symbolischen logischen Löser zu verbessern."
date = 2026-08-11
draft = false
math = true
toc_open = true
tags = ["LLMs", "Machine Learning", "Activation Steering", "Neuro-symbolic AI", "NLP"]
categories = ["Artificial Intelligence", "Research"]
+++

## Syllogistisches Denken von LLMs verbessern

Large Language Models (LLMs) wissen viel über die Welt, was häufig ihrer Fähigkeit strikte logische Folgerungen zu machen in die Quere kommt, insbesondere wenn diese nicht mit dem Weltwissen kompatibel sind. Ein hervorragendes Beispiel ist **syllogistisches denken**, bei dem LLMs Probleme mit dem so genannten *content effect* haben. Content effect wird das oben beschriebene Phänomen genannt, nämlich wenn eine logisch valide Schlussfolgerung verworfen wird da sie inhaltlich nicht mit dem Weltwissen des LLMs übereinstimmt.  

In diesem Post erläutere ich zwei Methoden, um content effect zu verringern: Activation Steering und neuro-symbolosch logische Löser. Dieser Post basiert auf einem Gruppenprojekt, welches ich zusammen mit 2 Freunden für einen Universitätskurs in Natural Language Processing gemacht habe. Leider hatten wir keine Zeit mehr, unser Projekt für [Aufgabe 11](https://sites.google.com/view/semeval-2026-task-11) der SemEval 2026 einzureichen, auf der unser Projekt basiert hat. Selbstverständlich habe ich vor der Veröffentlichung dieses Posts das Einverständnis meiner Co-Autoren eingeholt. Ein paar Absätze sind direkte Zitate aus unserem Report.

### Was ist syllogistisches Denken?

Eine Schlussfolgerung wird aus mindestens einer Premisse gezogen. Dabei geht es ausschließlich um die logische Validität, unabhängig davon, ob Prämissen oder die Schlussfolgerung in der Realität wahr ist. LLMs vermischen oft faktische und logische Validität, was zu falschen Schlussfolgerungen führt.

{{< alert type="info" title="Beispiel Syllogismus" >}}
Ein Tier welches Milch produziert, ist eine Kuh. Eine Ziege produziert Milch. Daher ist eine Ziege eine Kuh.
{{< /alert >}}

Das obrige Beispiel ist logisch korrekt, aber faktisch nicht, und stimmt nicht mit dem Weltwissen eines LLMs überein. Daher tendieren LLMs dazu, diesen Syllogismus als falsch zu bewerten. Dieses als content effect bekanntes Phänomen ist das schwerwiegenste Problem, welches unsere SemEval Aufgabe lösen sollte. .[^Lampinen_CE] 

[^Lampinen_CE]: See e.g. Lampinen et al., [Language models, like
humans, show content effects on reasoning tasks](http://arxiv.org/abs/2207.07051)

Wir haben uns früh dazu entschlossen, mit kleinen open-weight LLMs zu arbeiten. Zwar haben große state-of-the-art (SOTA) Modelle eine deutlich höhere Grundfähigkeit, aber kleine Modelle sind wichtig für edge-deployments, resourcenbeschränkte Umgebungen und low-latency inferenz. Wir haben *Llama-3.2-1B-Instruct* und *Llama-3.1-8B-Instruct (8 bit)* verwendet.

## Bestehende Literatur zum content effect in LLMs

Das Benchmarking formaler logischer Deduktion und logischer Konsistenz in LLMs hat sich zu einem aktiven Forschungsgebiet der NLP-Forschung entwickelt. In der bisherigen Literatur wird häufig die Anfälligkeit für „Belief Bias“ (z. B. content effect) hervorgehoben, bei denen Modelle semantische Plausibilität gegenüber deduktiver Gültigkeit priorisieren. Darüber hinaus hat sich gezeigt, dass LLMs menschenähnliche Denkfehler produzieren, indem sie beispielsweise *figurativen Effekten* zum Opfer fallen, wie etwa der Sensitivität gegenüber der Reihenfolge von Argumenten und der syntaktischen Anordnung von Variablen. [^Bertolazzi_Inf]  

Es wurden auch Anstrengungen in verschiedene Strategien zur Minderung dieser Probleme investiert: *activation steering*, eine einfache Maßnahme, die das Schlussfolgern durch die Manipulation interner Aktivierungen beeinflusst [^Valentino_AS], kann die Abhängigkeit von Weltwissen verringern, was wiederum den Inhaltseffekt reduziert. Ein weiterer Forschungsansatz befasst sich mit neuro-symbolischer KI, bei der ein LLM als semantischer Parser mit einer nachgeschalteten Engine für formales Schlussfolgern kombiniert wird. Das LLM liest die Argumente im Klartext und wandelt sie in die Eingabe um, die ein logischer Löser erwartet. Der Löser berechnet dann zuverlässig die logische Korrektheit [^Pan_Slv]. Außerdem wurden Anstrengungen unternommen, das syllogistische Schlussfolgern ausschließlich durch Prompt-Engineering zu verbessern [^Ranaldi_CoT].

[^Bertolazzi_Inf]: See e.g. Bertolazzi et al., [A Systematic Analysis of Large Language Models as Soft Reasoners: The Case of Syllogistic Inferences](https://doi.org/10.18653/v1/2024.emnlp-main.769)

[^Pan_Slv]: See e.g. Pan et al., [Logic-Lm: Empowering Large Language Models with Symbolic Solvers for Faithful Logical Reasoning](https://aclanthology.org/2023.findings-emnlp.248/)

[^Valentino_AS]: See e.g. Valentino et al., [Mitigating Content Effects on Reasoning in Language Models through Fine-Grained Activation Steering](https://doi.org/10.48550/arXiv.2505.12189)

[^Ranaldi_CoT]: See e.g. Ranaldi et al., [Improving Chain-of-Thought Reasoning via Quasi-Symbolic Abstractions](https://aclanthology.org/2025.acl-long.843/)

## Der SemEval 2026 Datensatz

Der SemEval 2026 Aufgabe 11 Datensatz besteht aus binären Syllogismen mit je zwei Prämissen und einer Schlussfolgerung. Das LLM muss entscheiden, ob die Schlussfolgerung logisch valide ist. Zusätzlich hat jeder Syllogismus ein *plausability label*, welches angibt, ob die korrekte Schlussfolgerung mit dem Weltwissen eines LLMs übereinstimmt. Dieses Label wurde natürlich nur während dem Training verwendet, nicht während der Evaluation. Die Antwort des LLMs kann mit einem *validity label* überprüft werden.  

Der Datensatz ist nur auf Englisch verfügbar. Da die offizielle Evaluation auch weitere Sprachen beinhaltet, haben wir den Datensatz mit Hilfe von Meta's *facebook/nllb-200-3.3B* in alle Zielsprachen übersetzt. Weitere Zielsprachen waren unter anderem Deutsch und Italienisch, aber auch weniger in Trainingsdaten repräsentierte Sprachen wie Swahili oder Bengali. Insgesamt wird das System auf 12 verschiedenen Sprachen evaluiert.

## Auswertung der Ergebnisse

Die Organisatoren von Aufgabe 11 haben ein Evaluationsskript zur Auswertung bereitgestellt. Es kombiniert eine gewöhnliche Korrektheitsmetrik mit weiteren Metriken, um content effect zu erkennen.  

**Genauigkeit:**  
Genauigkeit ist der Anteil der Antworten bei denen das System die Validität des Syllogismus richtig eingeschätzt hat. Desto höher die Genauigkeit, desto besser.  

**Content Effect Metriken:**  
Außerdem gibt das Skript mehrere Metriken im Zusammenhang mit dem content effect zurück:

- *plausible-valid*: Syllogismen die sowohl plausibel als auch korrekt sind
- *impausible-valid*: Syllogismen die zwar unplausibel, aber korrekt sind
- *plausible-invald*: Syllogismen, die zwar plausibel, aber falsch sind
- *implausible-valid*: Syllogismen die sowohl unplausibel als auch falsch sind

Ein niedriger content effect ist besser.  

Der *Intra-plausability content effect* misst den Bias in Richtung eines spezifischen Labels innerhalb der gleichen Plausibilität, also beispielsweise ob ein System bei plausiblen Syllogismen eher dazu tendiert diese als korrekt zu klassifizieren, auch wenn sie das nicht sind. Er wird berechnet als:

$$
\begin{equation}
\text{Bias}_{\text{intra}} = \frac{1}{2} \left( \left| \text{ACC}_{\text{plausible,valid}} - \text{ACC}_{\text{implausible,valid}} \right| + \left| \text{ACC}_{\text{plausible,invalid}} - \text{ACC}_{\text{implausible,invalid}} \right| \right)
\end{equation}
$$

Der *Cross-plausibility content effect* misst, wie sehr das Modell sich auf die Plausibilität innerhalb eines korrektheit-Labels verlässt:  

$$
\begin{equation}
\text{Bias}_{\text{inter}} = \frac{1}{2} \left( \left| \text{ACC}_{\text{plausible,valid}} - \text{ACC}_{\text{plausible,invalid}} \right| + \left| \text{ACC}_{\text{implausible,valid}} - \text{ACC}_{\text{implausible,invalid}} \right| \right)
\end{equation}
$$

Der *Total content effect* wird aus dem Durchschnitt der beiden Plausibilitäten berechnet:  

$$
\begin{equation}
\text{TCE} = \frac{1}{2} \left( \text{Bias}_{\text{intra}} + \text{Bias}_{\text{inter}} \right)
\end{equation}
$$

Von wo ausgehend der *finale Score* berechnet werden kann:

$$
Score = \frac{\text{accuracy}}{1 + \log(1 + \text{TCE})}
$$

## Ansätze um syllogistisches Denken von Llama 3 zu verbessern

Wir haben zwei Ansätze versucht, deren Resultate im späteren Verlauf miteinander verglichen werden. Der Erste ist Activation Steering, der Zweite einen logischen Löser zusammen mit dem LLM zu nutzen.

### Activation Steering

Die Grundeidee von Activation Steering ist es, die internen Berechnungen eines LLMs zu manipulieren, indem interne Aktivierungen linear in die gewünschte Richtung verschoben werden. Um diese Richtung zu identifizieren, wird ein Satz gelableter Trainingsdaten $D = \{(x_i, y_i)\}_{i=1}^{N}$ benötigt, wobei positive Label logisch korrekte Schlussfolgerungen markieren, und negative Label durch belief bias entstandene negative Schlussfolgerungen.  

In meiner Erfahrung ist die folgende Erklärung für Activation Steering einfach zu verstehen: Stellen Sie sich ein Gehirn vor. Dieses Gehirn wird Medikamenten ausgesetzt, welche Gehirnaktivitäten in bestimmten Arealen verstärken oder verringern. Beispiele für solche Medikamente sind solche gegen Narkolepsie. Das ist was Activation Steering mit LLMs macht. Es beeinflusst die Aktivierungen des LLMs, um dessen Verhalten zu beeinflussen. Activation Steering ist vergleichbar zu dem was Anthropic in ihrem Experiment getan hat, welches [Golden Gate Claude](https://www.anthropic.com/news/golden-gate-claude) hervorgebracht hat.  

Der größte Vorteil von Activation Steering liegt in seiner geringen Rechenintensität. Es bedarf nur weniger Ressourcen um die korrekten Aktivierungen zu identifizieren, deutlich weniger als traditionelles Finetuning. Des Weiteren erhöht es nicht den Rechenaufwand während der Inferenz, und kann auf einer pro-Anfragen Basis angewandt werden.

#### Contrastive Activation Addition (CAA)

Contrastive Activation Addition (CAA) ist eine recht einfache Form des Activation Steerings. Alle Aktivierungen für gewollte und ungewollte Ausgaben werden getrennt voneinander gesammelt. Wenn $\mu^+$ der Durchschnitt der gewollten (positiven) Aktivierungen und $\mu^-$ der Durchschnitt der ungewollten (negativen) Aktivierungen ist, kann ihre Differenz als $\Delta \phi = \mu^{+} - \mu^{-}$ berechnet werden. Während der Inferenz können die Aktivierungen des Modells modifiziert werden, indem eine skalierte Version des Kontrastvektors angewandt wird: $\tilde{\phi}(x) = \phi(x) + \alpha \cdot \Delta \phi$. $\phi$ ist die unmanipulierte Aktivierung und $\tilde{\phi}$ die finale Aktivierung mit angewandtem Steering. Diese Intervention schiebt die Repräsentationen der hidden States in Richtung des Subspaces welcher mit ungebiasten logischen Schlussfolgerungen assoziiert wird.[^Rimsky_CAA]  

[^Rimsky_CAA]: See e.g. Rimsky et al., [Steering llama 2 via contrastive activation addition](https://doi.org/10.18653/v1/2024.acl-long.828)

#### Conditional Activation Steering (CAST)

Conditional Activation Steering (CAST) stellt eine Verbesserung über CAA dar. Im Gegensatz zu CAA subtrahiert CAST auch vom unmanipuliertem Kontrollvektor $\phi$ [^Valentino_AS]. Wir haben Bedingungsvektoren verwendet, um zu bestimmen, ob $\alpha$ positiv oder negativ sein sollte, wobei letzteres die Richtung des Kontrollvektors im Vergleich zu CAA invertiert.  

Wir haben mit zwei Bedingungsvektoren $\psi_{c}^{+}$ und $\psi_{c}^{-}$ gearbeitet, welche die durchschnittliche Summe valider und invalider Aktivierungen sind. Das Vorzeichen von $\alpha$ wird dadurch bestimmt, ob die Aktivierung ähnlicher zum durchschnitt der validen oder invaliden Aktivierungen während dem Training ist.

$$
f \left(\alpha, \phi(x), \psi_c^{+}, \psi_c^{-}\right) =
\begin{cases}
    - \alpha, & \text{ if } \operatorname{sim}(\phi(x), \operatorname{proj}_{\psi_c^{+}} \phi(x)) > \operatorname{sim} (\phi(x), \operatorname{proj}_{\psi_c^{-}} \phi(x)) \\
    \alpha, & \text{otherwise.}
\end{cases}
$$

**k-cast:** Ein Problem mit gewöhnlichen CAST Methoden ist Informationsverlust welcher dadurch entsteht, dass alle validen und invaliden Aktivierungen während dem Training zusammen gerechnet werden. Dadurch gehen Details in der Aktivierung verloren. k-CAST verwendet k-nearest-neighbor clustering, um während der Inferenz vergleichbare Aktivierungen aus dem Training zu finden. Das Vorzeichen von $\alpha$ wird anhand der $k = 5$ nächsten Nachbarn mit den ähnlichsten Aktivierungen bestimmt.

### Neuro-symbolic logischer Löser

Der zweite Ansatz ist es, das LLM mit einem logischen Löser zu verknüpfen. Der Löser überprüft die Logik des Syllogismus, was korrekte Resultate garantiert, wenn die Eingaben korrekt waren. Das LLM muss den Syllogismus in syntaktisch und semantisch korrekte Eingaben für den Löser parsen.  

Wir haben jeden Satz auf exakt einen Qualifier aus `{all, none, some, some_not}` und zwei Prediktoren `(a,b)` gemapped, was zu einer kompakten symbolischen Repräsentation unabhängig von der Sprache des Syllogismus führt.  

![Extrahierung von Eingaben durch das LLM](/images/posts/improving_syllogistic_reasoning/ns-solver_extraction.png?width=600 "Extrahierung von Eingaben durch das LLM" )

#### Dreischrittige Pipeline zur Datenextraktion

Für jeden Syllogismus sollte das LLM einen *quantifier* und zwei *terms* von jedem der drei Sätze extrahieren. Aus allen drei Sätzen gleichzeitig zu extrahieren erhöhte Formatierungsfehler und vermischte gelegentlich Informationen aus mehreren Sätzen, weswegen Extrahierung Satz-für-Satz ausgeführt wurde. Um konsistente Benennung zu fördern, haben wir eine kurze `used_predicates` Liste geführt, welche im Prompt integriert wurde und Wiederverwendung früherer Prädikatennamen gefördert hat.  

Weil der logische Löser unterschiedliche Strings als unterschiedliche Prädikate behandelt, mussten extrahierte Begriffe normalisiert werden. Des Weiteren haben wir den vollständigen Syllogismus in jedem Prompt inkludiert, ebenfalls um stabile Benennung über alle Sätze hinweg zu fördern.  

Schlussendlich validierten wir Ausgaben und gaben sie im 3-tuple Format `(quantifier=..., a=..., b=...)` an den Löser weiter. Syntaktisch inkorrekte extrahierungen waren selten, aber wenn sie auftraten, sind wir auf zufälliges raten zurückgefallen.

#### Schlussfolgerungen mit dem Z3 Löser validieren

Wir haben den [Z3](https://www.microsoft.com/en-us/research/project/z3-3/) Löser für SMT (Satisfiability Modulo Theories) von Microsoft verwendet, um zu bestimmen, ob die Schlussfolgerung korrekterweise aus den Prämissen folgt. Gegeben korrekt geparster Eingaben, gibt Z3 ein binäres Urteil (`valid` oder `invalid`) zurück, welches wir als die finale Ausgabe unserer Pipline verwendeten.

#### Umgang mit mehrsprachigen syllogistischen Daten

Um die durch unsere Aufgabe geforderte Unterstützung verschiedenster Sprachen sicherzustellen, haben wir sprach-abhängiges Preprocessing vom sprach-unabhängigen extrahieren getrennt. Wir detektieren die verwendete Sprache automatisch und trennen die 3 Sätze mittels ICU sentence boundary detection, bevor die Eingaben in die extraction Pipeline gegeben werden. Die besten Ergebnisse wurden mit einem englischen Prompt erzielt, unabhängig von der Sprache des Syllogismus.  

Nach der Extrahierung wurden die Prädikate mit sprachspezifischen Filtern normalisiert (beispielsweise Artikel entfernt). Diese Schritte ergaben in den allermeisten Fällen brauchbare Eingaben für den Solver.

#### Auswahl des richtigen LLMs

Wir haben Modelle der Llama Familie verwendet. Llama-3.2-1B-Instruct stellte sich als unverlässlich für die strukturierte Extraktion heraus (bspw. gibt es häufig `some` statt `some_not` aus), weswegen wir zusätzlich Llama-3.1-8B-Instruct quantisiert auf 8-bit verwendet haben. Beide Modelle sind schnell und brauchen wenige Ressourcen, welches niedriglatente Anwendung in Ressourcen-beschränkten Umgebungen ermöglicht.

## Ergebnisse: Activation Steering vs. Logische Löser

Die Experimente wurden mit 5 verschiedenen Seeds (10, 20, 30, 40, 42) ausgeführt. Für Activation Steering wurde der Datensatz 90/10 zwischen Training und Evaluation aufgeteilt. Da das Training nur aus der Berechnung der Kontrollvektoren besteht, geschieht es schnell. Auf einer einzelnen Nvidia A100 benötigte Training und Evaluation für alle Sprachen sowie nur für englischsprachige Eingaben insgesamt nur wenige Minuten, inklusive der Evaluation des logischen Lösers. Letzterer benötigt kein Training.  

Die Evaluation beinhaltet als Referenz außerdem eine Baseline, in der das LLM nur gepromptet wurde.  

![Diagramm mit den Evaluierungsergebnissen von Activation Steering im Vergleich zu neuro-symbolischen Solvern auf englischen Trainingsdaten.](/images/posts/improving_syllogistic_reasoning/results_english.png?width=600 "Ergebnisse für rein englische Daten. Für Accuracy und Score gilt: Höher ist besser. Für TCE gilt: Niedriger ist besser. Die wichtigste Metrik ist der Score.")

![Diagramm mit den Evaluierungsergebnissen von Activation Steering im Vergleich zu neuro-symbolischen Solvern auf mehrsprachigen Trainingsdaten.](/images/posts/improving_syllogistic_reasoning/results_multi.png?width=600 "Ergebnisse für mehrsprachige Daten. Für Accuracy und Score gilt: Höher ist besser. Für TCE gilt: Niedriger ist besser. Die wichtigste Metrik ist der Score.")

### Activation Steering

Activation Steering ist **dauerhaft besser als die Baseline und neuro-symbolischen Ansätze**. Im ausschließend englischsprachigen Test wurde der beste Score von CAST erreicht, obwohl k-CAST eine höhere *accuracy* erreichte. Wir vermuten, dass es daran liegt, dass k-CAST nicht ausreichend viele Trainingsdaten zur Verfügung hat, um seine das Vorzeichen anhand der 5 nächsten Nachbarn **korrekt** zu bestimmen.  

Im **mehrsprachigen Test ist k-CAST besser als CAST**, vermutlich weil Aktivierung anhand des durchschnittlichen Vektors erheblich ungenauer wird, wenn die Aktivierungen für alle Sprachen summiert werden. Allerdings hat k-CAST mehr Rechenaufwand, da alle Aktivierungen der Trainingsdaten geladen werden müssen, um die k-nearest Nachbarn zu identifizieren, welche das Vorzeichen von $\alpha$ bestimmen.

### Neuro-symbolisch logische Pipeline

Mit Llama 1B konnten wir nur eine geringe Verbesserung gegenüber der Baseline beobachten. Wir vermuten, dass dies hauptsächlich an Fehlern in der Extrahierungspipeline liegt. Llama 8b (quantisiert auf 8-bit) produziert deutlich bessere Ergebnisse, was zeigt, dass die Fähigkeit des LLMs ein wichtiger Faktor für diesen Ansatz darstellt. Noch fähigere LLMs würden vermutlich noch bessere Ergebnisse erzielen.  

- **Llama 1B (Unter den Erwartungen geblieben):** Zeigte kaum Verbesserungen gegenüber der Baseline. Wir führen dies auf Fehler bei der Extraktion und Generierung der Eingaben für den Z3-Solver zurück, da das Modell mit 1B Parametern Schwierigkeiten hatte, Prämissen zuverlässig zuzuordnen.

- **Llama 8B (Verbesserte Genauigkeit):** Verbesserte die Genauigkeit (Accuracy) in beiden Dateneinstellungen erheblich, mit einem deutlichen Anstieg bei den englischen Daten.

- **Total Content Effect (TCE):** Interessanterweise erzielte Llama 1B unter den neuro-symbolischen Constraints einen besseren TCE als das 8B-Modell, obwohl es eine schlechtere Baseline bezüglich des TCE aufwies. Dies deutet darauf hin, dass die logischen Entscheidungen des kleineren Modells weniger von semantischer Plausibilität beeinflusst werden.

## Schlussfolgerung

In unserem Projekt haben wir gezeigt, dass wesentliche Verbesserungen der logischen Inferenz von LLMs mithilfe verschiedener Techniken möglich sind. Für mich persönlich stach Activation Steering als sehr vielseitiges Werkzeug mit geringem Overhead hervor, um ein Modell zu manipulieren. Im Gegensatz zu früheren Experimenten im Bereich Representation Engineering (wie Anthropics Golden Gate Claude), zeigt unsere Arbeit, wie Activation Steering kognitive Schlussfolgerungsfähigkeiten direkt steuern kann. Dies ist eine wichtige Demonstration auf dem Weg zur Spezialisierung von Modellen. In Zukunft würde ich gerne die Erklärbarkeit von KI (AI Explainability) weiter erforschen und untersuchen, wie ein besseres Verständnis genutzt werden könnte, um Modelle sicherer und besser zu machen.