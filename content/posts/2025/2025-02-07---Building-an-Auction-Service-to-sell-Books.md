---
title: "Cursor로 한시간 반만에 만든 책 경매 서비스" 
date: "2025-02-07T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/building-an-auction-service-to-sell-books-with-cursor"
category: "LLM"
tags:
  - "LLM"

description: "숫자로 보는 책 경매 서비스 회고"
---

# Cursor로 한시간 반만에 만든 책 경매 서비스

설 연휴 중에 새벽에 잠이 안와서 "책이나 좀 읽을까?" 하다가 책장에 이미 읽은 책들이 많다는 것을 발견했다. 그동안 주로 이런 책들을 알라딘에 중고서적으로 넘겼는데, 검색하니 Real MySQL 8.0이 3000원에 거래되고 있었다 
![Real MySQL 8.0 on 알라딘 중고서점](https://i.imgur.com/qAOLZBV.png)

"아니 이 좋은 책이 3천원이라고?" 라고 생각하면서 당근에 들어갔더니, 어떤분이 1권 2권을 통합해서 3만원에 판매하셨더라. 그래서 당근에 올리려다가 일일이 가격을 적고 하는게 귀찮아서 "수요가 있는 사람들이 직접 경매하게 해주는 서비스"를 만들어보기로 했다. 이유는 그냥 잠도 안오고 심심해서?

## 서비스 구조
![Service Architecture](https://i.imgur.com/2riJx3W.png)

- Flutter로 개발한 Web을 빌드해서 AWS S3로 올리고, CloudFront로 배포
- 서버는 Lambda와 API Gateway를 사용해서 개발 (API Gateway는 그림에서 누락)
- 경매 데이터는 Supabase에 저장

## 서비스 구현

모든 영역을 Cursor를 사용해서 구현했다. 어차피 1주일 뒤면 내릴 서비스이기 때문에 고민을 하고싶지 않았고, 여기에 너무 많은 시간을 투자하면 그냥 당근에 올리는게 더 효율적이라고 생각했기 때문이다. 먼저 데이터베이스 스키마를 Claude를 활용해서 만들었다. 

![Supabase Schema](https://i.imgur.com/ZUMR6kY.png)

그리고 Lambda와 연결해야하니, Supabase와 Lambda를 연결시킬 각종 파이썬 코드들을 Cursor를 이용해 만들었다. 나 혼자만 물건을 올릴 수 있고, 입찰자도 많지 않으니 사실 `users` 테이블은 필요 없는데, Claude가 작성한 코드를 그냥 넣다보니 생성됐고, 저걸 수정하는게 더 귀찮은 것 같아서 그냥 두었다. 그리고 호오오옥시나 이걸 서비스화 할 수도 있겠다는 나름의 기대감과 함께. 

![Lambda code](https://i.imgur.com/fw7mzkv.png)

람다다보니 각각의 기능별로 함수를 따로 만들어서 올렸다. 사용자 로그인을 구현하려다 시간이 너무 오래 걸릴 것 같아서, 입찰할 때 이메일을 입력하는 방식으로 구현했다 (`users` 테이블이 필요없는 이유 하나 추가)

클라이언트 사이드도 모든것을 Cursor로 구현했다. 커밋 로그에서 볼 수 있듯 총 개발 시간은 1시간 24분 18초이다 

![Commit History](https://i.imgur.com/zFtvgj7.png)

다만 Cursor를 활용하면 누구든 웹사이트를 몇시간 안에 구축할 수 있어요! 라고 말하긴 어렵다. 나는 사내에서 개발자/비개발자를 대상으로 플러터 강의를 할 정도로 플러터에 익숙하기 때문이다. 특히 초기 세팅의 경우 다양한 파일들을 생성하고, 접근하고, 수정해야해서 Cursor의 Compose기능을 활용해서 개발했는데, 로그를 보면 꽤나 구체적으로 Cursor에 요청한 것을 볼 수 있다. 

![Freezed](https://i.imgur.com/gByP1JP.png)
![Widget](https://i.imgur.com/i0u3UJj.png)

그리고 데이터베이스 스키마를 주고, 어차피 이 데이터를 받아와서 화면에서 보여주어야하니, 이에 맞게 화면을 구성하라고도 시킨다 

![database schema](https://i.imgur.com/52ClcYV.png)

스크린샷에 잘 보이진 않는데, 데이터베이스 스키마는 위에 있는 Supabase Schema 이미지를 스크린샷으로 첨부했다. 

[이런식으로 완료된 경매사이트](https://auction.byeongjinkang.link) 최대한 간단하게 만들려고 노력했고 결과는 나름 성공적이었다.

### 나름의 최적화 

처음에 이미지가 나오는데 너무 오래걸려서 이미지를 png -> webp로 전환했고, 상품 리스트에 보이는 이미지와 상품 상세화면에 보이는 이미지의 크기를 분리했다. png를 webp로 변경하는 건 온라인 무료툴들이 계속 회원가입과 결제를 요구해서 기피하다가 Claude에게 문의하니 패키지를 소개해줘서 비교적 간단하게 로컬에서 진행했다. 

```
for file in *.png; do cwebp -lossless "$file" -o "${file%.png}.webp"; done\
```

webp로 줄이면 이미지 크기가 같은 경우에도 용량이 반으로 줄어든다. 특히 리스트에 보여주는 이미지는 약 1/4 수준이기 때문에 이미지 로딩 속도를 확실하게 개선할 수 있었다. LCP측정해서 가지고 있었으면 뭔가 지표로 보여줄 수 있었을 것 같은데 살짝 아쉽다. 

![png to webp](https://i.imgur.com/WiNacti.png)

나름 CDN도 달았다. 최근 Azure 크레딧을 조금 받아서, Azure에 업로드하고 CDN을 구축했다. AWS가 익숙하긴 한데 크레딧 받은걸 빨리 써야 더주기 때문에 최대한 여기저기 활용해야한다. 사실 Lambda대신에 Azure Functions를 쓰고 싶었는데, 이걸 공부하는데 시간이 너무 오래 걸리게되면 "재미로 그냥 빠르게 한번 만들어보자" 라는 취지에 어긋나는 것 같아서 기술 스택은 그냥 친숙한 AWS를 사용했다. 차이점이 있다면, AWS에서 S3에 CDN을 활용하려면 CloudFront로 이동해서 연결시켜주고, 권한을 부여하는 작업을 해야하는데, Azure는 BlobStorage내에 CDN을 활용할 수 있는 옵션이 있어서 편리했다. 

![Azure CDN](https://i.imgur.com/0FoV6Xp.png)

클라이언트 단에서 이미지 `preCache`도 진행했다. 크게 영향은 없을 수도 있지만 조금이라도 사용자에게 편리함을 제공하고 싶었다. precache는 존재한다는 건 알고있었고, 코드 구현은 귀찮아서 Cursor를 활용했다.

```dart
  Future<void> _precacheImages(BuildContext context, List<String> imageList) async {
    for (final image in imageList) {
      if (image.startsWith('http')) {
        // Precache network images
        await precacheImage(NetworkImage(image), context);
      } else {
        // Precache asset images
        await precacheImage(AssetImage(image), context);
      }
    }
  }
```

최적화 측면에서 이것저것 하고싶은게 많았지만, 엔지니어들의 가장 큰 실수가 "아무도 사용하지 않는 것을 최적화 하는 것"이라는 [일론 머스크의 인터뷰](https://www.instagram.com/easkate/reel/CwwVkxrrPeC/)가 떠올라서 여기까지만 하고 배포하기로 했다. 

## 서비스 소개

![Book List](https://i.imgur.com/ekY243B.png)

먼저 책 리스트에서 사진과 현재 입찰 최고가를 확인할 수 있다. 최고가 입찰자는 원래 화면에 보여주진 않았는데 "내가 입찰한 금액인지 기억이 나지 않는다"는 피드백을 받아서 추가했다. 좋은 피드백을 주신 글또의 인애님께 감사를!

![Feedback](https://i.imgur.com/M14q0st.png)

그리고 입찰 화면에는 교보문고 링크로 이동할 수 있는 버튼을 추가했다. 내가 열심히 책 설명을 적는 것보다 저기에 더 자세히 나와있고, 현재 책의 가격과 비교하면 입찰하는데 도움이 될거라고 판단했기 때문이다 

![Auction Page](https://i.imgur.com/M3v2vjw.png)

## 성과

생각보다 좋은 반응이 있었다. 재미로 GA도 달았는데, 지난 5일간 전세계 8개국에서 644명이 방문했다. 

![Active users by Country](https://i.imgur.com/yKAWNQf.png)

![Active User Count](https://i.imgur.com/bHBJlCE.png)

정말 소소하지만 나름 리텐션도 있다

![User Retention](https://i.imgur.com/kpxUUHl.png)

가장 높은 가격에 입찰된 책은 [블라디미르 코리코프의 단위 테스트](https://product.kyobobook.co.kr/detail/S000001805070)이다. 글또, 퓨처스, 링크드인, 커리어리에 홍보글을 올렸는데, 아무래도 네트워크에 개발자들이 많다보니 유발하리리의 사피엔스보다 단위테스트가 더 높은 가격에 입찰되었다. 

## 회고

개인적으로는 빠르게 실행해볼 수 있어서 재밌었다. 다음에 이런 기능이 필요한 분이 있다면 연락주시면 테이블만 날리면 돼서 한명의 물건 판매는 가능하다. 어쩌면 정말 서비스가 가능할지도?!? 