# Topologie Packet Tracer — Entreprise 2 sites

Topologie multi-sites réalisée dans le cadre du module M201
(routage d'un réseau d'entreprise).

## Équipements
- 2 routeurs PT8200 (R1, R2), 2 switchs 2960 (SW1, SW2)
- 4 PC et 1 serveur

## Technologies configurées
- VLAN 10 (Admin) et VLAN 20 (Utilisateurs)
- Routage inter-VLAN (router-on-a-stick)
- OSPF zone 0 entre les deux sites
- DHCP sur les routeurs
- EtherChannel LACP (Po1, Fa0/23-24) entre les switchs

## Tests validés
- Ping PC0 (site 1) → Server0 (site 2) : réussi
- EtherChannel : Po1(SU), Fa0/23(P) Fa0/24(P) sur les deux switchs

## Fichier
Ouvrir `depot.pkt` dans Cisco Packet Tracer.
