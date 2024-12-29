---
layout: post
title:  Terminal in Farbe und Bunt
date:   2024-12-29
categories: snippets
---

# Terminal in Farbe und Bunt

Ich hatte letztens den Anwendungsfall, dass ich in einem Docker-Container mehrere Prozesse parallel am Laufen hatte - jedoch hat ein Container naturgemäß nur ein Log. Statt mich für einen Prozess zu entscheiden, dessen Output ich anzeigen will, habe ich mich dazu entschieden, die Outputs aller Prozesse anzeigen zu lassen - eben gemischt, alles in einem Log. Natürlich ist das anfangs nicht besonders übersichtlich.

Mein Entrypoint-Skript sieht in etwa so aus (obfuskiert):

```bash
#!/bin/bash
/usr/bin/befehl1 2>&1 & 
befehl2 2>&1 &
tail -f /var/log/someapp/*.log &
tail -f /dev/null
```

Letztlich wird jeder Befehl in den Hintergrund verschoben mit `&`, davor aber noch der `stderr` in `stdout` umgeleitet (`2>$1`). Soweit nichts besonderes an dieser Stelle, das ist bei Linux-Befehlen recht gängig. Weiter unten wurde noch der `tail -f` einer oder mehrerer Logs ausgegeben, ebenfalls im Hintergrund. Der letzte Befehl `tail -f /dev/null` soll nur meinen Container am Leben halten, damit dieser sich nicht vorzeitig beendet. Es gibt andere Varianten mit `while` und `sleep` und solche Geschichten, aber das schien mir hier recht elegant mit `tail -f /dev/null`.

## Unterscheidung der Logs

Wie kann ich jetzt also in den Logs eine Unterscheidbarkeit herstellen, um zu wissen, welcher Prozess welche Zeile in den Output des Containers geschrieben hat? Das Internet hat mich darauf gestoßen, dass man ja `sed` dazu verwenden könnte, den Output jedes Prozesses an `sed` zu pipen und damit etwas zu prefixen:

```bash
#!/bin/bash
/usr/bin/befehl1 2>&1 | sed "s/^/[befehl1 ] /" & 
befehl2 2>&1 | sed "s/^/[befehl2 ] /" & 
tail -f /var/log/someapp/*.log | sed "s/^/[someapp ] /" & 
tail -f /dev/null
```

So kann ich zumindest schonmal die Logs dadurch unterscheiden, dass sie mit dem entsprechenden Präfix beginnen. Aber das hat mir noch nicht gereicht. Ich wollte farbige Präfixe zur noch besseren Unterscheidung.

Im Terminal kann man auch farbige Outputs erzeugen, z.B. mit `echo "\033[31m roter Text\033[0m, normaler Text"`. Natürlich habe ich versucht, das in den `sed` Befehl zu integrieren:

```bash
#!/bin/bash
/usr/bin/befehl1 2>&1 | sed "s/^/\033[31m[befehl1 ]\033[0m /" & 
...
```

Aber das hat nicht so funktioniert. Der `sed` mag keine solchen Steuerzeichen. Aber [diese Antwort auf stackoverflow](https://unix.stackexchange.com/a/655853) brachte den entscheidenden Hinweis: man schreibt den Steuerbefehl einfach in Hex! Und dann sieht die finale Lösung so aus:

```bash
#!/bin/bash
/usr/bin/befehl1 2>&1 | sed "s/^/\x1b[34m[befehl1 ]\x1b[0m /" & 
befehl2 2>&1 | sed "s/^/\x1b[36m[befehl2 ]\x1b[0m /" & 
tail -f /var/log/someapp/*.log | sed "s/^/\x1b[33m[someapp ]\x1b[0m /" & 
tail -f /dev/null
```

Und dann hatte alles seine Richtigkeit und Farbe 👉🌈. 