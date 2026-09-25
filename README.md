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
```
all:
  hosts:
    node1:
      ansible_host: 192.168.0.3
      ip: 192.168.0.3
      access_ip: 192.168.0.3
    node2:
      ansible_host: 192.168.0.24
      ip: 192.168.0.24
      access_ip: 192.168.0.24
    node3:
      ansible_host: 192.168.0.42
      ip: 192.168.0.42
      access_ip: 192.168.0.42
    node4:
      ansible_host: 192.168.0.14
      ip: 192.168.0.14
      access_ip: 192.168.0.14
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
