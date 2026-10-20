# Référentiels externes

Ces fichiers publics ont servi à construire les tables de référence de l'entrepôt : zones géographiques, catégories juridiques et codes AGS. Le jeu de données du projet n'en reprend qu'un extrait : les communes, Kreise et Länder où se trouvent les sociétés clientes.

## Fichiers sources (`sources/`)

| Fichier | Contenu | Source | Utilisé pour |
|---|---|---|---|
| `INSEE_categories_juridiques_2022.xls` | Nomenclature des catégories juridiques sur 3 niveaux | INSEE | Table `CATEGORIE_JURIDIQUE_DW`, hiérarchie Niveau 3 → 2 → 1 |
| `France_communes_departements_regions.xlsx` | Communes françaises, avec leur département et leur région | Code officiel géographique de l'INSEE | Zones françaises de `ZONE_GEO_DW` (Commune → Département → Région) |
| `Destatis_Gemeindeverzeichnis_30092026.xlsx` | Répertoire des communes allemandes (Gemeinden) avec leur code officiel (AGS / ARS) | Statistisches Bundesamt (Destatis) | Codes et noms des Gemeinden, Kreise, Regierungsbezirke et Länder |
| `Destatis_Verwaltungsgliederung_31122025.xlsx` et `..._30092026.xlsx` | Découpage administratif de l'Allemagne par Land : nombre de Regierungsbezirke, Kreise et Gemeinden | Destatis | Vérifier quels Länder ont des Regierungsbezirke (niveau facultatif) |

## Fichier préparé par l'équipe (`prepares/`)

`Allemagne_zones_geo.xlsx` reprend le répertoire Destatis au format de la table `ZONE_GEO_DW` : `CodeZG`, `NomZG`, `CodeParent`, `TypeZG`. Il a un onglet par niveau (Land, Regierungsbezirke, Kreise, Gemeinden), plus un onglet « Territoire allemand » qui les réunit, soit environ 11 000 zones.

Le code officiel allemand (AGS) est construit par emboîtement : chaque niveau supérieur est un préfixe du code. Par exemple, München `09162000` → Kreis `09162` → Regierungsbezirk `091` (Oberbayern) → Land `09` (Bayern). C'est cette propriété qui permet de calculer le parent de chaque zone. Dans l'entrepôt, les codes allemands sont préfixés par `DE` pour ne jamais entrer en collision avec les codes français.

## Licences

- Données INSEE : Licence Ouverte / Open Licence (Etalab).
- Données Destatis : © Statistisches Bundesamt (Destatis), Datenlizenz Deutschland – Namensnennung – Version 2.0.
