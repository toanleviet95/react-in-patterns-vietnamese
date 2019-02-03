# Cách giao tiếp giữa các component
>Bản thân mỗi component đều có `state`, `input` và `ouput`

![Communication](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-02/communication.jpg)

### *Input
>Input của component chính là `props`
```javascript
// Title.jsx
function Title(props) {
  return <h1>{ props.text }</h1>;
}
Title.propTypes = {
  text: PropTypes.string
};
Title.defaultProps = {
  text: 'Hello world'
};

// App.jsx
function App() {
  return <Title text='Hello React' />;
}
```
Khi tạo một component việc cần làm là xác định `propTypes` - Kiểu dữ liệu mà chúng ta mong muốn cho `props` của component. Sẽ có những dòng báo lỗi cảnh cáo trên console nếu bạn quên xác định `propTypes` của component. `defaultProps` cũng cần thiết khi bạn muốn gán một giá trị mặc định cho `props` trong trường hợp chúng ta quên truyền giá trị cho `props`

Trong một số trường hợp `props`  mà bạn  truyền vào cũng có thể là một component khác
```javascript
function SomethingElse({ answer }) {
  return <div>The answer is { answer }</div>;
}
function Answer() {
  return <span>42</span>;
}

// later somewhere in our application
<SomethingElse answer={ <Answer /> } />
```

Chúng ta cũng có thể sử dụng thuộc tính `props.children` khi mong muốn truyền một component con trong một component cha
```javascript
function Title({ text, children }) {
  return (
    <h1>
      { text }
      { children }
    </h1>
  );
}
function App() {
  return (
    <Title text='Hello React'>
      <span>community</span>
    </Title>
  );
}
```
>Cần lưu ý rằng trong component `Title` nếu bạn không return về  `{ children }` thì component con sẽ không được render

Trong phiên bản React `v16.3` khi input là một component thì được gọi là `context`. Luôn có một `context` giúp các component có thể tiếp cận với nhau

### *Output
Dễ hình dung thì `output` của một component chính là đoạn HTML mà nó render ra - Những gì mà chúng ta có thể nhìn thấy. Tuy vậy `props` có thể là mọi thứ bao gồm cả `function` vì thế ta có thể lợi dụng điều này để truyền  dữ liệu ra ngoài

```javascript
function NameField({ valueUpdated }) {
  return (
    <input
      onChange={ event => valueUpdated(event.target.value) } />
  );
};
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { name: '' };
  }
  render() {
    return (
      <div>
        <NameField
          valueUpdated={ name => this.setState({ name }) } />
        Name: { this.state.name }
      </div>
    );
  }
};
```
>Ngoài ra chúng ta có thể sử dụng `lifecycle` của component trong trường hợp bạn muốn  `trigger` một `process`

```javascript
class ResultsPage extends React.Component {
  componentDidMount() {
    this.props.getResults();
  }
  render() {
    if (this.props.results) {
      return <List results={ this.props.results } />;
    } else {
      return <LoadingScreen />
    }
  }
}
```
Trong ví dụ trên chúng ta `trigger` một request ngay trong `componentDidMount`. Màn hình `Loading` sẽ hiện ra khi chưa có dữ liệu và sẽ hiện ra  danh sách kết quả khi dữ liệu đã được trả về

### Tóm tắt
Có thể xem một component trong React như một `chiếc hộp đen kín` có `input`, `ouput` và `lifcycle` trong đó. Đây là một trong những ưu điểm mà React mang lại giúp chúng ta dễ dàng tiếp cận một cách trừu tượng và dễ hiểu
