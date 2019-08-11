# Dữ liệu đầu vào kiểm soát và không kiểm soát được
>Hai thuật ngữ `kiểm soát (controlled)` và `không kiểm soát được (uncontrolled)` thường được dùng trong việc quản lý context của form.

`Controlled Input` là input nhận giá trị từ một nguồn duy nhất. Ví dụ bên dưới `App Component` chỉ có một field input duy nhất 

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return <input type='text' value={ this.state.value } />;
  }
};
```

Kết quả của đoạn code trên cho ta một input không thể thay đổi được state. Để làm cho input này hoạt động như ta mong đợi ta phải thêm `onChange` và xử lý cập nhật lại state. Việc này sẽ lại trigger cơ chế render mới và ta sẽ thấy được những gì mà ta nhập vào thẻ input

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
    this._change = this._handleInputChange.bind(this);
  }
  render() {
    return (
      <input
        type='text'
        value={ this.state.value }
        onChange={ this._change } />
    );
  }
  _handleInputChange(e) {
    this.setState({ value: e.target.value });
  }
};
```

Ngược lại `Uncontrolled Input` là một input mà chúng ta sẽ để cho browser xử lý việc user cập nhật. Chúng ta sẽ thêm một giá trị khởi tạo ban đầu bằng thuộc tính `defaultValue` và sau đó thì ta sẽ để browser chịu trách nhiệm giữ state của input

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return <input type='text' defaultValue={ this.state.value } />
  }
};
```

Thẻ input giờ đây sẽ trở nên vô dụng vì khi user cập nhật một giá trị nhưng component vẫn không biết điều đó. Chúng ta sẽ cần phải thêm thuộc tính `ref` để truy cập được input

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
    this._change = this._handleInputChange.bind(this);
  }
  render() {
    return (
      <input
        type='text'
        defaultValue={ this.state.value }
        onChange={ this._change }
        ref={ input => this.input = input }/>
    );
  }
  _handleInputChange() {
    this.setState({ value: this.input.value });
  }
};
```

Thuộc tính `ref` nhận vào một string hoặc callback. Đoạn code phí trên sử dụng một callback để lưu DOM của phần tử vào một biến local gọi là `input`. Kế đến khi `onChange` xử lý, chúng ta sẽ nhận được giá trị mới và cập nhật lại cho state

>Lưu ý: Việc lạm dụng thuộc tính `ref` không phải là một cách hay vì nó sẽ làm cho input của bạn trở nên khó kiểm soát hơn
>=> Hãy luôn cân nhắc việc sử dụng `Controlled Input` càng nhiều càng tốt

### Kết luận
Việc sử dụng `Controlled Input` hay `Uncontrolled Input` không phải là vấn đề quá quan trọng. Tuy nhiên nó vẫn đóng vai trò quyết định về luồng dữ liệu trong React component. Theo ý kiến cá nhân, việc sử dụng `Uncontrolled Input` là một kiểu `anti-pattern` và chúng ta nên tránh việc lạm dụng nó khi có thể
