@snap[midpoint]
# React api 연동해보기
@snapend

+++

**React api 연동해보기**

@ul

- 처음에는 react-thunk 를 고려해 봄
  - 너무 복잡한 단계가 생길 것 같음
- 그냥 심플하게 가자!

@ulend

+++

**React api 연동해보기**

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
@[16](react 생명주기를 이용한 ajax 호출)
@[19-22](async, await를 이용한 호출)


---

## Goodbye!

+++

## Adiós!

---

# code test

```
import React from 'react';

import PriceCounter from '../common/PriceCounter';

import './SelectedOption.css';

const SelectedOption = ({
  title = "선택항목",
  items,
  selectItemRemove,
  selectItemCntPlus,
  selectItemCntMinus
}) => (
  (items.length > 0)
  &&
  <div className="choice-items">
    {title && <strong>{title}</strong>}
    <ul>
      {items.map((item, idx) =>
        <li className="choice-item clearfix" key={item.itemOption}>
          <div className="pull-left">
            <div className="btn-group" role="group" aria-label="...">
              <button type="button" className="close" aria-label="Close" onClick={e => selectItemRemove(idx)}>
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            {' '}
            <span>{item.itemOption}</span>
          </div>
          <PriceCounter 
            plus={e => selectItemCntPlus(idx)}
            minus={e => selectItemCntMinus(idx)}
            count={item.itemCount}
            totalPrice={item.itemPrice * item.itemCount}
          />
        </li>
      )}
    </ul>
  </div>
);

export default SelectedOption;
```

---

### Flux Design

- Dispatcher: Manages Data Flow
- Stores: Handle State & Logic
- Views: Render Data via React

---

![Flux Explained](https://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)