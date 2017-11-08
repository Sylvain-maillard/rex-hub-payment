# DDD/Event Sourcing

Retour d'expérience sur les paiements de Voyages-sncf.com

![logo oui.sncf](images/LOGO_OUIsncf-Rvb-002-450x156.png)<!-- .element style="border: 0; background: None; box-shadow: None; width:300px" -->

---

# $ whoami_

Sylvain Maillard

* Lead dev Java chez Voyages-Sncf.com/Oui.sncf
* Dev java depuis 2001
* Object Oriented Paradigm
* Domain Driven Design

[@sylvain_m44](https://twitter.com/sylvain_m44) 

note:
Je vais commencer par me présenter rapidement comme ça c'est fait et on passe à autre chose.
Donc je suis sylvain maillard blabla etc

Pourquoi j'ai voulu faire cette présentation ? On entend beaucoup parler de ddd, d'event sourcing, de cqrs, mais il y a peu de retour sur la mise en place de ces techniques sur des projets stratégiques qui tournent en production.

Nous on a essayer de le faire sur la refonte de la brique de paiement du site voyages-sncf.com;

J'avais envie de partager notre expérience avec vous sur ce sujet, voir comment nous avons tenter d'implémenter ces concepts sur le hub de paiement, quels choix nous avons fait, quels problèmes nous avons eu et les leçons que nous en avons tiré.

---

![Page de paiement de voyages-sncf.com](images/2017-10-30%2023_22_58-Récapitulatif%20et%20paiement%20de%20votre%20commande.png)<!-- .element style="border: 0; background: None; box-shadow: None" -->

note: Je vous parler de cette page: 
Ceci est la page de paiement du site vsc. c'est un peu la partie emergée de l'iceberg du domaine paiement qui nous concerne ce soir. 

===

### Architecture existante

![Archi Legacy](images/archi-legacy.png)<!-- .element style="border: 0; background: None; box-shadow: None" -->

note: on a ce qu'on appelle un gros code legacy.
Ici, tout le code est placé dans une seule application. Cette application est responsable de tout le métier de la vente de billet de train.
Il s'agit d'une application historique (2001 la naissance quand même) qui est en train de devenir impossible a faire évoluer: on corrige d'un côté ça casse de l'autre...exemple du passage à Oui.

---

# Comment découper ce legacy ?

* le modèle Feature Team
* le DDD Stratégique

note: disclaimer: c'est moi qui fait les interprétations des concepts, c'est pas un truc qu'on m'a expliqué mais une perception de ma part.

---

# Le DDD "Stratégique"

* travail d'urbaniste
* recherche de "bounded context"

note: le ddd strategique est un outil qui va être utilisé au niveau du système d'information de l'entreprise, ou au moins sur l'ensemble du produit (dans le cas de VSC).
l'objectif est de définir des domaines fonctionels, les "bounded contexts"

---

# Qu'est ce qu'un bounded context ?

* notre périmètre de travail
* celui pour lequel un mot aura le même sens pour deux personnes différentes

note: par exemple, un code promo n'aura pas le même sens s'il est utilisé dans le cadre d'un paiement ou d'une vente..

---

le bounded context est lié à la culture d'entreprise

* une app == une équipe == un bounded context
-> efficacité.

note: La notion de bounded context n'est pas absolue, elle est valable dans la culture générale de l'entreprise. Ainsi, on aura
quelque chose d'efficace quand on a une équipe == une application == un bounded context.

---

# Le Bounded Context Paiement

note: Je vais maintenant parler de notre contenu.

---

# Une transaction ?

> "Une opération financière est un événement contractuel d'achat ou de vente pour échanger un actif contre paiement." 
> --[Wikipedia](https://fr.wikipedia.org/w/index.php?title=Transaction_financière)

note: Dans notre appli legacy, il y a une notion de transaction. 
Ceci ne s'applique pas pour notre bounded context: on parle de paiement, pas de transaction. 
La transaction est l'échange d'un bien contre une valeur (de l'argent)
d'après notre définition, interne à l'entreprise, nous devons uniquement nous occuper de la phase qui correspond au paiement:
récupération de l'argent, et non pas à la phase qui consiste à livrer le bien: c'est ce qu'on appelle la finalisation.
C'est un autre context. 

---

### Le paiement chez VSC/Oui.sncf

* Fortement lié à la sncf                                          
* Problématique de reconciliation comptable                        <!-- .element: class="fragment" -->
* Différents partenaire en fonction des moyen de paiement          <!-- .element: class="fragment" -->
* Problématique de sécurisation des cartes bancaires               <!-- .element: class="fragment" -->
* Problématique liée à la gestion de la fraude.                    <!-- .element: class="fragment" -->

note:
Le paiement n'est pas si simple qu'il y parait: C'est la sncf qui encaisse l'argent, pas oui.sncf/vsc
On utilise Atos pour la carte bancaire et un autre système pour paypal
On utilise un partenaire de scoring pour la gestion de la fraude
On a tout un backoffice à alimenter avec les flux financiers

---

# Design de l'application 

* le DDD "Tactique"

note: 
