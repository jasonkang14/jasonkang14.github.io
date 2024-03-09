---
title: "Styled Component vs Tailwind CSS"
date: "2023-06-28T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/styled-component-vs-tailwind-css"
category: "React"
tags:
  - "React"

description: "잘나가는 React styling 툴들의 장/단점 비교"
---

Styled Component가 다른 스타일 툴들과 무언가 차이점이 있다고 느꼈던 것은 Next.js에 Styled Component를 처음 적용했을 때입니다. 그냥 CSS를 적용하면 스타일이 바로 적용되는데, Styled Component는 잠깐 반짝임 후에 스타일이 적용되는 것을 경험했습니다. 저는 당시 스타일이 바로 적용될 수 있도록 `_document.tsx`에 별도의 설정을 추가로 해줬어야 해습니다. 이제는 [공식문서](https://nextjs.org/docs/app/building-your-application/styling/css-in-js)에서 추가설정을 어떻게 해줘야 하는지를 설명하고 있네요. 

위에서 언급한 "반짝임" 은 공식적으로 [Flash of Unstyled Content(FUOC)](https://en.wikipedia.org/wiki/Flash_of_unstyled_content)라고 불리는 문제입니다. 웹페이지가 외부에서 작성된 CSS Stylesheet를 불러오기 전에, 아주 잠깐동안 브라우저의 디폴트 스타일을 잠깐 보여주는 것입니다. 

일반적인 브라우저 렌더링 과정에 대해 생각해보면, 

1. 브라우저가 HTML을 파싱해서 DOM Tree를 생성하고 
2. CSS를 프로세싱 해서 CSS Object Model(CSSOM) Tree를 생성하고
3. DOM과 CSSOM Tree를 결합해서 Render Tree를 생성하고
4. 화면을 그리기 시작합니다. 

즉 자바스크립트를 실행하기 전에 스타일이 화면에 적용되는 것입니다. 하지만 CSS-in-JS라는 이름에서 유추할 수 있는 것처럼, styled component는 스타일링을 자바스크립트가 실행되는 런타임에 적용하기 때문에 자바스크립트가 실행되기 전까지 사용자는 스타일이 적용되지 않은 HTML 만으로 이루어진 DOM Tree를 보게 되는 것입니다. 

그럼 이제 자바스크립트가 실행되는 시점과 사용자가 화면을 보는 시점을 Client-Side Rendering(CSR)과 Server-Side Rendering(SSR)을 비교해서 알아보겠습니다. 

리액트 프로젝트의 디폴트인 Client-Side Rendering(CSR)에서는 자바스크립트 파일이 브라우저에서 실행됩니다. 따라서 HTML이 로딩 후와 자바스크립트 실행 전 아주 잠깐동안 HTML content에 스타일이 적용되지 않은 상태로 보일 수 있습니다. 하지만 Client Side Rendering에서 이 시간은 매우 짧기 때문에, 자바스크립트를 파싱하고 실행하는데 어마어마한 시간이 소요되는 경우가 아니라면 사용자가 위에서 언급한 "반짝임"을 경험할 가능성은 매우 낮습니다. 

하지만 Server-Side Rendering(SSR)에서는 조금 다릅니다. 서버에서 브라우저로 전송하는 HTML파일에는 리액트 컴포넌트의 최초 렌더를 포함하고 있습니다. 만약 styled-component가 적용하는 스타일이 서버의 response에 포함되어있지 않는다면, 스타일이 바로 적용되지 않을 수 있습니다. HTML은 브라우저에서 열리자마자 내용이 보이고, 그 이후에 자바스크립트를 로딩하고, 파싱하고, 실행시키기 때문입니다. CSS-in-JS를 사용하는 경우에는 자바스크립트가 실행되는 런타임에 스타일이 적용되기 때문에, 자바스크립트가 실행되기 전에는 사용자가 스타일이 적용되지 않은 화면을 봐야하는 것입니다. 

그리고 Client-Side Rendering에서 styled-component를 적용하는 경우에도, 스타일이 자바스크립트를 실행할 때 적용되기 때문에 Static CSS를 로딩하는 것보다 오래걸릴 수 있고, 자바스크립트 파일 자체가 커지면서 번들 사이즈가 커지는 문제가 발생할 수도 있습니다.

이런 문제가 있긴 하지만 styled-component도 많은 장점이 있습니다. 가장 좋은 점은 `scope`인데요, styled-component가 자체적으로 각각의 컴포넌트에 고유한 className을 부여하기 때문에, 여러 파일들에 `Button`, `Input` 등과 같이 같은 이름을 적용했다고 하더라도, style이 충돌하지 않고 정상적으로 적용되는 것입니다. 

그리고 styled component를 사용하면 dynamic styling도 적용할 수 있습니다. 

```typescript
const Button = styled.button`
  color: ${props => props.primary ? 'blue' : 'white'};
`;

```

styled component에서 제공하는 theme을 사용해서 속성을 나눌 수도 있고, React Component처럼 prop을 넘겨줄 수도 있습니다.

```typescript
const Button = styled.button<{isActive: boolean}>`
  background-color: ${props => props.isActive ? 'white' : 'grey'};
`;
```

그리고 미리 custom으로 작성해둔 css를 사용해서 스타일을 적용할 수도 있습니다. 

```typescript
const ActiveButton = css``
const InActiveButton = css``

const Button = styled.button<{isActive: boolean}>`
  ${props => props.isActive ? ActiveButton : InActiveButton};
`;
```

TailwindCSS는 빌드타임에서 스타일을 생성하기 때문에 위에서 언급한 runtime 관련 문제가 없습니다. 추가로 PurgeCSS를 활용하면, 사용하지 않는 style들을 프로덕션 빌드에서 제거하기 때문에, 최종적으로 생성되는 css파일이 매우 작아진다는 이점이 있습니다. 

또한 기본적으로 제공하는 utility class들을 사용하면 별도의 css를 따로 선언하지 않고 바로 스타일을 적용할 수 있습니다. 추가로 커스텀도 가능한데요, 

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    colors: {
      primary: "#1d2745",
      secondary: "#1de5d4",
      tertiary: "#f52c50",
      white: "#ffffff",
      mono1: '#f1f1f1',
      mono2: '#bebebe',
      error: '#d01e1e',
    },
    extend: {
      backgroundImage: {
        'dropdown-arrow': "url('/icons/ic-dropdown.svg')"
      },
      backgroundSize: {
        'dropdown-arrow': '1.5rem'
      },
    },
  },
  plugins: [],
}
```

theme을 사용해서 color와 같은 디폴트 속성들을 미리 선언해둘 수 있고, 아이콘과 관련된 스타일도 별도로 설정할 수 있습니다. 추가로 styled component의 dynamic styling과 유사하게 활용할 수 있습니다. 

```typescript
const active = 'text-primary bg-white';
const inActive = 'text-white';

export default function Category({ children, isActive, handleCategoryClick }: CategoryProps): ReactElement {
  return (
    <button
      className={`rounded-3xl border border-white p-2.5 ${isActive ? active : inActive}`}
      type="button"
      value={children.id}
      onClick={handleCategoryClick}
    >
      {children.name}
    </button>
  );
}
```

Tailwind CSS는 미리 선언된 class들을 사용해서 스타일을 적용할 수 있다는 장점이 있지만, 위에서 보신 것처럼 className에 추가되는 사항이 많아지면 HTML이 커진다는 단점이 있습니다. 그리고 저처럼 디폴트로 선언된 스타일 클래스를 잘 기억하지 못하는 분들은 [공식문서](https://tailwindcss.com/docs/flex-basis)에서 숫자와 클래스를 매칭해야하는 문제가 늘 존재합니다.

글을 마치면서 두 툴을 비교한 테이블을 전달드립니다 


|     || Styled Components | Tailwind CSS |
|-----|--|-------------------|--------------|
|**접근 방식** | | CSS-in-JS: JavaScript로 작성된 스타일이 컴포넌트에 범위 지정됩니다. | 유틸리티 우선 CSS: 유틸리티 클래스를 사용해 마크업에 직접 스타일이 적용됩니다. |
|**커스터마이징 가능성** | | 매우 높음: 각 컴포넌트는 프롭스에 기반한 독특한 스타일을 가질 수 있습니다. | 매우 높음: Tailwind의 기본 설정은 프로젝트 요구 사항에 맞게 쉽게 구성할 수 있습니다. |
|**성능** | | 런타임에서 CSS를 생성하기 때문에 성능 비용이 발생할 수 있습니다. 하지만 작은 앱에서는 종종 눈에 띄지 않으며, 좋은 관행을 통해 완화할 수 있습니다. | 일반적으로 성능이 더 좋습니다. 모든 클래스는 빌드 시간에 생성됩니다. 또한 PurgeCSS는 프로덕션 빌드에서 사용되지 않는 스타일을 제거합니다. |
|**동적 스타일링** | | 동적 스타일링에 맞춤. 스타일은 프롭스 또는 전역 테마에 기반해 쉽게 변경할 수 있습니다. | 동적 스타일링을 달성하려면 더 많은 작업이 필요합니다. 주로 상태에 따라 클래스를 전환하기 위해 JavaScript와 함께 사용됩니다. |
|**학습 곡선** | | 보통: CSS와 JavaScript에 익숙하다면, styled-components를 상대적으로 쉽게 습득할 수 있습니다. | 가파름: 유틸리티 우선 접근법은 전통적인 CSS와 다르며, 이해하고 익숙해지는 데 시간이 걸립니다. |
|**테마** | | 통합 테마 지원이 잘되어 있어, styled-components의 전체 접근법과 잘 통합됩니다. | 테마 설정을 통한 지원이 있으며, 테마 값에 따라 유틸리티가 생성됩니다. |
|**도구** | | Webpack 또는 Parcel과 같은 CSS-in-JS를 처리할 수 있는 빌드 도구가 필요합니다. | Webpack, Parcel, Gulp 등 PostCSS를 처리할 수 있는 빌드 도구가 필요합니다. |
|**서버 사이드 렌더링** | | SSR에 대한 내장 지원이 있지만, 스타일이 없는 콘텐츠의 깜박임을 방지하기 위해 추가 설정이 필요합니다. | SSR에 대해 추가 설정이 필요하지 않습니다. |
