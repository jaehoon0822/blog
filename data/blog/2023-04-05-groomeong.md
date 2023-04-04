---
title: 'Groomeong 3week 회고'
date: '2023-04-05'
tags: ['project', 'groomeong', '회고']
draft: false
summary: 'Project 진행하며 3주 동안의 회고'
---

# Groomeong 3주차 회고

> 벌써 `3week` 의 회고이다. <br/> `3week` 동안 프로젝트의 핵심인 지도상의 기능을 구현하게 되었다. <br/>
> 이를 구현하면서 느낀점을 기록해 본다.

구현사항은 다음과 같았다.

---

1. 행정동 및 행정구를 검색하여 가게 위치가 나올것
2. 검색시 해당 행정구로 이동될것
3. `side list` 에는 해당하는 가게의 목록이 나올것
4. 지도상에 가게의 `lat`, `lng` 을 사용하여, 마커로 나오게 할것
5. `side list` 의 가게를 클릭할시, 해당하는 마커가 표시될것.
6. 마커를 찍었을때 `side list` 에 해당하는 가게가 표시될것.
7. 행정구를 구분할 수 있도록, 해당 구 마다 `Polygon` 을 생성하여 경계를 나눌것
8. `Polygon` 을 클릭 및 호버시 해당 구의 이름이 나타날것
9. 선택된 마커에 `infoWindow` 를 사용하여 가게의 정보 및 이동 버튼을 생성할것
10. `Main Page` 행정구 입력한후 검색버튼을 누르면, `Map Page` 로 이동하며, 해당 구로 `fitBounds` 한다.
11. `Main Page` 에서 `행정구` 를 검색하면 하단에 `PopupBox` 가 생성되며, 해당 구의 가게를 `별점순` 으로 나타난다.
12. `PopupBox` 에서 나온 가게를 클릭시 해당 가게로 이동할것

---

해당부분을 구현하기 위해 사용할 기술 `stack` 으로 `@react-google-mpas/api` 를 사용한다.

기존의 `google map` 를 `react` 로 만들어져 사용이 좋으며, `component` 분리에도 용이할것 같아 사용하게 되었다.

> 무엇보다, 다운로드수가 다른 라이브리리들 보다 가장 높았다.

내가 이 기능들을 구현하면서 가장 먼저 생각이 든것을 각 `Component` 를 쪼개고, 쪼갠 컴포넌트의 상태값을 어떻게 연결시킬지가 가장 고민이 되었다.

각 `Component` 는 다른 `Component` 로 이루어져 있지만, 그 상태값들은 서로 공유하며 유기적으로 이어져 있다.

예를들면 `Marker Component` 가 클릭되면, `ListBox Component` 의 색상이 변하며, 클릭되었다는 점을 알려주어야 한다.

이렇게 각 떨어져 있는 `Component` 를 연결해 주기 위해서는 `Global State` 를 사용하는것이 좋다는 생각이 들어 사용할 `State` 를 정리하였다.

> 전역 상태관리는 `Recoil` 을 사용하여 구현한다.

```ts
/* google map 관련 state */
export interface IMapState {
  isLoaded: boolean
  map: google.maps.Map | null
  shop?: IAutocompleteShopsOutput | null
  codes: number[]
}

export const mapState = atom<IMapState>({
  key: 'mapState',
  dangerouslyAllowMutability: true,
  default: {
    isLoaded: false,
    map: null,
    shop: null,
    codes: [],
  },
})

/* polygon 관련 state */
export interface IPolyInfo {
  code?: number
  bounds: google.maps.LatLngBounds | null
}

export const polygonState = atom<IPolyInfo>({
  key: 'polygonState',
  dangerouslyAllowMutability: true,
  default: {
    code: undefined,
    bounds: null,
  },
})

/* search 시 저장될 검색 문자열 state*/
export const searchState = atom<string>({
  key: 'searchState',
  default: '',
})
```

위의 `state` 를 사용하여 각 상태값을 `Component` 마다 받아서 처리하게 한다.
다음의 상태값을 살펴보자.

### MapState

```ts
export interface IMapState {
  isLoaded: boolean
  map: google.maps.Map | null
  shop?: IAutocompleteShopsOutput | null
  codes: number[]
}
```

- `isLoaded` 는 `Map` 이 `Load` 되면 `true` 값을, 아니면 `false` 값을 가지는 `property` 이다.

> 이를 통해 많이 발생하는 "`google is not defined`" `Error` 를 쉽게 처리 할 수 있다.

- `map` 은 `google map` 을 통해 생성된 `Map` 을 가진다.
  이 `map` 을통해 `Map` 만이 가진 `pendTo` 같은 메서드를 사용할 목적으로 저장해둔다.

- `shop` 은 `Marker` 와 `sideList` 간의 연결을 위해 사용한다.

- `Marker` 의 `shopId` 와 `sideList` 가 가진 `sideListBox` 의 `shopId` 와 같을 경우에 동작하도록 만들 것이다.

- `codes` 는 선택된 지역구의 `code` 를 담은 배열이다.
  이 배열안에 지역구 `code` 가 있다면 해당 지역구는 활성화 된다.

### PolyInfoState

```ts
export interface IPolyInfo {
  code?: number
  bounds: google.maps.LatLngBounds | null
}
```

- `Polygon` 관련 `state` 를 모아놓은것이다. <br/>
  `Polygon` 은 각 지역구의 공간을 만들어주므로, 해당 지역구의 `code` 가 필요하다.

- `Polygon` 을 그리기 위해서는 해당 좌표의 목록이 필요하다. 이러한 목록을 하나의 공간으로 만들어 준 `bounds` 를 저장한다.

> `bounds` 를 만들어주는 `logic` 은 다음과 같다.

```ts
export const getLatLngBounds = (map: IDataProps) => {
  const bounds = new google.maps.LatLngBounds() // LatLngBounds 객체를 생성한다.
  map.geometry.forEach((coord) => {
    bounds.extend(coord) // coord 를 LatLngBounds 에 extend 한다.
  })
  return bounds // extend 된 bounds 를 반환한다.
}
```

위처럼 `google.maps.LatLngBounds` 를 통해 만들어진 `instance` 를 사용하여, <br/>
`coord` 를 하나로 묶은 `bounds` 생성한다.

이 `bounds` 는 매우 유용하게 사용되는데, <br/>
`polygon` 생성시 `paths` 를 통해 전달되어 공간을 그려주는 역할을 해준다.

또한, `polygon` 에 `fit` 하게 좌표이동이 되도록 할때, `google.maps.Map` 에 `fitBounds` 를 사용하여 `bounds` 를 전달하면, 해당 좌표로 딱 맞게 이동된다.

이는 `구현사항중 2번` 을 구현할때 매우 유용하게 사용될 값이다.

### `bounds` 예기치 못한상황발생

> 이 부분은 조금더 `refectoring` 이 진행되어야 한다. <br/>
> 실재 `code` 구현시, 생각과는 다르게 `boudns` 값을 가져올 방도가 생각나지 않았다. <br/>

상황은 이렇다. <br/>
`search` 시 해당 검색결과를 가져옴과 동시에 `fitBounds` 를 통해 해당 지역 `Polygon` 을 받아 `bounds` 를 넘겨줄 생각이었다.

하지만 예상과는 다르게, `Polygon Component` 에서 `bounds` 를 담을 `Trigger` 가 `onClick` 및 `hover` 밖에 존재하지 않았다.

이를 해결하기 위해 `useEffect` 를 사용해 `isActive` 라는 전역변수를 생성한후 `isActive` 시에 `bounds` 값을 `GlobalState` 에 넣으려 했었다.

이는 매 렌더링마다 평가가 이루어지므로 좋은 방식이 아닐뿐더러, 실제로 `무한순회` 되는 상황이 발생했다.

위의 `bounds` 를 처리 하기 위한 생각한 방법으로는, `component` 가 `mount` 될때, `code` 를 식별자로 하는 `bounds` 배열을 만드는 것인데, 사실 그렇게 좋은 방법은 아닌듯 하여 다른 방법으로 구현했다.

이를 해결하기 위해 따로 `shops` 를 `fetch` 한후 새로운 `bounds` 를 만들어 `fitBounds` 할수 밖에 없었다.

이러한 사항은 더 많이 생각하며 처리할 필요성을 느낀다.
현재로써는 위 방법말고는 마땅한 해결책이 생각나지 않았다.

### SearchState

```ts
export const searchState = atom<string>({
  key: 'searchState',
  default: '',
})
```

`searchState` 는 매우 간단하다. <br/>
그저, 검색 문자열을 받아, 떨어져 있는 `Component` 에서 사용가능할 수 있도록 만들어진 `state` 이다.

이는 `Map Page`, `Main Page` 의 검색처리를 하기에 매우 좋은 방법이라고 생각이 들었다.

이렇게 만들어진 `state` 들을 통해 각 `Component` 간의 연결을 사용하여 만든 것은 다음과 같다.

![map gif](/images/2023/04/map.gif)

각 지역구마다 `geoJson` 을 사용하여 각 경게를 나누어 처리했으며, 각 `state` 를 연계하여 각 컴포넌트마다 유기적으로 처리할 수 있도록 구현되었다.

### 그래서 느낀것은?

사실 작업하면서, `Component` 를 지속적으로 쪼개는것이 과연 옳은일인가? 하는 생각이 들었다.

코드 작업시 어떠한 `trigger` 가 있을때만, `state` 저장이 되도록 로직이 구성이 되드라..

> ex) onClick event 같은...

하지만, `trigger` 가 발생하지 않은 시점에도 해당 `data` 가 필요하게 되는 상황이 지속적으로 발생하게 되었고, 그러한 처리를 위해 어쩔수 없이 `data` 를 재가공하는 `함수` 를 작성할 수 밖에 없었다.

위 상황에서 만약 `component` 를 쪼개지 않고 만들었다면, 간단하게 `props` 를 넘기기만해도 해결되었을 상황들이 있어 보였다.

아직, `Component` 를 쪼개고 `State` 를 처리하는데 부족함이 많아서 그런것이겠지만, <br/>

한가지 확실한것은, 과하면 좋지 않을것 같다는 것이다.
이러한 부분들 역시 고려하기 위해서는 `Component` 작성시 고려될 여러사항에 대해 예측되어야 가능할것이다.

그렇게 예측가능한 부분까지 올라가려면 역시...

`Component` 를 쪼개고 맛보고 즐겨봐야지!!
