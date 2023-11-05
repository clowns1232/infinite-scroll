# 옵저버 패턴을 적용한 무한 스크롤

옵저버 패턴과 [thecatapi](https://thecatapi.com/)를 사용하여 스크롤을 내릴 때 마다 고양이를 불러오는 페이지를 만들었습니다

# 간단한 동작 원리

IntersectionObserver를 사용해서 최하단에 있는 div가 화면에 나오면 이를 감지하여 새로운 사진을 불러옵니다. <br/>

순서는 아래와 같습니다.

1. 첫 렌더링이 완료된 후 옵저버 생성
2. 옵저버 Element가 화면에 감지될 경우 obsHandler() 실행
3. obsHandler() 함수가 page값 변경
4. useEffect 훅에 의해 getPost() 실행
5. 데이터 조회 API 호출
6. 데이터 렌더링 완료

# 코드 설명

## 상태값 정리

```javascript
// 고양이 사진 리스트
const [list, setList] = useState([]);

// 고양이 사진 리스트를 얼마나 추가했는지
const [page, setPage] = useState(1);

// 로딩 여부
const [load, setLoad] = useState(1);

// 업데이트가 완료되었는지 확인(중복 실행 방지)
// useRef는 업데이트가 되어도 리렌더링이 되지 않음
const preventRef = useRef(true);

// 옵저버가 관측하고 있는 대상
const obsRef = useRef(null);
```

## 첫 화면 렌더링 시 발생 이벤트

아래의 코드는 첫 렌더링 당시 동작하는 코드입니다.<br>
첫 이미지, 옵저버, 옵저버 이벤트, 감시 대상 추가

```javascript
// 처음 화면 렌더링 시
useEffect(() => {
  // 첫번째 고양이 사진을 추가한다.
  getCat();

  // 옵저버를 추가한다
  const observer = new IntersectionObserver(obsHandler, { threshold: 0.5 });

  // 감시 대상을 추가한다.
  if (obsRef.current) observer.observe(obsRef.current);
  return () => {
    // 화면에서 나갈 때 옵저버를 지운다
    observer.disconnect();
  };
}, []);

// 옵저버 이벤트
const obsHandler = (entries) => {
  const target = entries[0];

  // 관찰 대상이 관찰되고(target.isIntersecting)
  // 업데이트가 완료 되었을 때(preventRef.current)
  // 고양이 사진을 추가한다
  if (target.isIntersecting && preventRef.current) {
    preventRef.current = false;
    setPage((prev) => prev + 1);
  }
};

//글 불러오기
const getCat = useCallback(async () => {
  setLoad(true); //로딩 시작
  // 고양이 사진 불러옴
  const res = await axios({
    method: "GET",
    url: `https://api.thecatapi.com/v1/images/search`,
  });

  // 정상적으로 고양이 사진을 가져오면
  if (res.data) {
    setList((prev) => [...prev, { ...res.data[0] }]); //리스트 추가
    preventRef.current = true;
  } else {
    console.log(res); //에러
  }
  setLoad(false); //로딩 종료
}, [page]);
```

## 화면 코드

여기서 확인할 대상은 맨 아래의 옵저버 대상입니다.<br/>
해당 div가 화면에 보이는지 옵저버가 감시하여<br/>
보이면 옵저버 이벤트가 실행됩니다.

```javascript
<>
  <div className="wrap min-h-[100vh]">
    {list && (
      <>
        {list.map((li) => (
          <img
            key={li.id}
            className="opacity-100 mx-auto mb-6"
            src={li.url}
            alt={li.dke}
            width={"60%"}
            height={"50%"}
          />
        ))}
      </>
    )}
    {load && <div className="py-3 bg-blue-500 text-center">로딩 중</div>}
    <div ref={obsRef} className="py-3 bg-red-500 text-white text-center">
      옵저버 Element
    </div>
  </div>
</>
```
