# 의뢰 내용
카페24 사용한 웹 사이트 운영 중임. vimeo 비디오를 게시물에 임베드 하였으나 작동하지 않음.
# 원인
카페24에서 여러가지 이유로 iframe의 삽입을 제한하고 있음.
vimeo는 iframe으로 임베드 하는 것을 표준으로 제공하고 있음. embed,object 태그는 지원 종료되었음.
iframe을 제외한 태그로 비디오 임베드 시 vimeo 플레이어의 일부기능은 제한되어 사용할 수 없었음.
특히 의뢰인은 전체화면 기능을 원했으나, 플레이어의 전체화면 보기를 사용할 수 없었음.
iframe이 아닌 외부 소셜미디어 등에 삽입 시를 상정하여 전체 화면 기능을 쓰지 않는 채로 제공한다고 함.

# 해결
iframe 태그 사용 시 allowfullscreen 속성을 사용하여 vimeo 플레이어가 인식. 
전체화면 재생 기능이 visible됨. object 태그 사용 시에도 이 기능을 사용하기 위해서 vimeo 플레이어 자체의 소스 코드 리뷰함. iframe에서 설정된 allowfullscreen 속성을 iframe 노드에서 어떻게 인식하는지 등이 주요 조사 지점.
vimeo 플레이어 내부에서 document.fullscreenEnabled 속성을 검사하여 전체 화면 기능을 숨김.document.fullscreenEnabled 속성은 iframe 태그에 fullscreen 옵션을 지정해야 활성됨.
## 솔루션 1
서버의 backend에 접근하여, iframe 필터링 관련 코드를 수정하기.
**제외** 기존에 존재하는 코드를 수정하는 행위에 부담이 있음. 
작업환경이 갖춰져있지 않음
## 솔루션 2
vimeo 플레이어를 이용하지 않고, vimeo 비디오의 스트리밍 url을 사용하여 video 태그에 넣어 재생하기.
**제외** vimeo의 플레이어의 기능을 이용할 수 없음. 재생 통계 등, 임베드 도메인 제한 할 수 없어 스트리밍 주소 노출 시 누구나 재생가능함.
## 솔루션 3
1. 부모 DOM의 vimeo 관련 \<object\> element를 \<iframe\>으로 수정하는 js코드 작성
2. html 문서 생성. js코드 포함하여 페이지 로드와 함께 실행하도록 함 (object tag는 js를 실행하지 않음.)
3. html 문서를 서버에 업로드 (CORS 준수)
4. 게시물 작성시 업로드된 html 문서를 object 태그로 삽입함.
5. 게시물 조회 시 object 태그가 iframe으로 수정되어 정상 작동함.
## 솔루션 4
카페24 빌더관리자 기능을 사용하면 서버의 소스를 간단히 수정할 수 있음.(백엔드 아닌 프론트엔드 한정) 웹에서 직접 편집할 수 있는 기능을 제공함.
1. 모든 페이지에서 공통으로 실행되는 모듈의 js 코드 편집
2. vimeo object 태그를 iframe태그로 변경하는 코드 저장
3. 적용

```javascript
////////////////vimeo 비디오 모듈  
window.addEventListener('DOMContentLoaded', function() {  
      // class="vimeo"인 모든 <object> 태그 선택  
      const vimeoObjects = parent.document.querySelectorAll('object.vimeo');  
      vimeoObjects.forEach(function(objectElement) {  
        // 새로운 <iframe> 태그 생성  
        const iframeElement = parent.document.createElement('iframe');  
          
        // 모든 속성 복사  
        Array.from(objectElement.attributes).forEach(attr => {  
          if (attr.name === 'data') {  
            iframeElement.setAttribute('src', attr.value); // data 속성은 src로 변환  
          } else {  
            iframeElement.setAttribute(attr.name, attr.value); // 나머지 속성은 그대로 복사  
          }  
        });  
  
  
        // <object>를 <iframe>으로 교체  
        objectElement.parentNode.replaceChild(iframeElement, objectElement);  
      });  
    });
```
