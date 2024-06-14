## 背景

React Router の[チュートリアル](https://reactrouter.com/en/main/start/tutorial)を TypeScript で試していたときに、
useRouteError を使用する[Handling Not Found Errors](https://reactrouter.com/en/main/start/tutorial#handling-not-found-errors)のところで TypeScript のエラーが発生して少し詰まったので、その解決方法を記載します。

## 前提条件

以下の環境を前提とします。

- React 18.2.0
- TypeScript 5.2.2
- react-router-dom 6.23.1

## useRouteError とは

React Router を設定したプロジェクト内で、何かしらのエラーが throw されたときにレンダリングされる[errorElement](https://reactrouter.com/en/main/route/error-element#errorelement)内で、useRouteError を使用します。  
useRouteError を使用すると、throw されたエラー内容を取得することができます。例えば存在しないページにアクセスしたときは、NotFound のエラー情報が入ります。

## TypeScript のエラー内容

チュートリアルと同じように、useRouteError 経由で取得した error オブジェクトから status や statusText を使用すると、「'error''は 'unknown' 型です。」というエラーが出ます。
これは useRouteError が unknown 型を返していることが原因です。

前述したようにプロジェクト内で発生した何かしらのエラー情報が useRouteError から返ってくるので、unknown 型が定義されているのは納得です。

```jsx
import { useRouteError } from "react-router-dom";

export default function ErrorPage() {
  const error = useRouteError();
  console.error(error);

  return (
    <div id="error-page">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <!-- 'error''は 'unknown' 型です。 -->
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}
```

## 解決方法

このエラーの解決方法として、[isRouteErrorResponse](https://reactrouter.com/en/main/utils/is-route-error-response) 関数を使用して型の絞り込みを行います。
isRouteErrorResponse の内部では、引数として受け取った error オブジェクトが以下のような ErrorResponse の型と一致するかを判定してくれます。

```typescript
export type ErrorResponse = {
  status: number;
  statusText: string;
  data: any;
};
```

これを利用して、以下のように型の絞り込みを行うと、TypeScript エラーを回避できます。

```jsx
import { useRouteError, isRouteErrorResponse } from "react-router-dom";

export default function ErrorPage() {
  const error = useRouteError();
  console.error(error);

  return (
    <div id="error-page">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      {isRouteErrorResponse(error) && (
        <p>
          <i>{error.statusText || error.message}</i>
        </p>
      )}
    </div>
  );
}
```

## 参考文献

- [What's the correct type for error in useRouteError() from react-router-dom?](https://stackoverflow.com/questions/75944820/whats-the-correct-type-for-error-in-userouteerror-from-react-router-dom)
- [useRouteError](https://reactrouter.com/en/main/hooks/use-route-error)
- [isRouteErrorResponse](https://reactrouter.com/en/main/utils/is-route-error-response)
