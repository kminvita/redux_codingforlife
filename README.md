# redux_codingforlife

- state와 render의 관계
redux의 핵심은 store이다.
store는 정보가 저장되는 곳으로 글 목록에 대한 정보나 현재 선택한 글에 대한 정보들이 모두 store에 저장된다.

store안에는 state라고 하는 곳에 실제 정보가 저장된다.
여기서 중요한 점은 state에 직접 접속하는것이 금지되어있고 불가능하다.
항상 함수를 통해서 접근할 수 있다.

store를 만들때 제일 먼저 해야하는 것은 reducer라는 함수를 만들어서 공급해주는것이다.

```
function reducer(oldState, action){
    // ...
}
var store = Redux.createStore(reducer);
```
`createStore`를 통해 store가 생성된다. 꼭 `reducer`를 인자로 줘야한다.
이렇게 리듀서를 작성하는것이 리덕스의 핵심이라고 해도 과언이 아니다.

또 중요한것은 render이다.
render는 store안에 속해 있지 않으며,
리덕스와 상관없이 UI를 만들어주는 역할을 한다.
즉, 개발자가 짤 코드를 말한다.

store에 있는 state에 직접 접속하는것이 금지되어있다고 언급하였는데,
접속하기 위해서는 함수를 통해서 소통할 수 있다.
getState, dispatch, subscribe와 같은 함수를 통해 접근할 수 있다.
store가 은행이라면 getState, dispatch, subscribe는 창구 직원같은 역할을 한다.

```
function render(){
    var state = store.getState();
    // ...
    document.querySelector('#app').innerHTML =
        <h1>WEB</h1>
        ...
}
```
render함수는 이런식으로 생겼는데 store에 getState를 통해 state값을 가져올 수 있고,
innerHTML을 통해 state값을 이용하여 웹 페이지를 만든 다음 실행하면 실제 UI가 된다.

render는 내부적으로 getState를 통해서 state를 가져오고 그것을 다시 render에게 전달해준다.
이후 render가 동작해서 UI를 만든다.

(직접 접근하지 못하는 이유는 망가질 수 있기 때문이다.)

즉, render라는 함수는 언제나 state값을 참조해서 UI를 만드는 친구이다.

여기서 한단계 더 깊이 생각해보자면,
render함수는 언제나 state값을 참조해서 UI를 만드는 친구인데,
state값이 바뀔때마다 알아서 render함수를 호출하여 UI를 갱신한다면 얼마나 편리할까?

이때 사용하는 것이 subscribe 함수이다.

```
store.subscribe(render);
```
render함수를 subscribe에 사용하는 코드는 위와 같다.
subscribe에 render함수를 등록해놓으면 state값이 바뀔때마다 render함수가 호출되면서 UI가 새롭게 갱신된다.

즉, render함수를 통해서 state값을 참조해서 UI를 바꾸고
subscribe 함수를 통해서 state값이 변경될때마다 render를 호출해서 UI를 새롭게 갱신시켜준다.

- action과 reducer
애플리케이션은 정적이지 않다...
사용자가 input태그에 글을 작성할수도있고, submit버튼을 이용하여 추가된 글목록이 갱신되는 등
이러한 상황들을 예상할 수 있는데 이때 리덕스가 어떻게 동작하는지에 대해서 알아보자

```
<form onsubmit="
    // ...
    store.dispatch({type:'create', payload:{title:title, desc:desc}});
">
```
submit버튼에는 이러한 이벤트가 걸려있다고 가정해보자.
submit버튼을 클릭하면 store dispatch에게 객체를 전송하는데
이때 type:'create'라는게 중요하다.

저 객체 자체를 action이라고 한다.

action(===객체)가 dispatch에게 전달된다.

그럼 여기서 dispatch가 하는 역할이 뭘까?

dispatch는 두가지 일을 하는데
첫번째, reducer를 호출해서 state의 값을 바꾼다.
두번째, 첫번째 작업이 끝나면 subscribe를 이용하여 render함수를 호출해준다.

그럼 새로운 화면이 갱신된다.

그렇다면 dispatch가 reducer를 어떻게 다룰까?
dispatch가 reducer를 호출할때 두개의 값을 전달한다.
코드와 함께 살펴보자
```
function reducer(state, action){
    if(action.type === 'create'){
        var newContents = oldState.contents.concat();
        var newMaxId = oldState.maxId+1;
        newContents.push({id:newMaxId, title: action.payload})
        return Object.assign({}. state, {
            contents:newContents,
            maxId:newMaxId,
            mode:'read',
            selectedId:newMaxId
        });
    }
}
```
dispatch가 reducer를 호출할때 현재의 state값과 action데이터(=== 객체)를 전달한다.
여기서 action은 type이 create인 객체의 값이다.

두개의 값이 dispatch에 의해서 공급되고 reducer함수가 호출된다.
action의 type이 create라면 그때 리턴해주는 값은 state의 새로운 값을 리턴해준다.

즉, reducer는 state를 입력값으로 받고 action을 참조해서 새로운 state값을 만들어내서 리턴해주는 state를 가공해주는 가공자 역할을 한다.
reducer가 리턴하는 값이 새로운 state값이 된다.

그럼 state값이 변경되었으니 render가 변경되어야하기때문에
subscribe에 등록되어있는 구독자들을 다 호출해준다.

render가 호출되면서 이전에 봤던 과정처럼 getState함수를 통해 새로 변경된 state를 가져온 다음
render가 UI를 갱신해준다.

즉, 새로운 state에 맞게 UI가 갱신된다.

getState를 통해 state를 가져오고
dispatch를 통해 값을 변경시키고
subscribe를 이용하여 값이 변경됐을때 구동될 함수들을 등록해준다.
또, reducer를 통해서 state의 값을 변경한다.

- 리덕스가 좋은 가장 중요한 이유
리덕스가 기존의 개발환경에 가져온 혁신적인 요소보다는
리덕스와 같은 시스템이 공통적으로 제공하는 혁명적인 부분 위주로 장점을 살펴보자.

컴포넌트가 독립적이지 않은 상태에서 서로 상호작용한다고 가정했을때
컴포넌트의 개수^2만큼의 로직이 필요하게 된다.

하지만 리덕스와 같이 중앙집중식으로 데이터를 관리하게 된다면
하나의 컴포넌트에서 어떤 데이터의 변화가 일어났을때
리덕스가 나머지 컴포넌트에게 변경을 해야한다는것을 알리고 나머지 컴포넌트는 알아서 변경을 하게 될것이다.
이러한 방식을 사용하게 된다면 컴포넌트개수*2 만큼의 로직이 있으면 되기때문에 훨씬 더 단순해진다.

추가로 Redux DevTools 확장기능을 이용하면 state의 시간의 변화에 따른 상태를 볼 수 있다.
확장기능을 사용하지 않았을때는 디버거를 통해서 현재의 상태만 볼 수 있지만,
확장기능을 사용하게 된다면 각각의 변화가 생겼을때 애플리케이션이 내부적으로 어떤 상태인지 볼 수 있다.
그리고 현재 애플리케이션의 상태를 파일로 다운로드 받은 후 import해주면 그 애플리케이션의 상태가 그대로 복원된다.

이렇게 redux를 이용하여 중앙집중식으로 관리하면 애플리케이션을 보다 쉽게 개발할 수 있다.