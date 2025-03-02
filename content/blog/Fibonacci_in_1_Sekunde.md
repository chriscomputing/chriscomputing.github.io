+++
title = 'Fibonacci in einer Sekunde'
date = 2025-02-28
draft = false
math = true
+++
Vor einiger Zeit schlug YouTube mir ein [Video](https://youtu.be/LXm6ygZ3h7A?si=2RG1Io6w2f44w2tR) vor, indem der Creator [Sheafification of G](https://www.youtube.com/@SheafificationOfG) versuchte, die größtmögliche Fibonacci Zahl innerhalb einer Sekunde zu berechnen. Das Video diewar Teil einer Serie, in der verschiedene Ansätze und verschiedene Programmiersprachen behandelt wurden. Ich empfehle sehr, die unterhaltenden und spannenden Videos zu schauen. Davon inspiriert beschloss ich, das ebenfalls zu versuchen, allerdings in Java. Nicht weil Java so eine weit verbreitete Sprache für wissenschaftliches Rechnen ist, sondern einfach weil ich länger kein Java mehr geschrieben habe. Meine Implementation befindet sich auf [Github](https://github.com/chriscomputing/fib-java).

## Der naive Ansatz
Die Fibonacci Reihe ist definiert als $fib(n) = fib(n-1) + fib(n-2)$, und $fib(0)= 0, fib(1) = 1$. Daher ist es naheliegend, Fibonacci rekursiv zu implementieren. Daraus ergibt sich der Pseudocode:
```
fib(n):
  if n == 0:
    return 0
  if n == 1:
    return 1
  return fib(n-2) + fib(n-1)
```
Es ist einfach zu sehen, dass dieser Algorithmus nicht effizient ist. Für jede Berechnung müssen alle Fibonacci Zahlen von 0 aus ermittelt werden, sodass dieser Algorithmus eine Laufzeit von $O(exp(n))$ hat.

## Dynamische Programmierung
Die offensichtliche Verbesserung der rekursiven Variante ist, $fib(n-2)$ und $fib(n-1)$ zu speichern um sich viele Rechenschritte sparen zu können. Das ist ein Trick, den viele intuitiv anwenden, wenn sie eine Fibonacci Zahl von Hand errechnen sollen. Im allgemeinen wird die Technik, Ergebnisse vorheriger Schritte wieder zu verwenden, dynamische Programmierung genannt.
```
fib(n):
  a = 0
  b = 1
  for i from 1 to n:
    tmp = a + b
    b = a
    a = tmp
  return a
```
Man könnte meinen, dieser Algorithmus würde Fibonacci in $O(n)$ berechnen, da die Addition in einem einfachen For-Loop stattfindet. Das ignoriert aber, dass die Operation der Addition auf der Bitebene ebenfalls lineare Zeit in Anspruch nimmt. Daher hat dieser Algorithmus (auf Computer angewandt) die Laufzeit $O(n²)$.

## Fast Squaring
Der vorherige Ansatz bietet bereits eine bedeutende Verbesserung, und erschöpft das Verbesserungspotential asympthotischer Laufzeiten in diesem Beitrag. Das bedeutet aber nicht, dass wir kein besseres Ergebnis mehr erzielen können. Zum Glück gilt diese Aussage:

$$
\begin{bmatrix}
    0 & 1 \\ 1 & 1
\end{bmatrix}^n =
\begin{bmatrix}
    F_{n-1} & F_n \\ F_n & F_{n+1}
\end{bmatrix}
$$

Wenn wir also die Matrix $\begin{bmatrix}0 & 1 \\ 1 & 1\end{bmatrix}$ mit $n$ potenzieren, erhalten wir oben rechts und unten links die n-te Fibonacci Zahl. Ein Beweis für diese Aussage findet sich [hier](https://usaco.guide/plat/matrix-expo?lang=cpp). Insbesondere gepaart mit [Exponentiation by Squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) ist es möglich, Fibonacci noch schneller zu berechnen.
```
fib(n):
  M = {{0,1}, {1,1}}
  M = exp_by_squaring(M, n)
  return M[0][1]
```
Exponentiation by Squaring hat den Vorteil, dass die Anzahl der benötigten Rechenoperationen von $O(n)$ im dynamischen Algorithmus auf $O(log(n))$ reduziert wird. Da für die letzte Multiplikation dennoch $O(n^2)$ Bitopertionen benötigt werden, bleibt die asympthotische Laufzeit (für Computer) dennoch gleich.

## Umsetzung in Java
In bester OOP Tradition habe ich ein BaseScheduler und BaseCalculator Interface geschrieben, sodass die konkreten Implementationen problemlos ausgetauscht werden können. Scheduler haben die Aufgabe, in möglichst wenig Schritten ein $n$ zu finden, welches das Programm innerhalb eines Threshholds von 0,002 Sekunden an eine Laufzeit von eine Sekunde bringt. Das ist nötig, weil ich nicht tagelang warten möchte, bis mein Laptop jede Zahl von $1$ bis $n$ ausprobiert hat. Calculator implementieren die bereits vorgestellten Algorithmen, um Fibonacci zu berechnen.

### Die Scheduler
Ich habe 3 Scheduler implementiert. Der Erste, genannt "Graph Scheduler" inkrementiert n um eine vorgegebene STEP_SIZE, bis die 1-Sekunden-Marke erreicht ist. Der Zweite, genannt "Binary Scheduler", führt eine Art binäre Suche aus, um möglichst schnell das korrekte n zu finden. Das funktioniert schon überraschend gut, aber kann noch verbessert werden. Dafür ist der "Ratio Scheduler" da. Dies ist sein Code:

```java
public class RatioScheduler implements BaseScheduler {

    private final double THRESHOLD;

    public RatioScheduler(double threshold) {
        this.THRESHOLD = threshold;
    }

    public long schedule(double time, long previous, double target) {
        if (this.THRESHOLD > Math.abs(time - target))
            return -1;

        double dampener = 1;
        double factor = Math.max(Math.min(target / time, 2.5), 0.5);

        if (factor < 1) {
            dampener = 1.02;
        } else {
            dampener = 0.98;
        }

        return (long) Math.ceil(factor * previous * dampener);
    }
}
```

Ratio Scheduler berechnet das Verhältnis zwischen gemessener Zeit und Zielzeit und leitet daraus ein Verhältnis, mit dem n multipliziert werden müsste ab. Außerdem bezieht Ratio Scheduler dabei einen Dämpfer mit ein, da es ansonsten für Laufzeiten nahe an einer Sekunde zu zu starken Oszillationen kam. 

### Die Calculator
Der Code für die Calculator ist entweder zu langweilig oder zu lang, und die Algorithmen bereits diskutiert. Daher verweise ich an dieser Stelle nur erneut auf das [Github Repository](https://github.com/chriscomputing/fib-java).  
Für die Implementation musste ich BigInteger verwenden, da Javas größter Datentyp, der long, bereits nach der 93sten Fibonacci Zahl überläuft. Ich habe zuerst den rekursiven und den dynamischen Algorithmus implementiert, bevor ich mich Fast Squaring zugewandt habe. Für Letzteres habe ich die [Apache Commons Math Library](https://commons.apache.org/proper/commons-math/) verwendet, welche zahlreiche Methoden für Matrizen bietet. Leider können die Matrizen der Library nicht mit BigIntegern umgehen, sodass ich für diese Implementation BigFraction aus der Commons Math Library verwenden musste, da der Rest meines Codes BigInteger erwartet. Daher beinhaltet der Code für Fast Squaring auch eine eher unschöne Umwandlug in BigInteger.

## Ergebnisse
Damit zum spannenden Teil. Wie schlägt sich Java im Vergleich zu C? Für C verwende ich die [Implementationen von Sheafification of G](https://github.com/SheafificationOfG/Fibsonisheaf). Die folgenden Werte wurden ermittelt, indem ich mein Testprogramm auf meinem Laptop mit den jeweils verschiedenen Calculatorn ausgewählt ausgeführt habe. Das daraus gewonnene n habe ich dann einzeln erneut berechnen lassen und die dabei ermittelte Laufzeit für Java und C festgehalten. Dazu später mehr. Hier sind die Ergebnisse:

| Algorithmus | n | Java | C |
| ----------- | --- | ---- | - |
| Rekursiv | 39 | 1.096616593s | 0.161663483s |
| Dynamisch | 302.463 | 1.630596675s | 0.507222650s |
| Fast Squaring | 2.743.397 | 1.816605545s | 1.511263398s |

Es ist überraschend, wie groß der Unterschied zwischen dem dynamischen Algorithmus und Fast Squaring ist, obwohl beide Asympthotisch die gleiche Laufzeit haben. Das zeigt erneut, dass Laufzeitklassen lange nicht alles über einen Algorithmus aussagen. Weniger überraschend ist, dass C durchweg besser abschneidet als Java. Auffällig ist, dass keine Zeit für Java unter einer Sekunde ist. Wie kommt das? Wie oben bereits erwähnt, wurde $n$ mit Hilfe eines Schedulers in einer Serie von vielen (ca. 20 - 30) Fibonacci Berechnungen ermittelt. In dieser Serie hat mein Laptop $n$ tatsächlich innerhalb einer Sekunde berechnen können. Die hier aufgeführten Zeiten für Java und C wurden ermittelt, indem die Berechnung ein mal mit dem gegebenen $n$ ausgeführt wurde. Den Zeitunterschied erkläre ich vor allem durch predictive branching der CPU. Cache könnte auch eine Rolle gespielt haben. Ausgiebig recherchiert habe ich zu dem Phänomen allerdings nicht.

## Verbesserungsmöglichkeiten
Natürlich gibt es sowohl im Versuchsaufbau, als auch in der Implementation der Algorithmen Verbesserungsmöglichkeiten. Zum Beispiel habe ich die Werte durch einmaliges Ausführen auf meinem Laptop ermittelt, ein hochgradig unwissenschaftliches vorgehen. Es ist auch nicht ideal, dass zwei Implementationen BigInteger und eine BigFraction verwendet. Ich habe nicht getestet ob Apaches BigFraction Implementation effizienter addiert als BigInteger, trotzdem wäre es sicherlich präziser, einen einheitlichen Datentyp zu verwenden. Streng genommen sind weder BigInteger noch BigFraction optimale Datentypen. Beide sind immutable, sodass für jede Berechnung neuer Speicherplatz für das Ergebnis allokiert werden muss. Das ist ein signifikanter Overhead, ohne den mit Sicherheit größere $n$ möglich wären. Da es möglich ist, für ein gegebenes $n$ zu berechnen, wie viel Speicherplatz $fib(n)$ maximal benötigen würde, wäre es am besten, zu Beginn des Algorithmus einmal die nötigen Variablen zu allokieren und dann in-place zu modifizieren. Ich habe Versuche genommen, das umzusetzen. Allerdings ist Java nicht für Manipulationen auf Byte-Level gemacht (zum Beispiel sind Bytes in Java signed, weil sie trotz ihres Namens als kleine Dezimalzahlen betrachtet werden), weswegen ich die Idee aufgrund von Zeitdruck durch Klausuren verworfen habe.
