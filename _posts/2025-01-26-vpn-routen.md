---
layout: post
title:  Split-Tunneling in Ubuntu
date:   2025-01-26
categories: snippets
---

Das bei mir in Ubuntu hinterlegte VPN zu meiner Arbeit hatte die doofe Eigenschaft, dass der komplette Internetverkehr darüber gelaufen ist. Jeder Aufruf zu `google.de` oder `teams.microsoft.com` ging über das Arbeits-VPN, mit entsprechenden Drawbacks: langsame Verbindungen, Abbrüche, und unser IT-Capo war bestimmt nur bedingt glücklich damit, dass ich die Internetleitung der Firma unnötigerweise belastet habe. Aber das hat jetzt ein Ende - eingerichtet habe ich das sogenannte "Split-Tunneling" so:

Zunächst muss man in der VPN-Verbindung in Ubuntu festlegen, dass die Verbindung nur für Ressourcen innerhalb des Netzwerks verwendet werden soll. Dazu muss man den entsprechenden Haken setzen:

![vpn-conn](/assets/images/vpn-conn.png)

Das hatte erstmal zur Folge, dass keine der Ressourcen im Netzwerk der Firma erreichbar waren. Das lag daran, dass die entsprechenden Routen dafür gefehlt haben. Zwei Subnetze mussten in die Routingtabelle eingetragen werden, damit man die IP-Adressen der Maschinen in der Firma erreichen kann (hier mit Beispielwerten, `<Verbindungsname>` ist der Name, wie er auch in den Einstellungen angezeigt wird):

```bash
sudo nmcli connection modify "<Verbindungsname>" +ipv4.routes "10.10.10.0/24"
sudo nmcli connection modify "<Verbindungsname>" +ipv4.routes "10.10.20.0/24"
```

Damit sind die Routen in der VPN-Verbindung eingetragen. Auffällig hierbei: In der GUI (wo man auch den Haken gesetzt hat, dass die Verbindung nur für Netzwerkressourcen verwendet werden soll) sind die Routen jetzt auch sichtbar. Aber wenn man den Dialog neu durch Speichern abschließt, sind die Routen wieder weg. Wohl ein Bug in Ubuntu.

Beim Verbinden mit dem VPN waren die Maschinen in der Firma jetzt auch erreichbar, aber nur unter ihren IP-Adressen, nicht unter ihren Namen. Was noch fehlt ist die Konfiguration der DNS-Server. Diese habe ich konfiguriert, indem ich die Datei `/etc/systemd/resolved.conf` angepasst habe. Dort kann man die DNS-Server hinterlegen, in meinem Fall die beiden Arbeits-DNS-Server. Den DNS-Server aus meinem eigenen Netzwerk musste ich hier nicht nochmal angeben.

```bash
...
[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4>
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNS=10.10.20.118 10.10.20.98
FallbackDNS=8.8.8.8
#Domains=
#DNSSEC=no

``` 

Die aktuell verwendeten DNS-Server kann man mit diesem Befehl herausfinden - damit habe ich meine Einstellungen kontrolliert:

```bash
~$ resolvectl --no-pager | grep Server
  Current DNS Server: 10.10.20.118
         DNS Servers: 10.10.20.118 10.10.20.98
Fallback DNS Servers: 8.8.8.8
Current DNS Server: 192.168.178.1
       DNS Servers: 192.168.178.1 
```

Endlich keine langsame Internetverbindung mehr, endlich keine Verbindungsabbrüche mehr (vor allem, wenn man während Teams das VPN einschaltet), mein PiHole greift wieder (dazu müsste ich auch mal einen Blogeintrag machen)... alles super. Ein "pet peeve" weniger, wie der Brite sagt.