# TP Projet

Créer un nouveau projet `my-project` à l'aide du stanza suivant : 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: <TODO>
  namespace: argocd
spec:
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  destinations:
  - namespace: '*'
    server: '*'
  sourceRepos:
  - '*'
```

Dans ce projet, créer une application `httpbin` qui déploie en mode `directory` un `Deployment`, un `Service`, une `Ingress` et un `ServiceAccount`. L'application sera déployée dans le namespace `httpbin` sur le cluster local.

Modifier le projet pour interdire votre repository et tentez un sync de l'application.

Re-authoriser uniquement le repository, et interdisez les objets de type `Ingress` à être créés. Syncez l'app. Analysez le résultat.

Autoriser maintenant uniquement les ressources de type `Deployment`, `Service`, `Ingress` et `ServiceAccount` à être créer dans votre projet. Syncez l'app. Que constatez-vous au niveau de l'IHM ?

Supprimez toutes les limitations vis-à-vis des objets à être déployés dans le cluster et blacklistez le namespace default.

Modifiez votre app pour déployer httpbin dans le namespace default. Constatez.

Repasser l'application dans le namespace httpbin et supprimez là depuis l'IHM.