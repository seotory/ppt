@transition[fade]

@snap[midpoint]
<h1>React api 연동해보기</h1>
@snapend

+++

### React api 연동해보기

<br/>

@ul

- 처음에는 **@color[#DC143C](react-thunk)** 를 고려해 봄
- 진입장벽, npmall 프로젝트는 크기가 작음
- 그냥 심플하게 가자!
- 그러나 프로젝트 크기가 커지면 다시 라이브러리를 고려해보자.

@ulend

+++

### React api 연동해보기

```
const props = {
  alertSender: alertSender(dispatch)
}

export default class Main extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      isApiLoad: false,
      mall: {}
    }
    this.handleLocationChange = this.handleLocationChange.bind(this);
  }

  async componentDidMount() {
    const { mallNo } = this.props.match.params;
    try {
      let res = await apiSupport.send({
        method: 'get',
        url: `${API_HTTP}/api/mall/${mallNo}`
      });
      this.setState({
        isApiLoad: true,
        mall: res.data.id ? res.data : null
      });
    } catch (e) {
      props.alertSender({
        type: 'danger',
        body: JSON.stringify(e)
      });
    }
  }

  componentWillMount() {
    const { history } = this.props;
    this.unsubscribeFromHistory = history.listen(this.handleLocationChange);
    this.handleLocationChange(history.location);
  }

  componentWillUnmount() {
    if (this.unsubscribeFromHistory) this.unsubscribeFromHistory();
  }

  handleLocationChange(location) {
    emitPageMove(location);
  }
  
  render() {
    // children 컴포넌트에 필요한 props를 셋팅해서 호출한다.
    const childrenWithProps = React.Children.map(this.props.children, child => 
      React.cloneElement(child, Object.assign({}, props, this.props, {mall: this.state.mall}))
    );

    return (
      ! this.state.mall
      ? 
      <h1>사용할 수 없는 테스트 상점입니다. url을 확인해주세요.</h1>
      :
      <div className="wrap">
        <Alert />
        <Header mall={this.state.mall} />
        <Nav mall={this.state.mall} />
        <section className="body">
          <div className="container">
            {this.state.isApiLoad && childrenWithProps}
          </div>
        </section>
        <Footer />
      </div>
    )
  }
}
```

@[16](react 생명주기 중 **componentDidMount** 이용한 ajax 호출)
@[16,19](async, await를 이용한 호출)
@[18,27](try-catch 문을 이용한 error 처리)
@[28-31](에러 발생시 알럿 생성)
@[23-26](결과를 받으면 state에 셋팅)
@[49](render 함수를 통해 html 생성)
@[66](api의 성공 여부에 따라 하위 컴포넌트 생성)

---

@transition[fade]

@snap[midpoint]
<h1>SPA 페이지 전환 문제</h1>
@snapend

+++

### SPA 페이지 전환 문제

- setTimeout, ajax의 @color[#DC143C](callback)이 문제

+++

### SPA 페이지 전환 문제

setTimeout 공통 라이브러리 작성  
`src/utils/timerSupport.js`

```
import { objectSize } from './coreHelper';

/**
 * 타이머 관리 object
 */
const timerSet = Object.create(null);

/**
 * 현재 등록된 타이머의 총 갯수
 */
export const size = () => objectSize(timerSet);

/**
 * 타이머를 추가하는 함수
 * {
 *   name: 타이머 명 (default: ex - tiemr_1535098895044),
 *   time: 타이머가 실행될 타임 설정 (default: 1초),
 *   actionFunc: 실행될 함수,
 *   afterFunc: actionFunc이 실행되고 후에 실행되는 함수,
 *              이 함수는 cancel, cancelAll이 동작할 때도 필수로 실행됨
 *   useCancel: 강제 cancel을 실행시킬지 결정하는 옵션 (default: true)
 * }
 * @param {Object} param0 
 */
export const add = ({
  name, 
  time = 1000, 
  actionFunc, 
  afterFunc = () => {},
  useCancel = true
}) => {
  if (!name) {
    name = 'timer_' + (+ new Date());
  }
  timerSet[name] = {
    timer: setTimeout(function () {
      delete timerSet[name];
      actionFunc();
      afterFunc();
    }, time),
    afterFunc,
    useCancel
  }
  return name;
}

/**
 * 타이머의 name을 알고 있다면 cancel 함수를 통해 취소 시킬 수 있다.
 * @param {*} name 
 */
export const cancel = name => {
  if (timerSet[name].useCancel) {
    console.info(`${name} 타이머가 취소되었습니다.`);
    clearTimeout(timerSet[name].timer);
    timerSet[name].afterFunc();
    delete timerSet[name];
  }
}

/**
 * 현재 등록되어 있는 모든 타이머를 cancel 시킴
 * SPA 페이지 바뀜 이벤트 발생시 실행되게 셋팅되어 있음
 */
export const cancelAll = () => {
  Object.keys(timerSet).map(name => cancel(name));
}
```

@[25-45](add 함수를 통해 타이머 추가를 실행)
@[64-66](cancelAll 함수를 통해 현재 실행 중인 모든 타이머 캔슬)

+++

### SPA 페이지 전환 문제

api 공통 라이브러리 작성  
`src/utils/apiSupport.js`

```
import axios from 'axios';

import { objectSize } from './coreHelper';

const CancelToken = axios.CancelToken;

/**
 * api 관리하는 오브젝트
 */
let apiSet = Object.create(null);

/**
 * 현재 동작중인 api의 총 갯수
 */
export const size = () => objectSize(apiSet);

/**
 * axios를 통해 api를 실행시키는 부분
 * @param {axios.config} config 
 */
export const send = async config => {
  let id = (+ new Date());
  config.cancelToken = new CancelToken(function (cancel) {
    apiSet[id] = function() {
      delete apiSet[id];
      cancel();
    };
  });
  let res;
  try {
    res = await axios(config);
    console.log(`axios success.`);
    delete apiSet[id];
  } catch(e) {
    console.log(`axios error.`);
    delete apiSet[id];
    throw e;
  }
  return res;
}

/**
 * 현재 동작중인 api reqeust를 모두 취소 시킴
 */
export const cancelAll = () => {
  for (let id in apiSet) {
    apiSet[id]();
  }
}
```

@[21-40](send 함수를 통해 api 실행)
@[45-49](cancelAll 함수를 통해 현재 실행 중인 모든 api 캔슬)

+++

### SPA 페이지 전환 문제

observer 패턴을 사용하기 위해 event bus 생성  
`src/utils/eventBus.js`

```
import events from 'events';

const PAGE_MOVE = 'PAGE_MOVE';

const eventEmitter = new events.EventEmitter();

export const onPageMove = func => {
  eventEmitter.on(PAGE_MOVE, func);
}

export const emitPageMove = location => {
  eventEmitter.emit(PAGE_MOVE, location);
}
```

@[11-13](이벤트를 발생시키는 함수)
@[7-9](이벤트가 발생되면 실행되는 함수)

+++

### SPA 페이지 전환 문제

실질적으로 이벤트가 발생되는 곳을 찾음

```
<Route
  path="/mall/:mallNo/product/new"
  render={props => (
  <DefaultLayout {...Object.assign({}, props)}>
    <ProductNew/>
  </DefaultLayout>
  )}
/>
```

Route를 통해 history 객체가 하위 컴포넌트에 props로 전달됨

+++

### SPA 페이지 전환 문제

이벤트 발생부

```
import React from 'react';

import Header from './Header';
import Nav from './Nav';
import Footer from './Footer';
import Alert from './Alert';

import * as apiSupport from '../../../utils/apiSupport';
import { emitPageMove } from '../../../utils/eventBus';
import { API_HTTP } from '../../../utils/globalData';

import { dispatch } from '../../../store';
import { alertSender } from '../../../actions/alertAction';

import './index.css';

// 하위에 넘겨줄 props를 만듦
const props = {
  alertSender: alertSender(dispatch)
}

export default class Main extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      isApiLoad: false,
      mall: {}
    }
    this.handleLocationChange = this.handleLocationChange.bind(this);
  }

  async componentDidMount() {
    const { mallNo } = this.props.match.params;
    try {
      let res = await apiSupport.send({
        method: 'get',
        url: `${API_HTTP}/api/mall/${mallNo}`
      });
      this.setState({
        isApiLoad: true,
        mall: res.data.id ? res.data : null
      });
    } catch (e) {
      props.alertSender({
        type: 'danger',
        body: JSON.stringify(e)
      });
    }
  }

  componentWillMount() {
    const { history } = this.props;
    this.unsubscribeFromHistory = history.listen(this.handleLocationChange);
    this.handleLocationChange(history.location);
  }

  componentWillUnmount() {
    if (this.unsubscribeFromHistory) this.unsubscribeFromHistory();
  }

  handleLocationChange(location) {
    emitPageMove(location);
  }
  
  render() {
    // children 컴포넌트에 필요한 props를 셋팅해서 호출한다.
    const childrenWithProps = React.Children.map(this.props.children, child => 
      React.cloneElement(child, Object.assign({}, props, this.props, {mall: this.state.mall}))
    );

    return (
      ! this.state.mall
      ? 
      <h1>사용할 수 없는 테스트 상점입니다. url을 확인해주세요.</h1>
      :
      <div className="wrap">
        <Alert />
        <Header mall={this.state.mall} />
        <Nav mall={this.state.mall} />
        <section className="body">
          <div className="container">
            {this.state.isApiLoad && childrenWithProps}
          </div>
        </section>
        <Footer />
      </div>
    )
  }
}
```

@[9](전에 만든 eventBus.js를 import)
@[54, 62-64](실질적으로 page move가 일어나는 경우 발생하는 callback 함수)
@[63](callback이 발생할때 emitPageMove 함수를 실행)

+++

### SPA 페이지 전환 문제

이벤트 사용부

```
import React from 'react';
import {Route, Switch} from 'react-router-dom';

import RouterError from './components/RouterError'
import { DefaultLayout } from './components/layouts';
import { Product, ProductNew, Products, Cart } from './components/pages';

import { isBrowser } from './utils/coreHelper';
import { onPageMove } from './utils/eventBus';
import * as apiSupport from './utils/apiSupport';
import * as timerSupport from './utils/timerSupport';

if ( ! isBrowser() ) {
  global.window = {};
}

/**
 * 이벤트 버스를 통해 페이지 이동에 대한 세부 로직을 밖으로 꺼내옴
 */
onPageMove(location => {
  console.log(`PAGE MOVE EVENT`, location);
  timerSupport.cancelAll();
  apiSupport.cancelAll();
})

class App extends React.Component {
  render() {
    return (
      <Switch>
        <Route
          path="/mall/:mallNo/product/new"
          render={props => (
            <DefaultLayout {...Object.assign({}, props)}>
              <ProductNew/>
            </DefaultLayout>
          )}
        />
        <Route
          path="/mall/:mallNo/product/:prodNo"
          render={props => (
            <DefaultLayout {...Object.assign({}, props)}>
              <Product/>
            </DefaultLayout>
          )}
        />
        <Route
          path="/mall/:mallNo/cart"
          render={props => (
            <DefaultLayout {...Object.assign({}, props)}>
              <Cart/>
            </DefaultLayout>
          )}
        />
        <Route
          path="/mall/:mallNo"
          render={props => (
            <DefaultLayout {...Object.assign({}, props)}>
              <Products/>
            </DefaultLayout>
          )}
        />
        <Route component={RouterError}/>
      </Switch>
    );
  }
}

export default App;
```

@[9](eventBus.js 파일 import)
@[20-24](emitPageMove가 발생했을때 실행되는 함수를 onPageMove에 셋팅함)

---

![Flux Explained](https://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)