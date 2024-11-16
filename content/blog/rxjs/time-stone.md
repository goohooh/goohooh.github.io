---
title: "RxJS Strange: 대혼돈의 User Interface"
date: 2024-09-01 12:00:00
category: 'rxjs'
draft: false
---

> "타임스톤"은 시간을 자유자재로 다룰 수 있는 무척 강력한 도구지만, 잘못 사용할 경우 영원히 반복되는 시간에 갇히거나 존재가 소멸할 수 있습니다.

RxJS가 "타임 스톤"이라면, Observable-Hooks는 이를 사용하기 위한 "아가모토의 눈"이라 할 수 있습니다. 

![rxjs](./images/rxjs-logo.png)

## (1)그래, 많이 들어봤는데... 왜 써?

RxJS와 함께라면 그 어떤 UI 요구사항도 만족시킬 수 있습니다. UI가 복잡해 질수록 개발자가 관리해야 하는 상태는 늘어나고, 그럴 수록 버그의 가능성도 늘어납니다. 그리고 개발자는 아래와 같이 답하게 됩니다.

"너무 복잡해서 어려울 것 같아요. 만들더라도 시간이 많이 필요해요"

하지만 개발자가 힘들어질수록 유저는 편해지기 마련입니다. 유저에게 많은 UI 피드백을 제공할수록 개발 난이도는 올라가지만, 제품의 매력은 상승하기 마련입니다. 그러니 안 쓸 이유가 없지 않을까요?

단, 제약조건이 있습니다. 
- 학습 난이도가 높다.
- 조심해서 사용해야한다.

부작용을 잘 생각해야하지만, boundary를 잘 설정한다면 문제는 최소화 할 수 있습니다. 그리고 익숙해질수록 더 큰 범위의, 더 어려운 UI를 구현할 수 있습니다. RxJS를 한마디로 설명한다면 아래와 같이 표현할 수 있겠네요.

> "상상의 경계를 무너뜨릴 광기"
> <div style="text-align: right; font-style: italic;">닥터 스트레인지: 대혼돈의 멀티버스 中</div>

## (2)그래, 많이 들어봤는데... React랑 안맞지 않아?

RxJS하면 Angular를 떠올리는 경우가 많습니다. 아무래도 프레임워크 자체에서 rxjs를 지원하기 때문인데요. 그래서인지 React 진영에서는 그만큼 관심이 크지 않은 것 같습니다.(제가 느끼기엔 그렇습니다)

본격적으로 rxjs를 익히고 react에 적용하려 했을 때, 막막함을 느끼곤 했습니다. 각각의 메커니즘은 이해를 했는데, 이를 함께 사용하기가 까다로웠습니다. 동작한다 하더라도, `useEffect` 로 누더기 코드가 되곤했죠. 이는 react의 데이터는 **위에서 아래로** 흐르는 단방향성에서 기인됐는데요. React의 개발 철학이기 때문에 근본적으로 RxJS와 충돌할 수 밖에 없었습니다. 

- React :arrow_right:  **위에서 아래로 흐르는** 데이터를 UI로 표현
- RxJS :arrow_right: **위/아래/앞/뒤로 흐르는** 데이터를 핸들링

태생부터 맞지 않는 라이브러리를 동시에 사용하려니 불편함을 느낄 수밖에 없었죠. 그렇지만 각각 오래된 라이브러리기도 했고, 분명 누군가 이러한 불편함을 해결했으리라 생각했습니다. 다행히 몇몇 후보들이 있었고 그 중에서도 [observable-hooks](https://observable-hooks.js.org/)는 모든 불편함을 해소해주었습니다.
- :star: Observable-Hooks :arrow_right: rxjs 데이터가 hook을 통해 react 컴포넌트로 제공됨

# Observable-Hooks!

![time-stone](./images/agamoto-eye.gif)

> "타임스톤"은 무척 강력한 도구지만, 왠만한 실력자가 아닌 이상 쉽게 다룰 수 없습니다. "아가모토의 눈"은 이를 효율적으로 제어하기 위한 도구입니다.

RxJS가 "타임 스톤"이라면, Observable-Hooks는 이를 사용하기 위한 "아가모토의 눈"이라 할 수 있습니다. 

![observable-hooks](./images/observable-hooks-logo.jpg)

## 왜 써?

여러분은 어떤 이유로 프론트엔드 개발을 하시나요? 여러 이유 중에서도 저는 UI의 어려운 부분을 해결했을 때 큰 성취를 느낍니다. 복잡하고 까다로운 유저의 요구사항을 명쾌하게 풀어냈을 때 쾌감은 이루 말할 수 없습니다. DOM Event, User Event, State Management, Network API 호출과 비지니스 로직을 한꺼번에 다룸과 동시에 유저가 만족을 느끼는 Seamless하고도 Elegant한 UI 렌더링을 함께 달성하기란 무척 어렵습니다. 

생각하기만 해도 복잡한 이런 난관 속에서 RxJS가 빛을 발합니다. 제 경험상 일견 불가능해 보였던 요구사항도 RxJS를 사용해 손쉽게 해결한 경우도 많았습니다.

![reveal-time-stone](./images/reveal-time-stone.gif)

- DOM Event
- User Event
- Network API

위 3가지 모두 비동기 Action이며 프론트 개발자가 통제하기 어려운 영역입니다. 이 3가지 액션의 조합으로 특정 조건에 따라 UI를 렌더링 한다고 상상해볼까요? 대부분 머리가 지끈거리게 될겁니다. 물론 어찌어찌 동작하는 코드를 작성할 수 있겠습니다마는, 대체로 가독성이 무척 떨어지기 마련이죠. 비동기 코드를 다루기란 대체로 그렇습니다. 전통적인 [Pulling 방식의 코딩](https://rxjs-dev.firebaseapp.com/guide/observable#pull-versus-push)으로는, 비동기 액션들의 타이밍을 종잡기 쉽지 않거든요.


# 언제 써?

RxJS는 이러한 '타이밍'을 컨트롤하기 쉽게 도와줍니다. '타임스톤'에 비유를 든 것은 이때문이죠.

![thread-of-time](./images/Ancient-One-Timeline-Infinity-Stones-Banner.avif)

React

간단한 예시부터 살펴볼까요?

```js
const useGenerateZipFile = () => {
  return useMutation({
    mutationFn: (values) => fetch('/api/zip', {
      method: 'POST',
      body: JSON.stringify(values),
    })
  });
};

function fetchZipDownloadByRequestId(requestId: string) {
  return fetch(`/api/zip/${requestId}`);
}
```

```jsx
const MyComponent = () => {
  const methods = useForm();
  const { mutateAsync } = useGenerateZipFile();
  cons [requestId, setRequestId] = useState(null);

  const { data, isError, error } = useQuery(
    ['pollDownloadUrl', requestId],
    () => fetchZipDownloadAsyncByRequestId(requestId),
    {
      enabled: Boolean(requestId), // requestId가 존재하고 폴링이 활성화된 경우에만 실행
      refetchInterval: (data) => {
        if (data.downloadUrl) return undefined;
        return 5000; // 5초마다 폴링
      },
    }
  );

  useEffect(() => {
    if (data?.downloadUrl) {
      downloadFile(data.downloadUrl);
      setRequestId(null);
      toast.dismiss();
    }
  }, [data]);

  useEffect(() => {
    if (isError && error) {
      toast.error(error.message);
    }
  }, [isError, error]);

  const onSubmit = async (values) => {
    try {
      toast('Generating and compressing files');
      const response = await mutateAsync(values);
      
      // Zip 파일 생성이 성공하면 requestId를 상태로 저장하고 폴링 활성화
      setRequestId(response.requestId);
    } catch (err) {
      toast.error(err.message);
    }
  };

  return (
    <form
      onSubmit={(e) => {
        void methods.handleSubmit(onSubmit)(e);
      }}
    >
      {/* form elements */}
    </form>
  );
};
```


```jsx
const MyComponent = () => {
  const methods = useForm();
  const { mutateAsync } = useGenerateZipFile();
  const [onGenerateResponse, onPollingResponse$] = useObservableCallback((generateResponse$) =>
    generateResponse$.pipe(
      tap(() => {
        toast('Generating and compressing files');
      }),
      switchMap((response) => {
        return interval(1000 * 5).pipe(
          switchMap(() => fetchZipDownloadAsyncByRequestId(response.requestId)),
          catchError((error) => {
            toast.error(error.message);
            return EMPTY;
          }),
          takeWhile((res) => {
            return !isString(res.downloadUrl);
          }, true),
        );
      }),
    ),
  );

  useSubscription(onPollingResponse$, ({ download: { downloadUrl } }) => {
    if (downloadUrl) {
      downloadFile(downloadUrl);
      toast.dismiss();
    }
  });

  const onSubmit = async (values) => {
    await mutateAsync(values)
      .then(onGenerateResponse)
      .catch((err) => toast.error(err.message));
  }

  return (
    <form
      onSubmit={(e) => {
        void methods.handleSubmit(onSubmit)(e);
      }}
    >
    {/* ... */}
    </form>
  );
}
```

요구사항을 바꿔보겠습니다. 1초 마다 

하지만 막상 익히려 하면 생각 이상으로 [어렵다는 것](https://x.com/hoss/status/742643506536153088)이 선뜻 추천하기 어려운 이유이기도 합니다.

1) 개발자에게 추가적인 개발 패러다임을 익힐 것을 요구하고
2) 수 많은 operator를 익혀야 하며
3) 자칫 디버깅이 매우 어려워 질 수 있습니다.

# 조금씩 적용 개선해보자

보통 rxjs의 수많은 operator에 압도되곤 합니다. 모든 operator를 익히기보다, Observable의 메커니즘에 집중하여, 적절한 사용처를 찾는 것을 추천드립니다.

- 타이머, hotkey
- Provider: Boundary 설정 및 리렌더링 없는 역상태전파
- debug(): https://www.youtube.com/watch?v=ZxVN4597RX8

# 마무리

RxJS를 왜 써야하는지 고민하던 시기가 있었습니다. 커리어에 도움이 될 것 같으면서도 한편으로 크게 와닿지 않아서, 간단히 공부하고 말았습니다. 하지만 지금은 무척 즐겨 사용하는 도구 중 하나입니다. 복잡한 UI를 다루면서 효용성을 많이 깨닫고 끊임없이 공부(수련)중입니다. 



![training](./images/training.gif)

 닥터 스트레인지 조차 이 타임스톤을 자유자재로 다루기 위해 힘든 역경을 지나왔습니다. 그리고 마침내 최강의 소서러 슈프림으로 등극합니다.

RxJS 또한 익히기 위한 시간이 필요하지만, 익숙해질 경우 여러분들을 만능 프론트엔드 개발자로 발돋움 하도록 도와줄 것임을 장담합니다.
