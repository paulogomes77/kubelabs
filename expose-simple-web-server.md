# Permissa

Requisitos: Ter 2 nós: Master Node e Worker Node a funcionar em 2 VM num computador por exemplo.

A ideia é simples, criar um pod e expo-lo no host fisico, ou seja, o computador que tem as VM a correr. Neste lab, utilizou-se um Node Port Service. É possivel também expor uma app com um Load BAlancer ou um Ingress.

# Lab

#### 1. Criar o pod com um servidor web, por exemplo o NGINX.

```
$ kubectl run meunginx --image nginx
```

#### 2. Testar conectividade ao pod


```
$ kubectl get pod -o wide

NAME       READY   STATUS    RESTARTS   AGE    IP          NODE         NOMINATED NODE   READINESS GATES
meunginx   1/1     Running   0          126m   10.44.0.0   kubenode01   <none>           <none>
```

```
curl 10.44.0.0
```

#### 3. Expor o pod ao exterior

```
$ kubectl expose pod/meunginx --type="NodePort" --port 8080 --target-port=80 --dry-run=client >> meunginxservice.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: meunginx
  name: meunginx
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    run: meunginx
  type: NodePort
status:
  loadBalancer: {}

$ kubectl apply -f meunginxservice.yaml
```

#### 4. Testar conectividade ao pod dentro do cluster kubernetes

```
$ kubectl get service -o wide

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          44h   <none>
meunginx     NodePort    10.100.86.17   <none>        8080:31130/TCP   77m   run=meunginx

$ curl 10.100.86.17:8080
```



#### 5. Testar conectividade ao pod a partir do exterior, neste caso no host

```
$ ip a

3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 50:b7:c3:fd:fc:2b brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.107/24 brd 192.168.1.255

$ curl 192.168.122.234:31130
# curl (ip do host:porto criado pelo serviço - ver ponto 4.)

```

#### 6. Esquematizando de uma forma muito simplista...

```
HOST                               >  Kubernetes Service  >  Pod/Container
192.168.122.234:31130 (nodeport)   >  Porto 8080          >  Porto 80

```



