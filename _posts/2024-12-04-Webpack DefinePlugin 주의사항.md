---
title: Webpack DefinePlugin 주의사항
date: 2024-12-04 16:00:00 +09:00
last_modified_at: 2024-12-04 16:00:00 +09:00
categories: [Webpack]
tags:
  [
    Webpack,
    DefinePlugin,
    Trobuleshooting,
  ]
---


## [contenthash]
### [contenthash]를 쓰는 이유는?
- Webpack에서 번들 시 contenthash를 사용하는 주된 이유는 파일 [캐싱](https://webpack.js.org/guides/caching/#output-filenames){:target="_blank"} 효율성을 극대화하기 위해서입니다.

### 브라우저 캐싱 최적화
- 브라우저는 파일 URL이 동일한 경우, 기존에 다운로드한 파일을 캐시에서 불러옵니다. 
- 파일 내용이 변경되더라도 URL이 동일하면 브라우저는 캐시된 오래된 파일을 계속 사용할 수 있습니다.
- contenthash는 파일의 내용에 따라 고유한 해시 값을 생성하여 파일 이름에 추가합니다. 
- 따라서 파일 내용이 변경될 경우 해시 값도 변경되어 새로운 URL이 생성됩니다. 
- 결과적으로, 변경된 파일만 다시 다운로드되고, 변경되지 않은 파일은 캐시에서 재사용됩니다.

## 주의사항
### 동일한 코드를 빌드했는데 매번 [contenthash]가 변경 됨
> 첫번째 Build 결과물  

![image](https://github.com/user-attachments/assets/06864308-d7c2-4d10-936d-b1ab256de87a){:style="width:430px; height:404px;"}

> 두번째 Build 결과물(코드 변경 X)

![image](https://github.com/user-attachments/assets/e990a0d1-9bb1-47ce-a16c-bc08fd4d0c09){:style="width:430px; height:404px;"}

contenthash는 파일의 내용을 기반으로 해시값을 생성하기 때문에 동일한 코드를 빌드하면 항상 동일한 해시값이 출력되야 합니다.  
**현재는 계속 변경되고 있기 때문에 [contenthash]를 통한 캐싱 이점을 전혀 누릴 수 없는 상태입니다.**

> Bundle 파일 비교

![image](https://github.com/user-attachments/assets/72d0e8ca-963a-47c2-a022-d73a370cc7ef)

Bundle 파일의 내용을 비교해보면 `/var/folders/nl/~` 하위의 폴더명이 변경되는 걸 확인할 수 있습니다.  
이 폴더의 역할은 Mac OS의 임시 파일 & 캐시 파일 저장인데 서비스에 필요없는 영역입니다.  
원인은 환경변수를 React Context에서 사용할 수 있도록 정의하는 [DefinePlugin](https://webpack.js.org/plugins/define-plugin/){:target="_blank"} 설정 이였습니다.

### DefinePlugin 설정
> 문제가 된 DefinePlugin 설정

```javascript
const envPath = DEPLOY_ENV === 'development' ? './globals/.env_dev' : './globals/.env';
require('dotenv').config({ path: envPath });

new webpack.DefinePlugin({
    'process.env': JSON.stringify({
      ...process.env,
    }),
  }),
```

dotenv로 개발 환경에 따라 필요한 환경변수를 가져온 뒤 `…process.env` 를 정의하면서 Build 시스템 자체의 환경변수까지 모두 주입한 결과입니다.  
이 부분을 다시 필요한 환경변수만 정의하도록 바꿔준 뒤 Build의 출력물을 비교해보면 해시값이 변경되지 않는 걸 확인할 수 있습니다.  

> 수정한 DefinePlugin 설정

```javascript
const envPath = DEPLOY_ENV === 'development' ? './globals/.env_dev' : './globals/.env';
const env = require('dotenv').config({ path: envPath });

new webpack.DefinePlugin({
    'process.env': JSON.stringify({
      ...env.parsed,
    }),
  }),
```
### 동일한 코드를 빌드해도 [contenthash]가 변경되지 않음
> 첫번째 Build 결과물

![image](https://github.com/user-attachments/assets/810b0a09-1027-44b1-bd28-4813e1c707e3){:style="width:430px; height:404px;"}

> 두번째 Build 결과물(코드 변경 X)

![image](https://github.com/user-attachments/assets/7b591217-0f57-4a67-b1e5-62554cc064fa){:style="width:430px; height:404px;"}


### 코드에 변경이 일어난 파일만 [contenthash] 변경 됨 
이번에는 `App.tsx` 컴포넌트의 코드만 살짝 변경해보고 빌드한 결과물을 비교해보면 변경이 일어난 Bundle 파일만 [contenthash] 값이 변경된 걸 확인할 수 있습니다.
```jsx
function App() {
    const bundleTest = 'bundleTest';
    
    // ... 생략
}
```
> 첫번째 Build 결과물

![image](https://github.com/user-attachments/assets/3077c0c8-1365-421d-a1a9-8b8ee3b9824b){:style="width:430px; height:404px;"}

> 두번째 Build 결과물(코드 변경 X)

![image](https://github.com/user-attachments/assets/b56ccc65-dd37-4c32-b13c-01e183a63665){:style="width:430px; height:404px;"}

