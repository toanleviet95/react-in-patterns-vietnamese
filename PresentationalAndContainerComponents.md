# Tách component theo dạng hiển thị & xử lý logic
>Bạn đã từng thử đặt câu hỏi: "Khi tôi truyền dữ liệu làm sao để tôi có thể quản lý state và quản lý sự giao tiếp của component khi có sự thay đổi xảy ra ?"  
>Rất khó có câu trả lời cụ thể cho câu hỏi trên mà chỉ khi ta bắt tay vào làm thực tiễn một ứng dụng ta mới có thể đúc kết được kinh nghiệm từ đó. Có một kiến trúc mà đa số thường được sử dụng phổ biến giúp chúng ta tổ chức một ứng dụng React đó là "Tách component thành `Phần hiển thị (Presentation)` và `Phần xử lý logic (Container)`"

Hãy bắt đầu bằng một ví dụ minh họa sau. Ta có một component `Clock` và ta sẽ truyền vào prop một kiểu dữ liệu `Date` và hiển thị ra thời gian thực

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: this.props.time };
    this._update = this._updateTime.bind(this);
  }
  render() {
    const time = this._formatTime(this.state.time);
    return (
      <h1>
        { time.hours } : { time.minutes } : { time.seconds }
      </h1>
    );
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _formatTime(time) {
    var [ hours, minutes, seconds ] = [
      time.getHours(),
      time.getMinutes(),
      time.getSeconds()
    ].map(num => num < 10 ? '0' + num : num);

    return { hours, minutes, seconds };
  }
  _updateTime() {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};

ReactDOM.render(<Clock time={ new Date() }/>, ...);
```

Trong hàm khởi tạo của component, biến `time` là state được gán với giá trị là thời gian hiện tại. Bằng cách sử dụng `setInterval` để cập nhật trạng thái mỗi giây và component sẽ render lại. Hai hàm bổ trợ là `_formatTime` và `_updateTime` giúp ta làm cho `Clock` nhìn giống với một cái đồng hồ điện tử thực tế hơn. `_formatTime` xử lý bốc tách `hours`, `minutes` và `seconds` để đảm bảo theo định dạng số `two-digits`. `_updateTime` xử lý cập nhật thay đổi trạng thái của `time` theo mỗi giây

### Vấn đề gặp phải

Trông component trên có vẻ trở nên phức tạp. Ta có thể quan sát được có 2 vấn đề xảy ra:
- Bản thân component tự thay đổi state. Việc thay đổi thời gian trogn component có vẻ không phải là ý hay bởi vì chỉ có `Clock` nhận giá trị này và nếu như có một chỗ khác đang lệ thuộc vào dữ liệu này thì rất khó để có thể share component này để dùng chung
- `_formatTime` đang phải làm 2 việc. Vừa tách dữ liệu dạng `Date` vừa phải đảm bảo rằng giá trị này phải biểu diễn ra dạng số `two-digits`. Cách làm này có vẻ ổn tuy nhiên sẽ tốt hơn nếu như chúng ta mang đoạn code phần tách dữ liệu ra khỏi hàm này

### Tách thành phần Container để xử lý logic

`Container` là nơi xử lý về dữ liệu vì nắm được hình thù dạng dữ liệu và biết nó đến từ đâu. Cách làm này gọi là tách phần `business logic` ra khỏi code. `Container` sẽ nhận thông tin và xử lý thông tin sao cho trở nên dễ hiểu để có thể sử dụng cho phần `Presentation` dễ dàng hơn. `Higher-Order Components (HOCs)` cũng là một cách để ta tạo ra `Container` bởi vì nó cho phép ta thoải mái chèn vào phần xử lý logic. `ClockContainer` sẽ trông như sau:

```javascript
// Clock/index.js
import Clock from './Clock.jsx';

export default class ClockContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: props.time };
    this._update = this._updateTime.bind(this);
  }
  render() {
    return <Clock { ...this._extract(this.state.time) }/>;
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _extract(time) {
    return {
      hours: time.getHours(),
      minutes: time.getMinutes(),
      seconds: time.getSeconds()
    };
  }
  _updateTime() {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};
```

Chúng ta vẫn nhận vào một biến `time` dạng `Date` và thực hiện `setInterval`. Trong hàm `render` sẽ là component `Clock` và truyền vào thuộc tính `props` ba giá trị `hours`, `minutes`, `seconds`. Có thể thấy trong `ClockContainer` chỉ là xử lý chuyên về `business logic` mà không hề dính đến việc hiển thị

### Tách thành phần Presentation để xử lý hiển thị

những component `Presentation` sẽ chỉ quan tâm đến mặt hiển thị đẹp đẽ như thế nào và không hề có sự lệ thuộc nào về logic. Những component này sẽ được viết dưới dạng `Stateless Functional Component` và không hề có `state` trong đó

```javascript
// Clock/Clock.jsx
export default function Clock(props) {
  var [ hours, minutes, seconds ] = [
    props.hours,
    props.minutes,
    props.seconds
  ].map(num => num < 10 ? '0' + num : num);

  return <h1>{ hours } : { minutes } : { seconds }</h1>;
};
```

### Lợi ích từ việc tách ra Container và Presentation cho Component

Giúp làm tăng việc tái sử dụng những component. Có thể thấy component `Clock` từ ví dụ trên sau khi được viết về dạng `stateless` thì có thể sử dụng được nhiều nơi khác nếu cần trong project của bạn, khiến cho cách tổ chức code của bạn trở nên sáng sủa hơn và không bị quá nhiều lệ thuộc vào dữ liệu

`ClockContainer` chỉ xử lý về logic và không hề quan tâm đến việc hiển thị như thế nào nên ta có thể có nhiều cách render dữ liệu trong một container thông qua việc thay component ví dụ ta có thể thay cách hiển thị `two-digits` thành cách hiển thị `analog` bằng cách thay thế `Clock` bằng một component khác trong hàm render

Việc viết test cũng trở nên dễ dàng hơn khi ta chia tách `Container` và `Presentation`

### Kết luận
Hai khái niệm `Container` và `Presentation` không còn trở nên quá xa lạ trong các ứng dụng React hiện nay. Cách này giúp cho việc tổ chức ứng dụng và scale lên trở nên dễ dàng hơn
