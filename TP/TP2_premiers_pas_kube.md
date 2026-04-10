# Premiers pas sur Kubernetes

**Disclaimer :** J'ai eu quelques soucis avec mon image python sur DockerHub. 
J'ai donc réalisé ce compte rendu avec l'application 3 tiers du TP suivant.

## Les pods

Dans Kubernetes, un pod représente la plus petite unité d'exécution. 
C'est lui qui va héberger une application donnée.

```bash
>>> kubectl get pod
NAME                       READY   STATUS    RESTARTS      AGE
backend-58b9d7c985-87sc5   1/1     Running   1 (44h ago)   44h
frontend-64dfb7b67-z8t97   1/1     Running   0             44h
ma-release-postgresql-1    1/1     Running   0             44h
```

En exécutant la commande `kubectl get pod`, on peut lister tous les pods actifs et connaitre diverses informations telles que l'état actuel de chaque pod (Running, Terminating, ...) ou le nombre de restarts associé à son cycle de vie.

Si on constate un problème sur un pod donné, on peut aussi regarder les logs associés à l'application qu'il héberge : `kubectl logs <nom_pod>`.

Enfin, afin de lister toutes les dépendances d'une application, on peut exécuter l'une des commandes suivantes (en fonction du langage utilisé) :

- Node.js : `kubectl exec <nom_pod> -- npm list`

- Python : `kubectl exec <nom_pod> -- pip freeze`

- Java (Maven) : `kubectl exec <nom_pod> -- mvn dependency:tree`

```bash
>>> kubectl exec backend-58b9d7c985-87sc5 -- pip freeze
click==8.0.4
dataclasses==0.8
Flask==2.0.3
importlib-metadata==4.8.3
itsdangerous==2.0.1
Jinja2==3.0.3
MarkupSafe==2.0.1
psycopg2-binary==2.9.5
typing_extensions==4.1.1
Werkzeug==2.0.3
zipp==3.6.0
```

On obtient donc une liste avec l'ensemble des packages et les versions utilisés par l'application.

## Les deployments

Tout comme les pods, il est possible d'obtenir des informations au sujet des différents deployments actifs.

- Pour lister tous les deployments :

```bash
>>> kubectl get deployment
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    1/1     1            1           2d4h
frontend   1/1     1            1           2d4h
```

Ici, on a accès à des informations très générales sur les deployments tels que le nombre de pods que gèrent chaque deployment.

- Pour obtenir des informations plus poussées au sujet d'un deployment ou d'un pod : `kubectl describe deployment <nom_deployment>` ou simplement `kubectl describe <nom_pod>`.
