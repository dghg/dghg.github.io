---
layout: post
title: "CancelToken을 이용한 비동기 요청 취소"
date: 2020-12-08 00:00:00
excerpt: ""
tags:
- study
categories:
- react
---

구글, 네이버 등 웹 서비스의 검색창에 검색어를 입력하면, 입력된 검색어에 맞춰 적절한 검색어를 제시해 준다.
이는 검색어를 입력 받는 `Input`의 값이 변할 때 마다, API 요청을 통해 응답을 받아 화면에 렌더링 해주는 방식으로 구현해 볼 수 있다.
이런 방식으로 구현한 리액트 컴포넌트는 다음과 같다.

```
const SearchForm: React.FunctionComponent = ({}) => {
  const [value, setValue] = useState<string>('');
  const [results, setResults] = useState<Array<any>>([]);
  const handleChange = async (e: React.ChangeEvent<HTMLInputElement> ) => {
    setValue(e.target.value);
    const response = await axios.get(`API_URL?q=${e.target.value}`);
    if(response.status === 200) {
      setResults(response.data);
    }
  }
  return (
    <>
    <input onChange={handleChange} value={value}/>
    {results.length ? results.map((val, idx) => {return <p key={idx}>{val}</p>}) : null }
    </>
  )
}
```

또한 이런 방식을 사용하고 있는 것이 네이버와 구글이다.
![google](#https://github.com/dghg/dghg.github.io/blob/master/_posts/img/cancel1.PNG?raw=true)


하지만 이런 방식들의 경우, 다음과 같은 문제점이 있다.
- 당연히 리소스를 낭비한다. 요청을 여러개 보내며, 마지막 응답을 제외한 이전의 응답들은 쓸모가 없어진다.
- 만약 두번째 요청의 응답이 첫번째 요청의 응답보다 일찍 온다면? 이상한 데이터가 화면에 표시되는 버그가 발생할 수도 있을 것이다.

이런 문제점을 해결하기 위해, 새로운 요청이 발생하면 이전에 발생한 모든 요청을 취소할 필요가 있다.

 ### CancelToken
 `axios` 라이브러리는 요청을 취소하기 위해 `CancelToken`이라는 API를 제공해 준다.

 `CancelToken.source()` 함수를 통해 `CancelToken`을 만들 수 있다.
 이러한 `CancelToken`을 이용하면, 요청을 취소할 수 있다.
 `CancelToken`을 사용하여 구현한 리액트 컴포넌트는 다음과 같다.

```
const SearchForm: React.FunctionComponent = ({}) => {
  const [value, setValue] = useState<string>('');
  const [results, setResults] = useState<Array<any>>([]);
  const source = useRef<any>(null);


  const handleChange = async (e: React.ChangeEvent<HTMLInputElement> ) => {
    setValue(e.target.value);
    if(source.current !== null) { // already exists request
      source.current.cancel();
    }
    source.current = CancelToken.source();
    try {
      const response = await axios.get(`API_URL?q=${e.target.value}`, {cancelToken: source.current.token});
      if(response.status === 200) {
        setResults(response.data);
      }
    } catch (err) {
      console.log(err);
    }

  }
  return (
    <>
    <input onChange={handleChange} value={value}/>
    {results.length ? results.map((val, idx) => {return <p key={idx}>{val}</p>}) : null }
    </>
  )
}
```
이렇게 구현한다면, 다음과 같이 최종 응답만 받고 나머지는 *canceled* 된다.

![canceltoken](#https://github.com/dghg/dghg.github.io/blob/master/_posts/img/cancel1.PNG?raw=true)

이렇게 `CancelToken`을 활용해 불필요한 비동기 요청들을 관리한다면, 리소스 낭비들을 줄이는 데 도움이 될 수 있을 것이다. 끝 !

### Referneces

[Cancellation - Axios](#https://github.com/axios/axios#cancellation)
