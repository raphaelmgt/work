# Amélioration de l'application

## Création des secrets
Dans Kubernetes, les objets secrets permettent de stocker des informations sensibles telles que les variables d'environnement, les mots de passe, ...

Pour cela, j'ai exploré deux approches : 

- Dans la première approche , j'ai créé un fichier `secret.env` qui contient les différentes données à protéger.
On peut ainsi définir les secrets à l'aide d'un `kustomization.yaml` qui me sert simplement à la création des ces objets à l'aide de la commande : 

```bash
kubectl apply -k path_dossier_kustomization/
```

Ou si on ne souhaite pas utiliser de fichier yaml pour créer les secrets :

```bash
kubectl create secret generic secrets-backend --from-env-file=secrets.env
```

- La seconde approche quant à elle repose uniquement sur un fichier `secrets.yaml` qui crée les objets secrets à la la manière d'un deployment ou d'un statefulset avec la commande : 

```bash
kubectl apply -f ./path/secrets.yaml
```

Dans le second point, j'expose les données sensibles directement dans un fichier YAML (ce qui n'est pas une bonne pratique malheureurement). Malgré cela, j'utilise cette méthode pour créer les secrets car je n'arrive pas à connecter le secret à mon backend (le back et la bdd ne peuvent pas communiquer). 

Une fois les secrets définis (quelque soit la démarque), j'ai ensuite modifié le fichier `backend.yaml` en remplaçant chaque 

```yaml
- name: NOM_VARIABLE
  value: valeur_a_proteger
```

par 

```yaml
- name: NOM_VARIABLE
  valueFrom:
    secretKeyRef:
      name: nom_du_secret
      key: DB_USER_SECRET
```

## Mise en place d'une classe de Quality of Service (QoS) de type Guaranteed

Afin de connaitre la QoS d'une pod donné, il suffit d'exécuter la commande suivante :

```bash
kubectl get pod backend-7fcfbf44d5-92bdv -o jsonpath='{.status.qosClass}'
```

Par défaut, le pod hébergeant le backend possède une **QoS:BestEffort** car on a pas définit les ressources. On va ainsi se pencher sur la manière d'obtenir une QoS de type Guaranteed dans la suite.

### Implémentation

Afin de changer une classe QoS d'un Pod, il suffit de configurer les ressources CPU et Mémoire disponibles à l'intérieur de ce dernier. On rajoute donc les lignes suivantes dans le Deployment (au même niveau que l'image) :

```bash
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "200m"  
    memory: "256Mi"
```

**Remarque :** Si on ne précise que les limtes dans le fichier YAML, Kubernetes définit par défaut les requests à la même valeur. Le Pod aura une QoS Guaranteed dans ce cas.

## Mise en place de clustering sur les bases de données postgresql