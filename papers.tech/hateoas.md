# Hypermedia As The Engine Of Application State

Le terme State est de plus en plus démocratisé dans le monde du dévéloppement, en tout cas pour ce qui concerne le développement web. J'ai volontairement omis de préciser s'il s'agissait de la partie frontend, backend ou même mobile car ces 3 composantes sont concernées

## Qu'est ce qu'un state

Le state, ou état, est la représentation de la logique de l'application (en général, mais nous pouvons considérer comme le contexte dans lequel une action est effectuée). Il s'agit d'une couche transverse et haut niveau, partagé avec les différents composants de mon application. Cela ne parait pas clair et cela peut du coup très vite devenir problématique à concevoir. Ce n'est pas anodin que de nombreux outils permettent de manipuler tout cela pour réduire cette charge de développement.

### Exemple: application générique

Des exemples, je pourrais vous en fournir une infinité, mais partons plutôt de principe ou fondements.

Prenons l'exemple d'une application qui manipule un produit. Restons vague, c'est d'ailleurs tout l'intérêt des développements génériques.

Cette application permet donc de publier un produit, de le désactiver (et réactiver), de manipuler son prix. D'un point de vue interface graphique, prenons le choix de faire une page avec 3 boutons distincts.

1. La publication d'un produit permet de créer une fiche produit. Aucune contre indication pour créer des doublons. Un simple formulaire avec des champs requis… dont le prix.
2. La désactivation d'un produit ne peut se faire que sur un produit déjà enregistré. La désactivation d'un produit désactivé n'a aucune incidence sur le produit. De même que l'activation.
3. La modification d'un prix ne peut se faire que sur un produit activé

Nous commencons à percevoir des dépendances entre ces actions. L'idée n'est pas de débattre si la solution du state est pertinente mais de comprendre en quoi elle est utile.

**Côté serveur**

`GET /product?id=aec4b-457d4-fab78`
```json
{
    "_id": "aec4b-457d4-fab78",
    "status": "activated",
    "name": "sneaker",
    "shop": "wethenew",
    "pro": {
        "_id", "wtn-143423424"
    },
    "links": {
        "deactivate": {
            "method": "POST",
            "path": "product/deactivate"
        },
        "change_price": {
            "method": "PATCH",
            "path": "/product/price"
        }
    }
}
```

HATEOAS est une "manière" de gérer les actions possibles sur une entité en fonction de l'état de l'application (droits ou règle de métier par exemple). Nous pouvons donc très bien imaginer qu'un utilisateur n'ayant pas les droits de modifier le prix n'a pas accès à son action. Cela permet de gérer les droits côtés backend. Idéalement, le frontend récupérera les informations du produit et en fonction des actions disponibles, mettra à jour la page.

`GET /product?id=aec4b-457d4-fab78`
```json
{
    "_id": "aec4b-457d4-fab78",
    "status": "deactivated",
    "name": "sneaker",
    "shop": "wethenew",
    "pro": {
        "_id", "wtn-143423424"
    },
    "links": {
        "activate": {
            "method": "POST",
            "path": "product/activate"
        },
        "change_price": {
            "method": "PATCH",
            "path": "/product/price"
        }
    }
}
```

Là encore, cette solution n'est pas forcément optimale, pourquoi avoir une adresse spécifique pour l'activation et une autre pour la désactivation. Ce billet est là pour expliquer le concept du state basé sur les metadata d'une entité.

## Implémentations

Plusieurs implémentations existent pour gérer ce concept. Le premier parlera un peu plus aux développeurs frontend, dans le sens où le format JSON-LD est un format pouvant être utilisé côté SEO pour la déclaration sémantique de contenu de page.

Il met l'accent sur l'accession à des compléments d'informations plutôt qu'à la manipulation. Cela veut dire que les liens d'état géreront la pagination, la liste des images d'un produit, les offres d'une marketplace.

Chaque norme d'HATEOAS possède sa propre syntaxe et rend plus ou moins lisible en fonction de l'uage

HAL est la norme que j'ai utilisé dans les éléments de code de cet article. Il est possible de séparer les actions des éléments de navigations

```json
"actions": [{
    "path": "/product/1234567890/image",
    "method": "POST",
    "fields": [
        {"name": "name", "type": "string"},
        {"name": "alternateName", "type": "string"},
        {"name": "image", "type": "blob"}
    ]
}],
"links": {
    "current": "/current-url",
    "next": "/some-url",
    "related-model": "/related-url"
}
```

Il est possible d'ajouter une image à un produit mais également de naviguer dans une gamme de produit par exemple.