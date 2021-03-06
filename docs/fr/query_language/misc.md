---
machine_translated: true
---

# Diverses Requêtes {#miscellaneous-queries}

## ATTACH {#attach}

Cette requête est exactement la même que `CREATE`, mais

-   Au lieu de la parole `CREATE` il utilise le mot `ATTACH`.
-   La requête ne crée pas de données sur le disque, mais suppose que les données sont déjà aux endroits appropriés, et ajoute simplement des informations sur la table au serveur.
    Après avoir exécuté une requête ATTACH, le serveur connaîtra l'existence de la table.

Si la table a été précédemment détachée (`DETACH`), ce qui signifie que sa structure est connue, vous pouvez utiliser un raccourci sans définir la structure.

``` sql
ATTACH TABLE [IF NOT EXISTS] [db.]name [ON CLUSTER cluster]
```

Cette requête est utilisée lors du démarrage du serveur. Le serveur stocke les métadonnées de la table sous forme de fichiers avec `ATTACH` requêtes, qu'il exécute simplement au lancement (à l'exception des tables système, qui sont explicitement créées sur le serveur).

## CHECK TABLE {#check-table}

Vérifie si les données de la table sont corrompues.

``` sql
CHECK TABLE [db.]name
```

Le `CHECK TABLE` requête compare réelle des tailles de fichier avec les valeurs attendues qui sont stockés sur le serveur. Si le fichier tailles ne correspondent pas aux valeurs stockées, cela signifie que les données sont endommagées. Cela peut être causé, par exemple, par un plantage du système lors de l'exécution de la requête.

La réponse de la requête contient `result` colonne avec une seule ligne. La ligne a une valeur de
[Booléen](../data_types/boolean.md) type:

-   0 - les données de la table sont corrompues.
-   1 - les données maintiennent l'intégrité.

Le `CHECK TABLE` query prend en charge les moteurs de table suivants:

-   [Journal](../operations/table_engines/log.md)
-   [TinyLog](../operations/table_engines/tinylog.md)
-   [StripeLog](../operations/table_engines/stripelog.md)
-   [Famille MergeTree](../operations/table_engines/mergetree.md)

Effectué sur les tables avec un autre moteur de table provoque une exception.

Les moteurs de la `*Log` la famille ne fournit pas de récupération automatique des données en cas d'échec. L'utilisation de la `CHECK TABLE` requête pour suivre la perte de données en temps opportun.

Pour `MergeTree` moteurs de la famille, le `CHECK TABLE` query affiche un État de vérification pour chaque partie de données individuelle d'une table sur le serveur local.

**Si les données sont corrompues**

Si la table est corrompue, vous pouvez copier les données non corrompues dans une autre table. Pour ce faire:

1.  Créez une nouvelle table avec la même structure que la table endommagée. Pour ce faire exécutez la requête `CREATE TABLE <new_table_name> AS <damaged_table_name>`.
2.  Définir le [max\_threads](../operations/settings/settings.md#settings-max_threads) la valeur 1 pour traiter la requête suivante dans un seul thread. Pour ce faire, exécutez la requête `SET max_threads = 1`.
3.  Exécuter la requête `INSERT INTO <new_table_name> SELECT * FROM <damaged_table_name>`. Cette demande copie les données non corrompues de la table endommagée vers une autre table. Seules les données avant la partie corrompue seront copiées.
4.  Redémarrez l' `clickhouse-client` pour réinitialiser l' `max_threads` valeur.

## DESCRIBE TABLE {#misc-describe-table}

``` sql
DESC|DESCRIBE TABLE [db.]table [INTO OUTFILE filename] [FORMAT format]
```

Renvoie ce qui suit `String` les colonnes de type:

-   `name` — Column name.
-   `type`— Column type.
-   `default_type` — Clause that is used in [expression par défaut](create.md#create-default-values) (`DEFAULT`, `MATERIALIZED` ou `ALIAS`). Column contient une chaîne vide, si l'expression par défaut n'est pas spécifiée.
-   `default_expression` — Value specified in the `DEFAULT` clause.
-   `comment_expression` — Comment text.

Les structures de données imbriquées sont sorties dans “expanded” format. Chaque colonne est affichée séparément, avec le nom après un point.

## DETACH {#detach}

Supprime les informations sur le ‘name’ table du serveur. Le serveur cesse de connaître l'existence de la table.

``` sql
DETACH TABLE [IF EXISTS] [db.]name [ON CLUSTER cluster]
```

Cela ne supprime pas les données ou les métadonnées de la table. Lors du prochain lancement du serveur, le serveur Lira les métadonnées et découvrira à nouveau la table.
De même, un “detached” tableau peut être re-attaché en utilisant le `ATTACH` requête (à l'exception des tables système, qui n'ont pas de stocker les métadonnées pour eux).

Il n'y a pas de `DETACH DATABASE` requête.

## DROP {#drop}

Cette requête a deux types: `DROP DATABASE` et `DROP TABLE`.

``` sql
DROP DATABASE [IF EXISTS] db [ON CLUSTER cluster]
```

Supprime toutes les tables à l'intérieur de la ‘db’ la base de données, puis supprime le ‘db’ la base de données elle-même.
Si `IF EXISTS` est spécifié, il ne renvoie pas d'erreur si la base de données n'existe pas.

``` sql
DROP [TEMPORARY] TABLE [IF EXISTS] [db.]name [ON CLUSTER cluster]
```

Supprime la table.
Si `IF EXISTS` est spécifié, il ne renvoie pas d'erreur si la table n'existe pas ou si la base de données n'existe pas.

    DROP DICTIONARY [IF EXISTS] [db.]name

Delets le dictionnaire.
Si `IF EXISTS` est spécifié, il ne renvoie pas d'erreur si la table n'existe pas ou si la base de données n'existe pas.

## EXISTS {#exists}

``` sql
EXISTS [TEMPORARY] [TABLE|DICTIONARY] [db.]name [INTO OUTFILE filename] [FORMAT format]
```

Renvoie un seul `UInt8`- type colonne, qui contient la valeur unique `0` si la table ou base de données n'existe pas, ou `1` si la table existe dans la base de données spécifiée.

## KILL QUERY {#kill-query}

``` sql
KILL QUERY [ON CLUSTER cluster]
  WHERE <where expression to SELECT FROM system.processes query>
  [SYNC|ASYNC|TEST]
  [FORMAT format]
```

Tente de mettre fin de force aux requêtes en cours d'exécution.
Les requêtes à terminer sont sélectionnées dans le système.processus en utilisant les critères définis dans le `WHERE` la clause de la `KILL` requête.

Exemple:

``` sql
-- Forcibly terminates all queries with the specified query_id:
KILL QUERY WHERE query_id='2-857d-4a57-9ee0-327da5d60a90'

-- Synchronously terminates all queries run by 'username':
KILL QUERY WHERE user='username' SYNC
```

Les utilisateurs en lecture seule peuvent uniquement arrêter leurs propres requêtes.

Par défaut, la version asynchrone des requêtes est utilisé (`ASYNC`), qui n'attend pas la confirmation que les requêtes se sont arrêtées.

La version synchrone (`SYNC`) attend que toutes les requêtes d'arrêter et affiche des informations sur chaque processus s'arrête.
La réponse contient l' `kill_status` la colonne, qui peut prendre les valeurs suivantes:

1.  ‘finished’ – The query was terminated successfully.
2.  ‘waiting’ – Waiting for the query to end after sending it a signal to terminate.
3.  The other values ​​explain why the query can't be stopped.

Une requête de test (`TEST`) vérifie uniquement les droits de l'utilisateur et affiche une liste de requêtes à arrêter.

## KILL MUTATION {#kill-mutation}

``` sql
KILL MUTATION [ON CLUSTER cluster]
  WHERE <where expression to SELECT FROM system.mutations query>
  [TEST]
  [FORMAT format]
```

Essaie d'annuler et supprimer [mutation](alter.md#alter-mutations) actuellement en cours d'exécution. Les Mutations à annuler sont sélectionnées parmi [`system.mutations`](../operations/system_tables.md#system_tables-mutations) tableau à l'aide du filtre spécifié par le `WHERE` la clause de la `KILL` requête.

Une requête de test (`TEST`) vérifie uniquement les droits de l'utilisateur et affiche une liste de requêtes à arrêter.

Exemple:

``` sql
-- Cancel and remove all mutations of the single table:
KILL MUTATION WHERE database = 'default' AND table = 'table'

-- Cancel the specific mutation:
KILL MUTATION WHERE database = 'default' AND table = 'table' AND mutation_id = 'mutation_3.txt'
```

The query is useful when a mutation is stuck and cannot finish (e.g. if some function in the mutation query throws an exception when applied to the data contained in the table).

Les modifications déjà apportées par la mutation ne sont pas annulées.

## OPTIMIZE {#misc_operations-optimize}

``` sql
OPTIMIZE TABLE [db.]name [ON CLUSTER cluster] [PARTITION partition | PARTITION ID 'partition_id'] [FINAL] [DEDUPLICATE]
```

Cette requête tente d'initialiser une fusion non programmée de parties de données pour les tables avec un moteur de [MergeTree](../operations/table_engines/mergetree.md) famille.

Le `OPTMIZE` la requête est également prise en charge pour [MaterializedView](../operations/table_engines/materializedview.md) et la [Tampon](../operations/table_engines/buffer.md) moteur. Les autres moteurs de table ne sont pas pris en charge.

Lorsque `OPTIMIZE` est utilisé avec le [ReplicatedMergeTree](../operations/table_engines/replication.md) famille de moteurs de table, ClickHouse crée une tâche pour la fusion et attend l'exécution sur tous les nœuds (si le `replication_alter_partitions_sync` paramètre est activé).

-   Si `OPTIMIZE` n'effectue pas de fusion pour une raison quelconque, il ne notifie pas le client. Pour activer les notifications, utilisez [optimize\_throw\_if\_noop](../operations/settings/settings.md#setting-optimize_throw_if_noop) paramètre.
-   Si vous spécifiez un `PARTITION`, seule la partition spécifiée est optimisé. [Comment définir l'expression de la partition](alter.md#alter-how-to-specify-part-expr).
-   Si vous spécifiez `FINAL`, l'optimisation est effectuée, même lorsque toutes les données sont déjà dans une partie.
-   Si vous spécifiez `DEDUPLICATE`, alors des lignes complètement identiques seront dédupliquées (toutes les colonnes sont comparées), cela n'a de sens que pour le moteur MergeTree.

!!! warning "Avertissement"
    `OPTIMIZE` ne peut pas réparer le “Too many parts” erreur.

## RENAME {#misc_operations-rename}

Renomme une ou plusieurs tables.

``` sql
RENAME TABLE [db11.]name11 TO [db12.]name12, [db21.]name21 TO [db22.]name22, ... [ON CLUSTER cluster]
```

Toutes les tables sont renommées sous verrouillage global. Renommer des tables est une opération légère. Si vous avez indiqué une autre base de données après TO, la table sera déplacée vers cette base de données. Cependant, les répertoires contenant des bases de données doivent résider dans le même système de fichiers (sinon, une erreur est renvoyée).

## SET {#query-set}

``` sql
SET param = value
```

Assigner `value` à l' `param` [paramètre](../operations/settings/index.md) pour la session en cours. Vous ne pouvez pas modifier [les paramètres du serveur](../operations/server_settings/index.md) de cette façon.

Vous pouvez également définir toutes les valeurs de certains paramètres de profil dans une seule requête.

``` sql
SET profile = 'profile-name-from-the-settings-file'
```

Pour plus d'informations, voir [Paramètre](../operations/settings/settings.md).

## TRUNCATE {#truncate}

``` sql
TRUNCATE TABLE [IF EXISTS] [db.]name [ON CLUSTER cluster]
```

Supprime toutes les données d'une table. Lorsque la clause `IF EXISTS` est omis, la requête renvoie une erreur si la table n'existe pas.

Le `TRUNCATE` la requête n'est pas prise en charge pour [Vue](../operations/table_engines/view.md), [Fichier](../operations/table_engines/file.md), [URL](../operations/table_engines/url.md) et [NULL](../operations/table_engines/null.md) table des moteurs.

## USE {#use}

``` sql
USE db
```

Vous permet de définir la base de données actuelle pour la session.
La base de données actuelle est utilisée pour rechercher des tables si la base de données n'est pas explicitement définie dans la requête avec un point avant le nom de la table.
Cette requête ne peut pas être faite lors de l'utilisation du protocole HTTP, car il n'y a pas de concept de session.

[Article Original](https://clickhouse.tech/docs/en/query_language/misc/) <!--hide-->
