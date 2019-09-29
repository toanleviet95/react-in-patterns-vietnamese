# Nâng cao hơn với kỹ thuật Dependency Injection, React Context và Module System
>Trong thực tế có rất nhiều module hoặc component chúng ta viết bị lệ thuộc nhau. Việc quản lý sự phụ thuộc này cũng đóng vai trò quan trọng trong dự án. Có một kỹ thuật gọi là [`dependency injection`](http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript) giúp ta xử lý sự phụ thuộc trên

Xem một ví dụ sau:

```javascript
// Title.jsx
export default function Title(props) {
  return <h1>{ props.title }</h1>;
}

// Header.jsx
import Title from './Title.jsx';

export default function Header() {
  return (
    <header>
      <Title />
    </header>
  );
}

// App.jsx
import Header from './Header.jsx';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { title: 'React in patterns' };
  }
  render() {
    return <Header />;
  }
};
```

Nếu chúng ta muốn truyền một chuỗi `"React in patterns"` đến component `Title` thì cách thường làm sẽ là truyền từ `App` đến `Header` rồi từ `Header` đến `Title`. Trong ví dụ trên là đi qua 3 lớp component tuy nhiên nếu số lượng component trở nên nhiều hơn và phân cấp nhiều hơn thì việc truyền thuộc tính như một `proxy` sẽ trở nên khó kiểm soát hơn

Chúng ta đã đọc qua ![`HOC (Higher-order component)`](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-04/#higher-order-component) - Một cách để tách phần dữ liệu. Chúng ta thử vận dụng vào ví dụ trên về cách tách dữ liệu nhé:

```javascript
// inject.jsx
const title = 'React in patterns';

export default function inject(Component) {
  return class Injector extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ title }
        />
      )
    }
  };
}

// -----------------------------------
// Header.jsx
import inject from './inject.jsx';
import Title from './Title.jsx';

var EnhancedTitle = inject(Title);
export default function Header() {
  return (
    <header>
      <EnhancedTitle />
    </header>
  );
}
```

Biến `title` đã được giấu đi nhờ `HOC` khi ta truyền thuộc tính vào `Title` component

### React Context (phiên bản < 16.3)

Ở phiên bản 16.3 React giới thiệu một API gọi là ![`context`](https://facebook.github.io/react/docs/context.html) - `context` là thứ mà mọi component đều có thể tiếp cận được. Ta có thể xem nó như một `store` đơn giản

```javascript
// a place where we will define the context
var context = { title: 'React in patterns' };

class App extends React.Component {
  getChildContext() {
    return context;
  }
  ...
};
App.childContextTypes = {
  title: React.PropTypes.string
};

// a place where we use the context
class Inject extends React.Component {
  render() {
    var title = this.context.title;
    ...
  }
}
Inject.contextTypes = {
  title: React.PropTypes.string
};
```

>Lưu ý: Khi sử dụng context ta cần khai báo đúng tên object trong context với `childContextTypes` và `contextTypes`

Từ khái niệm `context` ta có thể tự viết một dependency object như sau:

```javascript
// dependencies.js
export default {
  data: {},
  get(key) {
    return this.data[key];
  },
  register(key, value) {
    this.data[key] = value;
  }
}
```

`App` component bây giờ sẽ trông như sau:

```javascript
import dependencies from './dependencies';

dependencies.register('title', 'React in patterns');

class App extends React.Component {
  getChildContext() {
    return dependencies;
  }
  render() {
    return <Header />;
  }
};
App.childContextTypes = {
  data: React.PropTypes.object,
  get: React.PropTypes.func,
  register: React.PropTypes.func
};
```

`Title` component đơn giản bây giờ chỉ cần lấy dữ liệu từ context object:

```javascript
// Title.jsx
export default class Title extends React.Component {
  render() {
    return <h1>{ this.context.get('title') }</h1>
  }
}
Title.contextTypes = {
  data: React.PropTypes.object,
  get: React.PropTypes.func,
  register: React.PropTypes.func
};
```

Một cách hay hơn nếu chúng ta không muốn chỉ ra cụ thể `contextTypes` ở mỗi component là hãy gói chúng vào một `HOC` hoặc hay hơn nữa là viết thành một `utility function` gọi là `wire` để hỗ trợ việc đó:

```javascript
// Title.jsx
import wire from './wire';

function Title(props) {
  return <h1>{ props.title }</h1>;
}

export default wire(Title, ['title'], function resolve(title) {
  return { title };
});
```

Hàm `wire` nhận vào một React component. Tham số thứ hai là mảng tên dependency bạn cần đăng ký. tham số thứ ba là hàm `mapper`. Giá trị trả về sẽ là một object và sử dụng như một `prop` trong component

```javascript
export default function wire(Component, dependencies, mapper) {
  class Inject extends React.Component {
    render() {
      var resolved = dependencies.map(
        this.context.get.bind(this.context)
      );
      var props = mapper(...resolved);

      return React.createElement(Component, props);
    }
  }
  Inject.contextTypes = {
    data: React.PropTypes.object,
    get: React.PropTypes.func,
    register: React.PropTypes.func
  };
  return Inject;
};
```

`Inject` bây giờ sẽ là một `HOC` cho phép ta truy xuất mọi context được khai báo trong mảng dependency và hàm `mapper` sẽ nhận dữ liệu và chuyển nó thành `prop` trong component

### React Context (phiên bản >= 16.3)

Từ phiên bản 16.3 trở đi Facebook khuyến khích ta không nên tiếp cận `context` theo cách cũ nữa mà thay vào đó là một chút thay đổi. Chúng ta có thể viết tách context ra một file riêng như sau:

```javascript
// context.js
import { createContext } from 'react';

const Context = createContext({});

export const Provider = Context.Provider;
export const Consumer = Context.Consumer;
```

`createContext` trả về một object có hai thuộc tính là `.Provider` và `.Consumer`. `Provider` sẽ nhận dữ liệu và `Consumer` sẽ dùng để truy xuất dữ liệu. `App` component sẽ vie6t11 như sau:

```javascript
import { Provider } from './context';

const context = { title: 'React In Patterns' };

class App extends React.Component {
  render() {
    return (
      <Provider value={ context }>
        <Header />
      </Provider>
    );
  }
};
```

Những component được bọc bên trong `Provider` và những component con của nó sẽ có thể chia sẻ nhau dữ lei6u5 từ `context`. Chúng ta sẽ sử dụng `Consumer` trong `Title` component như sau:

```javascript
import { Consumer } from './context';

function Title() {
  return (
    <Consumer>{
      ({ title }) => <h1>Title: { title }</h1>
    }</Consumer>
  );
}
```

>Lưu ý: `Consumer` sử dụng kỹ thuật `render prop` để gửi đi context

Sử dụng context theo hướng mới này sẽ giúp chúng ta dễ hiểu hơn, code rõ ràng và đẹp hơn

### Sử dụng Module System

Nếu bạn không muốn sử dụng `context` vẫn còn một cách khác tạo ra `injection` đó là tận dụng `Module System` của Node.js

Như chúng ta đã biết cơ chế `caching` của Module System được trích từ tài liệu ![Node.js](https://nodejs.org/api/modules.html#modules_caching):

>Module được cache sau mỗi lần chúng được load. Điều này nghĩa là mỗi lần gọi `require('foo')` chúng ta đều nhận được cùng một object
>Việc bạn gọi `require('foo')` cũng không làm cho đoạn code trong module được thực thi nhiều lần. Đây là một lưu ý khá quan trọng

Vậy việc này giúp ích gì cho việc `injection` của bạn ? Nếu bạn export một object cụ thể là một dạng `singleton` và mỗi khi module khác import vào sẽ đều cùng nhận một object. Điều này cho phép bạn `register` dependency và sau cùng bạn chỉ cần `fetch` chúng ở file khác

Hãy viết thử một `di.jsx` như sau:

```javascript
var dependencies = {};

export function register(key, dependency) {
  dependencies[key] = dependency;
}

export function fetch(key) {
  if (dependencies[key]) return dependencies[key];
  throw new Error(`"${ key } is not registered as dependency.`);
}

export function wire(Component, deps, mapper) {
  return class Injector extends React.Component {
    constructor(props) {
      super(props);
      this._resolvedDependencies = mapper(...deps.map(fetch));
    }
    render() {
      return (
        <Component
          {...this.state}
          {...this.props}
          {...this._resolvedDependencies}
        />
      );
    }
  };
}
```

Chúng ta sẽ lưu trữ `dependencies` như một biến global (Phạm vi chỉ global trong module chứ không phải toàn bộ ứng dụng). Kế tiếp là ta export hai hàm `register` và `fetch` để đóng vai trò `write` và `read`. Bạn có thể liên tưởng giống với `setter` và `getter` và kế đó là viết hàm `wire` dạng `HOC` như đã nói ở trên

Khi đã có được một file helper như `di.jsx`, công việc bây giờ là inject vào `app.jsx` và `Title.jsx`:

```javascript
// app.jsx
import Header from './Header.jsx';
import { register } from './di.jsx';

register('my-awesome-title', 'React in patterns');

class App extends React.Component {
  render() {
    return <Header />;
  }
};

// -----------------------------------
// Header.jsx
import Title from './Title.jsx';

export default function Header() {
  return (
    <header>
      <Title />
    </header>
  );
}

// -----------------------------------
// Title.jsx
import { wire } from './di.jsx';

var Title = function(props) {
  return <h1>{ props.title }</h1>;
};

export default wire(
  Title,
  ['my-awesome-title'],
  title => ({ title })
);
```

### Kết luận

`Dependency injection` là một kỹ thuật đáng để học. Rất nhiều người vẫn không nhìn thấy được việc quản lý tốt những dependency đóng vai trò quan trọng thế nào trong việc phát triển ứng dụng. Ngoài `Dependency injection` vẫn còn rất nhiều những kỹ thuật khác mà bạn có thể chọn cho mình một kỹ thuật để áp dụng
