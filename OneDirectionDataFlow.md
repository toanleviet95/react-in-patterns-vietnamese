# Dòng chảy dữ liệu đơn hướng
>`Dòng chảy dữ liệu đơn hướng (One direction data flow)` là một kiến trúc đặc trưng của React. Ý tưởng của kiến trúc này là component sẽ không chỉnh sửa lại dữ liệu mà nó nhận vào mà chỉ lắng nghe sự thay đổi dữ liệu và cung cấp một giá trị mới và không cập nhật trên dữ liệu thật. Component sẽ chỉ có vai trò render lại khi có giá trị mới

Hãy bắt đầu bằng một ví dụ minh họa sau. Ta có một component `Switcher` và khi click vào nó ta sẽ bật một biến cờ trong hệ thống

```javascript
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => this.setState({
      flag: !this.state.flag
    });
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

function App() {
  return <Switcher />;
};
```

Lúc này dữ liệu đã nằm trong component. Hãy thêm một `Store`

```javascript
var Store = {
  _flag: false,
  set: function(value) {
    this._flag = value;
  },
  get: function() {
    return this._flag;
  }
};

class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => {
      this.setState({ flag: !this.state.flag }, () => {
        this.props.onChange(this.state.flag);
      });
    }
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

function App() {
  return <Switcher onChange={ Store.set.bind(Store) } />;
};
```

`Store` object là một `singleton` hỗ trợ `setting` và `getting` giá trị từ `_flag`

![One-direction-1](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-07/one-direction-1.jpg)

Giả sử ta lưu giá trị của flag vào một back-end service thông qua `Store` thì khi user quay lại màn hình ta cần phải gán đúng giá trị khởi tạo ban đầu. Nếu user gán flag bằng `true` thì ta phải hiển thị `lights on` còn ngược lại mặc định là `lights off`. Điều này càng phức tạp hơn vì ta đang có dữ liệu ở hai nơi. UI và `Store` có state của chính nó và ta phải thực hiện giao tiếp từ `Store` vào `Switcher` và ngược lại

```javascript
// ... in App component
<Switcher
  value={ Store.get() }
  onChange={ Store.set.bind(Store) } />

// ... in Switcher component
constructor(props) {
  super(props);
  this.state = { flag: this.props.value };
  ...
```

Dòng chảy dữ liệu bây giờ sẽ thay đổi thành

![One-direction-2](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-07/one-direction-2.jpg)

Điều này dẫn đến ta phải quản lý 2 state thay vì 1 làm tăng sự phức tạp trong ứng dụng

Dòng chảy dữ liệu đơn hướng cho phép giải quyết vấn đề này. Nó loại bỏ nhiều chỗ mà ta quản lý state và xử lý tập trung một nơi thường là tại `Store`. Để làm được điều này ta cần phải điều chỉnh `Store` một chút. Ta cần một logic cho phép `đăng ký (subscribe)` những thay đổi

```javascript
var Store = {
  _handlers: [],
  _flag: '',
  subscribe: function(handler) {
    this._handlers.push(handler);
  },
  set: function(value) {
    this._flag = value;
    this._handlers.forEach(handler => handler(value))
  },
  get: function() {
    return this._flag;
  }
};
```

Kế đó là đặt `hook` vào `App` component và sẽ render lại mỗi khi `Store` thay đổi giá trị

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { value: Store.get() };
    Store.subscribe(value => this.setState({ value }));
  }
  render() {
    return (
      <div>
        <Switcher
          value={ this.state.value }
          onChange={ Store.set.bind(Store) } />
      </div>
    );
  }
};
```

Bằng việc thay đổi như trên ta sẽ có một `Switcher` component đơn giản hơn. Ta không cần `state` ở trong component này, vì vậy ta sẽ viết lại dưới dạng `stateless function`

```javascript
function Switcher({ value, onChange }) {
  return (
    <button onClick={ e => onChange(!value) }>
      { value ? 'lights on' : 'lights off' }
    </button>
  );
};

<Switcher
  value={ Store.get() }
  onChange={ Store.set.bind(Store) } />
```

### Kết luận
Lợi ích của kiến trúc này là các component chỉ thực hiện mặt hiển thị `Presentation` khiến việc phát triển ứng dụng trở nên dễ dàng hơn. Dòng chảy dữ liệu đơn hướng đã thay đổi rất mạnh mẽ đến suy nghĩ của chúng ta khi thiết kế một tính năng mới
