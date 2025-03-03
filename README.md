# 📚 UrLink

<div align="center">
  
  ![Group 1 (3)](https://github.com/user-attachments/assets/19540f6e-1863-4ad2-a809-05097272441c)

  [각자 진행]
</div>


## 개발 동기[각자 진행]


## 기술 스택
<div align="center">
  
| 프론트엔드 | 백엔드 | 빌드 | 테스트 |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | 
| <img src="https://img.shields.io/badge/React-3B4250?style=flat-square&logo=React&logoColor=#61DAFB"/> <br /> <img src="https://img.shields.io/badge/Zustand-3B4250?style=flat-square&logo=React&logoColor=#3B4250"/> <br /> <img src="https://img.shields.io/badge/Tailwind-3B4250?style=flat-square&logo=tailwindcss&logoColor=#06B6D4"/> <br /> <img src="https://img.shields.io/badge/Axios-3B4250?style=flat-square&logo=Axios&logoColor=#5A29E4"/> | <img src="https://img.shields.io/badge/Node-3B4250?style=flat-square&logo=Node.js&logoColor=#5FA04E"/> <br /> <img src="https://img.shields.io/badge/Puppeteer-3B4250?style=flat-square&logo=Puppeteer&logoColor=#40B5A4"/> <br /> <img src="https://img.shields.io/badge/Express-3B4250?style=flat-square&logo=Express&logoColor=#646CFF"/>| <img src="https://img.shields.io/badge/Vite-3B4250?style=flat-square&logo=vite&logoColor=#646CFF"/> | <img src="https://img.shields.io/badge/Vitest-3B4250?style=flat-square&logo=vitest&logoColor=#6E9F18"/> |

</div>


## 구현 세부사항 [같이 진행]
### 어떻게 북마크를 가져올까?
프로젝트를 시작하기 위해선 프로젝트 키워드인 사용자의 크롬 북마크를 수집해야 했습니다.
북마크를 가져오기 위해 chromeAPI를 사용하기 위해 익스텐션을 선택한 것이기 떄문에, chromeAPI를 사용해 북마크를 가져오고자 했습니다.
공식문서에서 chromeAPI.getBookmark()를 사용한 예제를 참고하여 가져오고자 했으나 dom을 직접적으로 건드리는 형태임을 확인했습니다.
공식문서의 예문처럼 Dom으로 컨트롤하기엔 react에서는 지양해야할 방향성이므로 다른방법을 모색하던 와중 순수하게 
chromeAPI.getBookmark()가 어떤 data를 가져오는지 의문이 들어 자료를 꺼내 확인 해보았습니다.

``` js
  useEffect(() => {
    chrome.bookmarks.getTree((treeList) => {
      console.log(treeList);
      getNewTree(treeList);
    });
  }, []);
```
  
```chromeAPI.getBookmark()``` 로 가져온 북마크는 사용자 북마크 폴더 개수에 따라 얼마든지 중첩이 가능한 구조였습니다.

![트리구조](https://github.com/user-attachments/assets/8694307c-720f-4d15-9c86-9edfc674d02b)

이러한 트리 구조는 React상태 관리시 불변성을 지키기 힘든 이슈가 있고, 불변성을 지키기 위해서 저희가 사용할 목적에 맞는 자료구조로의 개선이 필요했습니다. 
저희는 사용자 폴더가 얼마나 트리 구조인지 알수 없기 때문에, 재귀를 사용해 얼마나 폴더가 중첩되어 있든 필요한 정보만 추출하여 리스트 저장해야 했기 때문에 재귀 함수를 작성하고자 했습니다.
```js
재귀 함수....
```

### 어떻게 키워드가 포함된 문장을 가져올까?
크롤링을 통해 HTML을 뽑고 includes로 가져왔습니다.


### 너무 느린 크롤링 속도 어떻게 해결할까?
- 서버에서는 어떻게 개선했을까
- Promise.allSetteld ->



## 의사결정 방식 [같이 진행]


## 일정
#### 1차 개발 : 2025년 1월 13일 ~ 2025년 2월 2일
> 진행 사항
> - 협업 규칙 및 Git 플로우 규칙 설정
> - 프로젝트 esLint, pretteir, husky 설정
> - 서버 Express, Puppeteer 설정
> - 크롤링[Puppeteer] 서버 개발
> - 익스텐션 검색 로직 구현
> - 익스텐션 옵션페이지 추가

### 2차 개발 : 
> 진행 사항
> - 옵션 페이지 기획 변경
> - 리팩토링 진행
> - 전역상태 설정
> - fetch Aixos로 변경

## 팀원
- 심소은 : euns127@gmail.com
- 박성훈 : seonghoon.dev@gmail.com
- 이수보 : nullzzoa@gmail.com
