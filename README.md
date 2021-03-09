# 딤드 레이어

자주 사용하는 딤드 레이어(modal) 를 만들면서 고민하였던 부분들에 대해서 개인적으로 삽질한 내용을 정리해 봤습니다.

## translate 기법 vs inline-block 기법

#### translate 기법 ([here](https://git.linecorp.com/pages/joontop/layer2/1.html))

top, left 는 화면 기준, translateXY 는 모달 사이즈 기준

```css
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 300px;
  height: 300px;
}
```

이렇게 하면 300x300 짜리 박스를 화면 중앙에 위치 시킬수 있다.

##### 문제점

화면을 줄였을때도 화면과 모달이 기준이 되기 때문에 잘리는걸 볼수 있다.

#### inline-block 기법 ([here](https://git.linecorp.com/pages/joontop/layer2/2.html))

inline 속성을 가지는 요소들은 vertical-align:middle 로 중앙정렬을 시킬수 있다. after 넣은 요소를 박스의 100% 로 차지하게 만들어서 modal을 중앙에 위치시키는 방법이다.

```css
.modal_wrap {
  height: 100%;
}
.modal {
  display: inline-block;
  vertical-align: middle;
}
.modal_wrap:after {
  display: inline-block;
  vertical-align: middle;
  height: 100%;
  content: '';
}
```

이슈방지를 위해 넣어줘야하는 속성들이 있다.

#####

```css
.modal_wrap {
  ...
  white-space:nowrap; /* 컨텐츠가 화면을 넘치면 아래로 떨어지는 inline 성질 막기 */
  font-size:0; /* 모달과 after 사이에 여백을 제거 */
}
.modal {
  ...
  max-width:100%; /* 같은 텍스트가 왔을때 박스가 화면을 넘칠수도 있으므로 */
  font-size: xpx; /* font-size:0 은 상속이 되므로 다시 설정 */
}
```

딱히 얘기하지 않아도 디자인을 위해 modal 에 margin 을 좀 추가해준다.

##### 문제점

딱히 없다. 아직까진 이걸 젤 많이 활용한다.

## scroll 에 대한 고민

딤드 레이어를 열었을때 하딘에 비치는 일반 컨텐츠의 스크롤 (html) 을 동작하지 않게 만들어달라는 요구가 많다. 일반적으로 당연한것처럼 사용되어 왔다. 상황에 따라 사용했던 방법이 달랐다.

#### 지원 환경이 모바일이고 내부스크롤이 없을때 ([here](https://git.linecorp.com/pages/joontop/layer2/3.html))

레이어를 열었을때 document 의 touchmove event 를 막는다.

```javascript
document.addEventListener(
  'touchmove',
  function (e) {
    e.preventDefault();
  },
  { passive: false }
);
```

레이어를 닫았을때 다시 removeEventListener 해준다.

##### 문제점

- 고정픽셀 레이어 안에 mousemove 가 필요한 요소가 있을 경우

#### 내부스크롤이 없고 PC도 지원할때

wheel 이벤트도 막아버린다.

```javascript
document.addEventListener(
  'wheel',
  function (e) {
    e.preventDefault();
  },
  { passive: false }
);
```

##### 문제점

- 스크롤 막대를 직접 클릭해서 끌었을때는 막지 못한다.

#### 모바일, PC 지원이고 유동픽셀이고 컨텐츠가 길때 ([here](https://git.linecorp.com/pages/joontop/layer2/4.html))

레이어를 열었을때 html은 overflow:hidden, body는 가둬줄 영역으로 만든다.

```css
html {
  overflow: hidden;
}
body {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
```

컨텐츠의 현재 스크롤 위치를 가져와서 contents 의 translateY 로 넣어준다. (html에 속성을 넣으면 현재 스크롤 위치가 top 0 으로 이동하기 때문)

```javascript
var scrollTopPx = document.documentElement.scrollTop;
...('body').style.transform = `translateY(-${scrollTopPx}px)`;
```

레이어를 닫았을때 기억하고 있던 scrollTop 값을 다시 설정해 준다.

```javascript
window.scrollTo(0, scrollTopPx);
```
