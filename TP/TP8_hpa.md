# Autoscaling avec le Horizontal Pod Autoscaler

**Remarque importante :** Pour que le hpa fonctionne, il faut définir les ressources pour le backend ET le frontend.
Dans le cas où on ne définit les ressources CPU et mémoire que dans le deployment du backend, 

```bash
>>> kubectl get hpa
NAME   REFERENCE            TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
hpa    Deployment/backend   cpu: <unknown>/70%   3         10        3          4d7h
```

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

Une fois ce problème réglé, on peut maintenant observer 
```bash
kubectl get hpa
NAME          REFERENCE            TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
backend-hpa   Deployment/backend   cpu: 1%/70%   3         10        3          54m
```
