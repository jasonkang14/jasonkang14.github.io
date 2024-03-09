---
title: Azure Kubernetes(AKS)에 Certbot을 사용한 SSL 인증서 추가
date: "2022-09-12T19:41:37.121Z"
template: "post"
draft: false
slug: "/azure/ssl-with-certbot-for-aks"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Certbot을 사용해서 SSL 인증서를 추가한다."
---

[AKS ingress](https://jasonkang14.github.io/azure/adding-ingress-controller-to-aks)에 관해서는 이미 작성했고, 이제 SSL인증서를 추가해본다. Anycert와 같은 기관에서 인증서를 구입해도 되지만, 아직은 프로젝트 출시 전이기 때문에 굳이 돈을 쓸 필요가 없을 것 같아서 [Certbot](https://certbot.eff.org/)을 사용하도록 한다. 

Certbot은 무료 인증서인데, 내가 그 도메인을 소유하고 있다는 것만 증명할 수 있다면 인증서가 발급된다. 에러메세지를 통해서 보기에는 80 포트를 사용해서 도메인을 인증한다. 우선 certbot인증서를 위한 `letsencrypt`를 설치하고, `certbot`, `python3-certbot-nginx`를 설치한다. nginx를 사용하는 이유는 단순히 개인적으로 apache보다 nginx를 사용한 경험이 많기 때문이다.

```bash
sudo apt update
sudo apt-get install letsencrypt certbot python3-certbot-nginx -y
```

certbot 으로 인증서를 발급받을 수 있는 방법은 다양한데, 경험상 가장 쉬운 방법은 standalone이다. 서버를 잠깐 내렸다가 아래 명렁어를 실행하면 된다. 

```bash
sudo certbot --nginx -d whatever.domain.com
```

간단하게 인증서를 취득할 수 있다. 누군가는 `내 도메인이 아니어도 되는 거 아니야?` 라고 생각할 수 있는데, 80 포트를 사용해서 인증하기 때문에 A record등이 설정되지 않으면 아래와 같은 에러 메세지를 마주하게 된다. 

![certbot-domain-error](https://i.imgur.com/LecJsti.png)

그리고 본인이 소유한 도메인에서 A record를 해당 IP 주소로 잘 설정했다면, 아래와 같은 메세지를 마주하게 되고, 인증서가 제대로 생성된다. 

![certbot-success](https://i.imgur.com/frXa2HX.png)

파일들이 생성 되는 것은 맞지만, 해당 파일을 열어보면 결국 string으로 이루어져 있다. Kubernetes secret들에는 종류가 있는데, 일반적으로 환경변수로 사용되는 secret들은 [generic](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#create-a-secret)인 반면, SSL 인증서로 사용되는 secret은 [TLS](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#download-the-certificate-and-use-it)이다. 

따라서 certbot으로 생성된 파일들을 사용해서 TLS secret을 사용하고, namespace를 지정해준다.

```
kubectl create secret tls server --cert fullchain.pem --key privkey.pem
```

그리고 혹시 모르니 ingress pod를 restart해준다. 

이제 https 통신이 가능한지 curl을 통해 확인한다. 

![https-signin-sucess](https://i.imgur.com/r6nQYhL.png)

SSL 인증서가 제대로 적용된 것을 확인할 수 있다.