# Gestion de la persistance


## Les limites du Deployment pour la bdd

Lors du TP3, on a utiliser un deployment pour deployer la base de données PostgreSQL dans des pods.
Cela représente un problème car Kubernetes considère les pods provenant d'un deployment comme des unités jetables. 
Ils peuvent être créés et supprimés à volonté. Chaque fois qu'un pod donné plante ou est mis à jour, ce dernier est simplement remplacé par un nouveau pod. 

Cette approche n'est donc pas adaptée pour des applications qui changent constamment d'état (la base de données Postgres dans notre cas). L'idée est donc de remplacer le Deployment classique par un Statefulset.

## Réalisation et difficultés rencontrées

Pour plus de lisibilité, j'ai déplacé les fichiers yaml qui créent respectivement le Deployment et le Statefulset de la bdd dans deux dossiers distincts :

- un dossier `bdd_deployment` qui contient le fichier `postgresql_deployment.yaml` pour le Deployment

- et un dossier `bdd_statefulset` qui contient le fichier `postgresql_statefulset.yaml`pour le Statefulset.

Maintenant, il faut remplacer l'objet Deployment par le Statefulset et ainsi faire en sorte que la communication s'établisse entre le backend et l'objet Statefulset.
On doit alors réaliser les taches suivantes :

- changer le nom du service présent dans la variable d'environnement `DB_HOST` (on remplace `postgresql-dpmt` par `postgresql-sfs`) et appliquer les changements

- supprimer tous les objets lié au Deployment que l'on veut remplacer

```bash
kubectl delete deployment postgresql-dpmt --ignore-not-found
kubectl delete service postgresql-dpmt --ignore-not-found
```

- créer le Statefulset et son service associé

```bash
kubectl apply -f 3-tiers-application/bdd_statesulset/postgresql_statefulset.yaml
```

- supprimer le pod backend qui était connecté à l'ancienne base de données

```bash
kubectl delete pod <nom_pod_backend>
```

Si toutes les étapes se sont bien passées, un nouveau pod hébergeant le backend de notre application démarrera. Ce dernier sera alors connecté au Statefulset.
Pour s'en convaincre, il suffit de poster une nouvelle review puis de supprimer le pod postgresql-sfs-0 (`kubectl delete pod postgresql-sfs-0`). Une fois qu'il est remplacé, on remarque que l'enregistrement créé précédemment fut préservé. 
