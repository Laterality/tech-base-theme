---
layout: post
title: TypeScript로 React 개발하기
date: 2018-04-06
author: Latera
---

<!-- desc -->
과제로 웹 프로젝트를 진행하면서 front-end에 react를 적용해보기로 했습니다. 이미 Javascript 대신 TypeScript를 적용했을 때의 이점을 node.js에서 이미 경험했기 때문에 이를 프론트 개발에서도 맛보고 싶다는 욕구가 있었습니다.

Javascript를 사용하는 React는 `.js`파일 대신 `.jsx`라고 하는 파일에 코드를 작성하는데, 이와 유사하게 TypeScript에서는 '`.tsx` 파일로 React 환경을 지원하고 있습니다. 

마침 TeypScript 문서에도 TypeScript로 React 프로젝트를 작성하는 가이드를 제공하고 있어서 이를 번역 겸 정리해봤습니다.

<!-- desc -->

## 사전 설정

패키지 설치를 위해서 `node.js`와 `npm`이 필요합니다.

글이 작성되는 시점에서 제가 사용한 버전은 다음과 같습니다.

* node.js: v8.9.0
* npm: 5.5.1

## 프로젝트 구성

프로젝트명은 `react-ts`로 가정하고, 프로젝트 디렉터리는 다음과 같이 구성하였습니다

```
react-ts
├ dist/
└ src/
  └ components/
```

* `dist`: webpack 실행 후 결과물(bundle.js)이 저장될 디렉터리입니다.
* `src`: TypeScript로 작성된 코드는 모두 이 디렉터리에 위치합니다.
* `components`: React의 컴포넌트 코드가 위치합니다.

이제 npm으로 프로젝트를 초기화합니다.

```
npm init
```

### 패키지 설치

Webpack을 사용하기 위해 전역으로 설치합니다.

```
npm install -g webpack-cli
```

Webpack은 프로젝트에 작성된 코드와 필요한 경우 의존성을 갖는 다른 코드들을 하나의 `.js`파일로 묶어서 생성해주는 도구입니다. 기능이 굉장히 많아서 여기서 Webpack에 대해 자세히 살펴보긴 어려울 것 같네요, 기회가 된다면 Webpack에 관해서도 알아보도록 하겠습니다.

React와 React-DOM을 설치합니다.

```
npm install --save react react-dom @types/react @types/react-dom
```

`@types/` 접두사를 갖는 패키지는 기존의 Javascript로 작성된 패키지가 TypeScript에서도 정적 타이핑을 지원하도록 추가하는 각종 선언에 대한 패키지입니다.

다음은 개발 단계에서만 사용되는 패키지들을 설치합니다.

```
npm install --save-dev webpack typescript awesome-typescript-loader source-map-loader
```

`awesome-typescript-loader`는 Webpack을 사용하여 TypeScript 코드를 컴파일하기 위해 사용되는 패키지입니다. `source-map-loader`은 TypeScript코드를 디버깅하기 수월하도록 소스맵을 함께 생성하는 패키지입니다.

### TypeScript 설정 파일 추가

TypeScript 컴파일러는 프로젝트 루트 디렉터리의 `tsconfig.json`에 프로젝트 옵션을 설정합니다.

```json
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "commonjs",
        "target": "es5",
        "jsx": "react"
    },
    "include": [
        "./src/**/*"
    ]
}
```
옵션들에 대한 자세한 내용은 [링크](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html)를 참조하시면 됩니다.


### Webpack 설정 파일

Webpack을 사용하기 위해 필요한 설정 파일을 작성합니다. 프로젝트 루트 디렉터리의 `webpack.config.js` 파일을 다음과 같이 작성해주세요.

```javascript
module.exports = {
    entry: "./src/index.tsx",
    output: {
        filename: "bundle.js",
        path: __dirname + "/dist"
    },

    // Enable sourcemaps for debugging webpack's output.
    devtool: "source-map",

    resolve: {
        // Add '.ts' and '.tsx' as resolvable extensions.
        extensions: [".ts", ".tsx", ".js", ".json"]
    },

    module: {
        rules: [
            // All files with a '.ts' or '.tsx' extension will be handled by 'awesome-typescript-loader'.
            { test: /\.tsx?$/, loader: "awesome-typescript-loader" },

            // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
            { enforce: "pre", test: /\.js$/, loader: "source-map-loader" }
        ]
    },

    // When importing a module whose path matches one of the following, just
    // assume a corresponding global variable exists and use that instead.
    // This is important because it allows us to avoid bundling all of our
    // dependencies, which allows browsers to cache those libraries between builds.
    externals: {
        "react": "React",
        "react-dom": "ReactDOM"
    },
};
```

## 코드 작성

본격적으로 코드를 작성해보도록 하겠습니다.

먼저, `src/components` 디렉터리에 `Hello.tsx` 파일을 다음과 같이 작성합니다.

```typescript
import * as React from "react";

export interface HelloProps { compiler: string; framework: string; }

export const Hello = (props: HelloProps) => <h1>Hello from {props.compiler} and {props.framework}!</h1>;
```

위의 코드는 [stateless functional compnent](https://reactjs.org/docs/components-and-props.html#functional-and-class-components)를 사용하고 있는데, 다음과 같이 클래스 형태로도 작성할 수 있습니다.

```typescript
import * as React from "react";

export interface HelloProps { compiler: string; framework: string; }

// 'HelloProps'는 prop의 형태를 설명합니다..
// 상태가 설정되어 있지 않기 때문에 '{}' 타입을 사용합니다.
export class Hello extends React.Component<HelloProps, {}> {
    render() {
        return <h1>Hello from {this.props.compiler} and {this.props.framework}!</h1>;
    }
}
```

다음은 `src` 디렉터리에 `index.tsx`파일을 다음과 같이 작성합니다.

```typescript
import * as React from "react";
import * as ReactDOM from "react-dom";

import { Hello } from "./components/Hello";

ReactDOM.render(
    <Hello compiler="TypeScript" framework="React" />,
    document.getElementById("example")
);
```

앞서 작성한 `Hello` 컴포넌트를 import 할 때 상대 경로를 사용한 것이 보이는데, TypeScript의 import문은 상대경로를 사용하지 않은 경우에는 해당 모듈을 `node_modules` 디렉터리에서 찾기 때문입니다.

이제 컴포넌트를 출력할 페이지를 작성하겠습니다.

프로젝트의 루트 디렉터리에 `index.html` 파일을 다음과 같이 작성합니다.

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello React!</title>
    </head>
    <body>
        <div id="example"></div>

        <!-- Dependencies -->
        <script src="./node_modules/react/umd/react.development.js"></script>
        <script src="./node_modules/react-dom/umd/react-dom.development.js"></script>

        <!-- Main -->
        <script src="./dist/bundle.js"></script>
    </body>
</html>
```

`node_modules`의 React와 React-DOM의 파일을 사용하는 것이 보이는군요.
두 패키지는 웹페이지에서 사용되는 `.js`파일을 하나의 독립된 파일로 패키지에 제공하고 있습니다. 필요한 경우에는 다른 디렉터리로 옮겨 사용할 수도 있습니다.
아니면, CDN 서비스를 이용하는 방법도 있겠네요.

## 빌드

한방이면 됩니다.

```
webpack
```

이제 `index.html` 파일을 열면 'Hello from TypeScript and React!' 메시지가 출력되는 걸 보실 수 있습니다.

## Conclusion

아직 본격적으로 React 개발을 시작해보지 않아서 React를 TypeScript로 개발했을 때 어떤 이점이 있는지 느껴보진 못했습니다. 다만 재밌는 것은 React를 Javascript로 개발할 때에도 컴포넌트에 전달되는 데이터 타입을 명시하도록 하는데, 이런 점들에서 TypeScript와 React가 추구하는 방향이 같다고 보여집니다. 