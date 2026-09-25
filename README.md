# Домашнее задание к занятию «Как работает сеть в K8s»
## Задание 1
 - Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.
 - В качестве образа использовать network-multitool.
 - Разместить поды в namespace App.
 - Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
 - Продемонстрировать, что трафик разрешён и запрещён.

## Решение
### Создание стенда
<img width="967" height="301" alt="image" src="https://github.com/user-attachments/assets/f2f0923a-96fb-48ef-88da-fcef58c32c4b" />



### Создание inventory
```console
all:
  hosts:
    node1:
      ansible_host: "{{ ip_node1_host_host }}"
      ip: "{{ ip_node1 }}"
      access_ip: "{{ ip_node1_access }}"
    node2:
      ansible_host: "{{ ip_node2_host_host }}"
      ip: "{{ ip_node2 }}"
      access_ip: "{{ ip_node2_access }}"
    node3:
      ansible_host: "{{ ip_node3_host_host }}"
      ip: "{{ ip_node3 }}"
      access_ip: "{{ ip_node3_access }}"
    node4:
      ansible_host: "{{ ip_node4_host_host }}"
      ip: "{{ ip_node4 }}"
      access_ip: "{{ ip_node4_access }}"
  children:
    kube_control_plane:
      hosts:
        node1:
    kube_node:
      hosts:
        node2:
        node3:
        node4:
    etcd:
      hosts:
        node1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

### Применение конфигурации Ansible для узлов кластера и создание kubeconfig-файл для пользователя admin:
```console
root@kube-master-01:~/kubespray# ansible-playbook -i inventory/mycluster/hosts.yaml -u admin -b -v --private-key=/root/.ssh/id_rsa cluster.yml
------ ВЫВОД ------
TASK [network_plugin/calico : Check if inventory match current cluster configuration] *******************************************************************************************************************************************************
ok: [node1] => {
    "changed": false,
    "msg": "All assertions passed"
}
Tuesday 18 June 2024 10:15:23 +0000 (0:00:00.131)       0:19:43.797 ********
Tuesday 18 June 2024 10:15:23 +0000 (0:00:00.060)       0:19:43.858 ********
Tuesday 18 June 2024 10:15:23 +0000 (0:00:00.045)       0:19:43.903 ********

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node1                      : ok=754  changed=150  unreachable=0    failed=0    skipped=1280 rescued=0    ignored=8
node2                      : ok=514  changed=94   unreachable=0    failed=0    skipped=780  rescued=0    ignored=1
node3                      : ok=514  changed=94   unreachable=0    failed=0    skipped=779  rescued=0    ignored=1
node4                      : ok=514  changed=94   unreachable=0    failed=0    skipped=779  rescued=0    ignored=1
```


### Проверка нод
```console
admin@kube-master-01:~$ kubectl get nodes
NAME    STATUS   ROLES           AGE     VERSION
node1   Ready    control-plane   9m27s   v1.28.2
node2   Ready    <none>          8m34s   v1.28.2
node3   Ready    <none>          8m29s   v1.28.2
node4   Ready    <none>          8m29s   v1.28.2
```

### Проверка подов
```console
admin@kube-master-01:~$ kubectl get po -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5fb8ccdcd6-kmxjj   1/1     Running   0          14m
kube-system   calico-node-5qcv4                          1/1     Running   0          16m
kube-system   calico-node-7949w                          1/1     Running   0          16m
kube-system   calico-node-9msxv                          1/1     Running   0          16m
kube-system   calico-node-dgs7f                          1/1     Running   0          16m
kube-system   coredns-67cb94d654-bkv4s                   1/1     Running   0          14m
kube-system   coredns-67cb94d654-jkc25                   1/1     Running   0          14m
kube-system   dns-autoscaler-7b6c6d8b5b-2j9jg            1/1     Running   0          14m
kube-system   kube-apiserver-node1                       1/1     Running   1          17m
kube-system   kube-controller-manager-node1              1/1     Running   2          17m
kube-system   kube-proxy-hrgfk                           1/1     Running   0          16m
kube-system   kube-proxy-mgdwl                           1/1     Running   0          16m
kube-system   kube-proxy-q256j                           1/1     Running   0          16m
kube-system   kube-proxy-vcfr7                           1/1     Running   0          16m
kube-system   kube-scheduler-node1                       1/1     Running   1          17m
kube-system   nginx-proxy-node2                          1/1     Running   0          16m
kube-system   nginx-proxy-node3                          1/1     Running   0          16m
kube-system   nginx-proxy-node4                          1/1     Running   0          16m
kube-system   nodelocaldns-22lvx                         1/1     Running   0          14m
kube-system   nodelocaldns-82xrs                         1/1     Running   0          14m
kube-system   nodelocaldns-8wqtz                         1/1     Running   0          14m
kube-system   nodelocaldns-s6w8x                         1/1     Running   0          14m
```
### Развертывание Service + Deployment
```console
NAME                                      READY   STATUS    RESTARTS   AGE     IP             NODE    NOMINATED NODE   READINESS GATES
pod/deployment-backend-59f8c49bdb-zqcwb   1/1     Running   0          5m33s   10.233.75.3    node2   <none>           <none>
pod/deployment-cache-69b486bc58-8x8db     1/1     Running   0          2m13s   10.233.71.2    node3   <none>           <none>
pod/deployment-frontend-8cc5b69b4-lrxn4   1/1     Running   0          9m31s   10.233.74.66   node4   <none>           <none>

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/service-backend    ClusterIP   10.233.9.28     <none>        80/TCP    5m25s   app=backend
service/service-cache      ClusterIP   10.233.32.40    <none>        80/TCP    2m6s    app=cache
service/service-frontend   ClusterIP   10.233.17.252   <none>        80/TCP    9m20s   app=frontend
```
### Создание сетевого взаимодействия

#### Между frontend и backend
https://github.com/Arseny-Kornilov/Kubernetes_8_Kornilov/blob/main/network_front_back.yaml
