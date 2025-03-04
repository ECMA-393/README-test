# 📚 UrLink

<div align="center">
  
  ![Group 1 (3)](https://github.com/user-attachments/assets/19540f6e-1863-4ad2-a809-05097272441c)

  URLink는 북마크 제목만으로는 파악하기 어려운 내부에 키워드를 검색하여 필요한 정보를 찾아주는 서비스입니다.
</div>


## 개발 동기
일상적으로 인터넷을 사용하다 보면, 저장해둔 북마크가 어느새 쌓여가는 경험을 하게 됩니다. 그러다 보니 북마크의 제목만으로는 실제 페이지의 내용을 파악하기 어려워, 필요한 정보를 찾기 위해 다시 방문해야 하는 불편함이 생겼습니다. 더불어 기존의 서비스들은 주로 북마크 제목을 기반으로 검색 기능을 제공했기에, 세세한 내용 검색에는 한계가 있었습니다.

이러한 문제점을 자연스럽게 해결하고자, 사용자가 별도의 로그인 없이도 크롬 브라우저에 저장된 북마크를 chromeAPI.bookmarks로 불러와, 각 북마크 내부의 키워드까지 부드럽게 검색할 수 있는 익스텐션을 개발하게 되었습니다.

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
  
```chrome.bookmarks.get``` 로 가져온 북마크는 사용자 북마크 폴더 개수에 따라 얼마든지 중첩이 가능한 구조였습니다.

![트리구조](https://github.com/user-attachments/assets/8694307c-720f-4d15-9c86-9edfc674d02b)

이러한 트리 구조는 React상태 관리시 불변성을 지키기 힘든 이슈가 있고, 불변성을 지키기 위해서 저희가 사용할 목적에 맞는 자료구조로의 개선이 필요했습니다. 
저희는 사용자 폴더가 얼마나 트리 구조인지 알수 없기 때문에, 재귀를 사용해 얼마나 폴더가 중첩되어 있든 필요한 정보만 추출하여 리스트 저장해야 했기 때문에 재귀 함수를 작성하고자 했습니다.<br />
재귀는 함수를 지속적으로 호출하면서, 콜 스택에 중첩되며 메모리를 많이 차지하게 되고 이는 속도 저하를 야기할 수 있었습니다.

재귀 함수의 초기 버전은 forEach와 반복 조건 등 가독성 및 성능에서 떨어진다고 판단했고, 아래와 같이 조건을 중첩하고 불필요한 조건을 줄여 가독성과 성능을 향상 시켰으며, ```forEach```를 ```for...of```로 변경해 더욱 간결하고 중간에 반복을 종료할 수 있도록 리팩토링 했습니다.

```js
// 리팩토링 버전
const getNewTree = (nodeItems) => {
    const getNewTreeResult = [];

    function recursive(nodes) {
        for (const node of nodes) {
            if (node && typeof node === "object") {
                if (node.title && node.url) {
                    getNewTreeResult.push(node);
                }
                if (node.children) {
                    recursive(node.children);
                }
            }
        }
    }

    recursive(nodeItems);
    setUrlNewList(getNewTreeResult);
};
```

### 어떻게 키워드가 포함된 문장을 가져올까?
저희가 맨 처음 생각한 방법은 fetch를 통해 html을 받아온 후 그 html에서 일치하는 내용을 찾는 것이었습니다. 많은 웹이 SPA로 작성되어 있거나 네이버 블로그들은 보통 iframe으로 작성된 부분이 많이 있었기 때문에 해당 문제를 크롤링으로 해결해보고자 하였습니다.

Puppeteer의 헤드리스 브라우저 모드를 사용하여 인간적인 브라우징 패턴을 모방하였고, 요청 헤더를 조작하여 SPA페이지 로딩을 기다리고 iframe 내부 내용또한 읽어와 해당 컨텐츠 내에서 키워드를 검색할 수 있었습니다.

### 너무 느린 크롤링 속도 어떻게 해결할까?
- 서버에서는 어떻게 개선했을까
- Promise.allSetteld ->


### Docker Container Out of Memory [OOM] 문제
크롤링 서버를 AWS의 EC2를 사용하여 배포하기로 했고, EC2의 프리티어 계정을 사용하여 배포했습니다.
프리티어의 경우 EC2로 받은 인스턴스의 램 메모리가 1GB라는 제한이 있었고 Promise.allSetteld를 통해 크롤링을 여러번 요청하면 해당 많은 메모리를 차지해 OOM을 이기하며 서버가 다운되는 현상이 발견되었습니다.

### 크롤링으로 가져온 검색 결과를 한 번 더 검색할 순 없을까?
생각외로 북마크는 취향이 겹치는 부분이 있어서 검색결과가 중복되는 결과가 많앗기 때문에 이 중에서 다시 한 번 검색을 할 필요가 있어서 추가검색 기능을 제공해야 했다.
그런데 크롤링 너무 느려서 추가 검색은 조금 더 빠르게 하고 싶었다.
그래서 뭐 HTML 본문을 전부 가져와서 사용자에게 빠르게 제공했다.

파생된 문제로 HTML이 너무 길고, 용량 처리 문제로 어디에 저장할지도 문제였고,
이런 식으로 해결했다.

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
