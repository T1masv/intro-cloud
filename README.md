# Giga-README pour le code de déploiement Kubernetes

Ce README fournit une vue d'ensemble détaillée de la structure et de la fonction de votre code de déploiement Kubernetes. Le code utilise un mélange d'Ansible et de Helm pour automatiser le déploiement de plusieurs applications et services sur un cluster Kubernetes (spécifiquement Minikube dans ce cas).

## Structure du projet

Le projet est divisé en deux répertoires principaux :

* `ansible`: Contient les playbooks et les rôles Ansible pour provisionner l'infrastructure et déployer les applications.
* `k8s`: Contient les charts Helm pour déployer les applications sur Kubernetes.

### Répertoire `ansible`

Le répertoire `ansible` contient le playbook principal (`playbook.yaml`) et un ensemble de rôles Ansible :

* `playbook.yaml`: Ce playbook est le point d'entrée pour le déploiement. Il s'exécute localement et appelle les différents rôles pour configurer Minikube, Traefik, cert-manager, MySQL et les applications.
* `roles/`: Ce répertoire contient les rôles Ansible, chacun responsable d'une partie spécifique du déploiement :
    * `roles/minikube`: Ce rôle assure que Minikube est installé et en cours d'exécution.
    * `roles/traefik`: Ce rôle déploie Traefik, un reverse proxy et ingress controller, sur Kubernetes en utilisant Helm. Il applique également des middlewares Traefik pour manipuler les requêtes (strip-prefix).
    * `roles/cert-manager`: Ce rôle déploie cert-manager, un outil pour provisionner automatiquement les certificats TLS, sur Kubernetes en utilisant Helm. Il applique également des ClusterIssuers pour configurer les fournisseurs de certificats (Let's Encrypt) pour les environnements de production et de staging[cite: 2, 3].
    * `roles/mysql`: Ce rôle déploie une base de données MySQL sur Kubernetes en utilisant Helm.
    * `roles/applications`: Ce rôle déploie les applications (`subscription-service` et `user-service`) sur Kubernetes en utilisant Helm.

### Répertoire `k8s`

Le répertoire `k8s` contient les charts Helm et les configurations spécifiques à Kubernetes :

* `k8s/helm/`: Ce répertoire contient les charts Helm pour les différents composants :
    * `k8s/helm/cert-manager`: Contient les fichiers Helmfile et les valeurs pour déployer cert-manager. Il inclut également les configurations des ClusterIssuers pour Let's Encrypt (prod et staging)[cite: 2, 3].
    * `k8s/helm/mysql`: Contient les fichiers Helmfile pour déployer MySQL.
    * `k8s/helm/traefik`: Contient les fichiers Helmfile, les valeurs et les middlewares pour déployer Traefik.
    * `k8s/helm/subscription`: Contient le chart Helm pour l'application `subscription-service`[cite: 5, 6]. Il inclut les templates pour le déploiement, le service, le Horizontal Pod Autoscaler (HPA), les notes d'utilisation, le service account et l'ingress.
    * `k8s/helm/user`: Contient le chart Helm pour l'application `user-service`. Il a une structure similaire au chart `subscription-service`.

## Workflow de déploiement

Le workflow de déploiement est orchestré par Ansible. Voici les étapes principales :

1.  **Provisionnement de Minikube**: Le rôle `minikube` assure que Minikube est en cours d'exécution.
2.  **Déploiement de Traefik**: Le rôle `traefik` déploie Traefik et configure les middlewares. Traefik servira d'ingress controller pour exposer les applications.
3.  **Déploiement de cert-manager**: Le rôle `cert-manager` déploie cert-manager et configure les ClusterIssuers pour provisionner les certificats TLS.
4.  **Déploiement de MySQL**: Le rôle `mysql` déploie la base de données MySQL.
5.  **Déploiement des applications**: Le rôle `applications` déploie les applications `subscription-service` et `user-service`.

## Composants clés et leurs fonctions

* **Minikube**: Un outil qui permet d'exécuter Kubernetes localement, utile pour le développement et les tests.
* **Helm**: Un gestionnaire de paquets pour Kubernetes, qui simplifie le déploiement et la gestion des applications.
* **Helmfile**: Un outil pour déclarer, configurer et appliquer des déploiements Helm.
* **Traefik**: Un ingress controller qui gère l'accès externe aux services Kubernetes. Il fournit également des fonctionnalités de reverse proxy et d'équilibrage de charge.
* **cert-manager**: Un outil qui automatise la gestion des certificats TLS dans Kubernetes. Il permet de provisionner et de renouveler automatiquement les certificats à partir de sources comme Let's Encrypt.
* **Let's Encrypt**: Une autorité de certification qui fournit des certificats TLS gratuits[cite: 2, 3].
* **Ansible**: Un outil d'automatisation qui permet de provisionner l'infrastructure et de déployer les applications de manière déclarative.
* **ClusterIssuer**: Une ressource Kubernetes (cert-manager) qui représente une autorité de certification et est utilisée pour émettre des certificats[cite: 2, 3].
* **Ingress**: Une ressource Kubernetes qui expose les services HTTP et HTTPS à l'extérieur du cluster.

## Applications déployées

* **subscription-service**: Une application déployée via Helm[cite: 5, 6]. Son chart Helm inclut des ressources pour le déploiement, le service, le HPA, etc.
* **user-service**: Une autre application déployée via Helm. Elle a une structure de chart Helm similaire à `subscription-service`.

## Fichiers de configuration importants

* `ansible/playbook.yaml`: Le playbook Ansible principal.
* `ansible/roles/*/tasks/main.yaml`: Les fichiers de tâches principaux pour chaque rôle Ansible.
* `k8s/helm/*/helmfile.yaml`: Les fichiers Helmfile pour chaque composant.
* `k8s/helm/*/values.yaml`: Les fichiers de valeurs par défaut pour les charts Helm.
* `k8s/helm/*/templates/*.yaml`: Les templates Kubernetes utilisés par les charts Helm.
* `k8s/helm/cert-manager/cluster-issuers/*.yaml`: Les configurations des ClusterIssuers pour cert-manager[cite: 2, 3].
* `k8s/helm/traefik/middlewares/*.yaml`: Les configurations des middlewares Traefik.

## Utilisation

Pour utiliser ce code, vous aurez besoin des outils suivants :

* Ansible
* Minikube
* Helm
* Helmfile
* kubectl

Assurez-vous que ces outils sont installés et configurés correctement.

Pour déployer l'infrastructure et les applications, exécutez le playbook Ansible principal :

```bash
ansible-playbook ansible/playbook.yaml