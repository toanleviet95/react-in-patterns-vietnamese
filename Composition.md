# Tính kết hợp của component
>Một trong những ưu điểm lớn nhất của React chính là tính kết hợp

Làm một ví dụ sau. Ta có một `header` và muốn đặt một `navigation` bên trong. Vì vậy ta cần liên kết những component lại với nhau

```javascript
<App> -> <Header> -> <Navigation>
```

Cách tiếp cận mà thông thường ta hay sử dụng là

```javascript
// app.jsx
import Header from './Header.jsx';

export default function App() {
  return <Header />;
}

// Header.jsx
import Navigation from './Navigation.jsx';

export default function Header() {
  return <header><Navigation /></header>;
}

// Navigation.jsx
export default function Navigation() {
  return (<nav> ... </nav>);
}
```

Tuy nhiên với cách tiếp cận trên ta sẽ gặp 2 vấn đề:

`header` ngoài `navigation` ra còn có rất nhiều những phần tử khác như `logo`, `thanh tìm kiếm` hoặc `slogan`. Vì thế ta cần truyền component vào `App` mà không tao ra sự ràng buộc hoặc trường hợp ta cần sử dụng `header` mà không có `navigation` trong đó. Với cách làm trên ta sẽ vô tình tạo ra sự ràng buộc

Rất khó để test bởi vì việc `import` các component đã vô tình khiến ta phải khởi tạo ra rất nhiều đối tượng để có thể test được toàn bộ

Vì vậy ta sẽ sử dụng các thủ thuật sau để giải quyết vấn đề trên:

### Sử dụng `children` trong React

React cho chúng ta thuộc tính `children`. Đó là cách mà phần tử cha có thể dễ dàng tiếp cận với phần tử con. Với thuộc tính này sẽ làm mất tính ràng buộc cho `header`

```javascript
export default function App() {
  return (
    <Header>
      <Navigation />
    </Header>
  );
}
export default function Header({ children }) {
  return <header>{ children }</header>;
};
```
>Lưu ý: Nếu ta không gọi `children` trong `header` thì `navigation` sẽ không bao giờ được render

### Truyền phần tử con vào như một `prop`

Ta có thể lợi dụng việc truyền `prop` vào component để truyền vào đó một phần tử

```javascript
const Title = function () {
  return <h1>Hello there!</h1>;
}
const Header = function ({ title, children }) {
  return (
    <header>
      { title }
      { children }
    </header>
  );
}
function App() {
  return (
    <Header title={ <Title /> }>
      <Navigation />
    </Header>
  );
};
```

Kĩ thuật này rất hửu ích bởi vì một component như `header` sẽ không cần quan tâm phần tử con của nó là gì khi truyền vào như một `prop`

### HOC (Higher-order component)

`HOC` đã trở thành một cách khá phổ biến để gắn kết những phần tử trong React lại với nhau. Nhìn chung kĩ thuật này kah1 giống với một mẫu thiết kế hướng đối tượng đó là `decorator design pattern` bởi vì nó cho phép tạo ra một component gói các component khác bên trong

Kĩ thuật này cho phép chúng ta viết một function mà ở đó nhận vào tham số là một component gốc và trả về phiên bản chỉnh chu hơn của component đó đã được liên kết với các component khác

```javascript
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component {...this.props} />
      )
    }
  };

var OriginalTitle = () => <h1>Hello world</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);

class App extends React.Component {
  render() {
    return <EnhancedTitle />;
  }
};
```

Với kĩ thuật này trước hết sẽ render component gốc cùng với thuộc tính của component gốc. Với cách làm này ta có thể giữ nguyên được sự toàn vẹn của input truyền vào và đồng thời ta cũng có thể tùy chỉnh được input này từ bên trong `HOC` mà không làm mất đi những thuộc tính mà component gốc truyền vào. Nếu chúng ta có thêm một `configuration setting` để tùy chỉnh ta có thể làm như sau

```javascript
var config = require('path/to/configuration');

var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title }) => <h1>{ title }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

Với cách làm trên ta hoàn toàn có thể cô lập mọi component với nhau. Điều này hỗ trợ cho việc test các component dễ hơn. Một ưu điểm khác của kĩ thuật này đó là chúng ta có thể thêm vào những xử lý logic. Ví dụ như `OriginalTitle` cần dữ liệu lấy từ `remote server`. Chúng ta có thể truy xuất dữ liệu từ trong `HOC` và sau đó truyền dữ liệu vào component con như một `prop`

```javascript
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    constructor(props) {
      super(props);

      this.state = { remoteTitle: null };
    }
    componentDidMount() {
      fetchRemoteData('path/to/endpoint').then(data => {
        this.setState({ remoteTitle: data.title });
      });
    }
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
          remoteTitle={ this.state.remoteTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title, remoteTitle }) =>
  <h1>{ title }{ remoteTitle }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

`OriginalTitle` lúc này chỉ cần biết là nhận vào hai thuộc tính và render ra nên không cần quan tâm dữ liệu đến từ đâu

### Sử dụng `function` như một `children` và thuộc tính `render`

Gần đây cộng đồng React đang chuyển sang một hướng đi mới. Chúng ta sẽ thử truyền một object vào `children prop`

```javascript
function UserName({ children }) {
  return (
    <div>
      <b>{ children.lastName }</b>,
      { children.firstName }
    </div>
  );
}

function App() {
  const user = {
    firstName: 'Krasimir',
    lastName: 'Tsonev'
  };
  return (
    <UserName>{ user }</UserName>
  );
}
```

Cách làm trên tưởng chừng nhử đơn giản nhưng lại rất hữu ích. Ví dụ như bạn muốn hiển thị một danh sách `TODO` và chỉ muốn hiển thị những `TODO` ở trạng thái là `done` đồng thời bạn muốn giữ nguyên render của component `TodoList`

```javascript
function TodoList({ todos, children }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ children(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

function App() {
  const todos = [
    { label: 'Write tests', status: 'done' },
    { label: 'Sent report', status: 'progress' },
    { label: 'Answer emails', status: 'done' }
  ];
  const isCompleted = todo => todo.status === 'done';
  return (
    <TodoList todos={ todos }>
      {
        todo => isCompleted(todo) ?
          <b>{ todo.label }</b> :
          todo.label
      }
    </TodoList>
  );
}
```

`App component` lúc này sẽ quyết định hiển thị theo `status` nào mà ko làm mất đi tính toàn vẹn dữ liệu và công việc của `TodoList` là chỉ cần render và không cần phải xử lý logic render theo `status` gì. Kĩ thuật trên gọi là lợi dụng. Lần này thay vì lợi dụng `children` ta sẽ sử dụng một thuộc tính tên là `render`

```javascript
function TodoList({ todos, render }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ render(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

return (
  <TodoList
    todos={ todos }
    render={
      todo => isCompleted(todo) ?
        <b>{ todo.label }</b> : todo.label
    } />
);
```

Cả hai cách trên gồm dùng `function` như một `children` và thuộc tính `render` là một trogn những cách được nhiều người yêu thích. Tạo nên sự linh hoạt trong việc tái sử dụng code

```javascript
class DataProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = { data: null };
    setTimeout(() => this.setState({ data: 'Hey there!' }), 5000);
  }
  render() {
    if (this.state.data === null) return null;
    return (
      <section>{ this.props.render(this.state.data) }</section>
    );
  }
}
```

`DataProvider` sẽ không render gì hết. 5 giây sau nó sẽ update `state` và render một thẻ `section` dựa vào thuộc tính `render` trả về gì

```javascript
<DataProvider render={ data => <p>The data is here!</p> } />
```

Một ứng dụng khác khi sử dụng kỹ thuật này đó là giới hạn việc hiển thị UI cho một số user nhất định. Ví dụ như quyền `read:products`. Ta sẽ sử dụng kĩ thuật `render prop` như sau

```javascript
<Authorize
  permissionsInclude={[ 'read:products' ]}
  render={ () => <ProductsList /> } />
```

### Kết luận
Nếu bạn vẫn đang thắc mắc tại sao HTML vẫn còn ở đây ? Câu trả lời là bởi vì nó vẫn tạo ra được sự kết nối. React và JSX cũng như vậy. Vì vậy bạn hảy đảm bảo rằng bạn đã thực sự rành về tính kết nối của các component. Đó là một trong những ưu điểm lớn nhất mà React mang lại
