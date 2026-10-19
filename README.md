# TP Kubernetes — Premiers déploiements avec Minikube et Docker

## Binôme

- Vincent BURGEVIN
- Lucas TEAR

## Versions utilisées

| Outil    | Version  |
|----------|----------|
| Docker   | 29.0.1   |
| kubectl  | v1.34.1  |
| Minikube | v1.39.0  |

Environnement : Windows 11 Pro, pilote Minikube `docker`.

## Contenu du rendu

```
rendu-kubernetes/
├── README.md
├── reponses.md               réponses aux questions
├── manifests/
│   ├── pod.yaml              Pod autonome whoami-pod
│   ├── deployment.yaml       Deployment whoami, 3 réplicas
│   └── service.yaml          Service ClusterIP whoami-service
├── monapp/
│   ├── index.html            page personnalisée
│   └── Dockerfile            image basée sur nginx:1.27
├── monapp-deployment.yaml    Deployment monapp, 2 réplicas
└── preuves/                  captures d'écran
```

## Principales étapes

Toutes les commandes sont lancées depuis `rendu-kubernetes/`.

1. **Cluster** : `minikube start --driver=docker`, puis vérification avec `minikube status`, `kubectl get nodes -o wide` et `kubectl get pods -A`.
2. **Namespace et Pod** : création du namespace `demo`, choisi comme namespace courant. `kubectl apply -f manifests/pod.yaml`, inspection (`describe`, `logs`, `exec ... hostname`), puis suppression : le Pod ne revient pas.
3. **Deployment** : `kubectl apply -f manifests/deployment.yaml` crée 3 Pods `whoami`. Après suppression d'un Pod, le ReplicaSet en recrée un. Test de `kubectl scale` à 5, 0 puis 3 réplicas.
4. **Service** : `kubectl apply -f manifests/service.yaml`, vérification des endpoints, puis `kubectl port-forward service/whoami-service 8080:80` et `curl http://localhost:8080`.
5. **Image personnalisée** : `docker build -t monapp:1.0 ./monapp`, `minikube image load monapp:1.0`, `kubectl apply -f monapp-deployment.yaml`, puis `kubectl port-forward deployment/monapp 8081:80` et `curl http://localhost:8081`.
6. **Nettoyage** : suppression des ressources et du namespace, retour au namespace `default`, `minikube stop`.

## Preuves

| Fichier                              | Légende |
|--------------------------------------|---------|
| `preuves/01-cluster.png`             | Cluster Minikube démarré : nœud `minikube` `Ready` et Pods système dans `kube-system`. |
| `preuves/02-deployment.png`          | Deployment `whoami` avec 3/3 réplicas, son ReplicaSet et ses Pods. |
| `preuves/03-autoreparation.png`      | Suppression d'un Pod `whoami` et recréation automatique par le ReplicaSet. |
| `preuves/04-service-http.png`        | Endpoints de `whoami-service` et réponse HTTP de `curl http://localhost:8080`. |
| `preuves/05-image-personnalisee.png` | Pods `monapp` en `Running` et page personnalisée servie sur `http://localhost:8081`. |

## Difficultés rencontrées

- Le premier `minikube start` est long, car Minikube télécharge son image de base et les images de Kubernetes.
- Sous PowerShell, `curl` est un alias de `Invoke-WebRequest` : on utilise `curl.exe http://localhost:8080` pour obtenir la sortie brute.
- `kubectl port-forward` bloque le terminal : il faut un second terminal pour lancer `curl`.
