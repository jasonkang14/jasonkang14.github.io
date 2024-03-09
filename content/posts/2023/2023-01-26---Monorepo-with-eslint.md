---
title: monorepo에 eslint 세팅
date: "2023-01-26T14:35:37.121Z"
template: "post"
draft: false
slug: "/react/monorepo-with-eslint"
category: "React"
tags:
  - "React"
  - "Monorepo"

description: "컨벤션을 위한 린트 적용"
---

프런트엔드 개발자가 1명(본인)에서 5명으로 늘어나면서, 컨벤션이 필요하다. 그리고 개인적으로 설정한 새로 오신 분 덕분에 함수에 parameter가 1개이면 소괄호가 빠지는 등의 문제가 있어서 eslint를 설정했던 적이 있다. 이번에는 처음부터 잡고 들어가려고 한다. 

우선 기존의 프로젝트에 있던 eslint 관련 패키지들을 설치한다. tailwind 설정할 때 작성했던 것처럼 lint 또한 전체 프로젝트에 적용할 예정이기 때문에 project root에 설치한다

```npm
pnpm add -w -D eslint-config-airbnb eslint-config-airbnb-typescript eslint-import-resolver-typescript eslint-plugin-jsx-a11y eslint-plugin-import  
eslint-plugin-react-hooks
```

`.eslint.json`을 생성하고 아래와 같이 입력했다. 

```json
// .eslint.json

{
  "env": {
    "browser": true,
    "es2021": true
  },
  "root": true,
  "extends": ["plugin:react/recommended", "airbnb", "airbnb-typescript"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": "latest",
    "sourceType": "module",
    "project": "./tsconfig.base.json"
  },
  "plugins": ["react", "@typescript-eslint"],
  "rules": {
    "react/jsx-filename-extension": [2, { "extensions": [".js", ".jsx", ".ts", ".tsx"] }],
    "react/react-in-jsx-scope": "off",
    "react/jsx-props-no-spreading": "off",
    "react/jsx-no-useless-fragment": ["error", { "allowExpressions": true }],
    "react/self-closing-comp": "off",
    "react/no-array-index-key": "off",
    "react/require-default-props": "off",
    "react/jsx-wrap-multilines": "off",
    "react/jsx-sort-props": [
      "error",
      {
        "callbacksLast": true,
        "shorthandFirst": true,
        "multiline": "ignore",
        "ignoreCase": true,
        "reservedFirst": ["key"]
      }
    ],
    "@typescript-eslint/indent": ["off"],
    "@typescript-eslint/no-use-before-define": ["off"],
    "@typescript-eslint/space-before-blocks": ["off"],
    "@typescript-eslint/naming-convention": [
      "error",
      {
        "selector": "default",
        "format": ["camelCase", "PascalCase", "UPPER_CASE", "snake_case"]
      }
    ],
    "consistent-return": "off",
    "indent": "off",
    "implicit-arrow-linebreak": "off",
    "import/order": "off",
    "import/prefer-default-export": "off",
    "import/extensions": [
      "error",
      "ignorePackages",
      {
        "js": "never",
        "jsx": "never",
        "ts": "never",
        "tsx": "never"
      }
    ],
    "jsx-a11y/media-has-caption": ["off"],
    "no-alert": "off",
    "no-console": "off",
    "no-confusing-arrow": "off",
    "no-use-before-define": "off",
    "no-underscore-dangle": ["error", { "allowAfterThis": true }],
    "no-param-reassign": ["error", { "props": false }],
    "no-plusplus": "off",
    "max-len": ["error", { "code": 120 }],
    "operator-linebreak": "off",
    "object-curly-newline": ["off"],
    "radix": ["error", "as-needed"],
    "space-before-blocks": "off",
    "function-paren-newline": "off"
  },
  "settings": {
    "import/resolver": {
      "typescript": {}
    }
  }
}
```

`rules`에 있는 항목들은 프런트엔드 팀에서 소통해서 정한 규칙들이다. 그리고 `.prettierrc`도 설정했다. 일반적인 타입스크립트 설정과 다른것은 아마 아래 부분일텐데,

```json
{
  "project": "./tsconfig.base.json"
}
```

파일 이름을 바꿔서 모든 워크스페이스에 적용할 수 있는 타입스크립트 설정을 하고. 각 워크스페이스에서는 기본 config를 extends하는 식으로 작성했다. 

```json
// .prettierrc

{
  "trailingComma": "all",
  "tabWidth": 2,
  "semi": true,
  "singleQuote": true,
  "printWidth": 120,
  "useTabs": false,
  "bracketSpacing": true,
  "endOfLine": "auto"
}
```

이렇게하면 설정이 완료된다.
