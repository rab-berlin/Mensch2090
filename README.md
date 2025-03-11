# Mensch2090

Mensch-ärgere-dich-nicht auf dem Busch Microtronic 2090...

![Mädn](/pics/2090maedn.jpg)

## tl;dr

Ich will spielen! Na klar, dann lies kurz die _offiziellen Spielregeln_ (nächster Abschnitt) und beachte folgendes:

- Du brauchst ein Spielbrett. Ich hatte keins und hab mir eins selbst gemalt. Und Pöppel für zwei Spieler brauchst du auch.
- Programmstart mit **HALT - NEXT - 00 - RUN**. Erneuter Start nur mit RESET-Taste (weil auch die Pöppel im Speicherregister gelöscht werden müssen)
- Die LED an Ausgang 4 zeigt, wer dran ist. Wenn sie leuchtet, ist der Mensch dran. Wenn nicht, zieht der 2090.
- Zu Beginn leuchtet sie nicht, also fängt der Microtronic immer an.
- Die gewürfelte Augenzahl wird einstellig angezeigt.
  - Ist der Microtronic dran, berechnet er seinen Zug nach dem Wurf automatisch.
  - Ist der Mensch dran, wählt er nach dem Wurf, welcher Pöppel gezogen werden soll (Taste 1 - 4). Ungültige Züge werden nicht akzeptiert.
- Der Zug wird nach Berechnung (2090) oder Auswahl (Mensch) zweistellig angezeigt in der Form **XY**: **Pöppel X** zieht **Y Felder** weiter.
  - Die Anzeige **45** bedeutet z.B., **Pöppel 4** (der weitest fortgeschrittene) zieht **5 Felder** vor.
- Wenn jemand gewonnen hat, hält das Programm an.

Tipp: Achte gut darauf, jeden Zug auf dem Spielbrett richtig auszuführen. Wenn was nicht stimmt, müsstest du mühsam in allen Registern nachsehen, wo die einzelnen Pöppel wirklich stehen (das geht mit dem Microtronic NextGen ganz gut, mit dem echten Microtronic ist das schon schwieriger, insbesondere die Speicherregister lassen sich nicht so einfach überprüfen).


## Spielregeln

Die _offiziellen Spielregeln_ von *Mädn* dürften viele überraschen. Mich eingeschlossen...

- Zu Beginn sind nur 3 Pöppel im Häuschen, **ein Pöppel steht bereits auf dem Startfeld**.
- Dafür entfällt das 3-malige Würfeln, um eine 6 zu bekommen, generell.
- Im Ziel dürfen eigene Pöppel übersprungen werden.

So steht es geschrieben! Wer´s nicht glaubt, möge nachsehen. Ja, ich weiß. Gelernt haben wir alle die "Familienregeln".

Tatsächlich erleichtern diese Regeln die Implementierung beträchtlich. Insbesondere das "3x Würfeln" ist schwierig umzusetzen, da immer beachtet werden muss, ob noch irgendwo eine theoretische Zugmöglichkeit besteht, bevor man dreimal versuchen darf, eine 6 zu würfeln.

## Spielbrett

Das Spielbrett hat - angefangen vom Startfeld bis zur letztmöglichen Position im Ziel - insgesamt 44 Felder. Da der Microtronic 2090 Hexadezimalzahlen liebt, wollen wir ihn nicht schon vor Spielbeginn mit Dezimalzahlen ärgern. Hexadezimal geht die Reise eines einzelnen Pöppels also 

- von Feld 00 (im Häuschen)
- über Feld 01 (Startfeld) bis zu
- maximal Feld 2C (letztes Feld im Ziel, entspricht dezimal 44).

## Pöppel (Püppchen, Figur, Spielfigur...)

Die Positionen der vier Pöppel auf dem Spielbrett (P1-P4) liegen in den Registern 0-7:

- P1 in den Registern 0 und 1
- P2 in den Registern 2 und 3
- P3 in den Registern 4 und 5
- P4 in den Registern 6 und 7

P4 soll immer der weitest fortgeschrittene Pöppel sein, P1 hingegen immer der hinterste Pöppel. Das bedeutet, dass die Bezeichnungen wechseln können, wenn sich die Positionen der Pöppel relativ zueinander verändern. Dazu ein Beispiel:

- P1 und P2 sind noch im Häuschen, also beide auf Feld 00.
- P3 steht auf Feld 01 (Startfeld).
- P4 steht auf Feld 04.

```
Pöppel      Register        Feld
  P1          0  1          0  0
  P2          2  3          0  0
  P3          4  5          1  0
  P4          6  7          4  0
```

Nun würfelt der Spieler eine 5. Weil noch Pöppel im Häuschen sind, darf P4 nicht ziehen, P3 muss das Startfeld räumen.
```
Pöppel      Register        Feld
  P1          0  1          0  0
  P2          2  3          0  0
  P3          4  5          6  0              Feld 01 + 5 Augen = Feld 06 
  P4          6  7          4  0
```

Nun müssen die Pöppel neu geordnet werden, denn P3 ist - durch den Wurf - inzwischen auf dem Spielbrett weiter vorne als P4. Und diese Aufgabe erledigt...
 
## Das gute alte Bubblesort

Schon vor Beginn der Programmierung von _Mädn_ hatte ich den Gedanken, dass es für viele Tests und Funktionen vorteilhaft sein könnte, wenn die Pöppel eines Spielers immer in einer geordneten Reihenfolge, also sortiert wären.

Ein paar Beispiele, wann und wo stets geordnete Pöppel nützlich sein können:
- Um festzustellen, ob noch Pöppel im Häuschen sind, genügt es, den Pöppel P1 zu testen. Ist sein Feld 00, dann ist mindestens ein Pöppel noch im Häuschen.
- Um festzustellen, ob ein Spieler gewonnen hat, muss nur geprüft werden, ob der letzte Pöppel (P1) gerade das Feld 29 hex erreicht hat. Da alle anderen Pöppel (P2-P4) in der Sortierung vor ihm liegen, müssen sie bereits im Ziel sein.
- Alle Pöppel ziehen nur in eine Richtung (vorwärts, vorwärts, vorwärts!), also kann ein potentieller Zug nur durch diejenigen eigenen Pöppel blockiert sein, die *vor* der ziehenden Spielfigur stehen. 

Also war der erste Schritt - nur so zum Spaß - einen Sortier-Algorithmus auf dem Microtronic zu implementieren. 

Warum nicht Bubblesort?! Der Algorithmus ist einfach und verhältnismäßig kurz an Programmschritten, wenn man auf schrittweise Reduzierung der Schleifenlänge nach jedem Durchlauf verzichtet. Um 4 Pöppel zu sortieren, muss 3x eine Schleife durchlaufen werden, in der jeweils das größte Element nach oben steigt.

![Bubblesort als Programm für den 2090](/program/Bubble.pdf)

## Hammses nich ne Numma kleena?

In Berlin ein geflügeltes Wort, wenn man ausdrücken will, dass eine Sache möglicherweise übertrieben ist. Ist der volle Bubblesort-Algorithmus vielleicht gar nicht nötig? 

Es stellte sich heraus... Ja, es geht auch ne Numma kleena. Dazu musste ich die zu sortierende Pöppel-Menge genauer betrachten und feststellen, dass diese ja immer teil-sortiert ist (und sich insoweit von völlig unsortierten Mengen unterscheidet). Nur zwei Ereignisse können die Reihenfolge der Sortierung verändern: 

- eine eigene Figur zieht
- eine fremde Figur wird geschlagen

Wenn eine eigene Figur zieht, vergrößert sich nur das Feld eines einzigen Pöppels um die gewürfelte Augenzahl. Daher reicht ein einziger Durchlauf:

- P1 > P2?
  - Ja, dann tauschen
- P2 > P3?
  - Ja, dann tauschen
- P3 > P4?
  - Ja, dann tauschen

wonach die Reihenfolge aller Pöppel wieder korrekt, also aufsteigend ist. Drei Bubblesort-Durchläufe sind gar nicht nötig. 

Wenn eine fremde Figur geschlagen wird, wird dieser Pöppel auf Feld 00 gesetzt. Bubblesort müsste hier - schlimmstenfalls - 3x durchlaufen, bis die Reihenfolge wieder korrekt, also Feld 00 ganz nach unten in Register 0 und 1 gewandert ist. Einfacher (und schneller) geht's so:

```
Schlagen	CALL Umrechnung	        Zielfeld umrechnen in gegnerische Feldnummerierung
	        CALL Korrektur	
                EXRL                    gegnerische Pöppel einblenden
        	CALL TestAbP4	        Zielfeld und P4 gleich?
	        BRZ SchlagP4	        dann hau P4 weg
	        CALL TestAbP3	        Zielfeld und P3 gleich?
	        BRZ SchlagP3	        dann hau P3 weg
	        CALL TestAbP2	        Zielfeld und P2 gleich?
	        BRZ SchlagP2	        dann hau P3 weg
	        CALL TestAbP1	        Zielfeld und P1 gleich?
                BRZ SchlagP1	        dann hau P1 weg
	        GOTO SchlagenEnde	keine gegnerischen Pöppel zu schlagen
SchlagP4	MOV P32,P42	
	        MOV P31,P41	
SchlagP3	MOV P22,P32	
	        MOV P21,P31	
SchlagP2	MOV P12,P22	
	        MOV P11,P21	
SchlagP1	MOVI #0,P12	
	        MOVI #0,P11	
SchlagenEnde	EXRL                     eigene Pöppel wieder einblenden
```

Anstatt Bubblesort den geschlagenen Pöppel in drei Durchläufen nach unten sortieren zu lassen, rücken einfach die nachfolgenden Pöppel um eine Position auf - und am Schluss wird P1 auf 00 gesetzt.

## Relativitätstheorie

Jeder Spieler, Mensch oder Maschine, sieht erstmal nur sein eigenes Spielbrett. Das Häuschen ist also immer 00, das Startfeld immer 01, das letzte Feld vor den vier Positionen im Ziel immer 28 hex. Dadurch können die Programmteile für beide Spieler identisch bleiben. 

Nur beim möglichen und tatsächlichen Schlagen wird es nötig, die beiden Bezugssysteme ineinander umzurechnen. Dazu werden die Unterprogramme Umrechnung und Korrektur verwendet. 

Umrechnung addiert zur Pöppel-Position einfach immer ein "halbes Brett" (hexadezimal 14 Felder). Ein eigener Pöppel beispielsweise, der mit einer 6 aus dem Haus kommt und auf dem Startfeld 01 landet, steht für den gegnerischen Spieler ja auf Feld 15 hex, also 01 + 14 = 15 hex. 

Das Ganze funktioniert aber nicht mehr, wenn die eigene Position größer als ein halbes Brett ist (Position >= 15 hex), dann muss nach Umrechnung noch eine Korrektur um ein ganzes Brett erfolgen.

Zum Beispiel landet der eigene Pöppel nach einem (unglücklichen) Wurf direkt auf dem Startfeld des Gegners. Das lässt allgemein die Nervosität steigen, muss aber korrekt berechnet werden:

```
Umrechnung:        15 + 14 = 29         (15 = eigene Feldnummer)
Korrektur:         29 - 28 = 01         (01 = gegnerische Feldnummer)
```

## Würfel

![Microtronic würfelt](/pics/wuerfelt.jpg)

Das erste "eigene" Programm des Microtronic war ein "elektronischer Würfel" (Anleitungsbuch 1. Teil, Seite 11). Und ich kann mich noch gut erinnern, wie begeistert ich war. Der würfelt, guck mal! Ich habe bestimmt stundenlang immer neue Zahlen gewürfelt, damals Weihnachten 1981, und alle genervt, sie sollen auch mal auf den Knopf drücken...

![Allererstes Programm](/pics/erstes.jpg)

Mnemonisiert und leicht gekürzt:

```
Würfel	RND
	CMPI #0,AUGEN
	BRZ Würfel
	CMPI #6,AUGEN
        BRC Würfel
        ...
```


Über 40 Jahre war ich der Überzeugung, kürzer geht's nicht.

Geht's aber doch! Ein Vergleich mit 0 ist nicht nötig, stattdessen addieren wir einfach 1 zu einer Zufallszahl zwischen 0 und 5:

```
Würfel	RND
	CMPI #5,AUGEN
	BRC Würfel
	ADDI #1,AUGEN
        ...
```
Kleinvieh macht auch Mist, spart einen Befehl.

## Zugliste


geht's


## Strategie

Auf den ersten Blick sollte eine Strategie für Mädn relativ einfach zu formulieren sein. Wir haben ein Spiel mit vollständiger Information und einem eng begrenzten Regelwerk. Wir haben allerdings auch nur 256 Programmschritte :-)

Ich will nicht alles verraten, spiel doch einfach ein paar Runden mit dem 2090. Dann dürfte der Masterplan des Microtronic ziemlich rasch deutlich werden - eine Mischung aus Krümelmonster und Lauf, Forrest, lauf. 

Komplexere Strategien lassen sich leider nicht verwirklichen, also läuft der Microtronic öfter mal äußerst mutig direkt "vor die Flinte". Schlägt aber im Gegenzug auch erbarmungslos alles, was in Reichweite ist.

Mich hat er jedenfalls mehrmals brutalst abgezockt... 

## Ein Königreich für ein Register

Wer ist gerade dran, Computer oder Mensch? Irgendwo muss diese Information hinterlegt werden, und diese Information muss über alle Spielzüge erhalten bleiben. Das ist angesichts des begrenzten Register-Speichers gar nicht so einfach.

Der Microtronic hat nur 16 Arbeitsregister (und 16 Speicherregister, die wir in die Arbeitsregister einblenden können). Für alle vier eigenen Pöppel werden die Register 0-7 genutzt. Die gegnerischen Pöppel residieren in den entsprechenden Speicherregistern und werden bei Spielerwechsel einfach "aktiv" geschaltet mit EXRL. Dadurch kann ich im wesentlichen die gleichen Programmteile für beide Spieler nutzen. 

Zusätzlich brauchen wir für das Würfeln mit RND die Register D-F. Alles, was in diesen Registern vorher war, ist nach dem Würfeln der unaufhaltsamen Macht der Entropie zum Opfer gefallen (also weg). 

Die vier möglichen Züge (Zugliste) und der tatsächlich ausgeführte Zug teilen sich den Platz im Register E.

Das Register F steht als Hilfsregister für allerlei Zahlenwerk zur Verfügung. Zum Beispiel muss die Bubblesort-Schleife dreimal durchlaufen werden, das wird im Hilfsregister F abgezählt. 

Die Register 8 und 9 (Zielfeld) sowie die Register A und B (Testfeld) brauchen wir für Vergleiche, also werden diese Register auch überschrieben. Was immer da drin war oder ist, überlebt den Spielzug nicht.

Übrig bleibt als einziges Register C. Aber das brauchen wir auch noch für ein Flag zur Erkennung, ob das Startfeld geräumt werden muss (dazu mehr unter Test0).

Also alle Register belegt :-( Woher noch ein zusätzliches Register nehmen (dessen Inhalt den Spielzug übersteht)?

Man könnte natürlich ein Speicherregister dazu nutzen: Vor dem Spielzug mit MAS (move Arbeitsregister in Speicherregister) sichern und später mit EXRM, MAS und nochmal EXRM den Inhalt zurückholen. Braucht aber 4 wertvolle Programmschritte für jede Abfrage.

Geht's einfacher?

Ja. Wir nutzen die Ausgänge als "zusätzliches" Register. Mit DOT wird der Registerinhalt 0 oder 8 (Computer ist dran oder Mensch ist dran) auf die Ausgänge gelegt, also Ausgang 4 auf High oder Low geschaltet. Mit DIN können wir dann diesen Inhalt wieder einlesen, wann und wo immer es nötig ist - vorausgesetzt, wir haben ein Kabel von Ausgang 4 zu Eingang 4 gelegt. Im Prinzip haben wir damit einen Stack der Größe 1 realisiert.

## Multifunktional...

Funktionen im klassischen Sinne kennt der Microtronic nicht. Will man etwas Vergleichbares programmieren, müsste man gezielt ein bestimmtes Register mit dem Rückgabewert füllen und dieses Register nach Rückkehr aus dem Unterprogramm (also der "Funktion") auswerten. Dieses Auswerten wäre allerdings dann wieder mit zahlreichen Vergleichen verbunden; die Programmschritte würden schneller dahinschmelzen als ein Ed-von-Schleck in der Sonne.

Allerdings gibt es das Carry- und Zero-Flag (bzw. die totale Abwesenheit von Flags). Theoretisch also drei Zustände als Rückgabewerte einer Funktion. Praktisch nutzt das Unterprogramm Test0 diese Möglichkeiten:

```
Test0               CMPI #0,ZIELFELD2
	            BRC EndeTest0
	            CMPI #1,ZIELFELD1
EndeTest0	    RET
```

Test0 soll folgendes zurückgeben: 
- ob der Pöppel noch im Haus ist (Feld 00)
- oder ob der Pöppel auf dem Startfeld steht (Feld 01)
- oder ob der Pöppel "normal" auf dem Brett steht (Feld >= 02)

Und macht das auch:
- Kein Flag, wenn noch im Haus,
- Zero-Flag, wenn auf Startfeld,
- Carry-Flag, wenn normal auf dem Brett

## Gewinnerkennung

war geradezu lächerlich einfach. Durch die vor jedem Zug erfolgte Sortierung der Pöppel muss nur ein Kriterium überprüft werden: Ist der letzte Pöppel (P1) im Ziel, also auf Feld 29 hex angekommen? Falls ja, dann gewonnen. Fertig.
