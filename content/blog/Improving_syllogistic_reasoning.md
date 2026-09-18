+++
Title = "Syllogistisches Denken in LLMs verbessern"
description = "Lerne, content effect in LLMs zu verringern um syllogistisches Denken mittels Activation Steering und neuro-symbolischen logischen Löser zu verbessern."
date = 2026-08-11
draft = true
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

Es wurden auch Anstrengungen in verschiedene Strategien zur Minderung dieser Probleme investiert: *activation steering*, eine einfache Maßnahme, die das Schlussfolgern durch die Manipulation interner Aktivierungen beeinflusst [^Valentino_AS], kann die Abhängigkeit von Weltwissen verringern, was wiederum den Inhaltseffekt reduziert. Ein weiterer Forschungsansatz befasst sich mit neuro-symbolischer KI, bei der ein LLM als semantischer Parser mit einer nachgeschalteten Engine für formales Schlussfolgern kombiniert wird. Das LLM liest die Argumente im Klartext und wandelt sie in die Eingabe um, die ein logischer Solver erwartet. Der Solver berechnet dann zuverlässig die logische Korrektheit [^Pan_Slv]. Außerdem wurden Anstrengungen unternommen, das syllogistische Schlussfolgern ausschließlich durch Prompt-Engineering zu verbessern [^Ranaldi_CoT].

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

Der *Cross-plausibility content effect* misst wie sehr 