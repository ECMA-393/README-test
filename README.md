# 📚 UrLink

<div align="center">

  ![Group 1 (3)](https://github.com/user-attachments/assets/20c8066b-e725-4f7b-975b-5ebc11aa995f)

  URLink는 북마크 제목만으로는 파악하기 어려운 내부에 키워드를 검색하여 필요한 정보를 찾아주는 서비스입니다.
</div>


## 목차
- [1. 개발 동기](#1-개발-동기)
- [2. 기술 스택](#2-기술-스택)
- [3. 구현 세부사항](#3-구현-세부사항)
  - [3-1. 어떻게 북마크를 가져올까?](#3-1-어떻게-북마크를-가져올까)
  - [3-2. SPA, Iframe 해결](#3-2-spa-iframe-해결)
  - [3-3. 어떻게 키워드가 포함된 문장을 가져올까?](#3-3-어떻게-키워드가-포함된-문장을-가져올까)
  - [3-4. 너무 느린 크롤링 속도 어떻게 해결할까?](#3-4-너무-느린-크롤링-속도-어떻게-해결할까)
    - [3-4-1. 사용자 UI/UX를 통해 해결해보자.](#3-4-1-사용자-uiux를-통해-해결해보자)
    - [3-4-2. 로직 변경을 통해 크롤링 서버 OOM을 피해보자.](#3-4-2-로직-변경을-통해-크롤링-서버-oom을-피해보자)
  - [3-5. 크롤링으로 가져온 검색 결과를 한 번 더 검색할 순 없을까?](#3-5-크롤링으로-가져온-검색-결과를-한-번-더-검색할-순-없을까)
    - [3-5-1. 가져온 HTML을 어디에 저장할지의 문제](#3-5-1-가져온-html을-어디에-저장할지의-문제)
  - [3.6. 어떻게 익스텐션과 웹 페이지간 데이터를 연결 시킬 수 있을까](#3-6-어떻게-익스텐션과-웹-페이지간-데이터를-연결-시킬-수-있을까)
- [4. 의사결정 방식](#4-의사결정-방식)
- [5. 일정](#5-일정)
- [6. 팀원](#6-팀원)


## 1. 개발 동기
일상적으로 인터넷을 사용하다 보면, 저장해둔 북마크가 어느새 쌓여가는 경험을 하게 됩니다. 그러다 보니 북마크의 제목만으로는 실제 페이지의 내용을 파악하기 어려워, 필요한 정보를 찾기 위해 다시 방문해야 하는 불편함이 생겼습니다. 더불어 기존의 서비스들은 주로 북마크 제목을 기반으로 검색 기능을 제공했기에, 세세한 내용 검색에는 한계가 있었습니다.

이러한 문제점을 자연스럽게 해결하고자, 사용자가 별도의 로그인 없이도 크롬 브라우저에 저장된 북마크를 chromeAPI.bookmarks로 불러와, 각 북마크 내부의 키워드까지 부드럽게 검색할 수 있는 익스텐션을 개발하게 되었습니다.

## 2. 기술 스택
<div align="center">
  
| 프론트엔드 | 백엔드 | 빌드 | 테스트 |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | 
| <img src="https://img.shields.io/badge/React-3B4250?style=flat-square&logo=React&logoColor=#61DAFB"/> <br /> <img src="https://img.shields.io/badge/Zustand-3B4250?style=flat-square&logo=React&logoColor=#3B4250"/> <br /> <img src="https://img.shields.io/badge/Tailwind-3B4250?style=flat-square&logo=tailwindcss&logoColor=#06B6D4"/> <br /> <img src="https://img.shields.io/badge/Axios-3B4250?style=flat-square&logo=Axios&logoColor=#5A29E4"/> | <img src="https://img.shields.io/badge/Node-3B4250?style=flat-square&logo=Node.js&logoColor=#5FA04E"/> <br /> <img src="https://img.shields.io/badge/Puppeteer-3B4250?style=flat-square&logo=Puppeteer&logoColor=#40B5A4"/> <br /> <img src="https://img.shields.io/badge/Express-3B4250?style=flat-square&logo=Express&logoColor=#646CFF"/>| <img src="https://img.shields.io/badge/Vite-3B4250?style=flat-square&logo=vite&logoColor=#646CFF"/> | <img src="https://img.shields.io/badge/Vitest-3B4250?style=flat-square&logo=vitest&logoColor=#6E9F18"/> |

</div>


## 3. 구현 세부사항
### 3-1. 어떻게 북마크를 가져올까?
프로젝트의 출발점은 사용자가 입력한 키워드를 기준으로 크롬 북마크 데이터를 불러오는 것이었습니다.

이 프로젝트가 익스텐션 형태로 크롬의 API를 활용하기 위해 설계된 만큼, Chrome 공식 문서에서 제공하는 ```chromeAPI.getBookmark()``` 예제를 참고하여 구현을 진행하고자 했습니다. 
하지만 문서를 살펴본 결과, 해당 예제는 DOM을 직접적으로 조작하는 방식을 사용하는 것으로 확인되었습니다.

React 기반으로 개발 중이었기 때문에 DOM을 직접 조작하는 방식은 지양해야 했고, 이로 인해 공식문서에서 제공하는 방식과 다른 접근법이 필요했습니다. 
따라서 직접 DOM을 다루지 않고 React의 방식으로 처리할 수 있는 대체 방법이 필요했습니다.
방법을 구하기 전, 순수하게 ```chromeAPI.getBookmark()``` 가 어떤 data를 가져오는지 의문이 들어 자료를 꺼내 확인 해보았습니다.

``` js
  useEffect(() => {
    chrome.bookmarks.getTree((treeList) => {
      console.log(treeList);
      getNewTree(treeList);
    });
  }, []);
```
  
```chrome.bookmarks.get``` 로 가져온 북마크는 사용자 북마크 폴더 개수에 따라 얼마든지 중첩이 가능한 구조였습니다.

![트리구조](https://github.com/user-attachments/assets/3c03b73c-46b6-44b1-8927-189628164a12)

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

<br/>

### 3-2. 크롤링을 선택한 이유[SPA, Iframe 해결]
저희가 처음에 생각한 방법은 북마크 URL을 받아 해당 URL로 fetch 요청을 보내고, 응답으로 받은 HTML 문자열에서 사용자가 입력한 키워드를 찾는 것이었습니다. 그러나 이 방법은 몇 가지 문제점이 있었습니다.<br />
먼저, MPA(Multi Page Application)의 경우, fetch 요청으로 가져온 HTML에서 키워드를 쉽게 찾을 수 있었습니다. MPA는 완성된 페이지 전체를 렌더링하기 때문에 저희의 의도대로 HTML을 추출하고 일치하는 키워드를 찾는 것이 가능했습니다. 그러나 SPA(Single Page Application)의 경우, fetch 요청으로 가져오는 HTML은 렌더링되기 전의 비어있는 상태일 수 있어 키워드를 찾을 수 없었습니다. SPA는 HTML의 일부만 받아 점진적으로 렌더링하기 때문에 이와 같은 문제가 발생했습니다.<br />

<div align="center">
  
  | | MPA | SPA |
  | ----- | ------ | ------ | 
  | 렌더링 시점 | 완성된 페이지 전체를 렌더링 | HTML에 페이지의 일부분을 받아 점진적 렌더링 |
  
</div>

iframe의 경우도 마찬가지로, fetch 요청으로 응답받은 HTML에서는 iframe 내부의 내용을 접근할 수 없는 문제가 있었습니다. iframe 내부의 DOM은 독립적이기 때문에 innerText를 통해 접근할 수 없었습니다.<br />
이를 해결하기 위해 Puppeteer의 헤드리스 브라우저 모드를 사용했고, MPA뿐만 아니라 SPA와 iframe까지 대응할 수 있었습니다.
1. SPA 페이지 로딩 대기: Puppeteer를 사용하여 인간적인 브라우징 패턴을 모방하고, SPA 페이지가 완전히 로딩될 때까지 대기합니다. 이를 통해 페이지가 완전히 렌더링된 후의 HTML을 가져올 수 있습니다.
2. iframe 내부 내용 접근: Puppeteer를 사용하여 iframe 내부의 내용을 읽어올 수 있습니다. iframe이 발견되면, iframe의 src 속성을 통해 iframe 내부로 이동하여 크롤링을 시작합니다. 이를 통해 iframe 내부의 DOM에 접근하고, 일치하는 키워드를 찾을 수 있습니다.
이를 통해 SPA와 iframe 문제를 해결하고, 사용자가 입력한 키워드를 보다 더 넓은 범위로 대응할 수 있게 되었습니다.<br />

<br/>

### 3-3. 어떻게 키워드가 포함된 문장을 가져올까?
![image (3)](https://github.com/user-attachments/assets/9e1c519c-8c50-43e5-b5ca-e9d58a498d5d)
가져온 문자열은 다음과 같이 HTML 문서 구조 내에서 요소들 사이의 공백과 개행을 유지하기 위해 개행문자가 포함돼 있는 문자열이었습니다.
이를 해결하기 위해 정규 표현식을 사용하여 데이터를 가공했습니다.
```js
const getAllSentence = (innerText) => {
  return innerText
    .replace(/\n|\r|\t/g, " ")
    .split(/(?<=다\. |요\. |니다\. |\. |! |\? )/)
    .reduce((array, sentence) => {
      const trimmedSentence = sentence.trim();
      if (trimmedSentence) {
        array.push(trimedSentence);
      }
      return array;
    }, []);
};
```
위 코드는 개행 문자를 공백으로 대체한 후 각 문장을 구분하여 배열에 담는 로직으로 개행문자 제거 로직을 구현했습니다.

또한 find함수를 통해 키워드를 포함하는 문장과 일치하는 첫 번째 문장을 가져오는 작업을 진행했지만, find함수는로 가져온 하나의 문장이 너무 긴 문자열을 포함해서 클라이언트 측에서 이 문자열을 표시하기에 어려움이 따랐기 때문에 문장을 단어 단위로 자르고, 키워드인 단어를 찾아서 그 단어 기준으로 "특정 단어 수" 만큼 데이터를 가져오도록 하여 해결했습니다.<br />

<div align="center">

  ![스크린샷 2025-03-04 오후 5 22 21](https://github.com/user-attachments/assets/c15333f4-5f2e-40f7-a29b-278cabbd3908)
  
</div>

그러나 사용자에게 보여주어야 할 공간을 채우기 위해 단어의 길이와 개수를 계산해 특정 단어 수를 도출한 "특정 단어 수"를 사용해 조합한 문자열이 사용자가 느끼기에 부자연스러운 느낌을 주어 사용자 경험을 개선을 위해 한 문장을 모두 가져온 뒤 ```text-overflow: ellipsis;```를 적용해 문장을 공간에 맞게 처리할 수 있었습니다.

<br/>

### 3-4. 너무 느린 크롤링 속도 어떻게 해결할까?
북마크 내부의 DOM을 탐색하고 사용자가 입력한 키워드를 검색하기 위해선 크롤링이 필수적이었습니다. 다만, 크롤링에 걸리는 시간은 절대적으로 필요한 시간이며 이를 줄일 방법은 많지 않았습니다.<br />
크롤링이 끝나기를 기대하는 사용자의 이탈을 막기 위한 대책을 세워야 했고, 한 번에 여러 개의 크롤링 요청을 보낼 시 Docker로 감싼 AWS의 EC2에 올려 놓은 크롤링 서버가 받은 인스턴스의 램 메모리 1GB보다 많은 메모리를 사용하여 OOM(Out Of Memory) 오류로 서버가 종료되는 이슈도 있었습니다. 따라서 서버에 대한 대책도 필요했습니다.<br />

#### 3-4-1. 크롤링 요청을 나누어서 해결
기존 로직은 Promise.allSettled를 사용했습니다. allSettled는 all과 다르게 하나의 요청에서 에러가 발생해도 나머지 요청을 계속 진행하기 때문에, 모든 북마크에 대한 요청을 한 번에 처리하려고 했습니다. 그러나 사용자가 50개의 북마크를 가지고 있다면 50개를 한 번에 요청하게 되어 서버 과부하와 OOM 오류를 유발할 수 있었고, 사용자는 50개가 모두 완료될 때까지 결과를 볼 수 없었습니다.<br />
이를 개선하기 위해, Promise.allSettled를 5개 단위로 묶은 프로미스 배열을 사용하여 5개씩 병렬로 처리하고, 이 병렬 처리 그룹을 직렬로 처리하는 방식으로 변경했습니다. 이를 통해 서버 과부하를 줄이고, 사용자는 크롤링 결과를 순차적으로 빠르게 확인할 수 있게 되었습니다.<br />
이 방법을 사용하여 북마크 개수에 따라 EC2 인스턴스가 다운되는 현상을 방지하고, 사용자가 결과를 최대한 빨리 확인할 수 있도록 개선했습니다.<br />

<div align="center">
  <img src="https://github.com/user-attachments/assets/be184ec0-cd19-48ab-92ac-0dd421fe2ba1" />
</div>

<br/>

### 3-5. 크롤링으로 가져온 검색 결과를 한 번 더 검색할 순 없을까?
생각외로 북마크는 사용자 취향에 맞는 페이지들을 북마크 해놓은 것이어서 키워드 선정을 잘 못하게 되면 사용자가 원했던 결과보다 더욱 많은 결과가 검색되었습니다. 그래서 1차로 나온 검색 결과를 가지고 추가적으로 검색을 하면 사용자 편의성이 증가할 것이라고 생각했습니다.<br />
하지만 문제는 크롤링의 속도였습니다. Promise.race() 를 통해 느린 크롤링의 속도를 UX적으로 개선했는데, 다시 추가 검색을 통해 느린 속도를 경험한다면 사용자 이탈률이 높아질 것이라고 생각했습니다.<br /><br />
따라서 추가 검색은 1차 검색보다 빠른 속도를 사용자에게 제공해야 했고, 1차 검색 시 단순히 키워드가 포함된 문장만 가져오는 것이 아닌 페이지 HTML 본문에서 ```innerText```를 통한 텍스트를 전부 가져와 사용자가 추가 검색 시 크롤링이 아닌 가져온 HTML 본문에서 추가 키워드를 찾아 제공하기로 했습니다.


#### 3-5-1. 가져온 HTML을 어디에 저장할지의 문제
firebase, mongoDB 등 다양한 스토리지가 있지만 저희는 익스텐션 LocalStorage에 저장하기로 했습니다.<br />
익스텐션 LocalStorage는 크롬 익스텐션에서 제공해주는 스토리지로 최대 10mb까지 사용할 수 있고 사용하기에 간편하다는 장점이 있고 또한, 옵션 페이지를 적용한 저희 익스텐션에서 옵션 페이지와 익스텐션 간의 데이터 공유 시 LocalStorage를 사용해 편리하게 공유할 수 있기에 적합한 선택이라고 생각했습니다.<br /><br />
LocalStorage의 부족한 용량을 넘기면 생기는 에러를 피하기 위해 ```StorageManager API``` 를 사용하여 현재 용량을 확인하고, 만약 용량으 넘길 시 기존에 있던 HTML 본문 중 가장 오래된 요소를 지우고 최신 요소를 넣는 방법을 선택했습니다.
```js
searchBookmarkList: {searchKeyword1: {list}, searchKeyword2: {list}}
```
일반 웹 환경과 동일한 브라우저 기반이라 생각하여 ```localStorage.setItem()```을 사용하면 데이터를 저장할 수 있을 거라 기대했지만, 익스텐션 환경에서는 해당 함수가 정상적으로 호출되지 않았습니다.
```js
ReferenceError: localStorage is not defined
```
이 에러를 통해 익스텐션 환경에서는 localStorage를 사용할 수 없음을 확인했고, 크롬 익스텐션 전용 함수 목록을 살펴보던 중 chrome.storage API를 발견했습니다. 공식 문서에서도 window.localStorage 사용을 권장하지 않는다는 점이 명시되어 있어, 결국 chrome.storage API를 사용하여 데이터를 저장하게 되었습니다.

<br/>

### 3-5-2. 로컬 스토리지 용량이 초과되면 어떻게 될까?
크롬 익스텐션에서는 크롤링된 데이터를 chrome.storage.local에 저장하여 추가적인 검색이 필요할 때 빠르게 조회할 수 있도록 설계했습니다. 하지만 Chrome Storage의 용량 제한(기본 10MB)으로 인해 저장 공간이 가득 차는 문제가 발생할 수 있었습니다. 이를 해결하기 위해, 저장 공간이 부족하면 가장 오래된 데이터를 자동으로 삭제한 후 새로운 데이터를 저장하는 방식을 적용했습니다.
#### Storage 용량 초과 문제
<div align="center">
  <img width="700" src="https://github.com/user-attachments/assets/f30ca9bf-2d36-4bf4-83e2-d5dce8494df7"/>
</div>

</br>초기에는 데이터를 저장할 때 용량을 확인하지 않고 무조건 저장을 시도했습니다. 이로 인해 Storage가 가득 찬 상태에서도 계속 데이터를 추가하려다 보니 Unchecked runtime.lastError: QUOTA_BYTES quota exceeded 오류가 발생했습니다. 이를 해결하기 위해 저장 전에 현재 localStorage 사용량을 확인하고, 공간이 부족하면 가장 오래된 데이터를 삭제하는 방식을 사용했습니다.
#### Storage 사용량 확인
Storage에 저장된 데이터의 용량을 파악하기 위해 Chrome Storage API의 chrome.storage.local.getBytesInUse()를 활용했습니다. 이를 통해 현재 Storage가 얼마나 사용되고 있는지 실시간으로 확인할 수 있도록 했으며, 사용량이 10MB에 가까워지면 추가적인 데이터 저장 전에 자동으로 오래된 데이터를 삭제하는 방식으로 구현했습니다. 데이터를 저장할 때 무조건 저장을 시도하는 것이 아니라, 먼저 현재 Storage의 사용량을 확인한 후 저장할 공간이 충분한 경우에만 데이터를 추가하고, 만약 공간이 부족하면 가장 오래된 데이터를 삭제한 후 저장을 시도하도록 로직을 변경했습니다.

<br/>

### 3.6. 북마크들의 서버, 페이지가 유효하지 않은 경우
사용자가 어떤 북마크에 어떤 내용이 있는지 알고 있는 상태에서 검색을 진행할 경우가 있는데 사용자가 알고 있는 것과 달리 오래된 북마크의 경우 북마크 페이지가 유효하지 않거나, 서버가 응답하지 않는 경우 왜 검색이 제대로 되지 않는지에 대해 의아해 할 수 있기 때문에, 북마크들에 대한 에러처리가 필요했습니다.<br />
에러 처리를 하기 위해선 해당 URL에 대해 서버가 응답하지 않는지, 페이지가 없는지에 대해 판단할 필요가 있었고 다음과 같이 처리했습니다.
```js
catch (error) {
  if (!isCheckTrueThisUrl(decodedLink)) {
    return res
      .status(400)
      .send({ message: `[Invalid Characters in HTTP request]  ${error}` });
  } else {
    return res
      .status(500)
      .send({ message: `[ServerError occured] ${error}` });
  }
} 

// 정상 URL 확인 함수
const isCheckTrueThisUrl = (url) => {
  const urlRegex = /^(http|https):\/\/[\w\-]+(\.[\w\-]+)+[/#?]?.*$/;
  if (urlRegex.test(url)) {
    return true;
  } else {
    return false;
  }
};
```
북마크 URL 요청에 따라 크롤링을 해주는 Puppeteer에서 Error를 던져주기 때문에 ```try...catch```의 catch 부분에서 해당 에러에 대해 유효한 URL인지 정규식으로 판단 후 400번의 에러를 클라이언트로 전송합니다.<br />
그 반대로 URL은 정상이나 서버가 응답하지 않아서 크롤링을 진행할 수 없을 때는 클라이언트로 500번 에러를 전송합니다.<br />

<div align="center">
  <img width="500" src="https://github.com/user-attachments/assets/900e30ee-8b3d-41c5-a2c9-44856427ef68"/>
</div>
이렇게 전달받은 에러들은 사용자에게 어떠한 이유로 해당 북마크에 접근하지 못했고 총 몇개의 북마크를 가지고 오지 못했는지 표시되는 용도로 사용됩니다.<br />


<br/><br/>

### 3-7. 받아온 문장에서 일치하는 키워드를 하이라이팅 처리하기
문장에 포함된 특정 키워드를 하이라이팅하려면, 해당 키워드를 개별적인 HTML 태그로 감싸 CSS를 별도로 적용하는 처리가 필요했습니다.

하지만 React에서는 DOM을 직접 조작하는 방식을 지양하기 때문에, 기존에 받아온 데이터를 map으로 렌더링하는 과정에서 자연스럽게 HTML 태그를 추가할 수 있는 방법을 고민하게 되었습니다.

하이라이팅 처리를 위해서 ```dangerouslySetInnerHTML```이라는 속성을 사용하면 일반 DOM의 innerHTML과 유사한 방식으로 HTML을 삽입할 수 있지만, 이름에서 알 수 있듯 악성스크립트가 침투 되 수 있는 보안상 위험이 존재하기 때문에 가능한 사용을 피하고자 했습니다.

최종적으로는 이미 검색한 키워드를 기준으로 문장을 나누고(split), 그 사이사이에 하이라이팅할 키워드를 다시 넣어 렌더링하는 방식을 채택하였습니다. 이 과정에서 자바스크립트의 ```split```메서드를 활용하여 안전하고 효과적으로 키워드를 하이라이팅 처리할 수 있었습니다.
``` js
// 키워드를 기준으로 여러 개의 문자열나누어 붙인 로직
{url.title.split(keyword).map((item, index) => {
  if (index === 0 && !item) {
    return null;
  } else if (index === 0 && item) {
    return <span key={index}>{item}</span>;
  } else {
    return (
      <span key={index}>
        <span className="text-blue-600 font-bold px-1 inline-block mx-px">
          {keyword}
        </span>
        {item}
      </span>
    );
  }
})}
```

<br/><br/>

## 4. 협업 방식
- 코어 타임: 오전 10시 ~ 오전 12시<br />
  매일 정해진 시간에 오프라인 방식으로 모여 직접 스크럼 회의를 통해 협업 상황에서 발생할 수 있는 깃 전략, 컨벤션 문제들을 공유하고 같이 해결해 나갔습니다.<br /><br />
  <div align="center">
    <img width="500" src="https://github.com/user-attachments/assets/690f4024-061f-493f-a9b5-59c67ac1ae8f" />
  </div>

- Git Flow 전략 + Squas
h merge를 통한 깔끔한 커밋 기록<br />
```
  main, dev, feature 3가지 종류로 브랜치를 나누고 
  1. dev 브랜치로 이동
  2. dev 브랜치에서 `git pull origin dev`
  3. `git checkout -b feature branch`
  4. feature branch 에서 개발작업
  5. `git push origin newBranch`
  6. PR 제출 방향은 dev 브랜치 ← feature branch (작업 브랜치)
  7. PR 코드리뷰 완료 후 merge는 squash merge
  - 다른 사람이 dev 브랜치로 PR을 끝내면 나머지 사람들은 작업 브랜치에서 dev브랜치 pull 받기
  - merge시, 충돌 발생하면 merge 메세지에 충돌을 난 이유 적기
      
      ex) `merge: 어떤 파일 몇번째 라인 충돌 해결`
```

## 5. 일정

### 1차 개발 : 2025년 1월 13일 ~ 2025년 2월 2일
> 진행 사항
> - 협업 규칙 및 Git 플로우 규칙 설정
> - 프로젝트 esLint, pretteir, husky 설정
> - 서버 Express, Puppeteer 설정
> - 크롤링[Puppeteer] 서버 개발
> - 익스텐션 검색 로직 구현
> - 익스텐션 옵션페이지 추가

### 2차 개발 : 진행 중
> 진행 사항
> - 옵션 페이지 기획 변경
> - 리팩토링 진행
> - 전역상태 설정
> - fetch Aixos로 변경

## 6. 팀원
- 심소은 : euns127@gmail.com
- 박성훈 : seonghoon.dev@gmail.com
- 이수보 : nullzzoa@gmail.com
