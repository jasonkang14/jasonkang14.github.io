---
title: Context API -> React Query
date: "2022-07-12T23:21:37.121Z"
template: "post"
draft: false
slug: "/react/introducing-react-query"
category: "React"
tags:
  - "React"

description: "Context API를 들어내고 React Query를 사용해본다"
---

# TL;DR

### React Query의 사용으로 store가 가벼워졌고, request 에러 핸들링이 간편해졌다.

회사에 매우 간단한 과제가 있어서 빠르게 만들어서 제공하기 위해 [Context API](https://reactjs.org/docs/context.html)를 사용해서 빠르게 배포했다. 규모가 워낙 작아서 효율보다는 속도에 집중해서 작업했다. 하지만, 사업을 확장하게 되면서 간단한 기능만 가지고 있던 프로젝트에 코드를 추가하게 되었다. 자연스럽게 점점 더 `Context`가 추가되었고, `Context`의 단점을 강하게 겪을 수 있었다. 

`Context`의 가장 큰 단점은, 해당 context 내 store(정확히는 해당 context의 reducer의 state)의 값이 바뀌게 되면 그 context를 사용한다면, 지금 component에서 사용하지 않는 값이 변경되도 re-render가 일어난다는 것이다. 따라서 rendering효율을 고려하면 Context를 엄청 잘게 쪼개야한다. 

함수형 프로그래밍을 적용하면서, 자식 component에서 부모 component의 state를 바꾸지 않기 위해서 많은 변수들을 전역변수로(context로)옮겼고, [렌더링 효율이 개선되기는 했지만](https://jasonkang14.github.io/react/functional-programming-with-react-part-three)그래도 아직 기술 부채가 남아있다. 그래서 이참에 `Context API`를 들어내기로 했다. 

다른 프로젝트에 적용되어 있는 [redux](https://redux.js.org/)를 써볼까 하다가, 이왕 하는거 새로운 걸 해보자는 팀원들의 의견에 따라 각자 기술검토를 시작했다. 우리가 가진 선택지는 [Recoil](https://recoiljs.org/), [constate](https://github.com/diegohaz/constate), 그리고 [MobX](https://mobx.js.org/README.html)였다. 

MobX는 [예전에 한 번 써봤는데](https://jasonkang14.github.io/posts/Request-to-Server-using-MobX) 보급형 리덕스 느낌이라서 내가 하지 말자고 제안했다. 누군가는 납득하지 못 할 이유일 수 있지만, constate는 깃헙 레포의 fork와 star 수가 빈약하다. 그렇다면 남은 것은 `recoil`이었다. 메타에서 만든거니 리액트랑 잘 어울릴 것이라고 판단했다. 그리고 recoil을 사용하는 김에 요즘 우아한형제들과 카카오에서 쓴다는 [react-query](https://react-query-v2.tanstack.com/)를 사용해보자는 의견이 있어서 같이 도입해보기로 했다. 

단순히 코드만 볼 때 `react-query`의 장점은, fetch 성공 실패 관리가 쉽다는 것이다. 기존에 `redux-saga`로 작업했을 때는, `reducer`에 값을 넘겨줘야 했기 때문에 각각의 request마다 success와 fail state를 사용했다(나중에 에러는 axios interceptor를 사용해서 모두 통합하긴 했지만). `react-query`를 사용하면 success와 error핸들링이 쉽다는 장점이 있을 것 같았다. 

위에서 언급한 프로젝트에서 로그인을 예로 들면, `Context API`의 경우 login이 provider에서 이루어지기 때문에, 자연스럽게 login 결과가 reducer의 state에 담기게 되고, login component에서만 사용되어도 되는 로그인 성공 여부에 관한 값이 전역변수로 관리되게 된다. 지금 프로젝트는 서버에서 받아오는 사용자의 permission에 따라 접근할 수 있는 화면들이 달라진다. 따라서 유저 타입에 따라 전역변수로 관리할 변수가 비례해서 증가하게 된다. 하지만 `react-query`의 경우 request를 component 내부에서 하기 때문에, store가 가벼워질 수 있는 장점이 있다. 그리고 자동으로 retry를 해주기 때문에, 간혹 예상치 못한 문제로 request가 실패했을 때에도 에러핸들링에 용이할 것 같았다. 예제를 통해 살펴본다 

```typescript
// Login.tsx

export default function Login() {
	const navigate = useNavigate();
	const {
		requestLogin,
		userTypeA,
		userTypeB,
		userTypeC,
		userTypeD,
	} = useAuth(); 
		// custom hook의 syntax를 위해 useContext(AuthContext)를 useAuth()로 export한다.

	const handleLoginBtnClick = () => {
		requestLogin({username, password}) 
		// context의 provider에서 request가 이루어지고, 값은 reducer에 저장된다.
	}

	useEffect(() => {
		if (!userTypeA) return;
		navigate('/routeA');
	}, [userTypeA])

	useEffect(() => {
		if (!userTypeB) return;
		navigate('/routeB');
	}, [userTypeB])

	useEffect(() => {
		if (!userTypeC) return;
		navigate('/routeC');
	}, [userTypeC])

	useEffect(() => {
		if (!userTypeD) return;
		navigate('/routeD');
	}, [userTypeD])
}
```

예제코드를 `react-query`로 바꾸면 아래와 같이 사용 가능하다 

```typescript
// Login.tsx

export default function Login() {
	const navigate = useNavigate();

	const handleLoginBtnClick = () => {
		const { mutate, isLoading } = useMutation(requestLogin, {
			onSuccess: (data) => {
				if (data.userTypeA) {
					navigate('/routeA');
				}
				if (data.userTypeB) {
					navigate('/routeB');
				}
				if (data.userTypeA) {
					navigate('/routeC');
				}
				if (data.userTypeA) {
					navigate('/routeD');
				}
			},
			onError: () => {
				alert('login failed');
			},
		});
	}
}
```

`react-query`를 사용하면서 다양한 장점을 경험할 수 있는데, 
1. context로 관리되는 변수가 줄어들었기 때문에, AuthContext를 사용하는 다른 component에서의 불필요한 rendering을 막을 수 있다. 
2. loading 상태 관리가 수월하다. 기존에 context라면 loading state가 바뀔 때 다른 component에서 불필요한 렌더링이 일어났을 것이다.
3. retry수를 default options에 설정하면 알 수 없는 이유로 로그인이 실패할 때, 사용자가 재시도하지 않아도 request를 다시 전송하기 때문에 UX 측면에서도 유리하다고 생각했다. 

그렇다면 진짜 효율적인지 [Profiler](https://reactjs.org/docs/profiler.html)를 통해 확인해보도록 한다.

|![context-api](https://i.imgur.com/UXdARpb.png)|
| :-------------------------------------------: |
|                 context-api                   |

|![react-query](https://i.imgur.com/zCAjc69.png)|
| :-------------------------------------------: |
|                 react-query                   |

렌더링 시간이 약 **1.7ms** 줄어들고 비율로 계산하면 약 **63%** 가량 개선된다.

`react-query`를 사용할 이유는 충분히 증명된 것 같고, [다음 포스트](https://jasonkang14.github.io/react/introducing-recoil)에서 recoil의 타당성을 검토해보기로 한다