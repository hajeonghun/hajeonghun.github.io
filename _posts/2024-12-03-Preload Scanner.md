---
title: Preload Scanner와 Image Lazy Loading
date: 2024-12-03 16:30:00 +09:00
last_modified_at: 2024-12-03 16:30:00 +09:00
categories: [Browser]
tags:
  [
    Preload Scanner,
    Lazy Loading,
    Browser
  ]
---

## Preload Scanner
### Preload Scanner란?

`Preload Scanner`는 웹 브라우저가 웹 페이지를 로드할 때 성능을 최적화하기 위해 사용하는 중요한 구성 요소 중 하나입니다. `HTML 파서`와 협력하여 페이지 로딩을 가속화하는 역할을 합니다.

### HTML 파서 vs Preload Scanner

- **HTML 파서**: HTML 파서는 HTML 문서를 읽으면서 DOM 트리를 생성하고, 요소들을 순차적으로 렌더링합니다. 하지만, HTML 파서는 일반적으로 스크립트(`<script>`) 태그를 만나면 제어권을 JavaScript 엔진으로 넘긴 뒤 해당 스크립트를 실행한 후에야 파싱을 계속합니다. 이로 인해 페이지 로딩이 지연될 수 있습니다.
- **Preload Scanner**: Preload Scanner는 HTML 파서와 동시에 동작하는 비동기적인 프로세스입니다. HTML 파서가 DOM을 생성하는 동안, Preload Scanner는 파싱된 HTML 문서의 나머지 부분을 미리 훑어보면서 중요한 리소스(예: CSS, JavaScript, 이미지 등)에 대한 로드를 미리 시작할 수 있습니다.

| ![일반 <script src=””> 동작방식 (단 defer, async 스크립트 동작방식은 이와 다릅니다.)](/assets/img/posts/2024-12-03/PreloadScanner(1).png) |
|:-------------------------------------------------------------------------------------------------------------------:|
|                           *일반 <script src=””> 동작방식 (단, defer, async 스크립트 동작방식은 이와 다릅니다.)*                           |

| ![기본적인 형태의 HTML 마크업](/assets/img/posts/2024-12-03/PreloadScanner(2).png) |
|:------------------------------------------------------------------------:|
|                           *기본적인 형태의 HTML 마크업*                            |

| ![HTML Parser](/assets/img/posts/2024-12-03/PreloadScanner(3).png) |
|:------------------------------------------------------------------:|
|                           *HTML Parser*                            |

| ![Preload Scanner](/assets/img/posts/2024-12-03/PreloadScanner(4).png) |
|:----------------------------------------------------------------------:|
|                           *Preload Scanner*                            |

### Preload Scanner의 역할

1. **`리소스 발견 및 미리 로드`**: Preload Scanner는 아직 파서가 해당 부분에 도달하지 않았더라도 HTML 문서에서 중요한 리소스를 미리 찾아냅니다. 그런 다음, 브라우저는 해당 리소스(CSS, JS, 이미지 등)를 다운로드하기 시작하여 렌더링에 필요한 파일이 빨리 준비되도록 합니다.
2. **`렌더링 지연 최소화`**: Preload Scanner는 스크립트 태그(`<script>`)와 같은 로딩을 지연시킬 수 있는 요소를 분석하여, 파서가 멈추기 전에 필요한 리소스를 미리 가져오도록 합니다. 이를 통해, 파서가 스크립트를 처리하는 동안에도 브라우저는 필요한 리소스를 미리 준비하여 렌더링 지연을 최소화할 수 있습니다.
3. **`성능 최적화`**: Preload Scanner는 브라우저가 리소스를 미리 다운로드할 수 있게 하여 전체 페이지 로드 시간을 줄이고, 사용자 경험을 개선합니다. 특히, 중요한 스타일시트나 스크립트 파일이 필요할 때, 이러한 리소스들이 이미 다운로드되어 준비되어 있으므로 렌더링이 더 빠르게 이루어집니다.


> 💡 요약: Preload Scanner는 HTML 파서와 함께 동작하여 중요한 리소스를 미리 스캔하고 로드하는 기능을 담당합니다.   
> 이를 통해 페이지 로드 성능을 최적화하고, 사용자가 페이지를 더 빨리 볼 수 있도록 도와줍니다.


## 이미지 지연 로딩 (Image Lazy Loading)

### 지연 로딩(Lazy Loading) 이란?
웹페이지에서 이미지를 필요할 때만 로드하는 기법으로, 페이지 성능과 사용자 경험을 개선하기 위해 사용됩니다.  
기본적으로 브라우저는 페이지가 로드될 때 모든 이미지를 한 번에 다운로드하지만, 지연 로딩을 사용하면 사용자가 해당 이미지가 보이는 위치로 스크롤할 때만 이미지를 로드합니다.

### 구현의 용이성 및 성능 개선 효과

지연 로딩을 구현하는 것은 비교적 적은 노력으로도 상당한 성능 개선 효과를 볼 수 있는 기술입니다.  
특히 페이지의 초기 로드 시기 동안 성능과 사용자 경험을 향상시킬 수 있습니다.

### 동작 예시

| ![Lazy Loading이 적용된 동작화면](https://github.com/user-attachments/assets/34ceba57-e855-430b-9018-67130967a8fb) |
|:----------------------------------------------------------------------------------------------------------:|
|                                          *Lazy Loading이 적용된 동작화면*                                          |


### Network 요청

- 기존 Network 요청 수 : `222` / `260`  (`image request` / `total request` )

  ![image.png](/assets/img/posts/2024-12-03/LazyLoading(1).png)


- 지연로딩 적용 후 Network 요청 수 : `18` / `60` (`image request` / `total request` )

  ![image.png](/assets/img/posts/2024-12-03/LazyLoading(2).png)


### 적용 방법

- `<img>` 엘리먼트의 경우 `loading` 속성을 통해 브라우저에 로드 방법을 알려줄 수 있습니다.
  - `lazy` 옵션을 설정할 경우, 뷰포트(viewport) 일정 범위내에 이미지가 들어올때까지 이미지 로드를 연기합니다.
    (범위는 브라우저마자 조금씩 다를 수 있지만 스크롤 할 때 이미지 로드에 문제가 없을만큼 설정되어 있습니다)
  - `eager` 옵션을 설정할 경우, loading 속성의 default 값으로 이미지가 뷰포트(viewport) 외부에 있더라도 즉시 로드 됩니다.
- `background-image` 등을 통한 css 속성으로 이미지를 로드하는 경우에는, `JavaScript`를 통해 지연로딩을 구현해야 합니다.
  - [데이터 속성(data-\*)](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-\*)에 image url을 설정해주고 [Intersection Observer API](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API) 를 사용하여 엘리먼트가 뷰포트(viewport) 범위에 들어오는 경우 데이터 속성에 있는 url을 실제 style backogrund-image로 설정해줍니다.

```tsx
// <img> 엘리먼트의 경우...
<img loading="lazy" src="image.jpg" alt="..." />

/************************* *************************/

  // css속성 background-image를 사용하는 경우...
const profileObserverRef = useIntersect(lazyImageLoadIntersect, {
    root: null,
    rootMargin: '0px',
    threshold: 0.1,
  });

function lazyImageLoadIntersect(entry: IntersectionObserverEntry, observer: IntersectionObserver) {
  const div = entry.target as HTMLElement;

  observer.unobserve(div);

  div.style.backgroundImage = div.dataset.bg;
}

<div ref={profileObserverRef} data-bg="url(~~/profile.png)" />
```

### Lazy Loading 적용 시 얻는 이점

1. **`초기 페이지 로드 시간 단축`**
   - 페이지가 로드될 때, 이미지를 즉시 로드하지 않고 사용자가 스크롤 하여 해당 이미지가 뷰포트(viewport)에 나타나기 직전에 로드하게 됩니다.   
   - 이를 통해 초기 페이지 로드 시 필요한 네트워크 리소스와 브라우저의 렌더링 작업을 줄일 수 있습니다. 결과적으로 페이지가 더 빨리 사용자에게 표시될 수 있습니다.
2. **`네트워크 대역폭 절약`**
   - 사용자가 페이지의 일부만 보고 떠날 경우, 보이지 않는 이미지들은 로드되지 않기 때문에 불필요한 네트워크 대역폭을 절약할 수 있습니다.
   - 네트워크 대역폭은 한정된 자원입니다. 한 번에 너무 많은 리소스를 요청하면 네트워크에 경합과정이 생기며 전체 페이지 로드가 느려질 수 있습니다. 불필요한 이미지 로드를 지연시키면, 네트워크 자원을 더 중요한 초기 리소스(예: HTML, CSS, JavaScript) 로드에 집중시킬 수 있습니다. 이로 인해 필요한 리소스들이 더 빨리 다운로드되고, 페이지가 더 빨리 렌더링 될 수 있습니다
   - 네트워크 연결 상태가 좋지 않은 경우 페이지의 LCP(Largest Contentful Paint)를 개선할 수 있으며, 대역폭을 재할당하면 LCP 후보자가 더 빠르게 로드하고 페인팅하는 데 도움이 될 수 있습니다.
3. `브라우저 메모리 사용 최적화`
   - 이미지가 즉시 로드되지 않기 때문에 브라우저의 메모리 사용량을 최적화할 수 있습니다. 이는 브라우저가 한꺼번에 많은 이미지들을 처리할 필요가 없어 메모리 사용량이 줄어들고, 브라우저의 성능이 향상됩니다.
4. **`모바일 사용성 향상`**
   - 모바일 환경에서는 데이터 사용량이 중요한 요소입니다. Lazy Loading을 통해 데이터 사용량을 줄여, 모바일 사용자의 요금 부담을 낮추고, 더 나은 모바일 사용성을 제공합니다.

> 💡 요약: 이미지가 중요 렌더링 경로(CRP) 차단 리소스는 아니지만, 네트워크 자원을 절약하며 중요 렌더링 경로의 리소스들을 빨리 로드하고 처리해서 초기 페이지 로딩 시간이 단축에 도움을 줄 수 있습니다.

### 모든 이미지에 적용한다면?

> 🚫 **초기 뷰포트(viewport)에 있는 이미지를 지연 로딩하면 안됩니다.**  
> (대표적인 안티패턴으로 잠재적으로 **LCP 이미지를 지연**시킬 수 있습니다.)

지연로딩은 초기 렌더링 시 뷰포트(viewport) 범위 외부에 있는 요소들에만 적용해야 합니다.

현대에서 웹사이트는 다양한 디바이스(PC, 태블릿, 모바일 등)에서 접근이 가능하기 때문에 각 장치의 뷰포트(viewport) 범위 및 종횡비를 전부 고려하기란 사실 쉬운일이 아닙니다.

동일한 모델의 태블릿을 사용하더라도 누군가는 가로방향으로 사용하고, 누군가는 세로방향으로 사용하면 뷰포트의 수평, 수직공간에 노출되는 요소가 달라지기 때문입니다.

하지만, 확실히 사용하면 안된다고 말할 수 있는 요소들이 존재합니다. 예를 들어, 디바이스의 종류나 종횡비에 상관없이 `페이지 상단에 노출되는 요소` , `Hero Image` , `LCP 후보`가 될 수 있는 요소들이 있습니다.

지연로딩이 걸려있지 않은 요소의 경우, 중요 렌더링 경로(CRP) 중 `HTML 파싱단계`에서 보조 Parser라고 할 수 있는 `Preload Scanner`에 의해 빠르게 발견되어 리소스를 빠르게 `fetch` 합니다.

반면 요소에 지연로딩이 걸려있는 경우, 이미지의 최종 위치가 사용자의 뷰포트(viewport) 범위 내에 있는지 파악하기 위해 중요 렌더링 경로(CRP)중 `레이아웃 단계`를 마칠때 까지 기다린 이후 범위내에 있다면 그제서야 `fetch` 하기 시작합니다.

이는 `Preload Scanner` 의 이점을 전혀 누릴 수 없어서, 중요 이미지인데도 불구하고 늦게 `fetch` 되어 사용자 경험에 부정적인 영향을 미칠 수 있습니다.

---

> *Hero Image - 웹사이트에서 화면을 가득 채우는 큰 배너 이미지. 일반적으로 앞이나 가운데에 눈에 확 들어오도록 배치되기 때문에 방문자의 이목을 집중시킨다.*  
> *LCP(Largest Contentful Paint) - 사용자가 페이지를 처음 탐색한 시점을 기준으로 뷰포트에 표시되는 가장 큰 이미지, 텍스트 블록 또는 비디오의 렌더링 시간*

## 자문자답

`Question`  
🤷: css 파일 내에서 다른 css 파일을 불러올(@import) 경우에도 Preload Scanner에 의해 발견되어지나요?

<code class="language-plaintext highlighter-rouge" style="color: red;">Answer</code>   
🙎: 결론만 먼저 말씀드리자면 아닙니다. Preload Scanner는 오직 서버에서 응답받은 정적 HTML만 스캔할 수 있습니다.

`Question`  
🤷: React 컴포넌트에 있는 요소들은 발견할 수 있나요?

<code class="language-plaintext highlighter-rouge" style="color: red;">Answer</code>  
🙎: 아닙니다. CSR방식을 사용하는 경우에는 발견하지 못합니다. 보통 React를 통한 CSR방식에서는 빈 HTML구조에 “root” 같은 ID를 가진 엘리먼트를 주입하고 번들된 js파일을 load 하고 실행한 뒤 해당 엘리먼트의 하위 요소로 렌더링 하게 되는데요. 이 동작방식으로 인해 초기 서버에서 응답받은 HTML에서 Preload Scanner가 미리 발견하고 로드할 요소가 없습니다.

`Question`  
🤷: 이럴때는 그럼 어떤 방법으로 Preload Scanner의 이점을 누릴 수 있을까요?

<code class="language-plaintext highlighter-rouge" style="color: red;">Answer</code>  
🙎: 번들된 js를 주입하는 Template HTML에 미리 해당 요소들을 배치하는 방안도 있으며, preload 를 통해 필요한 리소스를 미리 로드할 수 있습니다. 그리고 페이지의 마크업이 꼭 클라이언트에서 렌더링 되야하는 이유가 있는게 아니라면 SSR 방식으로 마크업을 제공하는 방식을 고려할 수 있습니다.
