---
title: Virtual Machine vs Container
date: "2022-06-27T01:26:37.121Z"
template: "post"
draft: false
slug: "/cloud/virtual-machine-vs-container"
category: "Cloud"
tags:
  - "Cloud"

description: "Microservice Architecture(MSA) 도입 전 컨테이너와 Virtual Machine의 차이에 대해 공부한다"
---

# TL;DR

### application들의 OS가 다양한 경우 VM을 사용해야 하지만 application들이 같은 OS에서 구동된다면, Container를 사용하는 것이 더 가볍고 빠르다.

회사에서 운영하는 서비스가 커지면서 Microservice Architecture(MSA)를 도입하고자 한다. Azure를 사용중이라서 Azure에 있는 기능들을 최대한 사용하고 싶은데, 아직 감이 잘 오지 않는다.

지금은 VM 하나에서 [docker-compose](https://docs.docker.com/compose/)를 사용해서 Django, MySQL, Redis, Celery등을 컨테이너로 올려서 관리중인데, 컨테이너 하나에 문제가 발생하면 다른 곳에서 제 기능을 못하는 경우도 있고. Django app하나에서 문제가 발생하면 다른 Django app이 기능을 잘 못하기도 한다. 그나마 다행인 점은 django app들을 잘게 쪼개둔 상태인데, 이 app들을 각각의 microservice로 분리하고자 한다.

적용하기 전에 Azure Container와 Virtual Machine 중 어떤걸 선택할지 결정하기 위해 조금 공부해보려 한다.

| ![container vs vm](https://i.imgur.com/GBnBKbW.png) |
| :-------------------------------------------------: |
|            virtual machine vs container             |

위 그림을 비교할 때 가장 큰 차이는 Operating System이다. Virtual Machine의 경우 infrastructure 위에 hypervisor가 있다. Hypervisor는 VM을 생성하고 구동하는 소프트웨어이다. processor, RAM, storage, network와 관련된 모든 것들이 Hypervisor를 통해 가상화된다. 하나의 컴퓨터 안에서 여러개의 VM을 구동할 수 있게 하는 것이다. 그리고 이 여러개의 VM들은 각각 Guest OS를 사용하고, hypervisor를 통해 리소스들을 공유할 수 있게 된다. 각각 Guest OS를 사용하기 때문에 메모리나 스토리지에 오버헤드가 있다.

반면 Container는 infrastructure 위에 hypervirsor없이 Host OS를 가진다--Kernel이 중간에 있는데 그림에서 빠졌다 Kernel은 하드웨어와 소프트웨어를 연결시킨다. 단계가 하나 빠지니 구동이 더 빠르고, Host OS를 사용하기 때문에 container안에서 돌고있는 application들은 모두 같은 OS환경에서 구동된다. 따라서 VM과 비교했을 때 상대적으로 오버헤드가 없다. 컨테이너 안에 여러개의 application들이 구동된다고 할 때, 각 application들에게 공유되는 리소스들은 read-only이기 때문에 VM보다 가볍고 빠르다.

Azure에서 MySQL과 Redis를 따로 제공하고, celery로 이루어지는 태스크들을 django에서 이전시키면 지금 구동된 여러 docker container들이 쓸모없어진다. 처음에는 이미 사용하고 있는 VM을 API Gateway로 사용하고, 각각의 서버들로 redirect하려고 했지만, Azure에서 많은 기능들을 제공하고 있으니 Azure Container를 사용해보려고 한다.

참고자료

1. https://www.atlassian.com/microservices/cloud-computing/containers-vs-vms
2. https://www.ibm.com/cloud/blog/containers-vs-vms
3. https://www.vmware.com/topics/glossary/content/vms-vs-containers.html
