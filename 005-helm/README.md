# Helm

Dans ce TP, nous allons installer la chart helm chartmuseum au travers d'ArgoCD.

ChartMuseum est une application qui permet d'herbeger ses propres charts.

## Creation de l'application

Créer le namespace chartmuseum : `kubectl create ns chartmuseum`

Le repository de chartmuseum est `https://chartmuseum.github.io/charts`.

La dernière version de la chart est la `3.10.3`.

à l'aide du stanza suivant, créer une Application qui installe la chart avec les valeurs par défault : 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: chartmuseum
  namespace: argocd
spec:
  destination:
    namespace: chartmuseum
    server: https://kubernetes.default.svc
  project: default
  source:
    chart: TODO
    repoURL: TODO
    targetRevision: TODO
```

Une fois l'application créée, vérifier que les pods sont bien présents.

Lorsqu'une chart helm est installée manuellement, des secrets helm sont créés dans le namespace (ex: `helm list -n argocd`).

Listez les charts installées dans le namespace `chartmuseum`. Que constatez-vous ? Pourquoi ?

## Customisation de la chart

Les champs présents dans le stanza suivant peuvent être utilisés pour l'exercice suivant :

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  source:
    helm:
      fileParameters:
      - name: name_of_the_parameter
        path: path_of_the_file_paremeter
      parameters:
      - name: name_of_the_parameter
        value: value_of_the_parameter
      values: "helm_values"
      valuesObject:
        key: value
```

Il est possible de changer le nombre de pods du déploiement à l'aide du champ `replicaCount`

Passer en tant que **paramètre** de la chart le nombre de pods à 2 au niveau de l'application. Vérifier en regardant le nombre de pods.

Il est possible de spécifier des labels personnalisés aux ressources déployées par la chart à l'aide de la valeur `commonLabels`.

Passer en tant que **valeur** de la chart le commonLabel `foo=bar` au niveau de l'application. Vérifier la bonne application en réalisant la commande `kubectl get OBJECT --show-labels`

Passer également en tant que **valeur** de la chart, le nombre de replicas à 3. Pourquoi le nombre de replica ne change pas ?

à l'aide des différentes ressources utilisées tout au long de la formation, complétez l'application pour exposer l'application. Les valeurs de la chart peuvent être récupérées à l'URL suivante : `https://artifacthub.io/packages/helm/chartmuseum/chartmuseum` en cliquant le bouton `DEFAULT VALUES`. L'application sera accessible sur le `host` `chartmuseum.formation-argo.com` (pensez à l'ajouter l'entrée dans /etc/hosts). Le path restera `/` et le TLS sera désactivé.

Chartmuseum sera accessible sur l'URL `http://chartmuseum.formation-argo.com:8080/`.

## cleanup

supprimer l'application depuis l'IHM