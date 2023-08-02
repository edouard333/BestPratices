# Best-pratices
Documentation centralisé de bonnes pratiques.

# Général
## Versionning
Versionner le code avec la convention : `X.Y.Z`.

* `X` : version majeure, incompatibilité avec la version précédente ou refonte graphique.
* `Y` : version mineure, ajout de fonctionnalité.
* `Z` : correction de bug.

## Gitflow
Avoir deux branches :
* `main` : contient l'application en version stable.
* `developpement` : contient les commits pour le développement.

# Site web
## Endpoints
Etant donné que de base on ne peut manipuler que des `GET` et `POST`, les endpoints contiennent dans leur URL l'action.

* `GET URL/livre` : consulte liste de livre.
* `GET/POST URL/livre/creer` : ajoute un livre (`GET` est pour avoir la page web avec le formulaire et le `POST` quand on le valide).
* `GET URL/livre/{id_livre}` : consulte un livre selon son ID.
* `GET/POST URL/livre/{id_livre}/modifier` : modifie un livre selon son ID (`GET` est pour avoir la page web avec le formulaire et le `POST` quand on le valide).
* `GET/POST URL/livre/{id_livre}/supprimer` : supprime un livre selon son ID (`GET` est pour avoir la page web avec le formulaire et le `POST` quand on le valide).

## Tests
Selenium IDE pour les tests UI.
Idéalement, utiliser des ID sur les balises pour sélectionner l'élément (même si ce n'est pas intuitif le ID sur un label ou un button).


# API REST
Il ne faut pas se faire avoir : l'API REST ne doit pas refléter les tables d'une quelconque base de données, mais chaque endpoints doit refléter le processus métier. Il peut y avoir des ressources, mais les opérations dessus doivent refléter le processus métier (si un processus demande d'utiliser plusieurs tables DB, dans l'API cela ne doit pas être d'office le cas).

## Verbes HTTP et status code de réponse cas nominaux
Respecter les règles d'API RESTfull.

Les verbes :
* `POST` : créer une ressource OU rechercher une ressource (filtre dans le `body`).\
  Réponse nominale :
  * `200` : liste des ressources selon le filtre de recherche.
  * `201` : ressource créée avec réponse. La réponse est l'ID de la ressource créée.
  * `204` : ressource créée sans réponse.
* `GET` : consulter une ressource (ou liste de ressource).\
Réponse nominale :
  * `200` : la ressource (ou une liste de ressource).
* `PUT` : modifier une ressource.\
  Réponse nominale :
  * `204` : ressource modifiée sans réponse.
* `PATCH` : modifier partiellement une ressource.\
  Réponse nominale :
  * `204` : ressource modifiée sans réponse.
* `DELETE` : supprimer une ressource.\
  Réponse nominale :
  * `204` : ressource supprimée sans réponse.

**Note** :
Certains recommandent que pour un *create* et *edit* on retourne la ressource. Mais nous laissons au client le soin de le récupérer via le endpoint adéquat. Cela permet également de forcer un test API en consommant le endpoint de récupération de la ressource. Cela évite également qu'un même endpoint réalise plusieurs actions.

Les versions de l'API sont dans l'URL : `/api/v1/`.

## Endpoints
* `/api/v1/ressource/` : à propos d'une ressource.
* `POST /api/v1/livre/` : ajoute un livre.
* `POST /api/v1/livre/recherche` : recherche de livre selon un filtre. Cela retourne une liste.
* `GET /api/v1/livre/` : consulte la liste des livres.
* `GET /api/v1/livre/{id_livre}` : consulte un livre selon son ID.
* `PUT /api/v1/livre/{id_ressource}` : modifie un livre selon son ID.
* `PATCH /api/v1/livre/{id_ressource}/titre` : modifie le titre d'un livre selon son ID.
* `DELETE /api/v1/livre/{id_ressource}` : supprime un livre selon son ID.

Cela représente un [CRUD](https://fr.wikipedia.org/wiki/CRUD#:~:text=L%27acronyme%20informatique%20anglais%20CRUD,informations%20en%20base%20de%20données.) sur une ressource.

**Note** : pour un `PUT` et `PATCH` nous ne pouvons pas vérifier si un certain nombre d'entrées sont modifié pour si l'utilisateur envoie les mêmes informations, ce qui fait qu'il n'y a aucune modification (cela n'a pas de sens, mais cela en a quand on valide par sécurité un formulaire) et on ne veut pas une erreur dans ce cas, sinon cela prête plus à confusion.

S'il y a un corps à la requête et/ou réponse, le corps doit d'office être un `JSON`, même pour un simple `string`.

## Status code cas non-nominaux
Concernant les status code d'erreur (ceux nominaux sont avec les verbes).
* `400` : erreur (générique) de la part du client.
* `401` : il faut être authentifié/connecté.
* `403` : accès interdit (même authentifié).
* `404` : n'existe pas. Par exemple quand on veut récupérer une ressource via son ID et que rien n'existe.\
Dans le cadre d'une liste, on retourne simplement un tableau vide `[]` et non `404` (plus simple à code des deux côtés, le serveur retourne simplement le résultat et le client ne fait que consulter un tableau vide au lieu de `404/null`).
* `409` : conflit avec la ressource sur le serveur, elle existe déjà, 2 comptes ne peuvent pas avoir le même identifiant, etc.
* `422` : la ressource envoyées n'est pas correctement formatée.
* `500` : erreur serveur (interne) qui n'est pas dû au client.

## Pagination
Paramètres : `offset` et `limit` pour API.\
`page` paramètre pour UI.\
Avoir les paramètres dans l'URI mais après le `?` comme ça pas une nouvelle action et rend bien optionnel les info. (cas `compte/{id_compte}` et `compte/{offset}`, on fait comment la diff entre les deux routes ?) Cela permet également de garder des requêtes `GET`.\
Les informations de `taille`, `limit`, `offset` de la réponse sont dans le header dans les champs personnalisés, comme ça le corps de la réponse reste un tableau d'objet.

## Documentation
Avoir des commentaires sur les controllers au minimum.

Utiliser la documentation d'[OpenAPI](https://swagger.io/specification/).\
Ajouter la documentation :
1. Paramètres URL (s'il y en a), même pour ceux optionnels (pagination par exemple).
2. Réponses possibles, cas nominaux : `200`, `201`, etc.
3. Body de la requête, s'il y en a.
4. Réponses possibles, cas non-nominaux : `400`, `401`, etc.


## Tests
Via Postman via "run collection" pour tester l'API.
Liste des vérifications, des plus simple ou fondamentale au plus complexe et chronophage.

**Tests "techniques"** :
1. Tester un maximum d'endpoint (idéalement tous). Et s'il y a de l'authentification, que cela fonctionne.
2. Vérifier le **status code** attendu de la réponse : `200/GET`, `201/POST`, etc...
3. Vérifier que le corps de la réponse est un `JSON` (s'il y a un corps).
4. Vérifier que le corps de la réponse est le **type attendu** : `empty`, `object` ou `array`.
5. Vérifier que pour une réponse contenant une liste il y ait des éléments.
6. Vérifier de récupérer le **bon nombre d'élément** dans la liste.
7. Vérifier que les données de la réponse sont **correctement structurées** : `object`, ok mais `CompteDTO` ?
8. Vérifier que les **données de la réponse** sont ceux attendus.
9. Vérifier des **cas non-nominaux mes techniques** : pas de body dans la requête, utiliser un mauvais verbe, donner des champs `null`.

**Tests "business"** :

10. Vérifier un **processus métier** intégralement. Ceux nécessitant l'utilisation de plusieurs endpoints (car, normalement, sinon cela a pu être testé dans les tests techniques).
11. Les tests sont **relancables facilement** (sans devoir faire une opération manuelle entre).
11. Vérifier les **cas nominaux** (simple).
12. Vérifier les cas nominaux particuliers.
13. Vérifier les cas non-nominaux simples : ne pas renseigner un champ 
10. Utiliser de mauvaises données volontairement : champ vide (null ou "") pour des champs obligatoires, deux fois le même nom (qui doit être unique) de compte.
