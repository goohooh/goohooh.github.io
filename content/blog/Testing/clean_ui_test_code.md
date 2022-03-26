---
title: MSW를 활용한 Clean UI Test Code
date: 2022-03-26 12:30:00
category: 'Testing'
draft: false
---

UI 통합테스트를 작성하다보면 몇가지 문제점을 마주칩니다. (Jest, React, Typescript 환경 예시)

```tsx
import { mocked } from 'ts-jest/utils';
import { getPostsHook } from './hooks'

jest.mock('./hooks');
const mockedGetPostsHook = mocked(getPostsHook);

it('유저는 자신이 작성한 Post의 리스트를 볼 수 있다', () => {
  mockedGetPostsHook.mockReturnValueOnce(Promise.revolve([
    { title: 'a', ... },
    { title: 'b', ... },
  ]));

  render(Component)

  const items = await screen.findByRole('listitem');

  expect(items.length).toBe(2);
});
```

단벌의 테스트 케이스라면 위 모킹은 문제가 되지 않습니다. 하지만

- 테스트 케이스가 늘어날 경우
- `getPostsHook`을 여러곳에서 사용할 경우
- 모킹할 함수가 늘어나거나, 모킹할 데이터 구조가 복잡해질 경우

다음과 같은 문제가 발생합니다.

1. 모킹 코드때문에 테스트 코드 자체의 가독성이 떨어진다. *단 몇 줄의 테스트코드를 위해 몇십 줄이 넘는 Mocking코드가 하나의 파일*에 필요하다.
2. 데이터 스키마가 바뀔 경우 관리하기 어렵다

> Jest에서 제공하는 Manual Mock을 사용하면 해결되지 않나?

맞습니다. 위 문제들을 *완화*시킬 수 있습니다.

```ts
// __mocks__/hooks.js
const POSTS = [{ title: 'a' }, { title: 'b' }]

export const getPostsHook = () => {
  return Promise.resolve(POSTS)
}
```

테스트코드는 이렇게 변합니다.

```tsx
jest.mock('./hooks')

it('유저는 자신이 작성한 Post의 리스트를 볼 수 있다', () => {
  render(Component)

  const items = await screen.findByRole('listitem')

  expect(items.length).toBe(2)
})
```

:tada: 깔끔해졌군요! 이제 끝일까요?

## 파편화

A라는 유저는 2개의 Post를 작성했다고 가정해보겠습니다. 반면 B라는 유저가 0개의 Post를 작성하였고, 디자인팀은 이런 경우 다른 UI가 그려지길 원합니다.

2개의 다른 케이스를 커버하기 위해 2벌의 테스트를 작성해야합니다. 아까 모킹했던 함수`getPostsHook`는 이에 대응할 수 있을까요?

단순히 생각하면 `__mocks__/hooks.ts` 의 코드를 수정하면 됩니다. 인자를 넘겨 다른 값을 리턴하도록 말이죠. 옳은 방법일까요?

함수는 2개로 파편화 됐습니다. 개발자가 관여하지 않는 이상, 테스트용 함수는 원본 함수의 변경을 감지하지 못합니다. 실제 함수에 버그가 생겨도 테스트용 함수는 제 몫을 하지 못합니다.

## MSW와 MSWJS/data

[MSW](https://mswjs.io/)는 최근들어 주목받는 라이브러리입니다.

> Mock by intercepting requests on the network level. Seamlessly reuse the same mock definition for testing, development, and debugging.

네트워크 호출을 가로채서 seamless한 모킹 경험을 제공하는 것이 핵심입니다. 그리고 [@mswjs/data](https://github.com/mswjs/data) 라는 모듈을 추가적으로 제공합니다.

> Model and query your mock data (fixtures).

간단히 이야기하자면, MSW Ecosystem으로 우리만의 서버를 만들 수 있게됐습니다! 테스트 환경에서든, 백엔드가 준비되지 않은 개발 환경에서든 말이죠.(후자의 경우 [Mocking으로 생산성까지 챙기는 FE 개발 – tech.kakao.com](https://tech.kakao.com/2021/09/29/mocking-fe/) 에서 자세하게 살펴보실 수 있습니다)

아래는 테스트환경의 간단한 예시 코드입니다. 최대한 책임을 분리하여 작성했습니다.

```js
// schema
import { factory, primaryKey } from '@mswjs/data'

export const schema = {
  post: {
    id: primaryKey(String),
    title: String,
  },
}

// seed
export const runMigration1 = db => {
  db.post.create({ id: '1', title: 'Hello' })
  db.post.create({ id: '2', title: 'Bye' })
}

// db
import { factory } from '@mswjs/data'
import { migration1 } from './seed'
import schema from './schema'

const db = factory(schema)
runMigration1(db)

export default db

// server
import { rest } from 'msw'
import { setupServer } from 'msw/node'
import db from './db'

// setupServer의 인자로 들어간 handler도 파일을 분리할 수 있지만
// 편의를 위해 한곳에 작성했습니다.
export const server = setupServer(
  rest.get('/posts', (req, res, ctx) => {
    const posts = db.post.getAll()
    return res(ctx.json(posts))
  })
)
```

간단하게 만드려 했지만, 사실 꽤나 품이 많이 듭니다.

```js
/* setupTest */
import { server } from './server'

// 테스트 전에 모든 요청을 intercept하는 layer 생성
beforeAll(() => server.listen())

// 요청 interception layer를 제거하여 side effect 차단
afterAll(() => server.close())

/* 테스트 파일 */
test('Post 데이터를 렌더링 한다', () => {
  render(Component)

  const items = await screen.findByRole('listitem');

  expect(items.length).toBe(2);
})
```

이제 더이상 테스트 코드 안에 Mocking 코드는 존재하지 않습니다. 이로써

- 가독성 확보
- 코드 파편화 방지(원본 코드를 그대로 사용)
- 그리고 Fixture의 형태, 필드 타입, 관계 정의 및 관리 용이성 확보
  가 가능해졌습니다.(아직 설명 하지 않았지만 mswjs/data로 데이터간 relation도 정의할 수 있습니다)

## 실전 예시

조금더 실전적인 시나리오를 작성해 봅시다. User와 Post관계를 정립하고, Post를 작성한 경우, 작성하지 않은 경우 2벌의 테스트를 작성해야하고 유저의 세션관리도 필요합니다.

```js
// schema
import { manyOf, nullable, oneOf, primaryKey } from '@mswjs/data';

export const schema = {
  user: {
    id: primaryKey(String),
    name: String,
    posts: manyOf('post')
  },
  post: {
    id: primaryKey(String),
    title: String,
  },
}

// seed
export const runMigration = () => {
  const posts = [
    db.post.create({ id: '1', title: 'Hello' })
    db.post.create({ id: '2', title: 'Bye' })
  ]

  db.user.create({ id: '1', name: 'has-posts', posts })
  db.user.create({ id: '2', name: 'no-posts', posts: [] })
}

// session
// 프로젝트 별로 세션관리는 상이하며, 이에따라 구현은 달라질 수 있습니다
import crypto from 'crypto';

let sessionStorage = {};

export function getAuthToken(request) {
  return request.headers.get('authorization')?.split(' ')[1];
}

export function getSessionUser(request) {
  const token = getAuthToken(request);
  return sessionStorage[token];
}

export function setSessionUser(user) {
  const token = crypto.randomBytes(16).toString('hex');
  sessionStorage[token] = user;
  return token;
}

export function clearSession() {
  sessionStorage = {};
}

// handlers
export const handlers = [
  rest.post("/login", (req, res, ctx) => {
    const { login } = req.variables;

    const user = db.account.findFirst({
      where: { id: { equals: login } },
    });

    const token = setSessionUser(user);

    return res(ctx.json({ token, user }));
  }),
  rest.get("/posts", (req, res, ctx) => {
    const user = getSessionUser(req);
	  return res(ctx.json({ posts: user.posts }))
  }),
]
```

클라이언트의 로그인 또한 프로젝트 구현에 따라 달라질 수 있습니다. 테스트용 로그인 helper를 만들었다 가정을 합니다

```js
test('Post를 작성한 유저는 모든 Post를 볼 수 있다.', () => {
  renderWithLogin(Component, 'has-posts')

  const items = await screen.findByRole('listitem');

  expect(items.length).toBe(2);
})

test('Post를 작성하지 않은 유저는 `글쓰기` 버튼을 노출한다.', () => {
  renderWithLogin(Component, 'no-posts')

  const button = await screen.findByRole('button', { name: '글쓰기' });

  expect(button).toBeInDocument();
})
```

## 다 좋은데, 구조가 너무 복잡해지는 아닌가요?

MSW를 통해

- 테스트 코드 가독성
- 모킹데이터 관리 용이성

을 얻어냈지만, 비용 또한 만만치 않습니다. 이미 보셨듯 꽤나 공수가 들어가는 작업입니다. 서버로직이 바뀐다면? 이에따라 관리해야할 코드가 늘어나기도 합니다. 그렇기에 모든 테스트를 MSW로 작성하고 이를 유지보수하기란 쉽지 않습니다.

## 결론

- 복잡한 Mocking 데이터가 많이 필요한 테스트
- 실패하면 안되는 Critical한 UI 테스트

앞선 Trade-Off를 고려하여, 위와 같은 경우 MSW를 활용하여 UI 통합 테스트를 작성하시길 추천드립니다.

후... 생각보다 긴 여정이었네요. 그럼에도 안정적인 서비스를 제공하기 위해 테스트는 필수입니다. MSW로 여러분들의 프로덕트가 더 견고해지는데 도움이 됐으면 좋겠습니다.(그리고 야근도 줄어들길 기원합니다 :pray:)
