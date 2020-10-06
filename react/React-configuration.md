## babel

```
npm i -D @babel/core babel-loader @babel/preset-react @babel/preset-env
```

바벨은 기본적으로 아무일도안한다. 플러그인이 있어야 동작한다!

- @babel/core: transfile esnext to es5
- @babel/preset-react: transfile jsx to js
- @babel/preset-env: es6문법 사용을 위한 plugin들의 집합체
- babel-loader: for using webpack

## webpack

```
npm i -D webpack webpack-dev-server webpack-cli html-webpack-plugin
```

- webpack: bundler
- webpack-dev-server: hot-reload기능 사용 가능. (webpack --watch로도 오토빌드는 가능)
- webpack-cli: commandline interface에서 webpack을 사용할 수 있도록 하기 위함
- html-webpack-plugin: html파일을
