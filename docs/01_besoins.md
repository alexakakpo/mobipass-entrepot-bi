# Besoins des décideurs : magasin Ventes

## Les décideurs

Deux responsables des ventes, l'un basé à **Lyon**, l'autre à **Berlin**. Leurs besoins sont identiques, mais chacun ne s'intéresse qu'aux données de son pays, sur le plan géographique comme financier. Ils forment donc une seule classe de décideurs, avec deux périmètres.

## Besoins exprimés par le client

| Réf. | Besoin | Type de restitution |
|---|---|---|
| B3.1 | Quantités achetées, montants réellement payés et bénéfices, par jour, mois et année | Analyse interactive |
| B3.2 | Mêmes mesures selon les produits (produit, gamme, secteur) et les employés (profil, classe de trajet, métier ROME pour la France) | Analyse interactive |
| B3.3 | Mêmes mesures selon les sociétés (type, catégorie juridique, positionnement géographique) | Analyse interactive |

Le client nous a laissé le soin de proposer **au moins deux rapports emblématiques**. Nous en avons proposé trois, chacun construit à partir d'une attente du cahier des charges :

| Réf. | Rapport proposé | Attente du client à laquelle il répond | Fréquence |
|---|---|---|---|
| B4.1 | Tableau de bord des ventes : quantités, montant payé et bénéfice par mois, comparés à l'année précédente | Mesures demandées, vision à moyen et long terme | Mensuelle |
| B4.2 | Gammes par territoire : classement des gammes par montant payé pour chaque région (FR) ou Land (DE) | « Identifier les gammes de produits les plus adaptées à chaque lieu » | Mensuelle |
| B4.3 | Ventes par type de société et catégorie juridique | « Maximum d'informations sur les sociétés » | Mensuelle |

## Décisions prises avec le client (entretiens des 13 et 14 octobre)

- **Base de comparaison commune.** Les montants sont restitués dans une même base pour comparer les pays. Convention retenue : le **hors taxes**.
- **Restitution TTC possible.** Le client veut aussi pouvoir afficher les prix **TTC**, au choix du décideur.
- **Tarif appliqué.** Le tarif ne dépend pas de l'agence : c'est celui du **pays de la société** de l'employé, valable à la date de l'achat.

## Ce qui a été volontairement écarté

- **Analyses non demandées :** le trimestre, les tranches d'âge des salariés, les classes d'autonomie des produits et le statut des générations ont été retirés du magasin, pour s'en tenir au besoin exprimé.
- **Achats chez le partenaire Alltricks :** ils restent dans l'entrepôt, mais ne sont pas analysés dans ce magasin. On ne connaît ni les produits ni les montants de ces achats.
