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

Pour se faire, allez dans le fichier /etc/nginx/conf.d/default.conf en vous connectant dans le pod via ```oc exec -it nginx -- /bin/bash```
Récupérez le nom du fichier (/usr/share/nginx/html/index.html)

Modifiez le via :
```echo "texte" > /usr/share/nginx/html/index.html```

Créez un fichier configmap index.html :
https://kubernetes.io/docs/concepts/configuration/configmap/

Suivant la documentation, montez le fichier index.html de la cm dans le pod nginx. 

Vérifiez que cette modification fonctionne. 

Pour plus simplement créer la configmap, vous pouvez utiliser la commande :

``` oc create configmap maconfigmap --from-file=index.html ```

### Correction

oc create configmap --from-file=canard.png --from-file=index.html canard-website --dry-run=client -o yaml > cm.yaml

oc create -f .\cm.yaml

oc apply -f podcm.yaml

## Demo 3 : Mettre en place des variables d'environnement avec une Config map 

L'appel de la variable d'environnement se fait depuis le pod :
```
        - name: demoenv
          valueFrom:
            configMapKeyRef:
              name: env-demo          # The ConfigMap this value comes from.
              key: demo
```
Ce qui laisse entendre que nous avons une configmap nommée env-demo et on récupère la valeur uniquement de la clé demo. 

## Exercice 5 : Mise en place d'un secret comme variable d'environnement :
https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables

Créez un secret qui va contenir un mot de passe pour une base de donnée 

Créez un pod avec harbor.kakor.ovh/public/mariadb:latest qui va monter ce secret dans la bonne variable d'environnement. 

Vérifiez que votre pod est bien en running et que le mot de passe fonctionne en vous connectant sur le pod et sur la db via la commande mariadb. 

### Correction 

Création du pod :
oc run mariabd --image=harbor.kakor.ovh/public/mariadb:latest --dry-run=client -o yaml > exo5/pod.yaml

Création du secret :

oc create secret generic mariadb-secret --from-literal=password=B4teau123!  --dry-ru
n=client -o yaml > exo5/secret.yaml

oc logs mariadb 

oc rsh mariadb 

mariadb -u root -p

## Exercice 6 

Nettoyez l'existant

Doc Kubernetes:
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Mettez en place un nouveau deployment, celui-ci doit se nommer mysql. Ce deployment doit posséder 1 replicas uniquement et utiliser l'image harbor.kakor.ovh/public/mysql:8.4.5 . Montez sur ce deployment un secret nommé "mysql-password" qui permettra de stocker le mot de passe de la base de donné du deployment via la variable d'environnement MYSQL_ROOT_PASSWORD

Une fois que le pod créé est en Running, modifiez le deployment pour utiliser la version harbor.kakor.ovh/public/mysql:9.3.

Une fois en Runnning mettez l'image harbor.kakor.ovh/public/mysql:15.8 . Puis revenez à la version précédente du deployment sans modifier le fichier yaml.

Tentez de supprimer les pods du deployment, que se passe t-il et pourquoi ? 

### Correction 

oc create deployment mysql-coorection --image=harbor.kakor.ovh/public/mysql:8.4.5 --replicas=1 --dry-run=client -o yaml

```
oc apply -f deploiment.yaml
oc apply -f secret.yaml 

autre option 

oc apply -f exo6/ 

oc set image deployment/mysql-coorection mysql=harbor.kakor.ovh/public/mysql:9.3
oc get pod
oc get rs
oc set image deployment/mysql-coorection mysql=harbor.kakor.ovh/public/mysql:15.6
oc get pod
oc rollout undo deployment mysql-coorection
oc get pod
oc delete pod mysql-coorection-5c759bbdc7-dvs44
oc get pod
oc delete deployment mysql-coorection
```