# Projet Final AJC DevOps - Groupe 4

# III-1 : Déploiement des différentes applications dans un cluster Kubernetes.




## **Prérequis :**
1) Avoir un cluster minikube fonctionnel

2) Installer Longhorn (gestionnaire de volumes persistants) sur minikube :
```sh
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.3.1/deploy/longhorn.yaml
```
3) Activer l'addon MetalLB sur le minikube (et donner à MetalLB une plage d'adresses sur le même réseau que la VM) :
```sh
minikube addons enable metallb
minikube addons configure metallb
```

Tout est prêt pour lancer le déploiement :
```sh
kubectl apply -f .
```
Pour simuler la résolution DNS, faire un :
```sh
kubectl get svc -n icgroup
```
Et rajouter les External IP des trois services dans le fichier de résolution de noms de la machine du browser (pour windows : C:\Windows\System32\drivers\etc\hosts)

Exemple :
```sh
192.168.56.100 ic-webapp.ajc
192.168.56.101 odoo.ajc
192.168.56.102 pgadmin.ajc
```
### Accès au portail ic-webapp sur le browser : http://ic-webapp.ajc

## **Infos de connection aux différentes applis** :
* Odoo : admin / admin
* PGadmin : admin@admin.com / pgadm-password