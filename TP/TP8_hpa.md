# Autoscaling avec le Horizontal Pod Autoscaler

**Remarque importante :** Pour que le hpa fonctionne, il faut définir les ressources pour le backend ET le frontend.
Dans le cas où on ne définit les ressources CPU et mémoire que dans le deployment du backend, on a :

```bash
>>> kubectl get hpa
NAME   REFERENCE            TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
hpa    Deployment/backend   cpu: <unknown>/70%   3         10        3          4d7h
```

Ici, le hpa affiche `cpu: <unknown>/70%`

On peut aussi constater l'erreur suivante dans la description du HPA 

```bash
>>> kubectl describe hpa
...
Events:
  Type     Reason                        Age                  From                       Message
  ----     ------                        ----                 ----                       -------
  Normal   SuccessfulRescale             40m (x2 over 41m)    horizontal-pod-autoscaler  New size: 3; reason: Current number of replicas below Spec.MinReplicas
  Warning  FailedComputeMetricsReplicas  38m (x12 over 41m)   horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu resource metric value: failed to get cpu utilization: missing request for cpu in container frontend of Pod frontend-64dfb7b67-pdlq8
  Warning  FailedGetResourceMetric       66s (x160 over 41m)  horizontal-pod-autoscaler  failed to get cpu utilization: missing request for cpu in container frontend of Pod frontend-64dfb7b67-pdlq8
```

Une fois ce problème réglé, on peut maintenant observer la charge cpu gérée par le hpa.

```bash
kubectl get hpa
NAME          REFERENCE            TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
backend-hpa   Deployment/backend   cpu: 1%/70%   3         10        3          54m
```

## Observer le scaling

**Observations**: 

- Lorsque l'on crée le load-generator, la charge cpu augmente rapidement et parvient même à dépasser la limite (ici 70 %).
Les pods n'arrivent visiblement pas à gérer tout l'afflut entrant (ce qui est l'effet désiré par la création du load-generator).

- Le hpa va alors progressivement créer de nouveaux pods hébergeant le backend afin de faire face à l'augmentation de la charge. 
On peut observer ce processus grâce à un `kubectl get pods --watch`.

- Enfin, la charge cpu diminue progressivement jusqu'à se stabiliser autour des 65 %.

```bash
>>> kubectl get hpa --watch
NAME          REFERENCE            TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
backend-hpa   Deployment/backend   cpu: 59%/70%   3         10        3          6h6m
backend-hpa   Deployment/backend   cpu: 114%/70%   3         10        4          6h6m
backend-hpa   Deployment/backend   cpu: 126%/70%   3         10        7          6h6m
backend-hpa   Deployment/backend   cpu: 115%/70%   3         10        7          6h7m
backend-hpa   Deployment/backend   cpu: 82%/70%    3         10        7          6h7m
backend-hpa   Deployment/backend   cpu: 66%/70%    3         10        7          6h7m
backend-hpa   Deployment/backend   cpu: 65%/70%    3         10        7          6h7m
backend-hpa   Deployment/backend   cpu: 66%/70%    3         10        7          6h8m
backend-hpa   Deployment/backend   cpu: 65%/70%    3         10        7          6h9m
backend-hpa   Deployment/backend   cpu: 66%/70%    3         10        7          6h10m
backend-hpa   Deployment/backend   cpu: 63%/70%    3         10        7          6h10m
backend-hpa   Deployment/backend   cpu: 64%/70%    3         10        7          6h10m
backend-hpa   Deployment/backend   cpu: 65%/70%    3         10        7          6h11m
backend-hpa   Deployment/backend   cpu: 67%/70%    3         10        7          6h11m
backend-hpa   Deployment/backend   cpu: 65%/70%    3         10        7          6h11m
backend-hpa   Deployment/backend   cpu: 64%/70%    3         10        7          6h12m
backend-hpa   Deployment/backend   cpu: 65%/70%    3         10        7          6h12m
backend-hpa   Deployment/backend   cpu: 67%/70%    3         10        7          6h12m
backend-hpa   Deployment/backend   cpu: 68%/70%    3         10        7          6h12m
```

```bash
>>> kubectl get pods --watch
NAME                        READY   STATUS    RESTARTS       AGE
load-generator              0/1     ContainerCreating   0              0s
load-generator              1/1     Running             0              2s
backend-57f5cbb695-mjszs    0/1     Pending             0              0s
backend-57f5cbb695-mjszs    0/1     Pending             0              0s
backend-57f5cbb695-mjszs    0/1     ContainerCreating   0              0s
backend-57f5cbb695-mjszs    0/1     Running             0              2s
backend-57f5cbb695-xl6mp    0/1     Pending             0              0s
backend-57f5cbb695-xl6mp    0/1     Pending             0              0s
backend-57f5cbb695-zrzlm    0/1     Pending             0              0s
backend-57f5cbb695-zrzlm    0/1     Pending             0              0s
backend-57f5cbb695-knf9h    0/1     Pending             0              0s
backend-57f5cbb695-knf9h    0/1     Pending             0              0s
backend-57f5cbb695-xl6mp    0/1     ContainerCreating   0              0s
backend-57f5cbb695-zrzlm    0/1     ContainerCreating   0              0s
backend-57f5cbb695-knf9h    0/1     ContainerCreating   0              0s
backend-57f5cbb695-zrzlm    0/1     Running             0              1s
backend-57f5cbb695-xl6mp    0/1     Running             0              2s
backend-57f5cbb695-knf9h    0/1     Running             0              2s
backend-57f5cbb695-mjszs    1/1     Running             0              23s
backend-57f5cbb695-zrzlm    1/1     Running             0              22s
backend-57f5cbb695-xl6mp    1/1     Running             0              23s
backend-57f5cbb695-knf9h    1/1     Running             0              23s
```

## Observer le scale-down

**Observations**: Une fois que l'on a supprimé le générateur de charge, la charge cpu va brusquement diminuer jusqu'à atteindre l'état de départ (1 %).
Le hpa va alors supprimer toutes les instances qu'il avait précédemment créés afin de supporter la charge.

```bash
>>> kubectl get pods --watch
NAME                        READY   STATUS    RESTARTS       AGE
backend-hpa   Deployment/backend   cpu: 66%/70%    3         10        7          6h24m
backend-hpa   Deployment/backend   cpu: 67%/70%    3         10        7          6h24m
backend-hpa   Deployment/backend   cpu: 42%/70%    3         10        7          6h25m
backend-hpa   Deployment/backend   cpu: 1%/70%     3         10        7          6h25m
backend-hpa   Deployment/backend   cpu: 1%/70%     3         10        7          6h30m
backend-hpa   Deployment/backend   cpu: 1%/70%     3         10        5          6h30m
backend-hpa   Deployment/backend   cpu: 1%/70%     3         10        3          6h30m
```

```bash
>>> kubectl get pods --watch
NAME                        READY   STATUS    RESTARTS       AGE
backend-57f5cbb695-mjszs    1/1     Running             0              23s
backend-57f5cbb695-zrzlm    1/1     Running             0              22s
backend-57f5cbb695-xl6mp    1/1     Running             0              23s
backend-57f5cbb695-knf9h    1/1     Running             0              23s
load-generator              1/1     Terminating         0              18m
load-generator              1/1     Terminating         0              18m
load-generator              0/1     Error               0              19m
load-generator              0/1     Error               0              19m
load-generator              0/1     Error               0              19m
backend-57f5cbb695-xl6mp    1/1     Terminating         0              23m
backend-57f5cbb695-mjszs    1/1     Terminating         0              23m
backend-57f5cbb695-xl6mp    1/1     Terminating         0              23m
backend-57f5cbb695-mjszs    1/1     Terminating         0              23m
backend-57f5cbb695-zrzlm    1/1     Terminating         0              23m
backend-57f5cbb695-knf9h    1/1     Terminating         0              23m
backend-57f5cbb695-zrzlm    1/1     Terminating         0              23m
backend-57f5cbb695-mjszs    0/1     Error               0              24m
backend-57f5cbb695-xl6mp    0/1     Error               0              24m
backend-57f5cbb695-knf9h    0/1     Error               0              24m
backend-57f5cbb695-zrzlm    0/1     Error               0              24m
```