# background

## background-image

## background-repeat

설정된 이미지의 크기가 화면보다 작으면 자동으로 이미지가 반복 출력된다. (`background-repeat`의 기본값이 `repeat`이다.)  
`background-repeat: no-repeat;` 으로 반복 출력을 멈출 수 있다.

`background-repeat: no-repeat, repeat;` 처럼 x, y축 다르게 설정할 수 있다.

## background-size

- corver: 배경이미지의 크기 비율을 유지한 상태에서, 부모요소의 width, height 중 큰 값에 배경이미지를 맞춘다.
  > 이미지의 일부가 보이지 않을 수 있지만 깨지지않은 사진으로 화면을 꽉 채울 수 있다.
- contain: 배경이미지의 크기 비율을 유지한 상태에서 부모 요소의 영역에 배경이미지가 보이지 않는 부분없이 전체가 들어갈 수 있도록 이미지 스케일을 조정한다.
  > 이미지의 모든 부분이 보이나, 부모요소에 빈 공간이 생긴다.

## background-attachment

화면이 스크롤되더라도 배경이미지는 스크롤되지 않고 고정되어 있게 하려면 background-attachment 프로퍼티에 fixed 키워드를 지정한다.

> parallax view를 만들 때 사용한다.

## background-position

이미지의 x, y좌표를 설정할 수 있다.

## background-color

요소의 배경색을 지정한다. `transparent` 설정 가능.

## shorthand

```css
background: color || image || repeat || attachment || position;
```
