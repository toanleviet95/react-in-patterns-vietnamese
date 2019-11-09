# React và những vấn đề về tổ chức mã nguồn
>Cách đây vài năm sau khi Facebook cho ra cú pháp JSX thì một vài người đã đưa ra nhận xét là việc này đã đi ngược lại với vấn đề [`Tổ chức mã nguồn`](https://en.wikipedia.org/wiki/Separation_of_concerns#HTML,_CSS,_JavaScript). Họ nói rằng JSX là một cách pha trộn giữa HTML, CSS và JS điều mà hầu hết mọi người đều muốn chia tách mã nguồn để dễ quản lý

Bài viết này sẽ chứng minh React vẫn có thể tổ chức mã nguồn tốt qua việc phân tách rõ ràng giữa `style`, `logic` và `markup`

### Styling
Việc sử dụng cách styling cũ vẫn có thể áp dụng chỉ khác chỗ về tên thuộc tính `class` được chuyển thành `className`. Chúng ta hoàn toàn có thể style theo kiểu `external` là tách thành một file .css riêng. Hướng tiếp cận này vẫn đảm bảo được tính chia tách mã nguồn mà vẫn dùng được JSX

```javascript
// assets/css/styles.css
.pageTitle {
  color: pink;
}

// assets/js/app.js
function PageTitle({ text }) {
  return <h1 className='pageTitle'>{ text }</h1>;
}
```

Mọi việc trở thành vấn đề khi người ta bắt đầu nói nhiều về `CSS trong JS` từ năm 2014. Tuy nhiên đến những năm tiếp theo thì điều này không phải là quá tệ. Hãy xem một ví dụ sau:

```javascript
function UserCard({ name, avatar }) {
  const cardStyles = {
    padding: '1em',
    boxShadow: '0px 0px 45px 0px #000'
  };
  const avatarStyles = {
    float: 'left',
    display: 'block',
    marginRight: '1em'
  };
  return (
    <div style={cardStyles}>
      <img src={avatar} width="50" style={avatarStyles} />
      <p>{name}</p>
    </div>
  );
}
```

Trong đoạn code trên nhiều người sẽ phản đối việc pha trộn CSS vào markup và để giải quyết vấn đề này ta sẽ thử giữ nguyên cấu trúc của `UserCard` và tổ chức chia tách code thành hai component con là `Card` và `Avatar`

```javascript
function Card({ children }) {
  const styles = {
    padding: '1em',
    boxShadow: '0px 0px 45px 0px #000',
    maxWidth: '200px'
  };
  return <div style={styles}>{children}</div>;
}
function Avatar({ url }) {
  const styles = {
    float: 'left',
    display: 'block',
    marginRight: '1em'
  };
  return <img src={url} width="50" style={styles} />;
}
```

`UserCard` component bây giờ sẽ trông đơn giản hơn vì ta sẽ không còn nhìn thấy những style CSS nữa

```javascript
function UserCard({ name, avatar }) {
  return (
    <Card>
      <Avatar url={avatar} />
      <p>{name}</p>
    </Card>
  );
}
```

Như chúng ta thấy, vấn đề hoàn toàn nằm ở việc tổ chức code của bạn và React hoàn toàn giúp ta tổ chức thành những component có thể tái sử dụng được một cách hoàn toàn linh hoạt. Có nhiều thư viện phổ biến khác hỗ trợ việc `CSS trong JS` có thể kể đến là [`Glamorous`](https://glamorous.rocks/) và [`styled-components`](https://www.styled-components.com/)

### Logic

Logic mà ta thường gặp trong thực tế đó là click vào một button và hiển thị ra một message. Ta có thể minh họa một ví dụ sau:

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      users: null,
      error: null
    };
  }
  componentDidMount() {
    this.setState({ loading: true }, () => {
      fetch('https://jsonplaceholder.typicode.com/users')
        .then(response => response.json())
        .then(users => this.setState({ users, loading: false }))
        .catch(error => this.setState({ error, loading: false }));
    });
  }
  render() {
    const { loading, users, error } = this.state;

    if (isRequestInProgress) return <p>Loading</p>;
    if (error) return <p>Ops, sorry. No data loaded.</p>;
    if (users) return users.map(({ name }) => <p>{name}</p>);
    return null;
  }
}
```

Nhìn đoạn code phía trên có thể thấy có quá nhiều logic phức tạp trong một `App` component bao gồm cả việc truy xuất dữ liệu và hiển thị dữ liệu. Có nhiều cách để tổ chức mã nguồn tốt hơn nhưng cách hay nhất mà tôi chọn là [`FaCC (Function as Child Components)`](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-04/README.md#function-as-a-children-render-prop). Ta sẽ việt một `FetchUsers` component để thực hiện riêng việc API request

```javascript
class FetchUsers extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      users: null,
      error: null
    };
  }
  componentDidMount() {
    this.setState({ loading: true }, () => {
      fetch('https://jsonplaceholder.typicode.com/users')
        .then(response => response.json())
        .then(users => this.setState({ users, loading: false }))
        .catch(error => this.setState({ error, loading: false }));
    });
  }
  render() {
    const { loading, users, error } = this.state;

    return this.props.children({ loading, users, error });
  }
}
```

Bằng cách tách `FetchUsers` chúng ta sẽ mang về một `App` component dạng `stateless` như sau:

```javascript
function App() {
  return (
    <FetchUsers>
      {({ loading, users, error }) => {
        if (loading) return <p>Loading</p>;
        if (error) return <p>Ops, sorry. No data loaded.</p>;
        if (users) return users.map(({ name }) => <p>{name}</p>);
        return null;
      }}
    </FetchUsers>
  );
}
```

Lúc này markup đã được chia tách khỏi logic. Chúng ta hoàn toàn có thể tái sử dụng lại `FetchUsers` component ở bất cứ đâu bạn muốn

### Markup

Một cách mà ta thường thấy trong việc đặt logic vào markup như sau:

```javascript
function CallToActionButton({ service, token }) {
  return <button onClick={ () => service.request(token) } />;
}

<CallToAction server={ service } token={ token } />
```

Trong đoạn code trên bạn hoàn toàn có thể mang phần xử lý logic ra khỏi phần hiển thị như sau:

```javascript
function CallToActionButton({ onButtonClicked }) {
  return <button onClick={ onButtonClicked } />;
}

<CallToAction server={ () => service.request(token) } />
```

### Kết luận
Theo suy nghĩ của tôi React không bao giờ đi ngược lại với cách tổ chức mã nguồn như mọi người thường thấy mà React hoàn toàn có thể giúp bạn tổ chức mã nguồn tốt hơn một khi bạn đã nắm rõ cách phân tách mã nguồn thật hợp lý và rõ ràng
