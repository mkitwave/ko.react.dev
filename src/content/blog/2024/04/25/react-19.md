---
title: "React 19 Beta"
author: The React Team
date: 2024/04/25
description: 이제 npm에서 React 19 베타를 사용할 수 있습니다! 이 글에서는 React 19의 새로운 기능에 대한 개요와 이를 도입하는 방법에 대해 설명하겠습니다.
---

April 25, 2024 by [The React Team](/community/team)

---

<Note>

이번 베타 릴리스는 라이브러리들이 React 19를 준비하기 위해 제공되었습니다. 애플리케이션 개발자는 18.3.0으로 버전을 업그레이드하고, 라이브러리들이 피드백에 따라 변경되며 React 19가 안정화될 때까지 기다려 주세요.

</Note>

<Intro>

이제 npm에서 React 19 베타를 사용할 수 있습니다!

</Intro>

[React 19 베타 업그레이드 가이드](/blog/2024/04/25/react-19-upgrade-guide)에서 애플리케이션을 React 19 베타로 업그레이드하는 단계별 지침을 공유했습니다. 이 글에서는 React 19의 새로운 기능에 대한 개요와 이를 도입하는 방법에 대해 설명하겠습니다.

- [React 19의 새로운 기능](#whats-new-in-react-19)
- [React 19의 개선 사항](#improvements-in-react-19)
- [업그레이드하는 방법](#how-to-upgrade)

주요 변경 사항 목록은 [업그레이드 가이드](/blog/2024/04/25/react-19-upgrade-guide)를 참고하세요.

---

## React 19의 새로운 기능 {/*whats-new-in-react-19*/}

### 액션(Actions) {/*actions*/}

React 애플리케이션의 보편적인 사용 사례는 데이터를 변경하고 응답을 기반으로 상태를 업데이트하는 것입니다. 예를 들어, 사용자가 이름을 변경하기 위해 폼(form)을 제출하면, API 요청을 하고 응답을 처리합니다. 이전에는 대기 상태, 오류, 낙관적 업데이트, 순차적 요청을 직접 처리해야 했습니다.

예를 들어, `useState`로 대기 상태와 오류 상태를 처리할 수 있었습니다.

```js
// 액션 도입 전
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    } 
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

React 19에서는, 트랜지션에서 비동기 함수를 사용하여 대기 상태, 오류, 폼 및 낙관적 업데이트를 자동으로 처리하는 기능이 추가됩니다.

아래는 `useTransition`을 사용해 대기 상태를 처리하는 예시입니다.

```js
// 액션 기반으로 대기 상태 사용
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

비동기 트랜지션은 `isPending` 상태를 즉시 true로 설정하고, 비동기 요청을 수행하고, 트랜지션이 실행된 이후 `isPending`을 false로 전환합니다. 이러한 방식은 데이터가 변경되는 도중에도 현재 UI의 반응성 및 상호작용을 유지할 수 있습니다.

<Note>

#### 컨벤션에 따라, 비동기 트랜지션을 사용하는 함수는 "액션(Actions)"이라고 부릅니다. {/*by-convention-functions-that-use-async-transitions-are-called-actions*/}

액션은 데이터 제출을 자동으로 관리합니다.

- **대기 상태**: 액션은 요청이 시작될 때 시작하여, 최종 상태 업데이트가 커밋되면 자동으로 재설정되는 대기 상태를 제공합니다.
- **낙관적 업데이트**: 액션은 새로운 Hook [`useOptimistic`](#new-feature-optimistic-updates)을 지원합니다. 이를 통해 요청이 제출되는 동안 사용자에게 즉각적인 피드백을 표출할 수 있습니다.
- **오류 처리**: 액션은 오류 처리 기능을 제공합니다. 이를 통해 요청이 실패할 때 Error Boundary를 표시하고, 낙관적 업데이트 값을 자동으로 원본 값으로 되돌릴 수 있습니다.
- **폼(form)**: 이제 `<form>` 요소의 `action`과 `formAction` 프로퍼티에 함수를 전달할 수 있습니다. `action` 프로퍼티에 함수를 전달하면 기본적으로 액션이 사용되며, 제출 이후에 자동으로 폼을 재설정합니다.

</Note>

액션을 기반으로 구축된 React 19는, 낙관적 업데이트를 관리하기 위한 [`useOptimistic`](#new-hook-optimistic-updates) Hook과 액션의 일반적 사용을 위한 [`React.useActionState`](#new-hook-useactionstate) Hook을 도입하였습니다. `react-dom`에서는 폼을 자동으로 관리하기 위해 [`<form>` 액션](#form-actions)을 추가하고, 폼에서 액션의 일반적인 경우를 지원하기 위해 [`useFormStatus`](#new-hook-useformstatus)를 추가하고 있습니다.

단순한 React 19 예제로 위의 내용을 확인해보겠습니다.

```js
// <form> action 프로퍼티와 useActionState 사용하기
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

다음 섹션에서는 React 19의 새로운 액션 기능들을 하나씩 살펴보겠습니다.

### 새로운 Hook: `useActionState` {/*new-hook-useactionstate*/}

액션의 일반적인 사용을 위해 `useActionState` Hook을 추가하였습니다.

```js
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // 액션의 어떤 결과든 반환할 수 있습니다.
      // 이 예제에서는 오류를 반환합니다.
      return error;
    }

    // 성공한 경우
    return null;
  },
  null,
);
```

`useActionState`는 함수(액션)를 받아 호출할 래핑된 액션을 반환합니다. 이는 각 액션이 조합되기에 가능합니다. 래핑된 액션이 호출되면, `useActionState`는 액션의 마지막 결과를 `data`로, 액션의 대기 상태를 `pending`으로 반환합니다.

<Note>

이전 Canary 릴리즈에서 `React.useActionState`를 `ReactDOM.useFormState`라고 불렀습니다. 그러나 이번 버전에서는 `useFormState`를 폐기하고 이름을 변경하였습니다.

자세한 내용은 [#28491](https://github.com/facebook/react/pull/28491)을 참고하세요.

</Note>

자세한 내용은 [`useActionState`](/reference/react/useActionState)를 참고하세요.

### React DOM: `<form>` 액션 {/*form-actions*/}

액션은 `react-dom`을 위한 React 19의 새로운 `<form>` 기능과도 통합되어 있습니다. 액션 기반의 자동 폼 제출을 위해 `<form>`, `<input>`, `<button>` 요소의 `action`과 `formAction` 프로퍼티에 함수를 전달할 수 있습니다.

```js [[1,1,"actionFunction"]]
<form action={actionFunction}>
```

`<form>` 액션이 성공적으로 수행되면, React는 제어되지 않는 컴포넌트들을 위해 폼을 자동으로 재설정합니다. 만약 `<form>`을 수동으로 재설정하고 싶다면, 새로운 React DOM API인 `requestFormReset`를 호출할 수 있습니다.

자세한 내용은 `react-dom` 문서의 [`<form>`](/reference/react-dom/components/form), [`<input>`](/reference/react-dom/components/input), `<button>`을 참고하세요.

### React DOM: 새로운 Hook: `useFormStatus` {/*new-hook-useformstatus*/}

디자인 시스템에서, 프로퍼티 드릴링 없이 해당 컴포넌트가 속한 `<form>`의 정보에 접근해야 할 때가 있습니다. Context를 사용해 구현할 수도 있지만, 더 쉽게 다루기 위해 `useFormStatus` Hook을 추가하였습니다.

```js [[1, 4, "pending"], [1, 5, "pending"]]
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

`useFormStatus`는 Context provider처럼 부모 `<form>`의 상태를 읽습니다.

자세한 내용은 `react-dom` 문서의 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus)를 참고하세요.

### 새로운 Hook: `useOptimistic` {/*new-hook-optimistic-updates*/}

데이터를 변경할 때 자주 사용되는 UI 패턴은, 비동기 요청이 진행되는 동안 최종 상태를 낙관적으로 표시하는 것입니다. 이 동작을 쉽게 다루기 위해, React 19에서는 `useOptimistic` Hook을 추가했습니다.

```js {2,6,13,19}
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>이름: {optimisticName}</p>
      <p>
        <label>이름 변경:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

`useOptimistic` Hook은 `updateName` 요청이 진행되는 동안 `optimisticName`을 즉시 렌더링합니다. 변경 요청이 완료되거나 실패하면, React는 자동으로 `optimisticName`을 `currentName` 값으로 전환시킵니다.

자세한 내용은 [`useOptimistic`](/reference/react/useOptimistic)을 참고하세요.

### 새로운 API: `use` {/*new-feature-use*/}

React 19에서는 렌더링에서 리소스를 읽는 새로운 API, `use`가 도입되었습니다.

예를 들어 `use`로 promise를 읽는 경우, React는 promise가 완료될 때까지 일시적으로 중단됩니다.

```js {1,5}
import {use} from 'react';

function Comments({commentsPromise}) {
  // `use` 는 promise가 완료될 때까지 중단됩니다.
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // Comments에서 `use`가 중단되면,
  // 아래의 suspense boundary가 표시됩니다.
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```

<Note>

#### `use`는 렌더링 중 생성된 promise를 지원하지 않습니다. {/*use-does-not-support-promises-created-in-render*/}

렌더링 중 생성된 promise를 `use`에 전달한다면, React는 아래와 같은 경고를 표시합니다.

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

A component was suspended by an uncached promise. Creating promises inside a Client Component or hook is not yet supported, except via a Suspense-compatible library or framework.

</ConsoleLogLine>

</ConsoleBlockMulti>

이 문제를 해결하려면 promise 캐싱을 지원하는 supense 기반 라이브러리 또는 프레임워크에서 promise를 전달해야 합니다. 향후 렌더링에서 promise를 더 쉽게 캐싱할 수 있는 기능을 제공할 계획입니다.

</Note>

`use`로 Context를 읽을 수도 있으며, 얼리 리턴과 같이 조건부로 Context를 읽는 것도 가능합니다.

```js {1,11}
import {use} from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }
  
  // 얼리 리턴에 의해 
  // useContext를 사용할 수 없습니다.
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```

`use` API는 Hook과 유사하게 렌더링에서만 호출될 수 있습니다. Hook과 다른 점은, `use`는 조건부로 호출될 수 있습니다. 향후 `use`를 활용해 렌더링에서 리소스를 사용하는 더 많은 방법들을 제공할 예정입니다.

자세한 내용은 [`use`](/reference/react/use)를 참고하세요.


## React 서버 컴포넌트 {/*react-server-components*/}

### 서버 컴포넌트 {/*server-components*/}

서버 컴포넌트는 번들링 전, 클라이언트 애플리케이션 또는 SSR 서버와 '분리된 환경'에서 컴포넌트를 미리 렌더링할 수 있는 새로운 방법입니다. 여기서 '분리된 환경'은 React 서버 컴포넌트에서 '서버'를 의미합니다. 서버 컴포넌트는 CI 서버에서 빌드 시 한 번 실행되거나 웹 서버에서 각 요청에 대해 실행될 수 있습니다.

React 19 includes all of the React Server Components features included from the Canary channel. This means libraries that ship with Server Components can now target React 19 as a peer dependency with a `react-server` [export condition](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md#react-server-conditional-exports) for use in frameworks that support the [Full-stack React Architecture](/learn/start-a-new-react-project#which-features-make-up-the-react-teams-full-stack-architecture-vision). 

React 19에는 Canary 버전의 모든 React 서버 컴포넌트 기능을 포함합니다. 즉, 서버 컴포넌트와 함께 제공되는 라이브러리들은 [풀스택 React 아키텍쳐](/learn/start-a-new-react-project#which-features-make-up-the-react-teams-full-stack-architecture-vision)를 지원하는 프레임워크에서 사용하기 위해 React 19를 하위 종속성으로 타겟팅할 수 있습니다. (참고: `react-server` [export condition](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md#react-server-conditional-exports))

<Note>

#### 서버 컴포넌트를 지원하려면 어떻게 해야 하나요? {/*how-do-i-build-support-for-server-components*/}

React 19의 React 서버 컴포넌트는 안정적이며 메이저 버전 간에 충돌되지 않지만, React 서버 컴포넌트 번들러 또는 프레임워크를 구현하는 데 사용되는 기본 API는 유의적 버전을 따르지 않으므로 React 19.x의 마이너 버전 간에 충돌될 수 있습니다.

번들러 또는 프레임워크로서 React 서버 컴포넌트를 지원하려면 특정 React 버전으로 고정하거나 Canary 릴리즈를 사용하는 것이 좋습니다. 향후에도 번들러 및 프레임워크와 협력하여 React 서버 컴포넌트를 구현하는 데 사용되는 API를 안정화할 예정입니다.

</Note>


자세한 내용은 [React 서버 컴포넌트](/reference/rsc/server-components)를 참고하세요.

### 서버 액션 {/*server-actions*/}

서버 액션은 클라이언트 컴포넌트가 서버에서 실행되는 비동기 함수를 호출할 수 있게 합니다.

"use server" 지시어로 서버 액션이 정의되면, 프레임워크는 자동으로 서버 함수에 대한 참조를 생성하고 해당 참조를 클라이언트 컴포넌트에 전달합니다. 클라이언트에서 해당 함수가 호출되면 React는 서버에 함수를 실행하라는 요청을 보내고 결과를 반환합니다.

<Note>

#### 서버 컴포넌트에는 지시어가 없습니다. {/*there-is-no-directive-for-server-components*/}

`"use server"` 지시어가 서버 컴포넌트를 나타낸다고 오해하는 경우가 많지만, 서버 컴포넌트에는 지시어가 없습니다. `"use server"` 지시어는 서버 액션에 사용됩니다.

자세한 내용은 [지시어](/reference/rsc/directives)를 참고하세요.

</Note>

서버 액션은 서버 컴포넌트에서 생성하여 클라이언트 컴포넌트에 props로 전달하거나 클라이언트 컴포넌트에 import해 사용할 수 있습니다.

자세한 내용은 [React 서버 액션](/reference/rsc/server-actions)을 참고하세요.

## React 19의 개선 사항 {/*improvements-in-react-19*/}

### `ref`를 prop으로 사용하기 {/*ref-as-a-prop*/}

React 19부터 함수 컴포넌트에서 `ref`를 prop으로 사용할 수 있습니다.

```js [[1, 1, "ref"], [1, 2, "ref", 45], [1, 6, "ref", 14]]
function MyInput({placeholder, ref}) {
  return <input placeholder={placeholder} ref={ref} />
}

//...
<MyInput ref={ref} />
```

새로운 함수 컴포넌트에는 더 이상 `forwardRef`가 필요하지 않으며, 새로운 `ref` prop을 사용하도록 컴포넌트를 자동으로 업데이트하는 codemod를 배포할 예정입니다. 향후 버전에서는 `forwardRef`를 더 이상 사용하지 않고 제거할 예정입니다.

<Note>

클래스 컴포넌트에 전달된 `refs`는 컴포넌트 인스턴스를 참조하므로 props로 전달되지 않습니다.

</Note>

### 하이드레이션 오류 개선 {/*diffs-for-hydration-errors*/}

`react-dom`의 하이드레이션 오류 표시 방식 또한 개선되었습니다. 이전에는 개발 환경에서 하이드레이션 불일치가 발생했을 때 구체적인 정보 없이 여러 에러를 보여주고 있었습니다.

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Warning: Text content did not match. Server: "Server" Client: "Client"
{'  '}at span
{'  '}at App

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: An error occurred during hydration. The server HTML was replaced with client content in \<div\>.

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: Text content did not match. Server: "Server" Client: "Client"
{'  '}at span
{'  '}at App

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: An error occurred during hydration. The server HTML was replaced with client content in \<div\>.

</ConsoleLogLine>

<ConsoleLogLine level="error">

Uncaught Error: Text content does not match server-rendered HTML.
{'  '}at checkForUnmatchedText
{'  '}...

</ConsoleLogLine>

</ConsoleBlockMulti>

React 19에서는 불일치 사항에 대해 하나의 오류 메세지로 표시합니다.

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Uncaught Error: Hydration failed because the server rendered HTML didn't match the client. As a result this tree will be regenerated on the client. This can happen if an SSR-ed Client Component used:{'\n'}
\- A server/client branch `if (typeof window !== 'undefined')`.
\- Variable input such as `Date.now()` or `Math.random()` which changes each time it's called.
\- Date formatting in a user's locale which doesn't match the server.
\- External changing data without sending a snapshot of it along with the HTML.
\- Invalid HTML tag nesting.{'\n'}
It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.{'\n'}
https://react.dev/link/hydration-mismatch {'\n'}
{'  '}\<App\>
{'    '}\<span\>
{'+    '}Client
{'-    '}Server{'\n'}
{'  '}at throwOnHydrationMismatch
{'  '}...

</ConsoleLogLine>

</ConsoleBlockMulti>

### `<Context>`를 provider로 사용하기 {/*context-as-a-provider*/}

React 19에서는 `<Context.Provider>` 대신 `<Context>`를 provider로 사용할 수 있습니다.


```js {5,7}
const ThemeContext = createContext('');

function App({children}) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );  
}
```

새로운 Context provider는 `<Context>`를 사용할 수 있으며, 이미 작성된 provider를 변경하는 codemod를 배포할 예정입니다. 이후 버전에서 `<Context.Provider>`는 폐기될 예정입니다.

### Cleanup functions for refs {/*cleanup-functions-for-refs*/}

We now support returning a cleanup function from `ref` callbacks:

```js {7-9}
<input
  ref={(ref) => {
    // ref created

    // NEW: return a cleanup function to reset
    // the ref when element is removed from DOM.
    return () => {
      // ref cleanup
    };
  }}
/>
```

When the component unmounts, React will call the cleanup function returned from the `ref` callback. This works for DOM refs, refs to class components, and `useImperativeHandle`. 

<Note>

Previously, React would call `ref` functions with `null` when unmounting the component. If your `ref` returns a cleanup function, React will now skip this step.

In future versions, we will deprecate calling refs with `null` when unmounting components.

</Note>

Due to the introduction of ref cleanup functions, returning anything else from a `ref` callback will now be rejected by TypeScript. The fix is usually to stop using implicit returns, for example:

```diff [[1, 1, "("], [1, 1, ")"], [2, 2, "{", 15], [2, 2, "}", 1]]
- <div ref={current => (instance = current)} />
+ <div ref={current => {instance = current}} />
```

The original code returned the instance of the `HTMLDivElement` and TypeScript wouldn't know if this was _supposed_ to be a cleanup function or if you didn't want to return a cleanup function.

You can codemod this pattern with [`no-implicit-ref-callback-return
`](https://github.com/eps1lon/types-react-codemod/#no-implicit-ref-callback-return).

### `useDeferredValue` initial value {/*use-deferred-value-initial-value*/}

We've added an `initialValue` option to `useDeferredValue`:

```js [[1, 1, "deferredValue"], [1, 4, "deferredValue"], [2, 4, "''"]]
function Search({deferredValue}) {
  // On initial render the value is ''.
  // Then a re-render is scheduled with the deferredValue.
  const value = useDeferredValue(deferredValue, '');
  
  return (
    <Results query={value} />
  );
}
````

When <CodeStep step={2}>initialValue</CodeStep> is provided, `useDeferredValue` will return it as `value` for the initial render of the component, and schedules a re-render in the background with the <CodeStep step={1}>deferredValue</CodeStep> returned.

For more, see [`useDeferredValue`](/reference/react/useDeferredValue).

### Support for Document Metadata {/*support-for-metadata-tags*/}

In HTML, document metadata tags like `<title>`, `<link>`, and `<meta>` are reserved for placement in the `<head>` section of the document. In React, the component that decides what metadata is appropriate for the app may be very far from the place where you render the `<head>` or React does not render the `<head>` at all. In the past, these elements would need to be inserted manually in an effect, or by libraries like [`react-helmet`](https://github.com/nfl/react-helmet), and required careful handling when server rendering a React application. 

In React 19, we're adding support for rendering document metadata tags in components natively:

```js {5-8}
function BlogPost({post}) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Josh" />
      <link rel="author" href="https://twitter.com/joshcstory/" />
      <meta name="keywords" content={post.keywords} />
      <p>
        Eee equals em-see-squared...
      </p>
    </article>
  );
}
```

When React renders this component, it will see the `<title>` `<link>` and `<meta>` tags, and automatically hoist them to the `<head>` section of document. By supporting these metadata tags natively, we're able to ensure they work with client-only apps, streaming SSR, and Server Components.

<Note>

#### You may still want a Metadata library {/*you-may-still-want-a-metadata-library*/}

For simple use cases, rendering Document Metadata as tags may be suitable, but libraries can offer more powerful features like overriding generic metadata with specific metadata based on the current route. These features make it easier for frameworks and libraries like [`react-helmet`](https://github.com/nfl/react-helmet) to support metadata tags, rather than replace them.

</Note>

For more info, see the docs for [`<title>`](/reference/react-dom/components/title), [`<link>`](/reference/react-dom/components/link), and [`<meta>`](/reference/react-dom/components/meta).

### Support for stylesheets {/*support-for-stylesheets*/}

Stylesheets, both externally linked (`<link rel="stylesheet" href="...">`) and inline (`<style>...</style>`), require careful positioning in the DOM due to style precedence rules. Building a stylesheet capability that allows for composability within components is hard, so users often end up either loading all of their styles far from the components that may depend on them, or they use a style library which encapsulates this complexity.

In React 19, we're addressing this complexity and providing even deeper integration into Concurrent Rendering on the Client and Streaming Rendering on the Server with built in support for stylesheets. If you tell React the `precedence` of your stylesheet it will manage the insertion order of the stylesheet in the DOM and ensure that the stylesheet (if external) is loaded before revealing content that depends on those style rules.

```js {4,5,17}
function ComponentOne() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="foo" precedence="default" />
      <link rel="stylesheet" href="bar" precedence="high" />
      <article class="foo-class bar-class">
        {...}
      </article>
    </Suspense>
  )
}

function ComponentTwo() {
  return (
    <div>
      <p>{...}</p>
      <link rel="stylesheet" href="baz" precedence="default" />  <-- will be inserted between foo & bar
    </div>
  )
}
```

During Server Side Rendering React will include the stylesheet in the `<head>`, which ensures that the browser will not paint until it has loaded. If the stylesheet is discovered late after we've already started streaming, React will ensure that the stylesheet is inserted into the `<head>` on the client before revealing the content of a Suspense boundary that depends on that stylesheet.

During Client Side Rendering React will wait for newly rendered stylesheets to load before committing the render. If you render this component from multiple places within your application React will only include the stylesheet once in the document:

```js {5}
function App() {
  return <>
    <ComponentOne />
    ...
    <ComponentOne /> // won't lead to a duplicate stylesheet link in the DOM
  </>
}
```

For users accustomed to loading stylesheets manually this is an opportunity to locate those stylesheets alongside the components that depend on them allowing for better local reasoning and an easier time ensuring you only load the stylesheets that you actually depend on.

Style libraries and style integrations with bundlers can also adopt this new capability so even if you don't directly render your own stylesheets, you can still benefit as your tools are upgraded to use this feature.

For more details, read the docs for [`<link>`](/reference/react-dom/components/link) and [`<style>`](/reference/react-dom/components/style).

### Support for async scripts {/*support-for-async-scripts*/}

In HTML normal scripts (`<script src="...">`) and deferred scripts (`<script defer="" src="...">`) load in document order which makes rendering these kinds of scripts deep within your component tree challenging. Async scripts (`<script async="" src="...">`) however will load in arbitrary order.

In React 19 we've included better support for async scripts by allowing you to render them anywhere in your component tree, inside the components that actually depend on the script, without having to manage relocating and deduplicating script instances.

```js {4,15}
function MyComponent() {
  return (
    <div>
      <script async={true} src="..." />
      Hello World
    </div>
  )
}

function App() {
  <html>
    <body>
      <MyComponent>
      ...
      <MyComponent> // won't lead to duplicate script in the DOM
    </body>
  </html>
}
```

In all rendering environments, async scripts will be deduplicated so that React will only load and execute the script once even if it is rendered by multiple different components.

In Server Side Rendering, async scripts will be included in the `<head>` and prioritized behind more critical resources that block paint such as stylesheets, fonts, and image preloads.

For more details, read the docs for [`<script>`](/reference/react-dom/components/script).

### Support for preloading resources {/*support-for-preloading-resources*/}

During initial document load and on client side updates, telling the Browser about resources that it will likely need to load as early as possible can have a dramatic effect on page performance.

React 19 includes a number of new APIs for loading and preloading Browser resources to make it as easy as possible to build great experiences that aren't held back by inefficient resource loading.

```js
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'
function MyComponent() {
  preinit('https://.../path/to/some/script.js', {as: 'script' }) // loads and executes this script eagerly
  preload('https://.../path/to/font.woff', { as: 'font' }) // preloads this font
  preload('https://.../path/to/stylesheet.css', { as: 'style' }) // preloads this stylesheet
  prefetchDNS('https://...') // when you may not actually request anything from this host
  preconnect('https://...') // when you will request something but aren't sure what
}
```
```html
<!-- the above would result in the following DOM/HTML -->
<html>
  <head>
    <!-- links/scripts are prioritized by their utility to early loading, not call order -->
    <link rel="prefetch-dns" href="https://...">
    <link rel="preconnect" href="https://...">
    <link rel="preload" as="font" href="https://.../path/to/font.woff">
    <link rel="preload" as="style" href="https://.../path/to/stylesheet.css">
    <script async="" src="https://.../path/to/some/script.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

These APIs can be used to optimize initial page loads by moving discovery of additional resources like fonts out of stylesheet loading. They can also make client updates faster by prefetching a list of resources used by an anticipated navigation and then eagerly preloading those resources on click or even on hover.

For more details see [Resource Preloading APIs](/reference/react-dom#resource-preloading-apis).

### Compatibility with third-party scripts and extensions {/*compatibility-with-third-party-scripts-and-extensions*/}

We've improved hydration to account for third-party scripts and browser extensions.

When hydrating, if an element that renders on the client doesn't match the element found in the HTML from the server, React will force a client re-render to fix up the content. Previously, if an element was inserted by third-party scripts or browser extensions, it would trigger a mismatch error and client render.

In React 19, unexpected tags in the `<head>` and `<body>` will be skipped over, avoiding the mismatch errors. If React needs to re-render the entire document due to an unrelated hydration mismatch, it will leave in place stylesheets inserted by third-party scripts and browser extensions.

### Better error reporting {/*error-handling*/}

We improved error handling in React 19 to remove duplication and provide options for handling caught and uncaught errors. For example, when there's an error in render caught by an Error Boundary, previously React would throw the error twice (once for the original error, then again after failing to automatically recover), and then call `console.error` with info about where the error occurred. 

This resulted in three errors for every caught error:

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Uncaught Error: hit
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...

</ConsoleLogLine>

<ConsoleLogLine level="error">

Uncaught Error: hit<span className="ms-2 text-gray-30">{'    <--'} Duplicate</span>
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...

</ConsoleLogLine>

<ConsoleLogLine level="error">

The above error occurred in the Throws component:
{'  '}at Throws
{'  '}at ErrorBoundary
{'  '}at App{'\n'}
React will try to recreate this component tree from scratch using the error boundary you provided, ErrorBoundary.

</ConsoleLogLine>

</ConsoleBlockMulti>

In React 19, we log a single error with all the error information included:

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Error: hit
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...{'\n'}
The above error occurred in the Throws component:
{'  '}at Throws
{'  '}at ErrorBoundary
{'  '}at App{'\n'}
React will try to recreate this component tree from scratch using the error boundary you provided, ErrorBoundary.
{'  '}at ErrorBoundary
{'  '}at App

</ConsoleLogLine>

</ConsoleBlockMulti>

Additionally, we've added two new root options to complement `onRecoverableError`:

- `onCaughtError`: called when React catches an error in an Error Boundary.
- `onUncaughtError`: called when an error is thrown and not caught by an Error Boundary.
- `onRecoverableError`: called when an error is thrown and automatically recovered.

For more info and examples, see the docs for [`createRoot`](/reference/react-dom/client/createRoot) and [`hydrateRoot`](/reference/react-dom/client/hydrateRoot).

### Support for Custom Elements {/*support-for-custom-elements*/}

React 19 adds full support for custom elements and passes all tests on [Custom Elements Everywhere](https://custom-elements-everywhere.com/).

In past versions, using Custom Elements in React has been difficult because React treated unrecognized props as attributes rather than properties. In React 19, we've added support for properties that works on the client and during SSR with the following strategy:

- **Server Side Rendering**: props passed to a custom element will render as attributes if their type is a primitive value like `string`, `number`, or the value is `true`. Props with non-primitive types like `object`, `symbol`, `function`, or value `false` will be omitted.
- **Client Side Rendering**: props that match a property on the Custom Element instance will be assigned as properties, otherwise they will be assigned as attributes.

Thanks to [Joey Arhar](https://github.com/josepharhar) for driving the design and implementation of Custom Element support in React.


#### 업그레이드하는 방법 {/*how-to-upgrade*/}
See the [React 19 Upgrade Guide](/blog/2024/04/25/react-19-upgrade-guide) for step-by-step instructions and a full list of breaking and notable changes.



