---
title: CSS 말줄임
date: 2021-03-27 19:03:37
category: css
thumbnail: { thumbnailSrc }
draft: false
---

> 말줄임은 UX에서 중요한 키워드이다. 브릿지 컨텐츠임을 암시하는 컨텐츠일 수 있고, 엔드페이지가 어딘가 존재하여 본문내용을 전부 확인할 수 있다는 뜻을 의미하기도 한다. CSS, JS를 이용한 말줄임이 가능하다.

## CSS 말줄임
CSS 말줄임은 기획/디자인적인 협의만 거치면 가장 **유지보수**, **확장성** 면에서 좋은 선택지다. 사실 반드시 JS 말줄임을 해야 하는 UX라면, 좋은 UX인지 다시한 번 생각해봐야 한다고 강하게 주장하고싶다. '말줄임'의 의도는 예쁜 UI가 아닌, UX가 우선되는 기획적 사고를 표현하는 수단으로서 의미가 더욱 크다고 생각한다.

렌더링 된 박스모델이 [BFC](https://developer.mozilla.org/ko/docs/Web/Guide/CSS/Block_formatting_context)를 형성한 요소에서 가능하다는것을 전제로 한다. 
#### [1줄 말줄임]

```css
.content {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
```

#### [2줄 이상 말줄임]

```css

.content {
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical; 
}
```

실무에서는 한 애플리케이션 내 N줄 이상의 말줄임이 잦을 수 있다.  
특히 모노레포 구조에서는 스타일에 대한 모듈화는 더더욱 필수적이며, 대표적으로 `scss` 환경의 `mixin` 문법을 사용해 모듈화 한다.

```scss
@mixin ellipsis_multi($line_clamp, $line_height) {
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: $line_clamp;
  -webkit-box-orient: vertical; 
  height: $line_height * $line_clamp;
}

$content_line_clamp: 2;
$content_line_heigt: 14px;
.content {
  /* other css property */
  @include ellipsis_muti($content_line_clamp, $content_line_height)
  max-height: $content_line_clamp * $content_line_height; /* for IE */
}

```

## JS 말줄임
위에서 주장한것처럼, CSS 말줄임으로 협의가 불가능한 경우일때만 사용하며, 다양한 라이브러리도 많고 바닐라로 구현할 수 있다. IE에서 CSS말줄임이 안되기 때문에(엄밀히 말하면 `before`, `after`를 이용해 억지로 구현은 가능하다) 용도에 맞게 쓰면된다. 반응형 대응시 다소 까다로울 수 있다.

```javascript
function truncateString(str, num) {
  if (str.length > num) {
    return str.slice(0, num) + "...";
  } else {
    return str;
  }
}

truncateString('12345678910111213', 10); // 1234567891...
```


## Reference
- https://developer.mozilla.org/ko/docs/Web/Guide/CSS/Block_formatting_context