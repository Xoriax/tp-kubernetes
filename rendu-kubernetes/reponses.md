# Réponses — TP Kubernetes

## Étape 3 — Cluster

- **Nœud Kubernetes** : un seul nœud nommé `minikube`, avec le rôle `control-plane`. C'est un conteneur Docker (visible avec `docker ps`) qui joue à la fois le rôle de plan de contrôle et de nœud de travail.
- **Trois composants système observés** (namespace `kube-system`) :
  - `kube-apiserver-minikube` : point d'entrée de l'API, c'est lui que `kubectl` interroge ;
  - `etcd-minikube` : base clé/valeur qui stocke l'état du cluster ;
  - `kube-scheduler-minikube` : choisit le nœud sur lequel placer chaque Pod.
  - On voit aussi `kube-controller-manager-minikube`, `kube-proxy-…`, `coredns-…` et `storage-provisioner`.

## Questions de synthèse

### 1. Pod autonome vs Pod géré par un Deployment

Un Pod autonome n'a aucun contrôleur au-dessus de lui : s'il est supprimé ou s'il tombe, personne ne le recrée (on l'a vérifié avec `kubectl delete pod whoami-pod`, il ne revient pas). Un Pod géré par un Deployment est créé par un ReplicaSet à partir d'un modèle (`spec.template`). Le Deployment garantit le nombre de réplicas voulu et permet de changer l'échelle (`kubectl scale`) et de faire des mises à jour progressives (rollout).

### 2. Rôle du ReplicaSet dans l'auto-réparation

Le ReplicaSet compare en permanence le nombre de Pods qui correspondent à son sélecteur (`app: whoami`) au nombre de réplicas demandé. Quand un Pod est supprimé, il en manque un : le ReplicaSet en crée un nouveau, avec un nouveau nom, pour revenir à 3. C'est ce qu'on observe avec `kubectl get pods -w`.

### 3. Rôle du Service et vérification de la cible

Les Pods sont éphémères et leur IP change. Le Service fournit un nom DNS et une IP stables (ClusterIP) et répartit le trafic entre tous les Pods qui portent le label `app: whoami`. Pour vérifier qu'il cible les bons Pods :

- `kubectl get endpoints whoami-service` doit lister autant d'adresses IP que de Pods `Running` ;
- ces IP doivent correspondre à celles affichées par `kubectl get pods -l app=whoami -o wide` ;
- `curl http://localhost:8080` (via `port-forward`) répond, et le champ `Hostname` change selon le Pod qui répond.

### 4. Pourquoi `minikube image load` et `imagePullPolicy: Never`

L'image `monapp:1.0` est construite dans le Docker de la machine hôte. Le cluster Minikube tourne dans son propre conteneur avec son propre runtime : il ne voit pas cette image. `minikube image load` la copie dans le runtime du nœud. L'image n'existe sur aucun registre (Docker Hub, etc.), donc sans `imagePullPolicy: Never`, Kubernetes pourrait essayer de la télécharger et échouer (`ErrImagePull` / `ImagePullBackOff`). `Never` lui dit d'utiliser seulement l'image déjà présente sur le nœud.
