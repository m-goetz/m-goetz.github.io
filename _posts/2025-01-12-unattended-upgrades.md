---
layout: post
title: Upgrades machen lassen mit unattended-upgrades
date: 2025-01-12
categories: raspi
---

Wer seinen eigenen kleinen Server betreibt, der vielleicht auch noch im Internet hängt, sollte es nicht verpassen, regelmäßig Updates einzuspielen. In Linux - und in meinem Fall genauer auf dem Raspberry Pi - kann man diesen Prozess zum Glück sehr gut automatisieren. Jedoch war bei der Einrichtung der `unattended-uprades`, wie das Package heißt, nicht ganz so einfach, wie ich mir das zu Anfang vorgestellt hatte. Aber der Reihe nach.

## Installation

Als Erstes installiert man natürlich das entsprechende Paket:

```bash
sudo apt update
sudo apt -y install unattended-upgrades
```

So weit, so einfach. Nach der Installation konfiguriert man die automatischen Upgrades mit dem nachfolgenden Befehl. Dabei folgt man einfach dem Assistenten und richtet sich das so ein, wie man das haben möchte:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

Erste Tests mit dem Debug-Befehl

```bash
sudo unattended-upgrade -d
```

haben aber gezeigt, dass ich nicht das gleiche Ergebnis erhalten habe, wie wenn ich die Upgrades manuell ausführe. Also ging es ans Feintuning.

## Konfiguration

In der Datei `/etc/apt/apt.conf.d/50unattended-upgrades` setzt man diverse weitere Einstellungen. Zuerst geht es mal darum, aus welchen Quellen man Pakete installieren möchte. Dazu modifiziert man die Liste der `Origins-Pattern` in dieser Datei. Ich habe die Liste um den einfachen Eintrag `"o=*,a=*";` erweitert.

```bash
Unattended-Upgrade::Origins-Pattern {
    ...
    "o=*,a=*";
};
```

Diesen Tipp hatte ich aus dem [RaspberryPi Forum](https://forums.raspberrypi.com/viewtopic.php?t=255901). Das bewirkt effektiv, dass Pakete aus allen Origins erlaubt sind. Das bedeutet für mich letztlich, dass ich ein volles Upgrade bekomme, wie wenn ich `sudo apt upgrade` ausführe. Das mag für den ein oder anderen zu riskant oder zu ungenau sein. Man könnte sich hier natürlich auch auf reine Security-Updates beschränken.

## Mails

Mir war es jetzt noch wichtig, dass ich irgendwie schnell und leicht mitbekomme, wenn Upgrades ausgeführt wurden. Der für mich einfachste Weg war, meinem Hauptbenutzer auf dem Raspi die Mail lokal zuzustellen und die Mails dann beim Login anzuzeigen. In der Datei `/etc/apt/apt.conf.d/50unattended-upgrades` trägt man den Nutzernamen des lokalen Linux-Users ein (hier in diesem Beispiel `harry`):

```bash
// Send email to this address for problems or packages upgrades
// If empty or unset then no email is sent, make sure that you
// have a working mail setup on your system. A package that provides
// 'mailx' must be installed. E.g. "user@example.com"
Unattended-Upgrade::Mail "harry";
```

Der Kommentar weist uns auch schon darauf hin, dass wir ein funktionierendes Mail-Setup brauchen. Ich habe dazu das Paket `bsd-mailx` intalliert. Zur Anzeige von Mails benutze ich `mutt`.

```bash
sudo apt -y install bsd-mailx
sudo apt -y install mutt
```

Jetzt fehlt nur noch eins: Ich will informiert werden, wenn ich mich einlogge. In der `.bashrc` Datei (oder `.zshrc`, je nach verwendeter Shell) trage ich ein:

```bash
mutt -z
```

Das `-z` bewirkt, dass es nur gestartet wird, wenn wirklich neue Mails vorliegen.

![mutt](/assets/images/mutt.png)

Seitdem kümmert sich mein Raspi ganz alleine um seine Upgrades und lässt mir immer seine Notizen da, wann er was gemacht hat - sehr entspannt! 