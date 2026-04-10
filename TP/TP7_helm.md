# Helm

## Difficultés rencontrés

- Il ne faut pas spécifier les variables d'environnement qui connectent le backend à la base de données postgres avant d'avoir ajouté CNPG en dépendance du chart. Kubernetes ne va pas créer les pods liés au backend et le deployment ne sera jamais READY.

```bash
>> kubectl get deployment
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
ma-release-backend-app   0/1     1            1           19m
```

- Parfois le cluster postgres ne semble pas terminer son initialisation pour une obscure raison. Lorsque cela se produit, il faut désinstaller puis réinstaller toute la stack pour que cela redémarre normalement.

- A de nombreuses reprises, le cluster ne crée pas la database reviews lors de l'initialisation ce qui fait planter le pod backend (malgré la présence de la variable d'environnement `DB=reviews`.)

## Chart Helm 3tier

Dans cette partie, on se penche sur l'intégration du frontend dans le chart helm.

Pour cela, j'ai créé trois fichiers permettant la création des objets Kubernetes liés à notre frontend :

- un fichier `templates/frontend-deployment.yaml` et un fichier `templates/frontend-service.yaml` qui vont définir le deployment et le service

- et un fichier `templates/ingress.yaml` pour exposer l'entièreté de notre application sur internet et de la rendre accessible via une URL et d'un navigateur.

Une fois tout cela fait, il faut modifier le fichier `values.yaml` en conséquence. Je l'ai donc divisé en deux "sections" pour plus de lisibilité : une section backend et une section frontend.

Enfin, si tout est bien paramétré, on peut installer toute la stack avec `helm install ma-release ./backend-app/` (ou `helm upgrade ma-release ./backend-app` si une partie de l'application à déjà été installée auparavant).

## Mise à jour et rollback

### Mise à jour

Une fois que l'on passe le backend à 2 réplicas et que l'on met à jour la release, on remarque qu'un nouveau pod backend se crée.

```bash
>>> kubectl get pod
NAME                       READY   STATUS    RESTARTS       AGE
backend-58b9d7c985-87sc5   1/1     Running   1 (165m ago)   165m
backend-58b9d7c985-rmvrm   0/1     Running   0              6s
```

On peut aussi obtenir tout un historique des mises à jours effectuées sur une release donnée grâce à :

```bash
>>> helm history ma-release 
REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION     
1               Mon Mar  16 13:40:15 2026        superseded      backend-app-0.2.0       1.1             Install complete
2               Mon Mar  16 16:25:43 2026        deployed        backend-app-0.2.0       1.1             Upgrade complete
```

### Rollback

Une fois qu'une mise à jour à été appliquée, il est possible de revenir à une version précédente : `helm rollback ma-release 1`.

```bash
>>> kubectl get pod
NAME                       READY   STATUS        RESTARTS       AGE
backend-58b9d7c985-87sc5   1/1     Running       1 (178m ago)   179m
backend-58b9d7c985-rmvrm   1/1     Terminating   0              13m
```

On remarque que le pod créé avec la mise à jour précédente est sur le point d'être supprimé, signe que le rollback fonctionne parfaitement.