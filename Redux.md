# Thư viện Redux
>[`Redux`](https://redux.js.org) là một thư viện đóng vai trò như một `state container` giúp chúng ta quản lý luồng dữ liệu. Được `Dan Abramov` giới thiệu vào năm 2015 tại hội nghị [`ReactEurope`](https://www.youtube.com/watch?v=xsSnOQynTHs). Redux có nhiều nét tương đồng với kiến trúc Flux

### Yếu tố chính trong Redux

![Basic-redux-architect](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-09/redux-architecture.jpg)

Tương tự với `Flux`, `view` sẽ dispatch `action` vào `store`. Điểm khác nhau giữa `Flux` và `Redux` là Redux chỉ có một `store` duy nhất


#### Action

`Action` trong Redux là một object với thuộc tính `type`. Ví dụ:

```javascript
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';
const action = {
  type: CHANGE_VISIBILITY,
  visible: false
}
```

Sẽ tốt hơn nếu ta sử dụng một constant `CHANGE_VISIBILITY` cho action type. `visible` chỉ là một meta data và không làm ảnh hưởng nhiều trong Redux, nó chỉ mang ý nghĩa khi trong ứng dụng bạn cần

Mỗi khi chúng ta muốn dispatch một action ta đều phải truyền một object như vậy. Tuy nhiên để tránh việc viết đi viết lại thì chúng ta sẽ có một `action creator` - một function trả về một object với tham số được truyền vào

```javascript
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});

changeVisibility(false);
// { type: CHANGE_VISIBILITY, visible: false }
```

Chúng ta sẽ không cần phải nhớ truyền vào một action type mà chỉ cần truyền vào tham số bạn muốn

#### Store

Redux cung cấp một helper hữu ích là `createSore` để tạo một store

```javascript
import { createStore } from 'redux';

createStore([reducer], [initial state], [enhancer]);
```

`reducer` là một hàm nhận vào `state` hiện tại và `action` rồi trả về một `state` mới. `initial state` khởi tạo dữ liệu cho `state`. `enhancer` giúp Redux liên kết với một `middleware` bên thử ba

Ngoài `createStore` ta còn có `getState`, `dispatch`, `subcribe` và `replaceReducer`. Và quan trọng nhất vẫn là `dispatch`

```javascript
store.dispatch(changeVisibility(false));
```

#### Reducer

`Reducer` là thành phần hay nhất của Redux. Trong reducer ta cần phải viết thành `pure function` để đảm bảo tính bất biến. Hai đặc trưng cơ bản của reducer:

* Phải viết dưới dạng `pure function` - function phải trả về chính xác giá trị output mỗi lần có input truyền vào

* Không nên có `side effect` - Những thứ như biến `global`, gọi `async` hoặc `promise` không được xuất hiện ỏ đây

Ví dụ:
```javascript
const counterReducer = function (state, action) {
  if (action.type === ADD) {
    return { value: state.value + 1 };
  } else if (action.type === SUBTRACT) {
    return { value: state.value - 1 };
  }
  return { value: 0 };
};
```

Không được xuất hiện `side effect` và cần phải trả về một object mới dựa trên `state` cũ và `action type`

#### Cách sử dụng Redux trong React

Ta cần sử dụng một module gọi là [`react-redux`](https://github.com/reactjs/react-redux) vì nó cung cấp cho ta hai thứ để giúp ta liên kết Redux với các component của React

* `<Provider>` component là một component nhận vào `store` và truyền xuống các component con bọc bên trong thông qua khái niệm `context` trong React

```javascript
<Provider store={ myStore }>
  <MyApp />
</Provider>
```

* `connect` là một hàm thực hiện `subcribe` những cập nhật vào `store` và render lại các component. Nó được thực hiện như một `HOC (Higher Order Component)`

```javascript
connect(
  [mapStateToProps],
  [mapDispatchToProps],
  [mergeProps],
  [options]
)
```

`mapStateToProps` là một hàm nhận vào `state` hiện tại và trả về một object và object này sẽ được gửi xuống component như một `prop`

```javascript
const mapStateToProps = state => ({
  visible: state.visible
});
```

`mapDispatchToProps` cũng là một hàm tương tự nhưng thay vì phải nhận vào `state` nó sẽ nhận vào một hàm `dispatch`

```javascript
const mapDispatchToProps = dispatch => ({
  changeVisibility: value => dispatch(changeVisibility(value))
});
```

`mergeProps` thực hiện việc gộp `mapStateToProps` và `mapDispatchToProps` cùng với những props mà bạn muốn. Ta có thể thực hiện hai action và gộp chúng lại thành một prop duy nhất

`options` cho phép bạn thiết lập tùy chỉnh cách kết nối

### Làm một ví dụ đơn giản

Ta sẽ viết thử một `counter app` sử dụng React và Redux

![Example-react-redux](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-09/redux-counter-app.png)

Hai nút `Add` và `Subtract` sẽ thay đổi giá trị trong `store`. Hai nút `Visible` và `Hidden` sẽ thực hiện việc ẩn hiện

#### Action

Chúng ta sẽ có 3 action chính: `ADD`, `SUBTRACT`, `CHANGE_VISIBILITY` đi kèm với `creator function` tương ứng:

```javascript
const ADD = 'ADD';
const SUBTRACT = 'SUBTRACT';
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';

const add = () => ({ type: ADD });
const subtract = () => ({ type: SUBTRACT });
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});
```

#### Store và Reducer

State khởi tạo sẽ có dạng:

```javascript
const initialState = {
  counter: {
    value: 0
  },
  visible: true
};
```

Reducer của bạn có thể sẽ được chia nhỏ thành nhiều phần trong project vì vậy Redux có hỗ trợ chúng ta một hàm hỗ trợ việc gom các reducer lại gọi là `combineReducers`:

```javascript
import { createStore, combineReducers } from 'redux';

const rootReducer = combineReducers({
  counter: function A() { ... },
  visible: function B() { ... }
});
const store = createStore(rootReducer);
```

Chúng ta sẽ viết một reducer cho việc `counter` và làm hai hành động `ADD` và `SUBTRACT` dựa trên `state` counter:

```javascript
const counterReducer = function (state, action) {
  if (action.type === ADD) {
    return { value: state.value + 1 };
  } else if (action.type === SUBTRACT) {
    return { value: state.value - 1 };
  }
  return state || { value: 0 };
};
```

> Lưu ý: Ở lần chạy đầu tiên `state` sẽ có giá trị `undefined` và `action` sẽ là `{ type: "@@redux/INIT" }`. Vì vậy ta cần phải có một giá trị khởi tạo cho `state` ví dụ: `{ value: 0 }`

Chúng ta sẽ viết tiếp một reducer cho việc ẩn hiện với action `CHANGE_VISIBILITY`:

```javascript
const visibilityReducer = function (state, action) {
  if (action.type === CHANGE_VISIBILITY) {
    return action.visible;
  }
  return true;
};
```

Và cuối cùng là gộp các reducer này lại bằng hàm `combineReducers` vào một `rootReducer`:

```javascript
const rootReducer = combineReducers({
  counter: counterReducer,
  visible: visibilityReducer
});
```

#### Selector

Trước khi đến phần viết React component ta sẽ đi qua một chút khái niệm về `Selector`. `Selector` là một hàm nhận vào `state` object và chọn lọc ra những thông tin mà ta thực sự cần. Ví dụ:

```javascript
const getCounterValue = state => state.counter.value;
const getVisibility = state => state.visible;
```

Trong phạm vi `counter app` chỉ là một app đơn giản nên ta không thể thấy hết được tầm quan trọng của việc sử dụng `Selector`. Tuy nhiên trong những project lớn việc sử dụng `selector` sẽ giúp bạn tiết kiệm những dòng code và đồng thời cho bạn thông tin đã được chọn lọc mà bạn nghĩ là cần thiết thay vì ta phải sử dụng toàn bộ `state` object

#### React component

Kế tiếp là việc trình bày UI

```javascript
function Visibility({ changeVisibility }) {
  return (
    <div>
      <button onClick={ () => changeVisibility(true) }>
        Visible
      </button>
      <button onClick={ () => changeVisibility(false) }>
        Hidden
      </button>
    </div>
  );
}

const VisibilityConnected = connect(
  null,
  dispatch => ({
    changeVisibility: value => dispatch(changeVisibility(value))
  })
)(Visibility);
```

`VisibilityConnected` component sử dụng connect truyền giá trị `null` vào `mapStateToProps` bởi vì ta không cần lấy dữ liệu này từ `store` chúng ta chỉ cần `dispatch` action cho hành động ẩn hiện

Component thứ hai sẽ là `CounterConnected`:

```javascript
function Counter({ value, add, subtract }) {
  return (
    <div>
      <p>Value: { value }</p>
      <button onClick={ add }>Add</button>
      <button onClick={ subtract }>Subtract</button>
    </div>
  );
}

const CounterConnected = connect(
  state => ({
    value: getCounterValue(state)
  }),
  dispatch => ({
    add: () => dispatch(add()),
    subtract: () => dispatch(subtract())
  })
)(Counter);
```

Chúng ta cần phải truyền cả `mapStateToProps` và `mapDispatchToProps` bởi vì ta muốn vừa đọc dữ liệu từ `store` vừa phải `dispatch` action. Vì vậy component trên sẽ có được 3 prop: `value`, `add` và `subtract`

Việc cuối cùng là viết một `App` component cha bọc hai component con ở trên:

```javascript
function App({ visible }) {
  return (
    <div>
      <VisibilityConnected />
      { visible && <CounterConnected /> }
    </div>
  );
}
const AppConnected = connect(
  state => ({
    visible: getVisibility(state)
  })
)(App);
```

### Kết luận

Redux là một kiến trúc tuyệt vời khi kế thừa được những cái hay nhất từ kiến trúc `Flux`. Sơ đồ hoàn chỉnh của kiến trúc Redux sẽ là:

![Complete-redux-architect](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-09/redux-reallife.jpg)
