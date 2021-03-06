---
machine_translated: true
---

# Dictionnaires Hiérarchiques {#hierarchical-dictionaries}

Clickhouse prend en charge les dictionnaires hiérarchiques avec un [touche numérique](external_dicts_dict_structure.md#ext_dict-numeric-key).

Voici une structure hiérarchique:

``` text
0 (Common parent)
│
├── 1 (Russia)
│   │
│   └── 2 (Moscow)
│       │
│       └── 3 (Center)
│
└── 4 (Great Britain)
    │
    └── 5 (London)
```

Cette hiérarchie peut être exprimée comme la table de dictionnaire suivante.

| id\_région | région\_parent | nom\_région        |
|------------|----------------|--------------------|
| 1          | 0              | Russie             |
| 2          | 1              | Moscou             |
| 3          | 2              | Center             |
| 4          | 0              | La Grande-Bretagne |
| 5          | 4              | Londres            |

Ce tableau contient une colonne `parent_region` qui contient la clé du parent le plus proche de l'élément.

Clickhouse soutient le [hiérarchique](external_dicts_dict_structure.md#hierarchical-dict-attr) propriété pour [externe dictionnaire](index.md) attribut. Cette propriété vous permet de configurer le dictionnaire hiérarchique comme décrit ci-dessus.

Le [dictGetHierarchy](../functions/ext_dict_functions.md#dictgethierarchy) la fonction vous permet d'obtenir la chaîne parent d'un élément.

Pour notre exemple, la structure du dictionnaire peut être la suivante:

``` xml
<dictionary>
    <structure>
        <id>
            <name>region_id</name>
        </id>

        <attribute>
            <name>parent_region</name>
            <type>UInt64</type>
            <null_value>0</null_value>
            <hierarchical>true</hierarchical>
        </attribute>

        <attribute>
            <name>region_name</name>
            <type>String</type>
            <null_value></null_value>
        </attribute>

    </structure>
</dictionary>
```

[Article Original](https://clickhouse.tech/docs/en/query_language/dicts/external_dicts_dict_hierarchical/) <!--hide-->
