# Choix de conception du magasin Ventes

## Démarche

J'ai suivi la démarche vue en cours, en trois niveaux :

1. **Conceptuel :** fait, mesures, dimensions, hiérarchies, paramètres et attributs faibles (formalisme multidimensionnel de F. Ravat), voir `modelisation/MD2_Ventes_schema_conceptuel.json`.
2. **Logique :** R-OLAP dénormalisé, c'est-à-dire une table par dimension et une table de fait.
3. **Physique (Oracle 11g) :**
   - chaque dimension est une vue matérialisée, avec sa clé primaire et un `CREATE DIMENSION` (niveaux, hiérarchies, attributs) ;
   - le fait est une vue matérialisée, avec sa clé primaire et ses clés étrangères ;
   - les agrégats forment un treillis partiel : seuls les nœuds utiles aux rapports sont matérialisés, et le plus agrégé est calculé à partir d'un nœud plus détaillé.

## Le fait

**Grain :** un produit, une version d'employé, un métier, un jour.

| Mesure | Calcul |
|---|---|
| Quantite_Achat_MD | Somme des quantités achetées |
| Montant_Achat_MD | Quantité × prix public HT |
| Montant_Achat_ReelPaye_MD | Montant public HT × (1 − % de réduction de la gamme, de la catégorie de l'employé et de l'année) |
| Benefice_Achat_MD | Montant public HT − montant réellement payé |
| Montant_Achat_TTC_MD, Montant_Achat_ReelPaye_TTC_MD | Montant HT × (1 + TVA du pays) |

**Prix.** Pour chaque achat, le prix est le tarif du produit **dans le pays de la société de l'employé**, valable à la date de l'achat. Les tarifs français, en TTC, sont ramenés en HT avec le taux de TVA du pays (20 % en France, 19 % en Allemagne), stocké dans la table des pays plutôt qu'écrit en dur.

## Les dimensions

| Dimension | Hiérarchies | Particularité |
|---|---|---|
| Temps | Jour → Mois → Année | Le mois est stocké sous la forme `AAAA-MM`, sinon « Mois → Année » ne serait pas une dépendance fonctionnelle. |
| Produit | Gamme → Secteur ; Marque ; Génération → Fabricant → Pays du fabricant | |
| Métier | Appellation → Fiche ROME → Domaine professionnel → Grand domaine | Un membre « Non renseigné » (NR) pour les salariés allemands et ceux sans fiche RH : aucun achat n'est perdu. |
| Employé | Catégorie → Classe de trajet ; Service → Société → Type ; → Catégorie juridique (3 niveaux) ; → NAF (4 niveaux) ; → géographie FR / DE | La société est incluse dans l'employé (choix de l'équipe), et l'employé est historisé. |

### Pourquoi historiser l'employé (type 2)

Un salarié peut changer de service, donc parfois de société. Si l'on gardait seulement sa situation actuelle, tous ses anciens achats basculeraient dans son nouveau service. Chaque affectation crée donc une **version** de l'employé (`IdVersionEmploye`), et un achat est relié à la version valable à sa date.

### La géographie France / Allemagne

| France | Allemagne |
|---|---|
| Commune → Département → Région → Pays | Gemeinde → Kreis → (Regierungsbezirk) → Land → Pays |

- La **région française** et le **Land allemand** sont au même niveau (NUTS 1). Ils sont réunis dans un seul paramètre, `CodeRegion`, pour pouvoir classer les territoires des deux pays sur un même axe.
- Le **Regierungsbezirk** (district) n'existe pas dans tous les Länder (Berlin, Hamburg…). Le lien Kreis → District est donc facultatif (cardinalité 0,1), avec `SKIP WHEN NULL` dans Oracle et une jointure externe `(+)` pour remonter au Land.
- Une société est soit française, soit allemande : les liens Société → Commune et Société → Gemeinde sont aussi facultatifs.

### Le NAF (activité des sociétés)

Les sociétés allemandes n'ont pas de code NAF, d'où une jointure externe. Un code peut aussi être rattaché directement à un niveau plus haut (par exemple 49.39A → 49). Les niveaux manquants restent vides, avec `SKIP WHEN NULL`.

## Les agrégats (treillis partiel)

| Agrégat | Nœud | Rapport | Calculé à partir de |
|---|---|---|---|
| MV_ACH_GAM_REG_MOIS | (Gamme, Région, All, Mois) | B4.2 | le fait |
| MV_ACH_PAYS_MOIS | (All, Pays, All, Mois) | B4.1 | MV_ACH_GAM_REG_MOIS |
| MV_ACH_TYPE_CJ_MOIS | (All, Type × Catégorie juridique, All, Mois) | B4.3 | le fait |

Chaque agrégat conserve `CodePays`, pour que le filtre de sécurité par pays reste applicable.

## Restitution Power BI

- **Connexion directe à Oracle,** en mode Import, à travers un tunnel SSH vers le serveur de l'université : un clic sur « Actualiser » recharge les données, sans export manuel.
- **Mesures DAX :** Quantité, Montant payé HT, Montant payé TTC, Bénéfice HT, et une mesure **Montant payé** qui bascule entre HT et TTC selon un segment.
- **Sécurité au niveau des lignes :** les rôles *Lyon* et *Berlin* filtrent `DIM_EMPLOYE_MD2[CODEPAYS]`, et le filtre se propage au fait par la relation.
- **Comparaison des années :** des colonnes `NumMois` et `NomMois` (le nom du mois trié par son numéro) permettent de comparer les années mois par mois. Le segment Année filtre les chiffres clés, mais pas le graphique de comparaison.

## Contrôles

- La somme des quantités de l'entrepôt est égale à celle du fait (2 730 = 2 730) : chaque achat a trouvé son tarif, sa réduction et sa version d'employé.
- Aucun doublon de clé dans le fait, aucune clé étrangère orpheline.
- Le rapport montant TTC / montant HT vaut bien 1,20 pour la France et 1,19 pour l'Allemagne.
