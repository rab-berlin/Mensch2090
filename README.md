# Mensch2090
Mensch-ärgere-dich-nicht auf dem Busch Microtronic 2090. Work in progress...

## Das gute alte Bubblesort
...

## Alles ist relativ

## Strategie

## Ein Königreich für ein Register

Wer ist gerade dran, Computer oder Mensch? Irgendwo muss diese Information hinterlegt werden, und diese Information muss über alle Spielzüge erhalten bleiben. Das ist angesichts des begrenzten Register-Speichers gar nicht so einfach.

Der Microtronic hat nur 16 Arbeitsregister (und 16 Speicherregister, die wir in die Arbeitsregister einblenden können). Für alle vier eigenen Pöppel werden die Register 0-7 genutzt. Die gegnerischen Pöppel residieren in den entsprechenden Speicherregistern und werden bei Spielerwechsel einfach "aktiv" geschaltet mit EXRL. Dadurch kann ich im wesentlichen die gleichen Programmteile für beide Spieler nutzen. 

Zusätzlich brauchen wir für das Würfeln mit RND die Register D-F. Alles, was in diesen Registern vorher war, ist nach dem Würfeln der unaufhaltsamen Macht der Entropie zum Opfer gefallen (also weg). 

Die vier möglichen Züge (Zugliste) und der tatsächlich ausgeführte Zug teilen sich den Platz im Register E.

Das Register F steht als Hilfsregister für allerlei Zahlenwerk zur Verfügung. Zum Beispiel muss die Bubblesort-Schleife dreimal durchlaufen werden, das wird im Hilfsregister F abgezählt. 

Die Register 8 und 9 (Zielfeld) sowie die Register A und B (Testfeld) brauchen wir für Vergleiche, also werden diese Register auch überschrieben. Was immer da drin war oder ist, überlebt den Spielzug nicht.

Übrig bleibt als einziges Register C. Aber das brauchen wir auch noch für ein Flag zur Erkennung, ob das Startfeld geräumt werden muss (dazu mehr unter Test0).

Also alle Register belegt :-( Woher noch ein zusätzliches Register nehmen (dessen Inhalt den Spielzug übersteht)?

Man könnte natürlich ein Speicherregister dazu nutzen: Vor dem Spielzug mit MAS (move Addressregister in Speicherregister) sichern und später mit EXRH, MAS und nochmal EXRH den Inhalt zurückholen. Braucht aber 4 wertvolle Programmschritte für jede Abfrage.

Geht's einfacher?

Ja. Wir nutzen die Ausgänge als "zusätzliches" Register. Mit DOT wird der Registerinhalt 0 oder 8 (Computer ist dran oder Mensch ist dran) auf die Ausgänge gelegt, also Ausgang 4 auf High oder Low geschaltet. Mit DIN können wir dann diesen Inhalt wieder einlesen, wann und wo immer es nötig ist - vorausgesetzt, wir haben ein Kabel von Ausgang 4 zu Eingang 4 gelegt.  

## Gewinnerkennung

war geradezu lächerlich einfach. Durch die vor jedem Zug erfolgte Sortierung der Pöppel muss nur ein Kriterium überprüft werden: Ist der letzte Pöppel (P1) im Ziel, also auf Feld 29 hex angekommen? Falls ja, dann gewonnen. Fertig.
