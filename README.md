# WeUse

微信小程序 API 助手

### Features

- [Promise 封装](#promise)
- [缓存时效实现](#缓存)
- [HTTP 请求封装](#网络请求)
- [Canvas 封装](#canvas)
- [Redux 状态管理](#redux)

## 安装

使用 npm 依赖(原生或 mpvue)

```bash
npm i weuse
```

使用本地依赖项

如无法使用 npm 依赖, 可下载 [latest release](../../releases/latest), 放至项目目录中进行相对路径引用.

## 使用

使用 npm 依赖

```javascript
const weuse = require('weuse')
```

TypeScript

```typescript
import * as weuse from 'weuse'
```

### Promise

**weuse.promise**

> Promise 相关封装

**weuse.promise.promisify(func: Function): Function**

将一个小程序方法 Promise 化

e.g.

```javascript
const res = await weuse.promisify(wx.getUserInfo)()
const { confirm, cancel } = await weuse.promisify(wx.showModal)({
  title: 'Test',
  content: 'This is a test'
})
```

**weuse.promise.map(values: any[], mapper: Function): Promise&lt;any[]&gt;**

对一个数组进行 map 操作并行执行每个返回的 Promise. 返回按传入顺序排序的执行结果

e.g.

```javascript
const urls = ['https://...', 'https://...', 'https://...']
const results = await weuse.promise.map(urls, url => weuse.request.get(url))
```

**weuse.promise.serial(values: any[]): Promise&lt;any[]&gt;**

顺序执行 Promise. 返回按传入顺序排序的执行结果

e.g.

```javascript
const results = await weuse.promise.serial([
  weuse.wx.login(),
  weuse.request.post({ url, data }),
  weuse.wx.setStorage({ key, data })
])
```

**weuse.wx**

> 包含全部已`Promisify`处理的 wx 方法对象. 调用签名同 wx 中的方法, 无需设置`success`,`fail`回调. 若设置回调, 则不会返回`Promise`.

e.g.

原调用

```javascript
wx.getUserInfo({
  success(res) {
    console.log(res.userInfo)
  }
})

wx.showModal({
  title: 'Test',
  content: 'This is a test',
  success({ confirm, cancel }) {
    //
  }
})
```

异步调用

```javascript
weuse.wx.getUserInfo().then(res => {
  console.log(res.userInfo)
})

// 使用await
const res = await weuse.wx.getUserInfo()
console.log(res.userInfo)

const { confirm, cancel } = await weuse.wx.showModal({ title: 'Test', content: 'This is a test' })
```

兼容调用

```javascript
weuse.wx.getUserInfo({
  success(res) {
    console.log(res.userInfo)
  }
})

weuse.wx.showModal({
  title: 'Test',
  content: 'This is a test',
  success({ confirm, cancel }) {
    //
  }
})
```

### 缓存

**weuse.storage**

> 支持 TTL 的缓存封装

**注意 get/del/touch/exists/ttl/expire/persist/flush 只能针对由 set 添加的缓存**

**weuse.storage.get(key: string): Promise&lt;any&gt;**

从本地缓存中获取指定 key 的内容. `Date` 类型会被反序列化. key 不存在返回 `undefined`

e.g.

```javascript
const token = await weuse.storage.get('token')
```

**weuse.storage.set(opts: Options): Promise&lt;void&gt;**

将数据存储在本地缓存中指定的 key 中

Options:

- key: string _本地缓存中指定的 key_
- data: any _需要存储的内容_
- ttl?: number _有效时间(秒). 为空或 0 则不会过期_

e.g.

```javascript
await weuse.storage.set({ key: 'token', data: token, ttl: 3600 })
```

**weuse.storage.del(key: string): Promise&lt;void&gt;**

从本地缓存中移除指定 key

e.g.

```javascript
await weuse.storage.del('token')
```

**weuse.storage.touch(...keys: string[]): Promise&lt;number&gt;**

检查指定的 key 是否已过期, 是则移除. key 不存在则忽略. 返回移除掉的 key 数量

e.g.

```javascript
const count = await weuse.storage.touch('token', 'name', 'key2')
```

**weuse.storage.exists(key: string): Promise&lt;boolean&gt;**

检查 key 是否存在

e.g.

```javascript
const exists = await weuse.storage.exists('token')
```

**weuse.storage.ttl(key: string): Promise&lt;number&gt;**

获取指定 key 的剩余有效时间(秒). 若 key 不存在返回-1; 若 key 不会过期返回-2

e.g.

```javascript
const ttl = await weuse.storage.ttl('token')
```

**weuse.storage.expire(key: string, ttl: number): Promise&lt;boolean&gt;**

更新指定 key 的有效时间. key 存在且设置成功则返回 true

e.g.

```javascript
const success = await weuse.storage.expire('token', 3600)
```

**weuse.storage.persist(key: string): Promise&lt;boolean&gt;**

移除指定 key 的有效时间让其不会过期. key 存在且设置成功则返回 true

e.g.

```javascript
const success = await weuse.storage.persist('token')
```

**weuse.storage.flush()**

移除所有已过期的缓存

e.g.

```javascript
await weuse.storage.flush()
```

### 网络请求

**weuse.request**

> 网络请求封装

**weuse.request(options: Options): Promise&lt;Response&gt;**

发起网络请求

Options:

- url: string
- query?: string|object
- method?: 'OPTIONS' | 'GET' | 'HEAD' | 'POST' | 'PUT' | 'DELETE' | 'TRACE' | 'CONNECT'
- data?: string | object | ArrayBuffer
- header?: object
- dataType?: string
- responseType?: 'text' | 'arraybuffer'

Response:

- data: string | object | ArrayBuffer
- statusCode: number
- header: object

e.g.

```javascript
const res = await weuse.request({ url: 'https://myapi.com/api/v1/items', method: 'GET' })
```

或者直接使用 http verb 小写名称当方法名

e.g.

```javascript
const res = await weuse.request.get('https://myapi.com/api/v1/items')

const { data, statusCode, header } = await weuse.request.get({
  url: 'https://myapi.com/api/v1/items',
  query: {
    name: 'abc',
    key: '12345'
  }
})

await weuse.request.post({
  url: 'https://myapi.com/api/v1/items',
  data: {
    type: 'Book',
    id: 1
  }
})
```

**注意当 status code 不为 2xx/3xx 时, Promise 会被自动 reject 掉, 抛出 HttpError(包含 code, message 内容)**

**weuse.request.defaults(opts: Options): Request**

返回一个包含默认设置的 request 对象. 可用来设置`baseUrl`, `header`等默认值

e.g.

```javascript
const request = weuse.request.defaults({
  baseUrl: 'https://myapi.com/api/v1',
  header: { 'x-uuid': '12345' },
  query: {
    token: '123456'
  }
})

const res = await request.get({
  url: '/items',
  header: { 'x-key': 'abcd' },
  query: {
    page: 3
  }
})
```

相当于

```javascript
const res = await weuse.request.get({
  url: 'https://myapi.com/api/v1/items',
  header: {
    'x-uuid': '12345',
    'x-key': 'abcd'
  },
  query: {
    token: '123456',
    page: 3
  }
})
```

**weuse.request.download(opts: Options): Promise&lt;Result&gt;**

分段下载文件

Options:

- url: string _下载地址_
- filePath?: string _保存路径_
- parts?: number _分段数 1~10_
- header?: object _请求头_
- query?: string|object _querystring_

Result:

- filePath?: string _下载文件保存路径_
- buffer?: ArrayBuffer _下载文件的二进制流_

e.g.

```javascript
const { filePath } = await weuse.request.download({
  url: 'https://some/large/file.jpg',
  parts: 3,
  filePath: wx.env.USER_DATA_PATH + '/file.jpg'
})

const { buffer } = await weuse.request.download({
  url: 'https://some/large/file.jpg',
  parts: 3
})
```

### Canvas

**weuse.canvas**

> 绘制 Canvas 封装

**weuse.canvas.draw(canvasId: string, options: Options): Promise&lt;void|string&gt;**

绘制 Canvas

Options:

- layers: Layer[] _图层. 从下到上绘制_
- default?: Style _默认样式. 可以在图层中覆盖_
- export?: Export _导出 canvas 到文件的设置_
- downloader?: Downloader _图片下载方法(不设置则使用 wx.getImageInfo)_

Style:

- stroke?: Stroke _描边_
- fill?: Fill _填充_
- font?: Font _文字_

Stroke: _描边样式_

- color?: Color
- lineWidth?: number
- lineDash?: Stroke.LineDash
- lineCap?: 'butt' | 'round' | 'square'
- lineJoin?: 'bevel' | 'round' | 'miter'
- miterLimit?: number

Fill: _填充样式_

- color?: Color
- shadow?: Fill.Shadow

Color: _颜色, 可以为字符串#fff, rgb(0,0,0)等_

- type: 'linear' | 'circular' | 'pattern' _渐变方式_

Font: _字体样式_

- family?: string _字体_
- size: number _字号_
- align?: 'left' | 'center' | 'right' _水平对齐_
- baseline?: 'top' | 'bottom' | 'middle' | 'normal' _垂直对齐_

Export:

- x?: number _指定的画布区域的左上角横坐标_
- y?: number _指定的画布区域的左上角纵坐标_
- width?: number _指定的画布区域的宽度_
- height?: number _指定的画布区域的高度_
- destWidth?: number _输出的图片的宽度_
- destHeight?: number _输出的图片的高度_
- fileType?: 'jpg' | 'png' _目标文件的类型_
- quality?: number _图片的质量_

Layer:

- type: 'rect' | 'arc' | 'image' | 'text' | 'path' _图层类型_
- x?: number _绘制起点 X_
- y?: number _绘制起点 Y_
- fill?: Fill | true _填充样式. 为 true 进行默认填充_
- stroke?: Stroke | true _描边样式. 为 true 进行默认描边_
- clip?: Clip _图层遮罩_
- transform?: Transform _图层变换_

Layer.Rect: _矩形_

- width: number _宽_
- height: number _高_
- anchor?: Vector2 _轴心点{x: [0, 1], y: [0, 1]}, 默认{x: 0, y: 0}_

Layer.Arc: _弧形_

- radius: number _半径_
- startAngle?: number _起始弧度(0 为三点钟方向)_
- endAngle?: number _结束弧度_
- counterClockwise?: boolean _逆时针_

Layer.Image: _图片_

- src: string _资源，可为网络链接或本地地址_
- width?: number _宽_
- height?: number _高_
- crop?: Image.Crop _裁剪_
- anchor?: Vector2 _轴心点{x: [0, 1], y: [0, 1]}, 默认{x: 0, y: 0}_

Layer.Path: _文本_

- text: string _内容_
- font?: Font _字体样式_
- maxWidth?: number _最大宽度_

Layer.Path: _路径_

- points: Point[] _路径点_
- close?: boolean _路径关闭_

e.g.

```xml
<view style='width:0px;height:0px;overflow:hidden;'>
  <canvas canvas-id='poster' style="width:750px;height:750px;"></canvas>
</view>
```

```javascript
const filePath = await weuse.canvas.draw('poster', {
  export: {
    width: 375,
    height: 500,
    fileType: 'jpg',
    quality: 0.8
  },
  default: {
    stroke: {
      color: 'black',
      lineWidth: 2
    },
    fill: {
      color: 'red'
    },
    font: {
      size: 22,
      align: 'center',
      baseline: 'middle'
    }
  },
  layers: [
    {
      type: 'rect',
      width: 100,
      height: 100,
      fill: true,
      stroke: {
        lineWidth: 1,
        color: 'green',
        lineDash: {
          pattern: [10, 20],
          offset: 5
        }
      }
    },
    {
      type: 'arc',
      x: 100,
      y: 100,
      radius: 50,
      fill: {
        color: 'purple'
      }
    },
    {
      type: 'path',
      x: 200,
      y: 200,
      points: [
        {
          type: 'linear',
          x: 220,
          y: 200
        },
        {
          type: 'arc',
          x: 250,
          y: 220,
          radius: 30
        },
        {
          type: 'quadratic',
          x: 300,
          y: 250,
          cpx: 260,
          cpy: 300
        },
        {
          type: 'cubic',
          x: 280,
          y: 180,
          cpx0: 260,
          cpy0: 250,
          cpx1: 270,
          cpy1: 220
        }
      ],
      stroke: true,
      fill: true,
      close: true
    },
    {
      type: 'text',
      text: 'Hello World!',
      x: 300,
      y: 300,
      fill: true
    },
    {
      type: 'image',
      src: 'https://placeimg.com/100/100/animals',
      x: 100,
      y: 300,
      clip: {
        // 圆角遮罩效果
        type: 'rect',
        x: 100,
        y: 300,
        width: 100,
        height: 100,
        radius: 4
      }
    },
    {
      type: 'image',
      src: 'https://placeimg.com/100/100/animals',
      x: 100,
      y: 400,
      clip: {
        // 圆形遮罩效果
        type: 'circular',
        x: 150,
        y: 450,
        radius: 50
      }
    }
  ]
})
await weuse.utils.saveImageToPhotosAlbum({ filePath })
```

效果如图:

![Imgur](https://i.imgur.com/3DqAgkL.jpg)

### 其他

**weuse.utils**

> 帮助方法封装

**weuse.utils.encodeQuery(query: object): string**

从 `object` 创建 querystring

e.g.

```javascript
const query = weuse.utils.encodeQuery({
  name: 'admin',
  size: 50,
  key: null,
  index: undefined
})
// 结果: name=admin&size=50
```

**weuse.utils.decodeQuery(query: object): object**

解析小程序页面传递的 querystring 对象. 兼容小程序码、二维码、普通链接

**weuse.utils.joinUrl(...urls: string[]): string**

连接 url. 会移除重复的 `/` 符号. url 开头跟结尾均可包含或缺省 `/` 符号. 返回结果结尾不含 `/` (除非整体仅为 `/`)

e.g.

```javascript
const url = weuse.utils.joinUrl('http://myapi.com/', '/api/v1', 'items/', '1')
// 结果: http://myapi.com/api/v1/items/1
```

**weuse.utils.authorize(scope: string): Promise&lt;void&gt;**

查询授权, 如果未授权则进行请求

**weuse.utils.getLocation(opts: Options): Promise&lt;Location&gt;**

检查权限并获取地理定位

**weuse.utils.chooseLocation(opts: Options): Promise&lt;Location&gt;**

检查权限并请求地理定位

**weuse.utils.saveImageToPhotosAlbum(opts: Options): Promise&lt;void&gt;**

检查权限并写入相册

**weuse.utils.clone(source: any): any**

深克隆

**weuse.utils.merge(source: any, target: any): any**

递归合并

### Redux

**weuse.redux**

> redux 风格的状态管理

**weuse.redux.createStore(reducer: Reducer): Store**

创建一个 Store

**weuse.redux.combineReducers(reducers: object): Reducer**

合并多个 Reducers

**weuse.redux.createProvider(store: Store, options: object): Provider**

创建一个 Provider

e.g.

```javascript
App(
  weuse.redux.createProvider(store, {
    onLaunch() {
      //...
      const state = this.$store.getState()
    }
  })
)
```

**weuse.redux.createConsumer(options: object, stateMapper?: StateMapper): Consumer**

创建一个 Consumer

e.g.

```javascript
Page(
  weuse.redux.createConsumer({
    onLoad() {
      //...
      this.$dispatch({ type: 'FETCH_DATA' })
    }
  })
)
// 注入state
Page(
  weuse.redux.createConsumer(
    {
      onLoad() {
        //...
        console.log(this.data.items)
        console.log(this.data.count)
      }
    },
    state => ({
      items: state.items,
      count: state.count,
      name: state.app.name
    })
  )
)
```

**weuse.redux.Provider(store: Store): ClassDecorator**

Provider 装饰器

e.g.

```typescript
@Provider(store)
class MyApp {
  onLaunch() {
    //...
    const state = this.$store.getState()
  }
}
App(new MyApp())
```

**weuse.redux.Consumer(stateMapper?: StateMapper): ClassDecorator**

Consumer 装饰器

e.g.

```typescript
@Consumer()
class MyPage {
  onLoad() {
    //...
    this.$dispatch({ type: 'FETCH_DATA' })
  }
}
Page(new MyPage())

// 注入state
@Consumer(state => ({
  items: state.items,
  count: state.count
}))
class MyPage {
  onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.count)
  }
}
```

**weuse.redux.Consumer.State(name?: string): PropertyDecorator**

State 绑定装饰器

e.g.

```typescript
@Consumer()
class MyPage {
  @Consumer.State
  items: any[]

  @Consumer.State('count')
  size: number

  onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.size)
  }
}
```

_注: State 绑定可与 Consumer 装饰器中的 StateMapper 参数共存, 若存在同名属性, 前者会覆盖后者_

**weuse.redux.Consumer.namespace(name?: string): Namespace**

子 State 绑定对象

**weuse.redux.Consumer.Namespace.State(name?: string): PropertyDecorator**

子 State 绑定装饰器

e.g.

```typescript
const app = Consumer.namespace('app')
const
@Consumer()
class MyPage {
  @app.State
  items: any[]

  @app.State('count')
  size: number

  onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.size)
  }
}
```

### Vuex

**weuse.vuex**

> vuex 风格的状态管理

**weuse.vuex.createStore(options: StoreOptions): Store**

创建一个 Store

**weuse.vuex.createProvider(store: Store, options: object): Provider**

创建一个 Provider

e.g.

```javascript
App(
  weuse.vuex.createProvider(store, {
    onLaunch() {
      //...
      const state = this.$store.state
    }
  })
)
```

**weuse.vuex.createConsumer(options: object, ...mappers: Mapper[]): Consumer**

e.g.

```javascript
Page(
  weuse.vuex.createConsumer({
    onLoad() {
      //...
    }
  })
)
// 注入state/getters/mutations/actions
Page(
  weuse.vuex.createConsumer(
    {
      async onLoad() {
        //...
        console.log(this.data.items)
        console.log(this.data.count)
        await this.login()
        console.log(this.data.user)
        this.setName('admin')
      }
    },
    {
      state: {
        items: 'items',
        count: state => state.count
      },
      getters: ['name'],
      mutations: {
        setUser: 'setName'
      }
    },
    {
      namespace: 'account',
      getters: ['user'],
      actions: ['login']
    }
  )
)
```

创建一个 Consumer

**weuse.vuex.Provider(store: Store): ClassDecorator**

Provider 装饰器

e.g.

```typescript
@Provider(store)
class MyApp {
  onLaunch() {
    //...
    const state = this.$store.state
  }
}
App(new MyApp())
```

**weuse.vuex.Consumer(...mappers: Mapper[]): ClassDecorator**

Consumer 装饰器

e.g.

```typescript
@Consumer()
class MyPage {
  onLoad() {
    //...
  }
}
Page(new MyPage())

// 注入state
@Consumer({
  state: ['items', 'count']
})
class MyPage {
  onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.count)
  }
}
```

**weuse.vuex.Consumer.State(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.State(func: Function): PropertyDecorator**
**weuse.vuex.Consumer.Getter(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Mutation(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Mutation(func: Function): PropertyDecorator**
**weuse.vuex.Consumer.Action(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Action(func: Function): PropertyDecorator**

State/Getter/Mutation/Action 绑定装饰器

e.g.

```typescript
@Consumer()
class MyPage {
  @Consumer.State
  items: any[]

  @Consumer.State('count')
  size: number

  @Consumer.State(state => state.count)
  count: number

  @Consumer.Getter
  max: number

  @Consumer.Action
  add: Consumer.ActionMethod

  @Consumer.Mutation
  setCount: Consumer.MutationMethod

  async onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.size)
    console.log(this.data.count)
    console.log(this.data.max)
    await this.add()
    this.setCount(100)
  }
}
```

_注: State/Getter/Mutation/Action 绑定可与 Consumer 装饰器中的 Mapper 参数共存, 若存在同名属性, 前者会覆盖后者_

**weuse.vuex.Consumer.namespace(name?: string): Namespace**

子模块绑定对象

**weuse.vuex.Consumer.Namespace.State(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.State(func: Function): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.Getter(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.Mutation(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.Mutation(func: Function): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.Action(name?: string): PropertyDecorator**
**weuse.vuex.Consumer.Namespace.Action(func: Function): PropertyDecorator**

子 State/Getter/Mutation/Action 绑定装饰器

e.g.

```typescript
const app = Consumer.namespace('app')
@Consumer()
class MyPage {
  @app.State
  items: any[]

  @app.State('count')
  size: number

  onLoad() {
    //...
    console.log(this.data.items)
    console.log(this.data.size)
  }
}
```

## 构建

```bash
npm run build
```

包含 source map

```bash
SOURCE_MAP=inline-source-map npm run build
```

## License

See [License](https://github.com/seancheung/weuse/blob/master/LICENSE)
