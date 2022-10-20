---
layout: post
current: post
cover: assets/images/azure-face-api/face_api.jpeg
navigation: True
title: Azure Face API 사용기
date: 2022-08-27 15:00:00
tags:
class: post-template
subclass: 'post'
author: Sue
---

## Gymboxx 안면 출입 서비스
<br>

안녕하세요. 서플라이스 Tech팀 백엔드 개발자 피수연입니다.

저희 헬스장 Gymboxx의 출입서버에서는 앱을 통한 바코드 인식과 안면인식을 통한 출입 서비스를 운영하고 있습니다.

짐박스 얼굴 출입 서비스는 앱을 켤 필요 없이 오직 안면 인식만으로 출입을 가능하게 하고있습니다. 따라서 회원들이 따로 휴대폰을 들고 다니지 않아도 편하게 헬스장을 출입할 수 있으며 회원들의 무단 출입을 막을 수도 있습니다.

얼굴 출입을 하기 위해서 먼저 회원의 얼굴을 사진을 찍어 등록합니다. 등록할 때, 회원의 얼굴 이미지에서 얼굴 특징을 감지하고 분석하여 데이터를 만든 뒤, 앞으로 회원이 출석 할 때 마다 저장된 회원 얼굴 데이터와 비교 후 동일한 얼굴로 인식이 되어야만 출입을 허가하고 있습니다.

이미지에서 얼굴을 감지하고 분석하기 위한 도구로 시중에 많은 Face Recognition API가 있습니다. 그 중에서 저희는 Microsoft Azure Cognitive services의 Face API를 사용하였습니다.

Azure Face API는 이미지에서 얼굴을 감지하고 분석하는 API입니다. 안면 인식을 통해 성별, 나이, 얼굴 표정을 통한 감정까지 인식할 수 있습니다. 또한 마스크 착용 시에도 얼굴을 감지할 수 있습니다. 

Azure Face API에 대한 설명은 간단하게 하고 사용기를 적어보겠습니다.

더 자세한 설명은 아래 링크에서 확인할 수 있습니다.

[Face API: 이미지에서 얼굴을 분석하는 AI 서비스](https://azure.microsoft.com/ko-kr/services/cognitive-services/face/#overview)
<br>
## Azure Face API 사용기
<br>
Azure Face API의 간단한 사용기만 언급할 것이므로 자세한 사용방법은 아래 링크에서 확인할 수 있습니다.

[빠른 시작: Face 서비스 사용](https://docs.microsoft.com/ko-kr/azure/cognitive-services/computer-vision/quickstarts-sdk/identity-client-library?pivots=programming-language-javascript&tabs=visual-studio)

먼저 얼굴을 등록합니다. 얼굴을 등록할때, Azure 클라우드 내 person group이란 것을 만들고 그 안에 감지된 얼굴을 등록하게 됩니다. 인식된 얼굴은 person group내 person으로 저장되고 고유한 personId를 부여받게 됩니다. 그다음 azure가 부여받은 personId를 person group내에서 얼굴을 식별할 수 있도록 학습시킵니다. 학습은 비동기적으로 진행됩니다.

테스트를 위해 아래 사진으로 person을 등록해 보겠습니다.

![등록 사진](assets/images/azure-face-api/ariana.jpeg)

얼굴을 등록하면 다음과 같은 personId를 부여받은 것을 확인할 수 있습니다.

```jsx
{ personId: '8baca216-cc92-4dca-af37-50f74586c5e1' }
```

azure에 등록된 person list에서도 확인해 볼 수도 있습니다.

```jsx
[
  {
    "name": "Ariana Grande",
    "personId": "8baca216-cc92-4dca-af37-50f74586c5e1",
    "persistedFaceIds": [
      "ecb30b20-faa7-4796-9fb2-6b0d352ea7fc"
    ]
  }
]
```

여기서 persistedFaceIds란 등록된 얼굴을 말하는데,  방금 하나를 등록했기 때문에 하나만 있는 것을 볼 수 있습니다. 같은 person에 대해서 얼굴을 추가적으로 등록할 수도 있습니다.

이제 얼굴을 추가했으니 얼굴 출입을 해보겠습니다. 얼굴출입은 출입할때 얼굴을 분석하여 faceId를 검출하고 Azure에 저장된 person group내에 해당 faceId와 일치하는 person을 식별하면 출입을 허용하도록 하였습니다.

동일인물의 다른사진으로 얼굴 출입을 해보겠습니다.

![출입 사진1](assets/images/azure-face-api/ari2.png)

얼굴을 분석했을때 다음과 같은 faceId를 얻을 수 있었습니다.

```jsx
[
  {
    faceId: '170970ee-db23-40fb-b7ef-78d4d7fe4614',
    recognitionModel: 'recognition_01',
    faceRectangle: { width: 179, height: 237, left: 48, top: 51 }
  }
]
```

그리고 해당 faceId를 person group내에 person인지 indentify를 해보았을 경우

```jsx
{
  faceId: '170970ee-db23-40fb-b7ef-78d4d7fe4614',
  candidates: [
    {
      personId: '8baca216-cc92-4dca-af37-50f74586c5e1',
      confidence: 0.93872
    }
  ]
}
```

얼굴 등록했을때 받은 personId와 0.93의 신뢰도로 동일한 인물임을 식별할 수 있었습니다. 이때 confidence는 해당 얼굴이 등록된 personId와 얼마나 비슷한지를 의미합니다.

마스크를 착용한 얼굴도 동일인물로 식별할 수 있을지 테스트를 해보았습니다.

![출입 사진2](assets/images/azure-face-api/ariana_mask2.png)

```jsx
{
  faceId: 'f8d10b65-6157-4cbd-b6ce-5e188df6a753',
  candidates: [
    {
      personId: '8baca216-cc92-4dca-af37-50f74586c5e1',
      confidence: 0.74237
    }
  ]
}
```

화장을 하지않고, 마크스를 착용한 얼굴임에도 0.74의 신뢰도로 동일인물로 식별하는 것을 확인할 수 있습니다.
<br>
### 다른 Face API는 어떨까?
<br>
Face recogition API으로 유명한 Luxand의 Face API를 사용해보았습니다. Luxand는 Azure와 마찬가지로 다양한 언어를 지원하고 AWS를 기반한 클라우드 서비스입니다. Luxand에서는 Dashboard를 제공해주어서 직접 코드를 작성하지 않아도 쉽게 테스트 해볼 수 있었습니다.
 
[Luxand](https://luxand.cloud/)

방법은 Azure에서와 같았습니다. 먼저 person을 등록하고 person에 face를 등록하는 방식입니다.

Azure Face api와의 신뢰도 비교를 위해 동일 사진으로 테스트를 해보았습니다.

같은 사진으로 person을 등록해보겠습니다.

![등록 사진](assets/images/azure-face-api/ariana.jpeg)

```jsx
{
  "status": "success",
  "uuid": "18eb51e9-26a2-11ed-b979-0242ac120002",
  "id": 292784,
  "message": "Successfully created",
  "url": "https://faces.nyc3.digitaloceanspaces.com/1904d48b-26a2-11ed-b979-0242ac120002.jpg"
}
```

person을 등록하고 나면 성공적으로 id를 받게 됩니다. Azure Face api와 마찬가지로 해당 id에 추가로 얼굴을 등록할 수 있습니다. 

이제 얼굴 인식을 해보겠습니다.

![출입 사진1](assets/images/azure-face-api/ari2.png)

```jsx
[
  {
    "id": 292784,
    "name": "Ariana Grande",
    "probability": 0.9995379447937012,
    "rectangle": {
      "left": 48,
      "top": 56,
      "right": 392,
      "bottom": 400
    }
  }
]
```

0.99의 신뢰도로 동일인물임을 식별하고 있습니다. Azure Face api보다 높은 신뢰도를 받았습니다.

![출입 사진2](assets/images/azure-face-api/ariana_mask2.png)

```jsx
[
  {
    "id": 292784,
    "name": "Ariana Grande",
    "probability": 0.9433196783065796,
    "rectangle": {
      "left": 97,
      "top": 87,
      "right": 287,
      "bottom": 277
    }
  }
]
```

해당 사진은 0.94의 신뢰도를 받았습니다. 

Luxand가 Face API만 전문으로 하는 기업이라 그런지 전체적인 신뢰도는 Azure보다 높았습니다.

<br>
## Azure Face API를 선택하게 된 이유
<br>

- 합리적인 가격

성능도 중요하지만 가격은 서비스를 선택하는데 아주 중요한 요소 중 하나입니다. Azure Cognitive Service의 무료서비스는 월별 30,000개의 트랜잭션(API 호출)을 무료로 제공하고 있고 표준 서비스는 1000개의 트랜잭션당 1달러로 아주 합리적인 금액 입니다. Face Storage도 매월 1,000개당 $0.01로 저렴한 편입니다.

Luxand의 가격 책정은 500개의 API호출까지만 무료로 제공하고 있습니다. Basic플랜은 한달에 $19에 10,000개의 API호출까지만 가능하고 500개의 얼굴만 저장할 수 있습니다. 헬스장에 다니는 회원들의 얼굴을 다 저장해야 한다고 가정하면 아마 가격이 배로 뛸 것 입니다. 

Azure Cognitive Service의 가격책정에 대한 자세한 정보는 아래 링크에서 확인할 수 있습니다.

[https://azure.microsoft.com/ko-kr/pricing/details/cognitive-services/face-api/](https://azure.microsoft.com/ko-kr/pricing/details/cognitive-services/face-api/)

- 좋은 성능

얼굴 인식 속도를 테스트 해보았을 때 API콜 당 700-1200ms내외로 빠릅니다. 저장된 얼굴의 개수에 따라 달라지겠지만 사용하면서 인식이 느리다고 느낀적은 없습니다. 그리고 평균 0.8-0.9 내의 신뢰도로 인식도 정확합니다.  

- 안정적인 클라우드 플랫폼

요즘 국내 유명한 기업들에서도 Face Recognition 서비스들을 출시하고 있지만, Azure Face API는 Microsoft의 제품이므로 더 신뢰할 수 있습니다. 

<br>
## 글을 마치며..
<br>
서플라이스에 입사해서 다양한 것들을 해볼 수 있어서 좋았는데 그 중 하나가 Face API를 사용해본 것입니다. 비록 간단하게 사용해보았지만 평소엔 접해보지 못 한 것들을 해보면서 재미있었습니다.  

서플라이스는 피트니스 시장에서 빠르게 성장하고 있는 스타트업입니다. 함께 경험을 쌓고 성장하실 분은 언제든 아래 공고에 지원 부탁드립니다.

• [채용 공고 보러가기](https://www.notion.so/17d545a40be3463784f3f4861a923c64)