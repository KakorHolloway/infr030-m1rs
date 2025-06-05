# infr030-m1rs

## Connection au cluster Openshift
Pour vous connecter au cluster Openshift mis à disposition : 

### Pour Windows

Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-03-10-010116/openshift-client-windows-4.15.0-0.okd-2024-03-10-010116.zip

Copiez le fichier oc.exe dans C:\Windows\System32

### Pour Linux 
Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-03-10-010116/openshift-client-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz

Dézippez et copiez le fichier oc dans /urs/bin
```
tar -xvf openshift-client-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz
cp oc /usr/bin/
```

### Pour tous les OS

Allez sur l'url https://console-openshift-console.apps.openshift.kakor.ovh authentifiez-vous avec l'utilisateur ipi-gp-x (le x étant le numéro de groupe) en choisisant "KeystoneIDP".

Une fois authentifié (attention à ne pas recharger la page même si l'affichage prends du temps), allez en haut à droite de la page et sélectionnez l'option "Copy login Command" et réauthentifiez vous. 

Cliquez sur le lien Display Token et copiez dans votre terminal sur VScode la ligne de commande qui à été donné avec oc. 

Pour vérifier que vous êtes authentifiés, lancez la commande ```oc get pod```

## Exercice 1

Dans votre namespace, créez un pod avec l'image docker pull ``` harbor.kakor.ovh/public/httpd:latest ``` en vous basant sur la documentation suivante :
https://kubernetes.io/docs/concepts/workloads/pods/

Avec la commande oc get pod, vérifiez que votre fonctionne bien. 

Ajoutez la section suivante dans la section container (au même niveau que image par exemple):
```
securityContext:
  allowPrivilegeEscalation: true
```

### Correction

```
oc apply -f exo1/podexo1.yaml
```

Vous pouvez vous connecter dans le pod avec :

```
oc exec -it httpd -- /bin/bash
ou 
oc rsh httpd
```

Lancez les commandes :
```
apt-get update && apt-get install curl 
curl localhost
```

## Demo 1 : les services

```
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    demo: httpd
spec:
  containers:
  - name: httpd
    image: harbor.kakor.ovh/public/httpd:latest
    securityContext:
      allowPrivilegeEscalation: true
    ports:
    - containerPort: 80
```

Service :

```
apiVersion: v1
kind: Service
metadata:
  name: httpd
spec:
  selector:
    demo: httpd
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
## Exercice 2

Supprimer les pods des exercices d'avant, déployez un nouveau pod nginx avec l'image ```harbor.kakor.ovh/public/nginx:latest```.

Créez un service associé à ce pod, en vous connectant dans le pod avec la commande ``` oc exec -it <monpod> -- /bin/bash ``` testez que le service fonctionne bien. 

### Correction

oc apply -f exo2/podservice.yaml

oc rsh nginx (équivalent à oc exec -it...) 

curl localhost # résultat du pod 
curl nginx # résultat du service

## Demo 2 Création d'un Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
    - host: nginx-correction.apps.openshift.kakor.ovh
      http:
        paths:
        - path: ''
          pathType: ImplementationSpecific
          backend:
            service:
              name: nginx
              port:
                number: 80
  tls:
  - {}
```

## Exercice 3

A partir des éléments de l'exercice 2, créez l'ingress vous permettant d'accéder depuis l'extérieur à votre pod nginx. 

Attention à ce que le nom de l'host soit singulier dans le cluster Openshift. 

(ex: monnginx-001.apps.openshift.kakor.ovh )

## Exercice 4 

Praparation : Sans rien toucher de l'exo 3, modifiez la page web hébergée par votre conteneur. 