@transition[fade]

@snap[midpoint]
<h1>React api 연동해보기</h1>
@snapend

+++

**React api 연동해보기**

<br/>

@ul

- 처음에는 **@color[#DC143C](react-thunk)** 를 고려해 봄
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

@[16](react 생명주기 중 **componentDidMount** 이용한 ajax 호출)
@[16,19](async, await를 이용한 호출)
@[18,27](try-catch 문을 이용한 error 처리)
@[28-31](에러 발생시 알럿 생성)
@[24-27](결과를 받으면 state에 셋팅)
@[49](render 함수를 통해 html 재작성)

---

## Goodbye!

+++

## Adiós!

---

![Flux Explained](https://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)