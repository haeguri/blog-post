---
title: CSS Box Model
---

## 소개

브라우저는 HTML 문서의 엘리먼트들을 화면에 그리기 위해 각 엘리먼트를 박스(box)로 표현한다. 이 때 모든 HTML 엘리먼트를 둘러싸고 있는 사각형 박스를 **CSS 박스 모델(CSS Box Model)**이라고 한다. [CSS 2.1 스펙](https://www.w3.org/TR/CSS2/box.html)에 의하면 박스 모델을 다음처럼 설명하고 있다.

> *CSS 박스 모델은 문서 트리(document tree)안의 엘리먼트들에 대해 만들어지고, 시각적 형식 모델(visual formatting model)에 따라서 배치되는 사각형 상자를 말한다.* 

하나의 박스는 컨텐츠 영역(content area), 패딩 영역(padding area), 테두리 영역(border area), 마진 영역(margin area)으로 구성된다. 네 가지의 영역이 박스를 어떻게 구성하는지는 아래의 그림을 통해 쉽게 파악해볼 수 있다.

{% img /images/css-box-model/css-box-model.png 300 238 %}

## 컨텐츠 영역

컨텐츠 영역에는 텍스트나 이미지 등 엘리먼트의 실제 내용이 있다. 컨텐츠 영역의 크기를 명시적으로 정해주는 것은 `box-sizing` 속성이 기본 값인 `content-box`으로 되어 있을 때만 가능하다. `box-sizing`이 기본 값으로 설정되어 있으면, `width`, `height`, `min-width`, `min-height`, `max-width`, `max-height`과 같은 크기를 지정하는 CSS 속성들은 컨텐츠 영역의 크기만 변경하게 된다.

그런데 `width`, `height`과 같은 속성들은 주로 컨텐츠 영역의 크기를 변경하려고 하는 것이 아니라 엘리먼트(박스)의 크기를 지정하려고 사용하는 경우가 대부분일 것이다.

예를 들어서, `padding`, `border` 속성이 부여된 엘리먼트의 크기를 너비 200px, 높이 200px으로 하고 싶다면 다음처럼 소스를 작성하려고 할 수도 있다.

{% jsfiddle dwv5tkmh/25/ html,result %}

이 엘리먼트가 실제 화면에 보여지는 크기를 개발자 도구에서 확인해보면 다음과 같다. 

{% img /images/css-box-model/calculated-box-size.png 300 200 %}

엘리먼트의 너비는 바깥에서부터 `border`, `padding`, `width`를 더해서 240px으로 계산되고, 엘리먼트의 높이도 `border`, `padding`, `height`를 더해서 240px으로 계산된다.

이렇게 계산된 이유는 CSS 코드에서 `box-sizing` 속성으로 어떤 값도 주어지지 않아서 기본 값인 `content-box`로 설정됐기 때문이다. `box-sizing:content-box`로 설정된 엘리먼트는 `width`, `height` 속성을 적용해도 `padding`과 `border`는 포함하지 않고 **컨텐츠 영역**의 크기만을 지정한다.

## 패딩(padding) 영역

패딩 영역은 컨텐츠 영역을 감싸면서 엘리먼트의 패딩을 포함하는 투명한 영역이다. 패딩의 두께는 `padding-top`, `padding-right`, `padding-bottom`, `padding-left` 속성과 네 가지 속성을 축약한 버전인 `padding`으로 결정된다.

## 테두리(border) 영역

테두리 영역은 패딩 영역을 감싸면서 엘리먼트의 테두리를 포함하는 영역이며, 테두리의 두께는 `border-width`, `border`으로 결정된다. 

`box-sizing` 속성 값 중에는 `border-box`가 있는데, `box-sizing: border-box;`를 하게 되면 `width`, `height` 같은 엘리먼트(박스)의 크기를 변경하는 CSS 속성들이 **'border-box'**의 크기를 변경하게 한다. **'border-box'**는 컨텐츠, 패딩, 테두리 영역를 포함하는 박스를 말한다.

이제 앞서 살펴본 예제에서 `box-sizing: border-box;` 코드만을 추가한 버전을 살펴보자.

{% jsfiddle dwv5tkmh/26/ html,result %}

개발자 도구를 통해서 엘리먼트의 크기를 살펴보면 아래와 같다.

{% img /images/css-box-model/calculated-box-size-2.png 300 200 %}

`box-sizing`이 `border-box`로 되어 있기 때문에 `width`, `height`가 **'border-box'**의 크기를 변경했다. 그리고 컨텐츠 영역의 크기는 `border`, `padding`을 제외한 크기로 계산됐다.

## 마진(margin) 영역

마진 영역은 테두리 영역을 감싸면서 이웃 엘리먼트로부터 자신을 구분하기 위해 비어있는 공간인 마진을 포함하는 영역이다. 마진의 두께는 `margin-left`, `margin-right`, `margin-bottom`, `margin-left`와 네 가지를 축약한 버전인 `margin`으로 결정된다.

## 참고 자료

- MDN Box Model : https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Box_model
- MDN Box Model Introduction : https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model
- W3C CSS 2.1 Box Model : https://www.w3.org/TR/CSS2/box.html
- W3C CSS Box Model Level 3 : https://www.w3.org/TR/css-box-3/#box-model

