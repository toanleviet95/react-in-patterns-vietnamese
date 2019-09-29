# Styling - Các cách Style CSS trong React
>Có rất nhiều cách tiếp cận để giúp bạn styling trong ứng dụng React và trong phạm vi bài viết tôi sẽ giới thiệu các bạn những cách phổ biến nhất

### CSS class theo cách cũ

Cú pháp của JSX khá giống với HTML nde6n bạn hoàn toàn có thể dùng cách khai báo class CSS như kiểu cũ nhưng thay vì sử dụng thuộc tính `class` thì ta sẽ sử dụng `className`

```javascript
<h1 className='title'>Styling</h1>
```

### Inline styling

Inline styling cũng là một cách. Giống như HTML thì JSX cũng truyền tru7c6 tiếp vào thuộc tính `style` nhưng thay vì là một chuỗi giá trị thì thay vào đó là một object

```javascript
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline styling</h2>
```

Nếu bạn muốn giữ nguyên tên thuộc tính CSS như thông thường bạn hãy để tên vào dấu nháy còn không thì hãy tuân theo cách viết camel case. Tuy nhiên khi viết style trong javascript còn có một ưu d9ei6m3 khác giúp bạn linh hoạt hơn

```javascript
const theme = {
  fontFamily: 'Georgia',
  color: 'blue'
};
const paragraphText = {
  ...theme,
  fontSize: '20px'
};
```

Chúng ta có thể tạo ra một basic style trong `theme` và nhập chúng vào `paragraphText` đó là một cách tận dụng sức mạnh của Javascript vào CSS

### CSS module

[CSS module](https://github.com/css-modules/css-modules/blob/master/docs/get-started.md) là một cách viết tách phần CSS ra một module riêng lẻ. Điều này giúp bạn truy xuất CSS module ở nhiều nơi trong ứng dụng

```javascript
/* style.css */
.title {
  color: green;
}

// App.jsx
import styles from "./style.css";

function App() {
  return <h1 style={ styles.title }>Hello world</h1>;
}
```

Ngoài ra bạn vẫn có thể liên kết truy xuất các file CSS khác ngay trong một file CSS bằng `composes`

```javascript
.title {
  composes: mainColor from "./brand-colors.css";
}
```

### Styled-component

[`Styled-component`](https://www.styled-components.com/) là một hướng tiếp cận khác. Nó tạo ra một React Component ke6t11 hợp với việc Inline styling. Ví dụ bạn có thể tạo ra một component `Link` từ việc styling thẻ `<a>`

```javascript
const Link = styled.a`
  text-decoration: none;
  padding: 4px;
  border: solid 1px #999;
  color: black;
`;

<Link href='http://google.com'>Google</Link>
```

Ngoài ra vẫn có thêm cơ chế kế thừa styling. Bạn có thể tạo ra một component `AnotherLink` từ component `Link` có sẵn:

```javascript
const AnotherLink = styled(Link)`
  color: blue;
`;

<AnotherLink href='http://facebook.com'>Facebook</AnotherLink>
```

Theo tôi `Styled-component` là một hướng tiếp cận rất thú vị. Bạn có thể dễ dàng tạo ra một component mới mà không nhớ đến việc phải styling ra sao. Nếu công ty bạn có sẵn một `Design system` riêng thì đây chính xác là cách phù hợp nhất để tạo ra những component riêng phì hợp với sản phẩm công ty

### Kết luận

Có nhiều kiểu styling trong ứng dụng React. Mỗi cách đều có một thế mạnh riêng. Bạn nên cân nhắc lựa chọn cho mình một cách sao cho phù hợp với dự án của bạn nhất
