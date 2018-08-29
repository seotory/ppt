@transition[fade]

@snap[midpoint]
<h1>1. React 복습</h1>
@snapend

+++

### @css[color-point](React 복습)

컴포넌트 생성 1

```
var AppES5 = React.createClass({
  render: function() {
    return(
      <div>Hello World</div>
    );
  }
});
```

+++

### @css[color-point](React 복습)

컴포넌트 생성 2

```
class AppES6 extends React.Component{
  render() {
    return(
      <div>Hello World</div>
    );
  }
};
```

+++

### @css[color-point](React 복습)

컴포넌트 생성 3

```
const StateLess = () => {
  return (
    <div>Hello World</div>
  )
}
```

---

@transition[fade]

@snap[midpoint]
<h1>2. redux 복습</h1>
@snapend

+++

### @css[color-point](redux 복습)

![redux](https://cdn-images-1.medium.com/max/1600/1*87dJ5EB3ydD7_AbhKb4UOQ.png)

---

@transition[fade]

@snap[midpoint]
<h1>3. React api 연동해보기</h1>
@snapend

+++

### @css[color-point](React api 연동해보기)

<br/>

@ul[](true)
- 처음에는 **@css[color-point](react-thunk)** 를 고려해 봄
- 진입장벽, npmall 프로젝트는 크기가 작음
- 그냥 심플하게 가자!
- 프로젝트 크기가 커지면 다시 라이브러리를 고려해보자.
@ulend

+++

### @css[color-point](React api 연동해보기)

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
<h1>4. SPA 페이지 전환 문제</h1>
@snapend

+++

### @css[color-point](SPA 페이지 전환 문제)

<br/>

- setTimeout, ajax의 @color[#DC143C](callback)이 문제

+++

### @css[color-point](SPA 페이지 전환 문제)

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

### @css[color-point](SPA 페이지 전환 문제)

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

### @css[color-point](SPA 페이지 전환 문제)

observer 패턴을 사용하기 위해 event emitter 생성  
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

### @css[color-point](SPA 페이지 전환 문제)

실질적으로 이벤트가 발생되는 곳을 찾아보자!

`App.js` 파일 중에 아래의 코드가 있음...

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

Route를 통해 `history`객체가 하위 컴포넌트에 props로 전달됨

+++

### @css[color-point](SPA 페이지 전환 문제)

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

### @css[color-point](SPA 페이지 전환 문제)

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

@transition[fade]

@snap[midpoint]
![따봉](https://media.metrolatam.com/2018/05/08/rambomeme-bff57a46e4aec97ce611fba845ea13f3-1200x800.jpg)
@snapend

---

@transition[fade]

@snap[midpoint]
<h1>5. 본격 koa 사용해보기</h1>
@snapend

+++

### @css[color-point](본격 koa 사용해보기)

##### node의 web framwork 종류
- express
- **@css[color-point](koa)**
- hapi
- 등등등....

##### koa
- core는 매우 가벼움
- 미들웨어 중심의 web framwork
- 여러가지 미들웨어를 섞어서 구현해야함
- 성능은 express 보다 약간 빠름
- express는 callback 중심, koa는 async & await 중심

+++

### @css[color-point](본격 koa 사용해보기)

server.js 살펴보기

```
const fs = require('fs');
const path = require('path');

const Koa = require('koa');
const cors = require('koa-cors');
const logger = require('koa-logger');
const serve = require('koa-static');
const favicon = require('koa-favicon');
const bodyParser = require('koa-bodyparser');
const Raven = require('raven');
const program = require('commander');

const { routers } = require('./routers');
const { render, getStore } = require('./render/render');
const logo = require('./logo');

const DEFAULT_PORT = 3001;

const app = new Koa();
const template = fs.readFileSync(path.resolve(__dirname, '../build/index.html'), { encoding: 'utf8' });

program
.option('-p, --port [num]', 'server port')
.parse(process.argv);

const PORT = program.port || DEFAULT_PORT;

// Error logging. 
// https://sentry.io
// https://sentry.io/seotory/npmall/ 
Raven.config('https://5a11168212984e6ba49f1876938a2dfb@sentry.io/1266535', {
  environment: process.env.NODE_ENV,
  tags: {
    port: PORT
  }
}).install();

// 에러 발생시 sentry로 보냄
app.on('error', function (err) {
  Raven.captureException(err, function (err, eventId) {
      console.log('Reported error ' + eventId);
  });
});

// cors 지원
app.use(cors());

// 로깅 지원을 위한 설정
app.use(logger());

// post parser
app.use(bodyParser());

// 파비콘
app.use(favicon(path.resolve(__dirname, '../build/favicon.ico')));

// react app을 webpack으로 실행시키면 나오는 빌드 폴더를 기준으로 라우팅
app.use(serve(path.resolve(__dirname, '../build/')));

// router inject into app
routers
  .prefix('/api')
  .targetFiles(['./order','./product','./mall', './test'])
  .inject(app);

// React SSR 지원
app.use(async ctx => {

  const location = ctx.path;
  const rendered = render(location);
  console.log(rendered);

  let res = await ctx.get(`/api${location}`);
  console.dir(res);

  // 해당 문자열을, 템플릿에 있는 '<div id="root"></div> 사이에 넣어줍니다.
  const page = template
                .replace(`<div id="root"></div>`, `<div id="root">${rendered}</div>`)
                // .replace('window.__PRELOADED_STATE__=null', `window.__PRELOADED_STATE__=${JSON.stringify(state).replace(/</g, '\\u003c')}`);
  ctx.body = page;
});

console.log(logo.print());
console.log(`MODE >> ${process.env.NODE_ENV} << MODE`);
console.log(`=======================================================`);
console.log('  [SERVER START]');
console.log(`  http://localhost:${program.port || DEFAULT_PORT}`);
console.log(`=======================================================`);

app.listen(program.port || DEFAULT_PORT);
```

@[4-9](koa 및 koa의 미들웨어 import)
@[19](new Koa를 통해 app 인스턴스 생성)
@[45-58](미들웨어는 위와 같이 app.use를 사용함)
@[60-64](라우터를 통해 url 맵핑하는 부분)
@[90](서버 실행부)

+++

### @css[color-point](본격 koa 사용해보기)

router `product.js` 살펴보기

```
const Router = require('koa-router');

const db = require('./../db');

let router = new Router();

/**
 * 상점에 해당하는 상품 상세
 */
router.get('/mall/:mallNo/product/:prodNo', async ctx => {
  let res = await db.ts(conn => Promise.all([
    db.mapper(conn, 'selectProductByMall', ctx.params),
    db.mapper(conn, 'selectProductOption', ctx.params)
  ]));
  let product = res[0][0];
  let opsInfo = {};
  res[1].map(option => {
    option.hasSubOptions = false;
    if (option.depth == 1) {
      opsInfo[option.opNo] = option;
      opsInfo[option.opNo].subOptions = [];
    }
    if (option.depth == 2 && opsInfo[option.prntOpNo]) {
      opsInfo[option.prntOpNo].hasSubOptions = true;
      opsInfo[option.prntOpNo].subOptions.push(option);
    }
  });
  product.options = Object.keys(opsInfo).map(key => opsInfo[key]);
  ctx.body = product;
})

/**
 * 상품 삭제
 */
router.delete('/mall/:mallNo/product/:prodNo', async ctx => {
  await db.ts(conn => Promise.all([
    db.mapper(conn, 'deleteProduct', ctx.params),
    db.mapper(conn, 'deleteProductOptions', ctx.params),
    db.mapper(conn, 'deleteMallProduct', ctx.params),
  ]));
  ctx.body = 'success';
})

/**
 * 상점에 해당하는 상품 리스트 목록
 * TODO: 리스팅 기능
 */
router.get('/mall/:mallNo/products', async ctx => {
  ctx.body = await db.mapper('selectProductsByMall', ctx.params);
})

/**
 * 상품에 해당하는 옵션 정보 로드
 */
router.get('/mall/:mallNo/product/:prodNo/options', async ctx => {
  let res = await db.mapper('selectProductOption', ctx.params);
  let opsInfo = {};

  res.map(option => {
    option.hasSubOptions = false;
    if (option.depth == 1) {
      opsInfo[option.opNo] = option;
      opsInfo[option.opNo].subOptions = [];
    }
    if (option.depth == 2 && opsInfo[option.prntOpNo]) {
      opsInfo[option.prntOpNo].hasSubOptions = true;
      opsInfo[option.prntOpNo].subOptions.push(option);
    }
  });

  ctx.body = Object.keys(opsInfo).map(key => opsInfo[key]);
})

/**
 * form 데이터 저장
 */
router.post('/mall/:mallNo/product', async ctx => {
  let params = ctx.params;
  let formData = ctx.request.body;
  console.log(formData);

  await db.ts(async conn => {
    let res = await db.mapper(conn, 'insertProduct', formData);
    let prodNo = res.insertId;
    let promiseAry = [];

    if (formData.opYn == 'Y') {
      formData.options.map(option => {
        option.prodNo = prodNo;
        promiseAry.push(db.mapper(conn, 'insertProductOptions', option));
      })
    }
    promiseAry.push(db.mapper(conn, 'insertMallProduct', {mallNo: params.mallNo, prodNo: prodNo}));
    return await Promise.all(promiseAry);
  });
  ctx.body = 'success';
})

module.exports = router;
```

@[1](koa-router 미들웨어를 사용하기 때문에 import)
@[5](new Router를 사용해서 router 인스턴스를 생성)
@[10](router에 사용되는 api url 및 method를 명시)
@[11-14](db를 호출하여 데이터를 가져옴)
@[15-28](데이터 가공)
@[29](response를 만들어서 api 응답)

---

@transition[fade]

@snap[midpoint]
<h1>6. node에서 mysql 사용하기</h1>
@snapend

+++

### @css[color-point](node에서 mysql 사용하기)

커낵션 풀 생성

```
const mysql = require('mysql2');
const info = require('./info');

info.queryFormat = function (query, values) {
  if (!values) return query;
  query = query.replace(/\:(\w+)/g, function (txt, key) {
    if (values.hasOwnProperty(key) && values[key]) {
      return this.escape(values[key]);
    } else {
      return null;
    }
  }.bind(this));
  console.log(query);
  return query;
};

let pool = mysql.createPool(info);

module.exports = pool;
```

@[2](info에 접속 정보를 가지고 있음)
@[4-15](쿼리문의 패턴 매칭을 통해 param 값을 입력 시키기 위한 포맷)
@[17, 19](커넥션 풀 생성 후 풀 반환)

+++

### @css[color-point](node에서 mysql 사용하기)

사용하기 편하게 트랜잭션 lib를 제작

```
const dbPool = require('../db/pool');

let getConnection = () => new Promise((resolve, reject) => {
  dbPool.getConnection((err, conn) => {
    if (err) {
      reject(err);
    }
    resolve(conn);
  })
});

let beginTransaction = conn => new Promise((resolve, reject) => {
  conn.beginTransaction(function(err) {
    if (err) {
      reject(err);
    }
    resolve();
  });
});

let rollback = conn => (new Promise((resolve, reject) => {
  conn.rollback(function() {
    resolve();
  });
})).then(() => conn.release());

let commit = conn => (new Promise((resolve, reject) => {
  conn.commit(function(err) {
    if (err) {
      reject(err);
    }
    resolve();
  });
})).then(() => conn.release());

module.exports = func => getConnection().then(conn => {
  return beginTransaction(conn)
    .then(() => {
      return func(conn).catch(e => {
        console.log(e);
        rollback(conn);
        throw e;
      })
    })
    .then(result => {
      commit(conn);
      return result;
    })
    .catch(e => {
      console.log(e);
      rollback(conn);
      throw e;
    });
});
```

@[36](getConnection을 통해 connection 객채를 얻어옴)
@[37](beginTransaction으로 트랜잭션 시작을 알림)
@[36,39](인자로 보낸 함수를 실행시킴)
@[41,46,51](rollback 또는 commit으로 트랜잭션 종료)
@[25,34](사용했던 connection 반환)

+++

### @css[color-point](node에서 mysql 사용하기)

아래처럼 트랜잭션을 사용함

```
let res = await db.ts(conn => Promise.all([
  db.mapper(conn, 'selectProductByMall', ctx.params),
  db.mapper(conn, 'selectProductOption', ctx.params)
]));
```

---

@transition[fade]

@snap[midpoint]
<h1>7. pm2를 사용한 node app 관리</h1>
@snapend

+++

### @css[color-point](pm2를 사용한 node app 관리)

pm2 

- node.js 프로세스 관리 도구
- log 관리 기능은 좀 약함(자동 log 로테이션이 안됨)
- 노드 프로세스 cluster 지원

pm2는 global로 설치

```
npm install -g pm2
```

아래와 같이 실행시킴

```
pm2 start pm2.config.js
```

+++

### @css[color-point](pm2를 사용한 node app 관리)

pm2.config.js

```
console.log(process.env.NODE_ENV)

// 개발환경 셋팅
let app_dev = [{
  name      : 'koa',
  script    : './server/server.js',
  args      : ['-p', '3000'],
  env       : {
    "NODE_ENV": "development"
  },
  // log setting
  out_file        : './logs/koa.log',
  error_file      : './logs/koa_error.log',
  log_date_format : "YYYY-MM-DD HH:mm Z"
}];

let app_prod = [{
  name      : 'koa',
  script    : './server/server.js',
  args      : ['-p', '80'],
  env       : {
    "NODE_ENV": "production"
  },
  // log setting
  out_file        : './logs/koa.log',
  error_file      : './logs/koa_error.log',
  log_date_format : "YYYY-MM-DD HH:mm Z"
}];

module.exports = {
  apps : process.env.NODE_ENV == 'production'
       ? app_prod
       : app_dev
};
```

@[4-15](개발환경 pm2)
@[17-28](운영환경 pm2)
@[31-33](환경에 따라 앱 실행)

+++

### @css[color-point](pm2를 사용한 node app 관리)

기타 명령어

- pm2 logs
- pm2 monit
- pm2 restart all
- pm2 stop all
- pm2 scale <app_name> <number>
- pm2 pid [app_name]
- pm2 sendSignal <signal> <pm2_id|name>
- 등등등....

---

@transition[fade]

@snap[midpoint]
<h1>8. node app error 로깅</h1>
@snapend

+++

### @css[color-point](node app error 로깅)

sentry.io

- 개발자는 error 로그에 민감
- 서버 들어가서 log 파일 열어보긴 귀찮음
- 빠르고 편한 뭔가가 있으면 좋겠음
- 슬랙처럼 메세지의 양에 따라 과금

+++

### @css[color-point](node app error 로깅)

지원언어

![]()

+++

### @css[color-point](node app error 로깅)

sentry.io 셋팅하기 

```
const Raven = require('raven');

Raven.config('https://5a11168212984e6ba49f1876938a2dfb@sentry.io/1266535', {
  environment: process.env.NODE_ENV,
  tags: {
    port: PORT
  }
}).install();

app.on('error', function (err) {
  Raven.captureException(err, function (err, eventId) {
      console.log('Reported error ' + eventId);
  });
});
```

끝. 참 쉽죠?

---

![](https://image.fmkorea.com/files/attach/new/20160521/44021718/363301069/376446192/9e87449cc5d905ebd80e51150a99c6a3.png)