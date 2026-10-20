# Guide d'exécution

## 1. Créer et charger l'entrepôt (Oracle 11g)

Dans SQL Developer, connecté à **votre propre schéma**, lancez les scripts dans l'ordre avec **F5** (« Exécuter le script ») :

| Ordre | Script | Rôle |
|---|---|---|
| 1 | `sql/01_creation_ED.sql` | Crée les 29 tables de l'entrepôt (suffixe `_DW`) |
| 2 | `sql/02_contraintes_ED.sql` | Ajoute les contraintes métier (CHECK, déclencheurs) |
| 3 | `sql/03_insertion_ED.sql` | Charge le jeu de données (environ 12 000 lignes) |
| 4 | `sql/04_MD2_Ventes.sql` | Crée le magasin Ventes : dimensions, fait, agrégats, puis contrôles |

> **Attention.** Le script 01 supprime d'abord les tables du même nom (`DROP … PURGE`, suppression définitive). Vérifiez votre schéma avant de le lancer :
>
> ```sql
> SELECT USER FROM dual;
> ```

À la fin du script 4, la requête de contrôle doit renvoyer **2730 | 2730**.

Si votre compte n'a pas le droit `QUERY REWRITE`, retirez la ligne `ENABLE QUERY REWRITE` des deux agrégats concernés.

**Après chaque nouveau chargement de l'entrepôt**, rafraîchissez le magasin :

```sql
EXEC DBMS_MVIEW.REFRESH('DIM_TEMPS_MD2,DIM_PRODUIT_MD2,DIM_EMPLOYE_MD2,DIM_METIER_MD2,FAIT_ACHAT_MD2,MV_ACH_GAM_REG_MOIS,MV_ACH_PAYS_MOIS,MV_ACH_TYPE_CJ_MOIS');
```

## 2. Régénérer les données (facultatif)

```
python3 generation/gen_donnees.py
```

À lancer depuis la racine du dépôt. Le script réécrit `sql/03_insertion_ED.sql`. La graine est fixe, donc les données obtenues sont toujours les mêmes, et le générateur vérifie lui-même les règles de gestion.

Contenu du jeu de données :
- un historique d'octobre 2024 à octobre 2026, avec une activité allemande à partir de janvier 2026 ;
- 20 sociétés françaises et allemandes, 403 employés, 8 agences, 256 exemplaires ;
- 5 100 locations, 2 200 achats et 1 000 réservations ;
- une saisonnalité réaliste (pic au printemps et en été).

## 3. Ouvrir le rapport Power BI

1. Ouvrez un tunnel SSH vers le serveur Oracle (port local au choix, par exemple 1550).
2. Ouvrez `powerbi/MD2_Ventes.pbix` dans Power BI Desktop.
3. Si besoin, changez la source : **Transformer les données → Paramètres de la source de données**, serveur `localhost:<port>/<SID>`, identifiants Oracle en mode **Base de données**.
4. Cliquez sur **Actualiser**.
5. Pour tester la sécurité : **Modélisation → Voir comme → Lyon** ou **Berlin**.
