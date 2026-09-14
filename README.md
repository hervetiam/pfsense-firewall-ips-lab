# pfsense-firewall-ips-lab
pfSense Firewall Deployment &amp; Perimeter Defense Lab
# pfSense Firewall & IPS Lab

## À propos de ce projet

Ce projet est un laboratoire personnel que j'ai monté pour développer des compétences pratiques en sécurité réseau, dans le cadre de ma recherche de stage en cybersécurité. L'objectif était simple : sortir de la théorie et vraiment mettre les mains dans la configuration d'un pare-feu, comprendre comment il filtre le trafic, et voir concrètement ce qui se passe quand on l'attaque.

J'ai utilisé pfSense comme pare-feu/routeur, avec Kali Linux jouant le rôle d'un attaquant externe (sur le réseau WAN) et une machine Ubuntu comme cible protégée (sur le réseau LAN). Tout tourne en local dans VirtualBox, sur mon PC.

Ce lab est le premier d'une série de deux projets connectés. Le second, https://github.com/hervetiam/Splunk-SIEM-Lab-Insider-Threat-Detection.git, 
part de cette même infrastructure et y ajoute un SIEM (Splunk) pour détecter les menaces internes — celles qu'un pare-feu périmétrique 
comme pfSense ne peut pas voir. Ensemble, les deux projets illustrent le principe de défense en profondeur : un pare-feu qui filtre à la frontière, 
et un SIEM qui surveille l'intérieur.

## Objectifs

- Monter une infrastructure réseau segmentée (WAN/LAN) avec pfSense comme point de contrôle
- Comprendre le comportement par défaut d'un pare-feu stateful (deny by default)
- Configurer des règles de filtrage ciblées et comprendre l'ordre de traitement des règles
- Simuler une attaque réseau réelle (flood ICMP avec HPING3) et observer son impact
- Tester une mitigation par limitation de débit (Traffic Shaper / Limiters)
- Documenter l'ensemble du processus, y compris les erreurs et ce qu'elles m'ont appris

## Environnement

- Oracle VirtualBox (hôte Windows, 16 Go RAM)
- pfSense CE 2.9.0
- Kali Linux (image VirtualBox préconfigurée)
- Ubuntu Server 26.04 LTS

  
## Architecture
    Kali Linux (attaquant)
     10.0.0.10
          |
     intnet-wan
          |
    ┌──────────────┐
    │   pfSense     │
    │  WAN: 10.0.0.1│
    │  LAN: 192.168.1.1
    └──────────────┘
          |
     intnet-lan
          |
    Ubuntu Server (cible)
     192.168.1.100
