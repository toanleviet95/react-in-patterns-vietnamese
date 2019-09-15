# Kiến trúc Flux
>[`Flux`](http://facebook.github.io/flux) là một mẫu kiến trúc sử dụng cho UI và được Facebook giới thiệu tại hội nghị [F8](https://www.youtube.com/watch?v=nYkdrAPrdcw&feature=youtu.be&t=568). Từ đó mà có nhiều công ty đã từ ý tưởng đó mà đưa ra mẫu thiết kế xây dựng front-end. Có một thư viện được phát hành bởi Facebook đó chính là [`Redux`](https://redux.js.org). Nhờ vào mẫu thiết kế này chúng ta có thể vừa tạo ra ứng dụng nhanh hơn vừa tổ chức code tốt hơn

### Yếu tố chính trong kiến trúc Flux

![Basic-flux-architect](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-08/fluxiny_basic_flux_architecture.jpg)

Yếu tố chính vẫn là `dispatcher`. Nó đóng vai trò tổ chức các sự kiện trong hệ thống. Công việc của `dispatcher` là nhận thông báo rằng ta cần thực hiện `action` và truyền chúng vào `store`. `Store` quyết định việc thay đổi `state` hay không. Việc thay đổi `state` sẽ làm render lại những React components hay còn gọi là `view`. Nếu so sánh `Flux` với một kiến trúc nổi tiếng như `MVC` thì có thể thấy `store` tương tự với model

Các `action` đến `dispatcher` từ `view` giống như những `service`. Ví dụ một module thực hiện một HTTP request, khi nó nhận kết quả nó sẽ kích hoạt một `action` để báo rằng request đã thực hiện thành công

### Thực thi một kiến trúc Flux

#### Dispatcher

![Dispatcher](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-08/fluxiny_the_dispatcher.jpg)

```javascript
var Dispatcher = function() {
  return {
    _stores: [],
    register: function(store) {
      this._stores.push({ store });
    },
    dispatch: function(action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function(entry) {
          entry.store.update(action);
        })
      }
    }
  }
};
```

Điều đầu tiên ta nhận ra từ đoạn code trên là cần có phương thức `update` trong store. Vì vậy ta cần kiểm tra thêm phương thức này đã tồn tại chưa

```javascript
register: function (store) {
  if (!store || !store.update) {
    throw new Error('You should provide a store that has an `update` method.');
  } else {
    this._stores.push({ store: store });
  }
}
```

#### Liên kết view với store

![Store-change-view](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-08/fluxiny_store_change_view.jpg)

* Sử dụng context
`Context` là một cách để truyền prop đến component con mà không cần khai báo prop tại mỗi cấp bậc của cây component. Facebook khuyên chúng ta nên sử dụng context trong tình huốn cần truyền dữ liệu đến những `nested component`. Đây cũng là một cách giúp ta liên kết `view` với `store` tuy nhiên dùng cách này việc khai báo dữ liệu sẽ không được rõ ràng

* Sử dụng HOC (Higher Order Component)

`HOC` được [giới thiệu](https://gist.github.com/sebmarkbage/ef0bf1f338a7182b6775) bởi Sebastian Markbåge với ý tưởng tạo một component bọc ngoài kết quả trả về

```javascript
function attachToStore(Component, store, consumer) {
  const Wrapper = React.createClass({
    getInitialState() {
      return consumer(this.props, store);
    },
    componentDidMount() {
      store.onChangeEvent(this._handleStoreChange);
    },
    componentWillUnmount() {
      store.offChangeEvent(this._handleStoreChange);
    },
    _handleStoreChange() {
      if (this.isMounted()) {
        this.setState(consumer(this.props, store));
      }
    },
    render() {
      return <Component {...this.props} {...this.state} />;
    }
  });
  return Wrapper;
};
```

`Component` là view mà ta muốn gắn vào `store`. Hàm `consumer` nói rằng `state` của `store` sẽ được lấy ra và gửi vào `view`. Cách sử dụng HOC sẽ như sau:

```javascript
class MyView extends React.Component {
  ...
}

ProfilePage = connectToStores(MyView, store, (props, store) => ({
  data: store.get('key')
}));
```

Tuy nhiên hướng tiếp cận này sẽ khó khăn khi ta càng có nhiều hơn component được bọc vào trong và ta cũng cần phải truyền vào 3 yếu tố: `view`, `store` và `consumer` để ta có thể thiết lập ra sự kết nối

* Phương án cuối cùng?
Dựa vào HOC ở trên ta có thể có một cách làm tương tự tuy nhiên ta sẽ không phải lúc nào cũng truyền `store` vào. Nói tóm gọn là ta chỉ cần một hàm nhận vào `consumer` và hàm này sẽ được gọi mỗi khi có sự thay đổi trong `store`

Thay vì ta không trả về gì cả trong hàm register thì lần này ta sẽ trả về một `subcriber`

![Fluxing-store-view](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-08/fluxiny_store_view.jpg)

Lần này hàm `register` sẽ viết như sau:

```javascript
register: function (store) {
  if (!store || !store.update) {
    throw new Error(
      'You should provide a store that has an `update` method.'
    );
  } else {
    var consumers = [];
    var subscribe = function (consumer) {
      consumers.push(consumer);
    };

    this._stores.push({ store: store });
    return subscribe;
  }
  return false;
}
```

Vẫn còn một chút vấn đề đó là làm thế nào để `store` nói rằng `state` đã thay đổi. Vì vậy trong hàm `update` ta truyền `action` nhưng đồng thời cũng phải truyền một hàm `change`

```javascript
register: function (store) {
  if (!store || !store.update) {
    throw new Error(
      'You should provide a store that has an `update` method.'
    );
  } else {
    var consumers = [];
    var change = function () {
      consumers.forEach(function (consumer) {
        consumer(store);
      });
    };
    var subscribe = function (consumer) {
      consumers.push(consumer);
    };

    this._stores.push({ store: store, change: change });
    return subscribe;
  }
  return false;
},
dispatch: function (action) {
  if (this._stores.length > 0) {
    this._stores.forEach(function (entry) {
      entry.store.update(action, entry.change);
    });
  }
}
```

Lưu ý rằng ta sẽ phải push `change` cùng với `store` vào mảng `_stores`. Sau đó ở hàm `dispatch` ta sẽ gọi `update` và truyền đồng thời `action` cùng hàm `change`

Ta sẽ cần thêm `state` khởi tạo ban đầu và thực hiện trong hàm `subcribe`

```javascript
var subscribe = function (consumer, noInit) {
  consumers.push(consumer);
  !noInit ? consumer(store) : null;
};
```

Đầy đủ đoạn code của dispatcher sẽ là:

```javascript
var Dispatcher = function () {
  return {
    _stores: [],
    register: function (store) {
      if (!store || !store.update) {
        throw new Error(
          'You should provide a store that has an `update` method'
        );
      } else {
        var consumers = [];
        var change = function () {
          consumers.forEach(function (consumer) {
            consumer(store);
          });
        };
        var subscribe = function (consumer, noInit) {
          consumers.push(consumer);
          !noInit ? consumer(store) : null;
        };

        this._stores.push({ store: store, change: change });
        return subscribe;
      }
      return false;
    },
    dispatch: function (action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function (entry) {
          entry.store.update(action, entry.change);
        });
      }
    }
  }
};
```

#### Action
Cách thực hiện `action` thường theo chuẩn gồm 2 thuộc tính quan trọng là `type` và `payload`

```javascript
{
  type: 'USER_LOGIN_REQUEST',
  payload: {
    username: '...',
    password: '...'
  }
}
```

`type` nói cho ta biết đây là action gì và `payload` chứa thông tin liên quan đến event. Trong một vài tính huống thì `payload` có thể rỗng

```javascript
var createAction = function (type) {
  if (!type) {
    throw new Error('Please, provide action\'s type.');
  } else {
    return function (payload) {
      return dispatcher.dispatch({
        type: type,
        payload: payload
      });
    }
  }
}
```

`createAction` mang lại những lợi ích sau:
* Không cần phải nhớ chính xác `type` của action vì chỉ cần quan tâm payload
* Chúng ta không cần đi vào `dispatcher` để code mà `action` vẫn có thể nằm ở một nơi riêng trong project

![Action-creator](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-08/fluxiny_action_creator.jpg)

`Action creator` là một hướng tiếp cận kah1 là phổ biến mà mọi người thường hay sử dụng

#### Đầy đủ đoạn code thực thi

Chúng ta sẽ viết thêm một hàm `createSubscriber`. Trong đó ta sẽ đăng ký `store`

```javascript
var createSubscriber = function (store) {
  return dispatcher.register(store);
}
```

Ta sẽ đóng gói `createSubscriber` và `createAction` thành một module

```javascript
var Dispatcher = function () {
  return {
    _stores: [],
    register: function (store) {
      if (!store || !store.update) {
        throw new Error(
          'You should provide a store that has an `update` method'
        );
      } else {
        var consumers = [];
        var change = function () {
          consumers.forEach(function (consumer) {
            consumer(store);
          });
        };
        var subscribe = function (consumer, noInit) {
          consumers.push(consumer);
          !noInit ? consumer(store) : null;
        };

        this._stores.push({ store: store, change: change });
        return subscribe;
      }
      return false;
    },
    dispatch: function (action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function (entry) {
          entry.store.update(action, entry.change);
        });
      }
    }
  }
};

module.exports = {
  create: function () {
    var dispatcher = Dispatcher();

    return {
      createAction: function (type) {
        if (!type) {
          throw new Error('Please, provide action\'s type.');
        } else {
          return function (payload) {
            return dispatcher.dispatch({
              type: type,
              payload: payload
            });
          }
        }
      },
      createSubscriber: function (store) {
        return dispatcher.register(store);
      }
    }
  }
};
```

Với 66 dòng code trên ước tính sẽ chỉ mất 1.7 KB (795 byte khi minimize)

### Làm một ví dụ đơn giản

Ta sẽ vận dụng lại và làm một ví dụ đơn giản là một app counter
UI gồm có 2 nút:

```javascript
<div id="counter">
  <span></span>
  <button>increase</button>
  <button>decrease</button>
</div>
```

Thẻ `span` sẽ sử dụng để hiển thị giá trị của `counter`

#### View
```javascript
const View = function (subscribeToStore, increase, decrease) {
  var value = null;
  var el = document.querySelector('#counter');
  var display = el.querySelector('span');
  var [ increaseBtn, decreaseBtn ] =
    Array.from(el.querySelectorAll('button'));

  var render = () => display.innerHTML = value;
  var updateState = (store) => value = store.getValue();

  subscribeToStore([updateState, render]);

  increaseBtn.addEventListener('click', increase);
  decreaseBtn.addEventListener('click', decrease);
};
```

#### Store
Tạo ra biến `constant` định danh action

```javascript
const INCREASE = 'INCREASE';
const DECREASE = 'DECREASE';
```

`CounterStore` viết dưới dạng singleton

```javascript
const CounterStore = {
  _data: { value: 0 },
  getValue: function () {
    return this._data.value;
  },
  update: function (action, change) {
    if (action.type === INCREASE) {
      this._data.value += 1;
    } else if (action.type === DECREASE) {
      this._data.value -= 1;
    }
    change();
  }
};
```

`_data` là `state` khởi tạo ban đầu. `update` sẽ được `dispatcher` gọi. Các action sau khi thực hiện xong sẽ gọi `change()` khi kết thúc. `getValue` là phương thức public để trả về giá trị của counter

Giống như phần thực hiện kiến trúc flux ta sẽ có một module `Fluxiny` với hàm `create`. Kế đó ta sẽ ghép đoạn code từ `CounterStore` và phần `View` lại

```javascript
const { createAction, createSubscriber } = Fluxiny.create();
const counterStoreSubscriber = createSubscriber(CounterStore);
const actions = {
  increase: createAction(INCREASE),
  decrease: createAction(DECREASE)
};

View(counterStoreSubscriber, actions.increase, actions.decrease);
```

### Bản demo
JSBin: http://jsbin.com/koxidu/embed?js,output

Github: https://github.com/krasimir/fluxiny/tree/master/example
