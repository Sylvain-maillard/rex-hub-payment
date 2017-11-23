# Il était une fois...

===

![Page de paiement de voyages-sncf.com](images/2017-10-30%2023_22_58-Récapitulatif%20et%20paiement%20de%20votre%20commande.png)<!-- .element style="border: 0; background: None; box-shadow: None" -->

note:
Il était une fois un bon gros legacy.
Je vous parler de cette page: 
Ceci est la page de paiement du site vsc. c'est un peu la partie emergée de l'iceberg du domaine paiement qui nous concerne ce soir. 

===

## Architecture existante

![Archi Legacy](images/archi-legacy.png)<!-- .element style="border: 0; background: None; box-shadow: None" -->

note: on a ce qu'on appelle un gros code legacy.
Ici, tout le code est placé dans une seule application. Cette application est responsable de tout le métier de la vente de billet de train.
Il s'agit d'une application historique (2001 la naissance quand même) qui est en train de devenir impossible a faire évoluer: on corrige d'un côté ça casse de l'autre...exemple du passage à Oui.

===

## Douleurs 

* une seule application
* de multiples intervenants
* le run de la fonction de paiement ne repose que sur une seule personne
* le build est déporté sur une équipe dont ce n'est pas l'unique responsabilité
* des problèmes en production que le support a du mal à traiter.

===

## La problématique du paiement

* Moyens de paiement multiples                                     <!-- .element: class="fragment" -->
    - Cartes (VISA, CB, AMEX)
    - paypal
    - Sofort / Ideal (marchés européens)
* Différents partenaires en fonction des moyens de paiement        <!-- .element: class="fragment" -->
* Pas d'encaissement en propre: c'est la SNCF qui encaisse !       <!-- .element: class="fragment" -->
* Problématique de reconciliation comptable                        <!-- .element: class="fragment" -->
* Problématique de sécurisation des cartes bancaires               <!-- .element: class="fragment" -->
* Problématique liée à la gestion de la fraude.                    <!-- .element: class="fragment" -->

note:
Le paiement n'est pas si simple qu'il y parait: C'est la sncf qui encaisse l'argent, pas oui.sncf/vsc
On utilise un système d'Atos (sips1) pour la carte bancaire et un autre système (sips2) pour paypal et les autres moyens de paiement pour l'europe
On utilise un partenaire de scoring pour la gestion de la fraude
On a tout un backoffice à alimenter avec les flux financiers

---

# Le besoin
## Y voir plus clair

* le metier souhaite savoir comment fonctionne le paiement
* le métier souhaite améliorer la traçabilité des évènements liés aux paiements

note: actuellement, le metier doit demander aux developpeurs de lui expliquer comment fonctionne le paiement
ce n'est pas la bonne manière de faire. On veut rendre la connaissance au metier => solution: le DDD
Nous avons également des problèmes de traçabilité du fonctionnement en prod => solution: l'Event Sourcing. 

---

# Comment découper ce legacy ?

* le modèle Feature Team
* le DDD Stratégique

note: 
disclaimer: c'est moi qui fait les interprétations des concepts, c'est pas un truc qu'on m'a expliqué mais une perception de ma part.
comment adresser le problème à la fois technique et organisationnel ?

===

## La Feature Team PEGGAT

* des moyens alloués pour réduire les problèmes de suivi de prod
* décision d'extraire le paiement du legacy 

===

## Le DDD "Stratégique"

* travail d'urbaniste
* recherche de "bounded context"

note: le ddd strategique est un outil qui va être utilisé au niveau du système d'information de l'entreprise, ou au moins sur l'ensemble du produit (dans le cas de VSC).
l'objectif est de définir des domaines fonctionels, les "bounded contexts"

===

## Qu'est ce qu'un bounded context ?

* notre périmètre de travail
* celui pour lequel un mot aura le même sens pour deux personnes différentes
* le plus simple: une app == une équipe == un bounded context

note: par exemple, un code promo n'aura pas le même sens s'il est utilisé dans le cadre d'un paiement ou d'une vente..
La notion de bounded context n'est pas absolue, elle est valable dans la culture générale de l'entreprise. Ainsi, on aura
quelque chose d'efficace quand on a une équipe == une application == un bounded context.

===

## Le Bounded Context Paiement

note: Je vais maintenant parler de notre contenu.
L'analyse de notre bounded context va nous permetre d'avancer sur la conception de notre application

===

## Une transaction ?

> "Une opération financière est un événement contractuel d'achat ou de vente pour échanger un actif contre paiement." 
> --[Wikipedia](https://fr.wikipedia.org/w/index.php?title=Transaction_financière)

Le bounded context couvert par le hub de paiement concerne uniquement la partie "contre paiement"

note: Dans notre appli legacy, il y a une notion de transaction. 
Ceci ne s'applique pas pour notre bounded context: on parle de paiement, pas de transaction. 
La transaction est l'échange d'un bien contre une valeur (de l'argent)
d'après notre définition, interne à l'entreprise, nous devons uniquement nous occuper de la phase qui correspond au paiement:
récupération de l'argent, et non pas à la phase qui consiste à livrer le bien: c'est ce qu'on appelle la finalisation.
C'est un autre context. 

---

# Design de l'application

===

# DDD "Tactique"

* entité: une identité et un état (qui peut varier) 
    - exemple: un paiement                              <!-- .element: class="fragment" -->
* value object: objet immutable sans identité 
    - exemple: un montant de 45 €                       <!-- .element: class="fragment" -->
* domain service: un truc qu'on arrive pas à mapper en OO
    - exemple: un générateur d'id de transaction        <!-- .element: class="fragment" -->
* application service: orchestre les objets/services du domaine pour réaliser des uses cases
    - exemple: un use case de remboursement             <!-- .element: class="fragment" -->

note:
ddd tactique est une manière d'architecturer un bounded context.
on va utiliser des "briques de base" pour organiser notre appli. 
pas de domaine anémique etc.

---

# Architecture du Hub de paiement

=== 

## Zoom out

===

![Archi zoom out](images/archi-hub-zoom-out.png)<!-- .element height="70%" width="70%" style="border: 0; background: None; box-shadow: None" -->

===

## Archi "Hexagonale" ?

===

![Archi layers](images/archi-hub-layers.png)<!-- .element height="65%" width="65%" style="border: 0; background: None; box-shadow: None" -->

===

## Archi "Hexagonale" (détail)

===

![Archi Classes](images/archi-hub-classes.png)<!-- .element height="68%" width="68%" style="border: 0; background: None; box-shadow: None" -->

===

## Machine à état du paiement

[click here](https://doc-vsct.vsct.fr/display/FINALISATION/2502+-+HUB+-+payments#id-2502-HUB-payments-MachineàétatouDiagrammed'activités) <!-- .element data-preview-link -->

---

# Merci !