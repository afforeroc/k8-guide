# Kubernetes Guide - Labs 0, 1, 2

## Laboratorio 0 - Obtener el servicio de Container Registry

### Instalar IBM Cloud CLI y Container Registry CLI
* Mac/Linux: 
curl -sL https://ibm.biz/idt-installer | bash

* Windows: 
https://clis.ng.bluemix.net/download/bluemix-cli/latest/win64

* Verificar versión de IBM CLI: 
ibmcloud -v

* Instalar CR CLI: 
ibmcloud plugin install container-registry -r Bluemix

## Laboratorio 1 - Configurar y desplegar tu aplicación

### Iniciar sesión en IBM Cloud y en Container Registry
* Cuenta normal de IBM Cloud: 
ibmcloud login

* Cuendate federada de IBM Cloud: 
ibmcloud login --sso

* Iniciar sesión en Container Registry
ibmcloud cr login

### Comandos utiles de docker
* Ver todas las imagenes locales de docker: 
docker images

* Eliminar una imagen docker local: 
docker image rm -f <id_del_container>

### Generar y almacenar una imagen docker en Container Registry
* Acceder a la carpeta del laboratorio a trabajar:
cd Lab 1

* Crear un cluster:
ibmcloud cs cluster-create --name <name-of-cluster>

* Agregar un nombre de espacio para las imagenes de los contenedores
ibmcloud cr namespace-add <my_namespace>

* Generar una imagen-contenedor de la aplicación
docker build --tag us.icr.io/<my_namespace>/hello-world:1 .

* Guardar una imagen-contenedor en Container Registry:
docker push us.icr.io/<my_namespace>/hello-world:1

### Verificar el cluster y el worker
* Comprobar los estados de mis clusters: 
ibmcloud cs clusters

* Ver los estados de los workers para un cluster:
ibmcloud cs workers <yourclustername>

### Desplegar la aplicación
* Obtener el comando para establecer las variables de entorno:
ibmcloud cs cluster-config <yourclustername>

* Establecer variables de entornon en Mac/Linux: 
SET KUBECONFIG=<dir\file.yml>

* Establecer variables de entornon en Mac/Linux en Windows:
$env:KUBECONFIG="<dir\file.yml>"

* Desplegar la aplicación con la imagen-contenedor:
kubectl run hello-world --image=us.icr.io/nodejs-space/hello-world:1

* Obtener información del despliegue:
kubectl get pods

### Conectarse con servicio/contenedor
* Exponer el despliegue como servicio. Escuchando por el puerto 8080:
kubectl expose deployment/hello-world --type="NodePort" --port=8080

* Encontrar el puerto usado por el nodo worker:
kubectl describe service <name-of-deployment>
-> <nodeport>

* Obtener la IP publica del cluster:
bx cs workers <name-of-cluster>
-> <public-IP>

* Acceder al servicio/contenedor:
<public-IP>:<nodeport>


## Laboratorio 2 - Escalar y actualizar apps -- servicios, conjuntos de replicas, and cheque de salud

## Escalar la aplicación con replicas
* Escalar 10 replicas del despliegue:
kubectl scale --replicas=10 deployment hello-world

* Obtener información de la replicas:
kubectl get pods

## Actualizar y devolver cambios de la aplicación

* Construir y guardar una nueva imagen de la aplicación:
docker build --tag registry.ng.bluemix.net/<my_namespace>/hello-world:2 .
docker push registry.ng.bluemix.net/<my_namespace>/hello-world:2

* Actualizar la imagen del despliegue:
kubectl set image deployment/hello-world hello-world=registry.ng.bluemix.net/<namespace>/hello-world:2

* Verificar el estado del despliegue:
kubectl rollout status deployment/hello-world

* Devolver al último cambio:
kubectl rollout undo deployment/<name-of-deployment>

## Verificar el estado de la aplicación

* Aplicar un scritp para verificar la salud de la aplicación:
kubectl apply -f healthcheck.yml

* Obtener credenciales de Kubernetes:
kubectl config view -o jsonpath='{.users[0].user.auth-provider.config.id-token}'

* Iniciar el dashboard de Kubernetes:
kubectl proxy

* Abrir: 
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

* Borrar script de salud:
kubectl delete -f healthcheck.yml

## Laboratorio 3

### Construir las imagenes de Watson
* Construir la imagen de la aplicación watson:
docker build -t registry.ng.bluemix.net/<namespace>/watson ./watson

* Enviar la imagen wtason al Container Registry:
docker push registry.ng.bluemix.net/<namespace>/watson

* Construir la imagen de la aplicación watson-talk:
docker build -t registry.ng.bluemix.net/<namespace>/watson-talk ./watson-talk

### Crear un servicio de IBM Watson con el CLI de IBM Cloud
* Verificar organización y espacio de trabajo:
bx target --cf

* Crear el servicio de tone analyzer:
bx cf create-service tone_analyzer standard tone

* Verificar la creación del servicio:
bx cf services

### Vincular el el servicio de Watson con el cluster
* Vincular el cluster con el servicio (Este comando creara un secret para el servicio):
bx cs cluster-service-bind <name-of-cluster> default tone

* Verificar el secret creado:
kubectl get secrets

### Crear los pods y servicios
* Construir la aplicación con el archivo yml:
kubectl create -f watson-deployment.yml

* Verificar que el pod fue creado:
kubectl get pods

### Probar la aplicación de watson
* Obtener información del cluster:
bx cs workers <name-of-cluster>
-> public-IP

* Probar la aplicación:
http://<public-IP>:30080/analyze/"Today is a beautiful day"

## Links de referencia
An introduction to containers: https://github.com/IBM/container-service-getting-started-wt