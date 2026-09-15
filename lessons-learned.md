# Lessons Learned — pfSense Firewall & IPS Lab

## Concepts réseau

- **Internal Network vs NAT Network (VirtualBox)** : les réseaux internes se créent dynamiquement dès que deux VMs partagent le même nom d'interface, sans configuration centralisée préalable — contrairement aux NAT Networks qui doivent être créés explicitement.
- **Routage inter-VM** : chaque machine a besoin d'une route par défaut explicite (`ip route add default via ...`) pour atteindre des réseaux distants ; une IP statique seule ne suffit pas si la passerelle n'est pas définie.
- **WAN symbolique vs IP publique réelle** : dans un lab isolé, l'interface WAN reçoit une IP privée statique plutôt qu'une IP publique distribuée par un FAI — une simplification pédagogique qui n'affecte pas la validité du concept de filtrage démontré.

## pfSense

- **Deny by default** : aucune règle n'est nécessaire pour bloquer — c'est le comportement natif de toute interface pfSense tant qu'aucune règle Pass n'est ajoutée.
- **Anti-lockout rule** : pfSense autorise automatiquement l'accès au webConfigurator depuis LAN, mais pas depuis les autres interfaces (WAN, OPT1) — une règle explicite est nécessaire pour ouvrir l'accès administratif depuis une interface non-LAN.
- **Règles automatiques de sécurité** ("Block private networks", "Block bogon networks") s'exécutent avant les règles personnalisées et peuvent bloquer du trafic légitime dans un contexte de lab simulé — à connaître avant de déboguer une règle qui semble ne "jamais" s'appliquer.
- **Ordre des règles** : pfSense traite les règles de haut en bas et s'arrête à la première correspondance — l'ordre est déterminant.
- **Logging asymétrique** : les règles Block loggent par défaut, les règles Pass non — il faut activer explicitement le logging sur une règle Pass si on veut visibilité sur le trafic autorisé.
- **Rate-limiting du logging** : sous forte charge (flood), pfSense limite volontairement le volume de logs générés pour se protéger — un flood de millions de paquets ne produit que quelques lignes de log.
- **États (states) vs débit (rate)** : `max-src-states` limite les connexions simultanées distinctes, pas le débit de paquets au sein d'une même connexion — un Limiter (Traffic Shaper/dummynet) est nécessaire pour contrôler un flood au sein d'une seule state.

## Méthodologie / troubleshooting

- Lynx (navigateur texte) permet d'administrer pfSense sans interface graphique, mais se montre peu fiable sur des pages complexes (JS, CSRF tokens) — un accès réseau direct depuis un vrai navigateur reste préférable dès que possible.
- `easyrule` sur pfSense a une syntaxe stricte nécessitant 4 arguments explicites pour une règle TCP/UDP avec port (interface, protocole, source, destination, port séparés par espaces) — l'usage de `:` pour combiner IP et port n'est pas supporté.
- Toujours vérifier et cliquer sur "Apply Changes" après une modification de règle pfSense — les changements ne prennent effet qu'après validation explicite.
