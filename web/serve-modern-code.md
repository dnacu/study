# Serve modern code to modern browsers for faster page loads

## babel-preset-env

기본적으로 바벨의 동작은 현재(2020년도 기준) ES2015(es6)에서 ES2020의 코드를 ES5에서 사용할 수 있도록 트랜스파일링 해준다.

target에 따라서 이를 조율할 수 있는데, 그것에 대해 알아보겠다.

### targets option

targets를 설정하지 않아도 기본으로 동작하나, 이는 공식 문서에서 권장하지 않는 방법이다. 그 이유는 target을 특정했을 때에 오는 이점을 얻을 수 없기 때문이다.

> 최신브라우저만 지원할건데 targets를 설정 안해서 ie까지 지원하다 보니 polyfill이 많아져 번들 파일의 크기가 커지는 등의 단점을 말하는 듯하다.

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": "last 2 versions",
        "debug": true
      }
    ],
    "@babel/preset-react"
  ]
}
```

`targets: last 2 versions`

```json
{
  "android": "85",
  "chrome": "85",
  "edge": "85",
  "firefox": "80",
  "ie": "10",
  "ios": "13.4",
  "opera": "70",
  "safari": "13.1",
  "samsung": "11.1"
}
```

`targets: defaults`

```json
{
  "android": "85",
  "chrome": "84",
  "edge": "85",
  "firefox": "78",
  "ie": "11",
  "ios": "12.2",
  "opera": "70",
  "safari": "13.1",
  "samsung": "11.1"
}
```

`debug: true` 옵션을 통해 타겟 브라우저 리스트 및 추가된 플러그인들에 대한 정보를 얻을 수 있다.
둘을 보면 타겟 브라우저의 버전이 상이하며, 이는 번들코드가 서로 다를 수 있음을 의미한다.

> `targets`에 작성 가능한 문법은 [browserlist](https://github.com/browserslist/browserslist#queries) 를 보면 알 수 있다.

### useBuiltIns option

기본적으로 바벨은 트랜스파일링을 위해 필요한 모든 폴리필이 담긴 `@babel/polyfill` 을 가져와서 사용하는데, 이는 불필요한 폴리필까지도 코드에 포함될 수 있음을 뜻한다.

`useBuiltIns` 옵션을 활용하면 필요한 폴리필만을 가져오도록 설정할 수 있다.

> 바벨은 es6이상의 문법을 es5이하 문법들로 트랜스파일링 해주는데, 새롭게 생성된 메서드나 생성자까지는 지원해주지 않는다. 따라서 이들을 구현한 폴리필을 추가함으로써 낮은 버전만을 지원하는 브라우저에서도 잘 동작하도록 한다.

`false`  
기본값이다. `@babel/polyfill`을 가져온다. (다 가져온다.)

`entry`  
현재 타겟 브라우저에서 필요한 모든 폴리필을 가져온다.

`usage`  
실제로 필요한부분(코드에서 실제로 쓰는 부분)에 대해서만 가져온다.

> 현재 `@babel/polyfill`은 deprecated 상태이며, `core-js, regenerator-runtime`을 인스톨하여 직접 import해주어야 한다. `regenerator-runtime` 은 generator function을 위한 폴리필이다.

## use ES modules in modern browser

바벨을 통해 변환된 파일에는 낮은 버전의 브라우저를 지원하기 위한 여러가지 폴리필들이 포함되어 있다. 이는 파일의 크기를 크게 만든다.

최신 브라우저에서는 es6문법과 esmodule 방식을 지원하기 떄문에 추가적인 변환이 필요하지 않다. 하지만 통신 등의 이유로 파일을 번들링하는 것이 추천된다.(압축/난독화등을 포함하여)

최신 브라우저와 구버전 브라우저를 위한 번들 파일을 각각 생성해보자.

> ES moudle 방식은 93.x%의 브라우저 커버리지를 가진다. 잘 사용되지 않는 브라우저들(KaiOS, Baibu, UC browser 등)을 제외하면 꽤 높은 비율일 것이라 생각하며, 이는 점차 증가할 것이다.

### configuration

`React, Typescript`를 사용하는 프로젝트의 환경설정.

`BabelMultiTargetPlugin` 를 사용하여 babel설정만 다른 두 벌의 빌드파일을 쉽게 생성할 수 있다.

**webpack.config.js**

```js
  // ...

  plugins: [
    new HtmlWebpackPlugin({ template: 'index.html' }),
    new BabelMultiTargetPlugin({
      // !!! experimental option !!!
      // safari 10.1 에서 두 모듈(type=module, nomodule)이 중복 로드되는 현상 방지
      normalizeModuleIds: true,
      babel: {
        presetOptions: {
          targets: {
            browsers: ['last 2 versions', 'not IE', 'not dead'],
          },
          useBuiltIns: 'usage',
          corejs: { version: 3, proposals: true },
        },
      },
    }),
  ],
  module: {
    rules: [
      {
        test: /\.(jsx?|tsx?)$/,
        loaders: [BabelMultiTargetPlugin.loader(), 'ts-loader'],
        exclude: /node_modules/,
      },
      {
        test: /\.(css|scss)$/,
        loaders: ['style-loader', 'css-loader', 'sass-loader'],
      },
    ],
  },

  // ...
```

**.babelrc**

```json
{
  "env": {
    "modern": {
      "presets": [
        "@babel/preset-react",
        "@babel/preset-env",
        {
          "targets": {
            "esmodules": true
          }
        }
      ]
    },
    "legacy": {
      "presets": [
        "@babel/preset-react",
        "@babel/preset-env",
        {
          "targets": {
            "esmodules": false
          }
        }
      ]
    }
  }
}
```

**tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "es2019",
    "strict": false,
    "jsx": "react",
    "module": "ES6",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": false,
    "isolatedModules": false,
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "exclude": ["node_modules", "webpack.config.js", "public"]
}
```

> `module: CommonJS`로 두면 ts-laoder가 ts(x) 파일 컴파일 시에 모듈방식을 CommonJS 스타일로 변경한다. ES6방식을 그대로 두도록 하고, 낮은 브라우저 호환이 필요하다면 바벨 트랜스파일링 시에 동작할 수 있도록 하자.

## results

![image](https://user-images.githubusercontent.com/36905916/98625703-72c98800-2353-11eb-87cc-89867868f9de.png)

결과적으로 두 개의 번들링 파일이 생성되어 html파일에 들어갔다. es6를 지원하는 최신 브라우저에서는 `nomodule`옵션을 가진 스크립트를 무시하기 때문에 `bundle.modern.mjs` 파일만 다운로드된다.

> `type=module` 스크립트는 자동으로 defer속성이 정의되기 때문에 추가적으로 설정할 필요가 없다.

## features of module script

- 중복되어도 단 한번만 다운로드한다. (클래식 그렇지 않음)
- `Access-Control-Allow-Origin: *`등의 CORS정책에 알맞은 헤더가 필요하다. (클래식은 그냥 다운로드됨)
  > 그렇기 때문에 `file://`로 html에 접근하면 `.mjs`파일을 로드하지 못한다.

## 참고

- https://web.dev/codelab-serve-modern-code/
- https://velog.io/@widian/%EC%9B%B9%EC%97%90%EC%84%9C-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0
