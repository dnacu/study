# display/visibility/opacity

## display

| 프로퍼티값   | 키워드 설명                                                    |
| ------------ | -------------------------------------------------------------- |
| block        | block 특성을 가지는 요소(block 레벨 요소)로 지정               |
| inline       | inline 특성을 가지는 요소(inline 레벨 요소)로 지정             |
| inline-block | inline-block 특성을 가지는 요소(inline-block 레벨 요소)로 지정 |
| none         | 해당 요소를 화면에 표시하지 않는다 (공간조차 사라진다)         |

### block 레벨 요소

1. 항상 새로운 라인에서 시작한다.
2. 화면 크기 전체의 가로폭을 차지한다. ( `width: 100%;` )
3. `width, height, margin, padding` 지정 가능
4. block 레벨 요소 내에 inline 레벨 요소를 포함할 수 있다

- div
- h1~6
- p
- ol
- ul
- li
- hr
- table
- form

### inline 레벨 요소

1. 새로운 라인에서 시작하지 않으며 문장의 중간에 들어갈 수 있다. (줄바꿈 없이)
2. `content`의 너비만큼 가로폭을 차지한다.
3. `width, height, margin-top, margin-bottom` 프로퍼티를 지정할 수 없다.
   > 상하 여백은 `line-height` 프로퍼티로 지정한다.
4. inline 레벨 요소 뒤에 공백(엔터, 스페이스) 가 있으면 정의하지 않은 스페이스 4px이 자동 지정된다.
5. inline 레벨 요소 내에 block 레벨 요소를 포함할 수 없다.

- span
- a
- strong
- img
- br
- input
- select
- textarea
- button

## inline-block 레벨 요소

block, inline 레벨 요소의 특징을 모두 갖는다. inline 레벨 요소와 같이 한 줄에 표현되면서, `width, height, margin` 프로퍼티를 지정할 수 있다.

## visibility

| 프로퍼티값 키워드 | 설명                                                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| visible           | 해당 요소를 보이게 한다 (기본값)                                                                                                                          |
| hidden            | 해당 요소를 보이지 않게 한다. display: none;은 해당 요소의 공간까지 사라지게 하지만 visibility: hidden;은 해당 요소의 공간은 사라지지 않고 남아있게 된다. |
| collapse          | table 요소에 사용하며 행이나 열을 보이지 않게 한다.                                                                                                       |
| none              | table 요소의 row나 column을 보이지 않게 한다. IE, 파이어폭스에서만 동작하며 크롬에서는 hidden과 동일하게 동작한다.                                        |

## opacity

투명도
