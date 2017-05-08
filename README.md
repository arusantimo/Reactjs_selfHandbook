# Reactjs_selfHandbook
Reactjs 개인적인 사용법 성능 개선 방법 정리

## 1) 컴포넌트 Component

리엑트 가상 엘리먼트를 나타내주는 함수.<br>
Mounting -> updating -> Unmounting 의 순서로 진행된다.

### 1-1) 컴포넌트 라이프사이클

#### Mounting (순서가 보장) 컴포넌트 생성

1. construtor()
2. componentWillMount()
3. render()
--------------------------렌더전
4. componentDidMount()

#### Updating 컴포넌트 갱신

1. componentWillReciveProps() => props가 갱신되면 실행됨
2. shouldComponentUpdate() => 기본값이 존재하고 업데이트(false)를 취소할수 있다.
3. componentWiiUpdate() => setState 불가능
4. render()
5. componentDidUpdate() => 이전의 props과 갱신된 props값 비교 가능 setState가능

#### Unmountiong 컴포넌트 재거

1. componentWillUnmount()

### 컴포넌트 state, props

컴포넌트 인스턴스 속성 state, props가 있다. <br>
컴포넌트 안에서는 state로 값을 전달 받고 <br>
컴포넌트 끼리는 props로 값을 전달 받는다. props은 부모와 자식간의 수직적인 관계로 전달받는다.

### 1-2) setState의 이해

컴포넌트안에 setState()라는 함수가 존재한다. 하지만 완벽하지 못하다.
여러가지 이유가 있지만 ..
1. 값의 변화를 감지할때 연산자의 값이 아닌 reference 값을 기준으로 참/거짓을 리턴한다.
2. setState 호출 즉시 state가 변경되지 않고 비동기적으로 동작하므로 주의 해야 한다.

이렇게 만든 이유는 무분별한 render()를 방지하기 위한것이기도 하다. <br>


### 1-3) 컴포넌트 다양한 패턴

컴포넌트를 좀 더 효율적으로 사용하기 위해서 고려하는 사항들
- solid원칙 (컴포넌트(함수)를 한가지 역할 이외의 다른 기능을 담당하지 않게 한다.)
- dry유지
- 재사용가능한 컴포넌트 생성
- 컴포넌트역할의 명확성
- 똑똑한 컴포넌트(containers), 멍청한 컴포넌트(components) 분리...
- 컴포넌트가 150줄이 넘지 않아야 한다.

#### 무상태 컴포넌트 만들기

재사용성이 좋은 무상태 즉 state가 없는 컴포넌트.<br>
무상태 컴포넌트는 render에 필요한 데이터는 존재한다. 또한 ui를 조작하기 위한 state는 없다.
```js
// JS 자바스크립트
const Block = (props) => (
  <div className={props.className}/>
);
// or
const Block = (...props) => (
  <div {...props}/>
);
```
```ts
//TS 타입스크립트
interface IAppProps {
  className?: string;
};
class Block extends React.Component<IAppProps, any> {
  static defaultProps:IAppProps = {
    className: 'defaultValue'
  };
  public render(): JSX.Element {
    return (
      <div className={props.className}/>
    );
  }
};
```

#### 컨테이터 컴포넌트 만들기

무상태컴포넌트를 조합하여 app을 제작하기 위한 컴포넌트<br>
jsx문법이 적고 state관리를 통해서 무상태(멍청한) 컴포넌트를 조작한다. fetch를 통한 state관리도 가능하다.

```js
//JS 자바스크립트
class Main extends React.Component {
  state = { isChanged: false };
  handleClickButton = (e) => this.setState({ isChanged: true });
  render() {
    return (
      <div>
        <Button onClick={this.handleClickButton}/>
        <Block className={this.state.isChanged}/>
      </div>
    );
  }
}
```
```ts
interface IAppProps {
};
interface IAppState {
  isChanged: boolean;
};
//TS 타입스크립트
class Main extends React.Component<IAppProps, IAppState> {
  constructor(props:IAppProps)  {
    super(props:IAppProps);
    state:IAppState = {
      isChanged: false,
    };
  }
  private handleClickButton = (e) => this.setState({ isChanged: true });
  public render(): JSX.Element {
    return (
      <div>
        <Button onClick={this.handleClickButton}/>
        <Block className={this.state.isChanged}/>
      </div>
    );
  }
}
```

## 2) 성능 개선을 위한 방법들

### 2-1) Webpack DefinePlugin를 이용해서 production 빌드를 한다,

### 2-2) PureComponent의 의미없는 리렌더링 피하기
shouldComponentUpdate에서 렌더링을 선택적으로 하기위해서 방어 코드를 삽입해 준다. <br>
값의 변화를 감지할때 연산자의 값이 아닌 reference 값을 기준으로 참/거짓을 판단하므로 immutable한 값으로 변화를 감지하도록 한다.
```js
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

### 2-3) React Perf 사용
렌더링 성능 체크 툴 사용
```
$ npm install react-addons-perf
```
```js
import Perf from 'react-addons-perf'
Perf.start() // 기록시작
... //성능 체크할 부분
Perf.stop() // 기록끝
const measurements = Perf.getLastMeasurements()
Perf.printInclusive(measurements) //얼마나 시간을 들였는지 알려준다.
Perf.printExclusive(measurements) //얼마나 시간을 들였는지 알려준다.
Perf.printWasted(measurements) //시간낭비를 찾아서 알려준다.
Perf.printDOM(measurements) //모든 dom의 연산을 알려준다.
```

### 2-3) 메서드를 올바르게 선언

```js
안티패턴
  return (
    <div>
      <div onClick={(e) => this.doSomeThing(e)}></div>
      or
      <div onClick={this.doSomeThing.bind(this)}></div>
    </div>
  );
```
위와 같이 render() jsx문법안에 doSomeThing메서드를 만들어 놓으면 render 메서드가 호출 될 때마다 항상 새로운 익명 함수를 만들어 낸다.

```js
권장패턴
class Component extends React {
  doSomeThing = (e) => {
  }
  render() {
    return (
      <div onClick={this.doSomeThing}></div>
    );
  }
}
```
doSomeThing 메소드를 생성 할 때 arrow 함수를 사용하면 이미 doSomeThing 메소드를 Component의 범위에 바인딩하고 doSomeThing은 항상 동일하므로 우리 컴포넌트는 자신을 다시 렌더링하지 않는다.


### 2-4) 비용이 많이 들어가는 로직을 캐시화 한다.
