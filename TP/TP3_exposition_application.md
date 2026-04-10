# Exposition de l'application sur internet

## Exposition sur le réseau interne du cluster

Pour que notre application soit accessible sur le réseau interne au cluster, il faut créer un objet service. Cet objet Kubernetes permet :

- de transférer les requêtes qu'il reçoit aux pods en fonction des ressources disponibles (fonction de Load Balancing)
- d'associer 

Afin d'obtenir toutes les informations au sujet d'un service, il suffit d'exécuter la commande suivante : `kubectl get svc <nom_du_service>`

Ou simplement `kubectl get svc` pour avoir une liste de tous les services actifs sur le cluster.

```bash
>>> kubectl get svc 
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend                ClusterIP   10.233.7.79     <none>        5000/TCP   9h
frontend               ClusterIP   10.233.19.215   <none>        5000/TCP   9h
postgresql-dpmt        ClusterIP   10.233.58.52    <none>        5432/TCP   9h
```

Ici la colone CLUSTER-IP donne l'adresse IP assignée à chaque service. Ces adresses IP donnent l'accès aux applications depuis le cluster.

## Exposition sur internet

Maintenant que notre application est accessible depuis le cluster, on veut pouvoir y accéder depuis notre navigateur préféré et profiter de la magnifique IHM introduite par le frontend. Il faut ainsi associer une URL à notre application (ici, notre frontend).

Pour cela, on crée un objet ingress qui va nous permettre d'exposer notre application sur internet en le connectant au service associé au frontend.

De la même manière que les objets service, la commande `kubectl get ingress <nom_ingress>` permet d'obtenir les information au sujet d'un ingress donné. Pour lister tous les ingress, on peut simplement exécuter la commande `kubectl get ingress` directement dans le terminal.

```bash
>>> kubectl get ingress
NAME                      CLASS    HOSTS                                          ADDRESS   PORTS     AGE
frontend-ingress          nginx    frontend-tp-ingress.lab.sspcloud.fr                      80        9h
```

La colone HOST donne l'URL permettant d'accéder à notre application depuis le navigateur.
