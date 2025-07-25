# Multi-sources

L'objectif de ce TP est de déployer 2 applications multi-source avec les types directory et helm.

Le stanza suivant peut être utilisé pour réaliser les exercices : 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  project: default
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  sources:
    - repoURL: TODO
      path: TODO
      targetRevision: HEAD
```

# Directory

Dans 1 repository git, créer 2 dossiers : `workload` et `exposition`.

Dans le dossier `workload`, créer le fichier `deployment.yaml` : 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      serviceAccountName: httpbin
      containers:
      - image: docker.io/mccutchen/go-httpbin:v2.15.0
        imagePullPolicy: IfNotPresent
        name: httpbin
        ports:
        - containerPort: 8080
```

et le fichier `serviceaccount.yaml` :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: httpbin
```

Dans le dossier `exposition`, créer le fichier `service.yaml` : 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
    service: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 8080
  selector:
    app: httpbin
```

et le fichier `ingress.yaml` :

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
              number: 8000
```

Créer ensuite l'application ArgoCD `multi-source-directory` qui utilise 2 sources différentes pour déployer l'ensemble des manifestes dans le namespace `httpbin`. Créer le namespace si nécessaire.

Une fois l'application déployée, supprimer la depuis l'IHM.

## Helm

Créer un repository qui contiendra le ou les fichiers values de la chart

à l'aide du stanza suivant, déployer une application multi-source helm de la chart chartmuseum : 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  sources:
  - repoURL: 'https://chartmuseum.github.io/charts'
    chart: chartmuseum
    targetRevision: "3.10.3"
    helm:
      valueFiles:
      - $values/path/to/values.yaml
  - repoURL: 'TODO'
    targetRevision: TODO
    ref: values
```

Dans le fichier values.yaml, positionner le contenu suivant : 

```yaml
commonLabels:
    foo: bar
replicaCount: 3
ingress:
    enabled: true
    hosts:
    - name: "chartmuseum.formation-argo.com"
    path: "/"
    tls: false
```

Une fois la chart déployée et fonctionnel, splitter le fichier values.yaml en 3 fichiers : `common.yaml`, `workload.yaml` et `ingress.yaml`.

Referencez les dans l'Application.

Cleanup: supprimer l'application depuis l'IHM.