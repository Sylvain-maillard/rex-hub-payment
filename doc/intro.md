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
