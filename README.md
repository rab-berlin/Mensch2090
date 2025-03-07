# Mensch2090
Mensch-ärgere-dich-nicht auf dem Busch Microtronic 2090. Work in progress...

![Mädn](/pics/2090maedn.jpg)

## Spielbrett

Das Spielbrett hat - angefangen vom Startfeld bis zur letztmöglichen Position im Ziel - insgesamt 44 Felder. Hexadezimal geht die Reise eines einzelnen Pöppels also von Feld 01 bis zu (maximal) Feld 2C.

Die Positionen der vier Pöppel auf dem Spielbrett (P1-P4) liegen in den Registern 0-7:

- P1 in den Registern 0 und 1
- P2 in den Registern 2 und 3
- P3 in den Registern 4 und 5
- P4 in den Registern 6 und 7

Wenn P1 noch im Haus ist, dann steht in beiden Registern 0 und 1 eine 0. Steht er auf dem Startfeld, ist im Register 0 eine 1 und im (höherwertigen) Register 1 eine 0. Wenn er im Ziel auf der allerhöchsten Position ist, dann steht im Register 0 ein C und im Register 1 eine 2 - das Spielbrett hat insgesamt 2C hex, also 44 Felder.


## Das gute alte Bubblesort

Schon vor Beginn der Programmierung von Mädn hatte ich den Gedanken, dass es für viele Tests und Funktionen vorteilhaft sein könnte, wenn die Pöppel eines Spielers immer in einer geordneten Reihenfolge, also sortiert wären.

Alle Pöppel ziehen nur in eine Richtung (vorwärts, vorwärts, vorwärts!), also kann ein potentieller Zug nur durch diejenigen eigenen Pöppel blockiert sein, die *vor* der ziehenden Spielfigur stehen. 

Also war der erste Schritt - nur so zum Spaß - einen möglichst kurzen Sortier-Algorithmus zu implementieren. 


## Relativitätstheorie

Jeder Spieler, Mensch oder Maschine, sieht erstmal nur sein eigenes Spielbrett. Das Haus ist also immer 00, das Startfeld immer 01, das letzte Feld vor den vier Positionen im Ziel immer 28 hex. Dadurch können die Programmteile für beide Spieler identisch bleiben. 

Nur beim möglichen und tatsächlichen Schlagen wird es nötig, die beiden Bezugssysteme ineinander umzurechnen. Dazu werden die Unterprogramme Umrechnung und Modulo verwendet. 

Umrechnung addiert zur Pöppel-Position einfach immer ein "halbes Brett" (hexadezimal 14 Felder). Ein eigener Pöppel beispielsweise, der mit einer 6 aus dem Haus kommt und auf dem Startfeld 01 landet, steht für den gegnerischen Spieler ja auf Feld 15 hex, also 01 + 14 = 15 hex. 

Das Ganze funktioniert aber nicht mehr, wenn die eigene Position größer als ein halbes Brett ist (Position >= 15 hex), dann muss nach Umrechnung noch eine Korrektur um ein ganzes Brett erfolgen.

Zum Beispiel landet der eigene Pöppel nach einem (unglücklichen) Wurf direkt auf dem Startfeld des Gegners. Das lässt allgemein die Nervosität steigen, muss aber korrekt berechnet werden:

Umrechnung 15 + 14 = 29 
Modulo     29 - 28 = 01 


## Würfel

![Microtronic würfelt](/pics/wuerfelt.jpg)

Das erste "eigene" Programm des Microtronic war ein "elektronischer Würfel" (Anleitungsbuch 1. Teil, Seite 11). Und ich kann mich noch gut erinnern, wie begeistert ich war. Der würfelt, guck mal! Ich habe bestimmt stundenlang immer neue Zahlen gewürfelt, damals Weihnachten 1981, und alle genervt, sie sollen auch mal auf den Knopf drücken...

![Allererstes Programm](/pics/erstes.jpg)

Über 40 Jahre war ich der Überzeugung, kürzer geht's nicht.

Geht's aber doch! Ein Vergleich mit 0 ist nicht nötig, stattdessen addieren wir einfach 1 zu einer Zufallszahl zwischen 0 und 5:

```
Würfel	RND
	CMPI #5,AUGEN
	BRC Würfel
	ADDI #1,AUGEN
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
