# Kustomize

## Votre repository git source

Créer votre repository git `argocd-kustomize` sous github ou gitlab. Assurez-vous qu'il soit public.

Créer un repertoire base pour stocker les fichiers de base de votre application : 

```
argocd-kustomize/
└── base/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── kustomization.yaml
```

Ajouter les fichiers suivants :

base/deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpbin
  template:
    metadata:
      labels:
        app: httpbin
    spec:
      containers:
      - name: httpbin
        image: docker.io/kennethreitz/httpbin
        ports:
        - containerPort: 80
```

base/service.yaml:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpbin
spec:
  selector:
    app: httpbin
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

base/ingress.yaml:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: httpbin
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
      - pathType: ImplementationSpecific
        path: /httpbin(/|$)(.*)
        backend:
          service:
            name: httpbin
            port:
              number: 80
```

base/kustomization.yaml:

```yaml
resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
```

Ajouter maintenant un dossier `overlays/dev` : 

```
argocd-kustomize/
└── base/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── kustomization.yaml
└── overlays/
    └── dev/
        └── kustomization.yaml
        └── patch.yaml
```

overlays/dev/kustomization.yaml:

```yaml
resources:
  - ../../base
patches:
  - path: patch.yaml
    target:
      kind: Deployment

```

overlays/dev/patch.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
```

## Création de l'application ArgoCD

Inspirez vous du stanza suivant pour créer votre application et appliquer la sur le cluster :

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: httpbin-dev
  namespace: argocd
spec:
  destination:
    namespace: httpbin-dev
    server: https://kubernetes.default.svc
  project: default
  source:
    path: TODO
    repoURL: TODO
    targetRevision: HEAD
```

`kubectl apply -f application-dev.yaml`

Vérifier ensuite que votre application est bien déployée sur le cluster. Utilisez le sync si nécessaire.

## Modification du repository source

- Passer le nombre de replicas du déploiement à 1 à l'aide d'un commit
- à l'aide de la directive commonLabels de kustomize, ajouter le label `happy=newyear` aux ressources crées
- Repasser le nombre de replicas à 3, et enlever le label `happy=newyear` sans faire de commit
- Ajouter l'annotation `merry=christmas` sur l'ensemble des ressources, à l'aide de la directive commonAnnotations, mais en positionnant cette directive sur l'application et pas avec un commit. Utiliser la commande `kubectl explain application.spec.source.kustomize` pour vous aider
- De la même façon, prefixer le nom des ressources générées par `dev-`

## Ajout d'une 2nde application

- Dans le repository du déploiement, ajouter un 2nd overlay de prod
- Créer une nouvelle Application ArgoCD et prefixer les ressources par `prod-`
- Modifier le champ `spec.rules.http.paths[0].path` de l'ingress à l'aide d'un patch dans l'application pour faire en sorte que l'ingress soit admise dans le cluster.
Dans le champ patch (cherchez où le positionner), vous pouvez utiliser le stanza suivant : 

```yaml
        patch: |-
          - op: replace
            path: /spec/rules/0/http/paths/0/path
            value: /httpbin-prod(/|$)(.*)
```


## Bonus

Utiliser la directive `patchesJson6902` de kustomize pour mettre la variable d'environnement `LOG_LEVEL` à `debug` sur une des 2 applications.