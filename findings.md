# Findings — pfSense Firewall & IPS Lab

## Vue d'ensemble

Ce document présente les résultats des tests effectués sur le pare-feu pfSense, incluant la validation du comportement par défaut, 
une attaque simulée par déni de service (HPING flood), et les tentatives de mitigation.

## Architecture testée

- **Kali Linux** (WAN, 10.0.0.10) — simule un attaquant externe
- **pfSense** (WAN: `10.0.0.1`, LAN: 192.168.1.1) — pare-feu/routeur
- **Ubuntu Server** (LAN,192.168.1.100) — machine cible

## Test 1 — Comportement par défaut (deny by default)

**Commande** : ping 10.0.0.1 -c 4 (depuis Kali vers pfSense WAN)
**Résultat** : 100% packet loss

pfSense bloque par défaut tout trafic ICMP entrant sur l'interface WAN, sans qu'aucune règle explicite ne soit nécessaire pour ce comportement — illustration du principe de sécurité "deny by default".

## Test 2 — Création d'une règle d'autorisation ciblée

Une règle a été ajoutée sur WAN pour autoriser explicitement le trafic ICMP Echo Request vers Ubuntu (192.168.1.100) :
MESSAGE DU LOG : Action: Pass | Interface: WAN | Protocol: ICMP (Echo Request) Source: Any | Destination: 192.168.1.100

**Découverte** : la règle automatique **"Block private networks"** (activée par défaut sur WAN) bloquait ce trafic avant même d'atteindre notre règle personnalisée, car l'IP source de Kali (`10.0.0.10`) est une adresse privée. Cette règle a dû être désactivée pour les besoins du lab (dans un contexte réel, cette règle protège contre l'usurpation d'adresses privées venant d'Internet — elle ne devrait jamais être désactivée en production).

**Résultat après correction** : `ping 192.168.1.100 -c 4` → 0% packet loss.

## Test 3 — Attaque HPING flood

**Commande** : sudo hping3 --icmp --flood 192.168.1.100
<img width="942" height="467" alt="image" src="https://github.com/user-attachments/assets/e2c9f07d-b07c-4a08-b991-e86fbe5e2bba" />


**Observation clé sur le logging** : malgré des millions de paquets envoyés, seules 2 lignes de log sont apparues pour cet événement. pfSense (via le moteur `pf` de FreeBSD) applique un rate-limiting interne sur la génération de logs, pour éviter que le système de logging lui-même ne devienne un vecteur de déni de service. Ceci démontre une limite du logging pare-feu traditionnel face à des attaques volumétriques, et justifie l'intérêt d'un SIEM (voir lab 2 — Splunk) pour l'agrégation et la détection basées sur le volume plutôt que sur des logs événement-par-événement.

## Test 4 — Tentative de mitigation par Limiter (Traffic Shaper)

Un Limiter a été configuré (`icmp_flood_limit`, 10 Kbit/s) et associé à la règle ICMP.

**Découverte** : le paramètre `max-src-states` (limite de connexions simultanées) s'est révélé inefficace contre ce type de flood, car tous les paquets d'un flux ICMP flood partagent une seule et même state (même tuple source/destination/protocole) — le compteur de states ne dépasse jamais 1, peu importe le nombre de paquets envoyés. La limitation par débit (Limiter/dummynet) est l'approche appropriée pour ce type d'attaque, contrairement à une limitation par nombre de connexions.

## Test 5 — Blocage complet

La règle a été modifiée de **Pass** à **Block**.

ping 192.168.1.100 -c 4` → 100% packet loss confirmé.

**Résultat** : MESSAGE LOG 

Action: block Matched Rule: block drop in log quick on em0 inet proto icmp from any to 192.168.1.100 icmp-type echoreq
