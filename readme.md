# 1. storybook 셋팅

1. 스토리북 설치
```properties
npx -p @storybook/cli sb init --type react
```

2. 스토리북 경로 변경 (.storybook\main.js 파일 수정)
```js
module.exports = {
  stories: ["../src/**/*.stories.js"],
  addons: ["@storybook/addon-actions", "@storybook/addon-links"]
};
```

3. Knobs 애드온 설치
```properties
yarn add -D @storybook/addon-knobs
```

<b>설명</b>: 컴포넌트의 props 를 스토리북 화면에서 바꿔서 바로 반영시켜줄 수 있는 애드온

4. Knobs 애드온 적용 (.storybook\main.js 파일 수정)
```js
module.exports = {
  stories: ["../src/**/*.stories.js"],
  addons: [
    "@storybook/addon-actions",
    "@storybook/addon-links",
    "@storybook/addon-knobs/register"
  ]
};
```

# 2. 문서화

1. Docs 애드온 설치
```properties
yarn add -D @storybook/addon-docs
yarn add -D react react-is babel-loader
```

<b>설명</b>: MDX 형식으로 문서를 작성 할 수 있게 해주고, 컴포넌트의 props와 주석에 기반하여 자동으로 아주 멋진 문서를 자동생성해줍니다.

2. Docs 애드온 적용 (.storybook\main.js 파일 수정)
```js
module.exports = {
  stories: ["../src/**/*.stories.(js|mdx)"],
  addons: [
    "@storybook/addon-actions",
    "@storybook/addon-links",
    "@storybook/addon-knobs/register",
    "@storybook/addon-docs"
  ]
};
```

3. prop-types 설치
```properties
yarn add prop-types
```

4. props 표현
```js
import React from "react";
import PropTypes from "prop-types";

/**
 * 안녕하세요 라고 보여주고 싶을 땐 `Hello` 컴포넌트를 사용하세요.
 *
 * - `big` 값을 `true`로 설정하면 **크게** 나타납니다.
 * - `onHello` 와 `onBye` props로 설정하여 버튼이 클릭했을 때 호출 할 함수를 지정 할 수 있습니다.
 */
const Hello = ({ name, big, onHello, onBye }) => {
  return (
    <div>
      {big ? <h1>안녕하세요, {name}!</h1> : <p>안녕하세요, {name}!</p>}
      <div>
        <button onClick={onHello}>Hello</button>
        <button onClick={onBye}>Bye</button>
      </div>
    </div>
  );
};

Hello.propTypes = {
  /** 보여주고 싶은 이름 */
  name: PropTypes.string.isRequired,
  /** 이 값을 `true` 로 설정하면 h1 태그로 렌더링 됩니다. */
  big: PropTypes.bool,
  /** Hello 버튼 누를 때 호출할 함수 */
  onHello: PropTypes.func,
  /** Bye 버튼 누를 때 호출할 함수 */
  onBye: PropTypes.func
};

Hello.defaultProps = {
  big: false
};

export default Hello;
```

# 3. Typescript 적용하기

1. Typescript 의존성 라이브러리 설치
```properties
yarn add -D babel-preset-react-app react-docgen-typescript-loader typescript
```

<b>babel-preset-react-app</b>: create-react-app 에서 사용하는 babel preset 과 동일합니다. TypeScript를 적용 할 때 우리는 babel-loader 를 사용 할 것입니다. 참고로 babel-loader는 Storybook 프로젝트에 이미 설치가 되어있습니다.<br>

<b>react-docgen-typescript-loader</b>: 컴포넌트의 props 에서 사용된 TypeScript 타입들을 추출하여 문서로 만들어주는 도구입니다. 우리가 이전에 PropTypes 를 작성하여 DocsPage 에서 보여줬었던 것 처럼 말이지요.<br>

<b>typescript</b>: 프로젝트에서 TypeScript를 사용하기 위하여 필수적으로 설치해야하는 패키지입니다.<br>


2. tsconfig.json 파일 작성
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react"
  },
  "include": ["src"]
}
```

3. .storybook\main.js 파일 수정
```js
module.exports = {
  stories: ["../src/**/*.stories.(js|tsx|mdx)"],
  addons: [
    "@storybook/addon-actions",
    "@storybook/addon-links",
    "@storybook/addon-knobs/register",
    "@storybook/addon-docs"
  ],
  webpackFinal: async (config, { configType }) => {
    config.module.rules.push({
      test: /\.(ts|tsx)$/,
      use: [
        {
          loader: require.resolve("babel-loader"),
          options: {
            presets: [["react-app", { flow: false, typescript: true }]]
          }
        },
        require.resolve("react-docgen-typescript-loader")
      ]
    });
    config.resolve.extensions.push(".ts", ".tsx");
    return config;
  }
};
```