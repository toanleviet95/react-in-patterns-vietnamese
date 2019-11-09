# Tích hợp với thư viện bên thứ ba
>React rõ ràng là một sự lựa chọn hoàn hảo cho việc xây dựng UI. Tuy nhiên trong vài tình huống ta vẫn cần phải tích hợp thêm thư viện bên thứ ba. Chúng ta đều biết rằng React gặp khó khăn khi làm việc với `actual DOM` (thay vì virtual DOM). Ở bài viết này sẽ chỉ cách cho chúng ta tích hợp React với một jQuery plugin 

### Ví dụ
Chúng ta sẽ thử chọn một plugin gọi là `[tag-it](https://github.com/aehlke/tag-it)` của jQuery:

```javascript
<ul>
  <li>JavaScript</li>
  <li>CSS</li>
</ul>
```

![tag-it-example](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-12/tag-it.png)

Trước hết chúng ta cần include thư viện jQuery, jQuery UI và plugin `tag-it`. Cách sử dụng như sau:

```javascript
$('<dom element selector>').tagit();
```

Tiếp theo là tạo một `Tags component` đơn giản và gắn vào `App component`

```javascript
// Tags.jsx
class Tags extends React.Component {
  componentDidMount() {
    // initialize tagit
    $(this.refs.list).tagit();
  }
  render() {
    return (
      <ul ref="list">
      {
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        )
      }
      </ul>
    );
  }
};

// App.jsx
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { tags: ['JavaScript', 'CSS' ] };
  }
  render() {
    return (
      <div>
        <Tags tags={ this.state.tags } />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.querySelector('#container'));
```

### Force single-render

Điều trước tiên ta cần làm là `force single-render` cho Tags component bởi vì React thêm những phần tử trong actual DOM mà ta muốn sử dung jQuery. Chúng ta cần giữ cho React và jQuery phải cùng làm việc trên một DOM. Để thực hiện được `single-render` ta cần sử dụng `shouldComponentUpdate` như sau:

```javascript
class Tags extends React.Component {
  shouldComponentUpdate() {
    return false;
  }
  ...
```

Bằng cách trả về `false` chúng ta sẽ nói rằng việc render lại là không thể xảy ra. Bởi vì ta muốn sử dụng React để render lần đầu và sẽ không sử dụng nó lần thứ hai

### Khởi tạo plugin

React cung cấp cho ta một API cho phép tiếp cận actual DOM. Chúng ta sẽ sử dụng thuộc tính `ref` và `componentDidMount` sẽ là phương thức hợp lý mà ta sẽ khởi tạo plugin `tag-it` bên trong nó

```javascript
class Tags extends React.Component {
  ...
  componentDidMount() {
    this.list = $(this.refs.list);
    this.list.tagit();
  }
  render() {
    return (
      <ul ref='list'>
      {
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        )
      }
      </ul>
    );
  }
  ...
```

### Điều khiển plugin trong React

Chúng ta đang cần thêm một tag mới và `tag-it` đang chạy. Việc quan trọng là ta phải tìm cách giao tiếp dữ liệu với `Tags component` nhưng vẫn phải giữ được hướng tiếp cận `single-render`

Để minh họa việc này chúng ta sẽ cần thêm một input vào `App component` và một button để khi click vào ta sẽ truyền một chuỗi vào `Tags component`

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);

    this._addNewTag = this._addNewTag.bind(this);
    this.state = {
      tags: ['JavaScript', 'CSS' ],
      newTag: null
    };
  }
  _addNewTag() {
    this.setState({ newTag: this.refs.field.value });
  }
  render() {
    return (
      <div>
        <p>Add new tag:</p>
        <div>
          <input type='text' ref='field' />
          <button onClick={ this._addNewTag }>Add</button>
        </div>
        <Tags
          tags={ this.state.tags }
          newTag={ this.state.newTag } />
      </div>
    );
  }
}
```

Chúng ta sử dụng `state` cục bộ như là một cách để lưu giá trị của dữ liệu mới. Mỗi lần ta click một button ta sẽ cập nhật lại `state` và trigger render lại cho `Tags` component. Nhưng vì do trong `shouldComponentUpdate` ta đã buộc không cập nhật lại trên màn hình vì thế ta sẽ nhận một giá trị mới của thuộc tính `newTag` thông qua một `lifecycle method` khác đó là `componentWillReceiveProps`

```javascript
class Tags extends React.Component {
  ...
  componentWillReceiveProps(newProps) {
    this.list.tagit('createTag', newProps.newTag);
  }
  ...
```

`.tagit('createTag', newProps.newTag)` là một đoạn code jQuery thuần và `componentWillReceiveProps` là một nơi rất tốt để bạn có thể gọi những thư viện bên thứ ba

Sau đây là phần code được thực hiện đầy đủ:

```javascript
class Tags extends React.Component {
  componentDidMount() {
    this.list = $(this.refs.list);
    this.list.tagit();
  }
  shouldComponentUpdate() {
    return false;
  }
  componentWillReceiveProps(newProps) {
    this.list.tagit('createTag', newProps.newTag);
  }
  render() {
    return (
      <ul ref='list'>
      {
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        )
      }
      </ul>
    );
  }
};
```

### Kết luận
Dường như việc tích hợp thư viện bên thứ ba trở nên rất khó khăn trong cách vận hành của React. Tuy nhiên qua bài viết trên ta có thể thấy bằng những `lifecycle method` có sẵn, ta có thể áp dụng chúng vào việc xử lý quá trình render và tạo ra một cầu nối hoàn hảo giữa những đoạn code React và những đoạn code `non-React`
