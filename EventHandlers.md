# Xử lý sự kiện
>React cung cấp cho chúng nhiều hàm xử lý sự kiện. Nhìn chung nó khá giống với các hàm sự kiện mà một DOM thường có chỉ khác ở cách viết `camel case`

```javascript
const theLogoIsClicked = () => alert('Clicked');

<Logo onClick={ theLogoIsClicked } />
<input
  type='text'
  onChange={event => theInputIsChanged(event.target.value) } />
```
Thông thường ta sẽ xử lý sự kiện trong một component chứa phần tử mà `dispatch` sự kiện. Xem ví dụ bên dưới

```javascript
class Switcher extends React.Component {
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log('Button is clicked');
  }
};
```

>Vấn đề là không phải lúc nào ta cũng có một ngữ cảnh duy nhất. Vì thế việc sử dụng `this` trong ví dụ sau sẽ dẫn đến lỗi
```javascript
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
  }
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
    // leads to
    // Uncaught TypeError: Cannot read property 'state' of null
  }
};
```
>Thông thường ta sẽ sử dụng `bind` để giải quyết vấn đề chỉ định cho function

```javascript
<button onClick={ this._handleButtonClick.bind(this) }>
  click me
</button>
```

Bất tiện của cách viết trên là ta phải viết lại `bind(this)` nhiều lần như trên. Vì thế cách tiếp cận tốt nhất là ta có thể tạo ra nhiều `bindings` trong hàm `constructor` như sau

```javascript
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
    this._buttonClick = this._handleButtonClick.bind(this);
  }
  render() {
    return (
      <button onClick={ this._buttonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
  }
};
```

`constructor` là một nơi rất tốt để cho ta chỉ ra những hàm xử lý khác nhau. Ví dụ chúng ta có một form nhưng phải xử lý nhiều input có chung một kiểu function. Chúng ta có thể giái quyết như sau

```javascript
class Form extends React.Component {
  constructor(props) {
    super(props);
    this._onNameChanged = this._onFieldChange.bind(this, 'name');
    this._onPasswordChanged = this._onFieldChange.bind(this, 'password');
  }
  render() {
    return (
      <form>
        <input onChange={ this._onNameChanged } />
        <input onChange={ this._onPasswordChanged } />
      </form>
    );
  }
  _onFieldChange(field, event) {
    console.log(`${ field } changed to ${ event.target.value }`);
  }
};
```

### Kết luận
Chúng ta có thể có nhiều cách xử lý sự kiện trong React. Nếu ta đã quá quen với các hàm xử lý trong HTML thì việc làm quen xử lý sự kiện trong React không phải là quá khó
