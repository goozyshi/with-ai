# TypeScript 基础教程

## 基本概念

TypeScript 是 JavaScript 的超集，它添加了静态类型系统，让我们能在编写代码时就发现潜在问题，而不必等到运行时才发现错误。对于熟悉 Vue 的开发者来说，学习 TypeScript 可以让你的 Vue 应用更加健壮。

### 为什么需要 TypeScript

在 JavaScript 中，我们经常遇到这样的情况：

```javascript
const message = "Hello World!";
message(); // 运行时错误：TypeError: message is not a function
```

这个错误只有在运行时才会被发现。但使用 TypeScript：

```typescript
const message = "Hello World!";
message(); // 编译时错误：This expression is not callable.
```

TypeScript 可以在你编写代码时就提醒你这个错误，避免潜在的运行时问题。

## 静态类型检查

TypeScript 的核心优势就是静态类型检查，它可以在代码运行前发现可能的错误。

```typescript
// JavaScript中，这段代码会返回undefined并继续执行
const user = {
  name: "张三",
  age: 26,
};
console.log(user.location); // undefined

// TypeScript会警告你
const user = {
  name: "张三",
  age: 26,
};
console.log(user.location); // 错误：Property 'location' does not exist on type '{ name: string; age: number; }'
```

## 与 Vue 结合使用

在 Vue 项目中，你可以这样定义一个组件：

```typescript
interface UserInfo {
  id: string;
  name: string;
  age?: number;
  isActive: boolean;
}

// User.vue
const props = defineProps<{
  user: UserInfo;
  isEditable?: boolean;
}>();

const emit = defineEmits<{
  (e: "update", user: UserInfo): void;
  (e: "delete", id: string): void;
}>();
```

这样定义的组件具有完整的类型支持，IDE 会给你提供精确的代码补全和类型检查。

## 类型注解与类型推断

TypeScript 有强大的类型推断能力，通常不需要显式添加类型：

```typescript
// 类型推断
let message = "你好"; // TypeScript推断为string类型
let count = 42; // TypeScript推断为number类型

// 显式类型注解
let userName: string = "王五";
let isActive: boolean = true;
```

最佳实践是：**当 TypeScript 能正确推断类型时，不添加额外的类型注解**。

## 类型抹除

TypeScript 代码编译后会移除所有类型信息，变成普通的 JavaScript：

```typescript
// TypeScript
function greet(person: string, date: Date): void {
  console.log(`你好 ${person}, 今天是 ${date.toDateString()}!`);
}

// 编译后的JavaScript
function greet(person, date) {
  console.log("你好 " + person + ", 今天是 " + date.toDateString() + "!");
}
```

这意味着类型注解不会影响程序的运行时行为，只在开发阶段提供帮助。

## 降级（Downleveling）

TypeScript 可以将新版本的 JavaScript 代码转换为旧版本的代码，这个过程称为"降级"：

```typescript
// 原始代码（ES2015+）
`你好 ${person}, 今天是 ${date.toDateString()}!`;

// 降级后的代码（ES3/ES5）
"你好 " + person + ", 今天是 " + date.toDateString() + "!";
```

在 Vue 项目中，通常使用 Vite 或 webpack 来处理这些转换。

## 严格模式设置

TypeScript 提供了一系列严格检查选项，推荐新项目开启：

```json
{
  "compilerOptions": {
    "strict": true,
    // 也可以单独配置
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

其中最重要的两个设置：

1. **noImplicitAny**: 不允许变量被隐式推断为 any 类型
2. **strictNullChecks**: 更严格地处理 null 和 undefined 值

## 在 Vue 项目中使用组合式函数

TypeScript 与 Vue 的组合式 API 结合非常好：

```typescript
function useCounter(initialValue: number = 0) {
  const count = ref(initialValue);
  const increment = () => {
    count.value++;
  };
  const decrement = () => {
    count.value--;
  };

  return {
    count,
    increment,
    decrement,
  };
}

// 在组件中使用
const { count, increment, decrement } = useCounter(10);
```

# TypeScript 常见类型

## 原始类型: string、number 和 boolean

JavaScript 中有三个最常用的原始类型，TypeScript 也完全支持：

```typescript
// 字符串类型
let name: string = "张三";

// 数字类型 - JavaScript中所有数字都是number类型，不区分整数和浮点数
let age: number = 25;
let price: number = 99.9;

// 布尔类型
let isActive: boolean = true;
```

> 注意: 使用`string`、`number`、`boolean`这些小写形式，而非`String`、`Number`、`Boolean`。大写形式指的是 JavaScript 中很少使用的包装对象。

## 数组类型

TypeScript 提供两种定义数组的方式：

```typescript
// 方式1: 类型后跟[]
const numbers: number[] = [1, 2, 3, 4];
const names: string[] = ["张三", "李四", "王五"];

// 方式2: 使用泛型语法
const scores: Array<number> = [80, 90, 100];
```

在 Vue 项目中使用数组：

```typescript
// 定义一个todo列表
interface TodoItem {
  id: string;
  title: string;
  completed: boolean;
}

// 在组件中使用
const todos = ref<TodoItem[]>([]);
```

## any 类型

`any`是 TypeScript 特有的类型，表示"任意类型"。当使用`any`时，TypeScript 会关闭类型检查：

```typescript
let notSure: any = 4;
notSure = "可以是字符串";
notSure = false;
notSure.toFixed(); // 不会报错，虽然布尔值没有toFixed方法
```

虽然`any`提供了灵活性，但它失去了 TypeScript 最大的优势——类型安全。应尽量避免使用`any`。

### noImplicitAny

如果你没有指定类型，TypeScript 无法从上下文推断出类型时，会隐式设置为`any`类型。

启用`noImplicitAny`编译选项后，当变量被隐式推断为`any`类型时，TypeScript 会报错，促使你添加正确的类型注解。

## 变量的类型注解

你可以显式地为变量添加类型注解：

```typescript
let userName: string = "王五";
```

但大多数情况下，TypeScript 能够自动推断类型：

```typescript
// TypeScript自动推断userAge为number类型
let userAge = 30;
```

最佳实践是：**让 TypeScript 推断简单类型，只在必要时添加类型注解**。

## 函数类型

### 参数类型注解

```typescript
function calculateTax(income: number, taxRate: number) {
  return income * taxRate;
}

// 错误调用会被捕获
calculateTax("50000", 0.2); // 错误: 类型'string'不能赋给类型'number'
```

### 返回值类型注解

```typescript
function getFullName(firstName: string, lastName: string): string {
  return `${firstName} ${lastName}`;
}
```

TypeScript 通常能推断函数的返回类型，但显式添加返回类型有助于：

1. 防止无意的返回类型变化
2. 提高代码可读性
3. 提前发现实现与设计的不匹配

### 匿名函数

当 TypeScript 能够确定匿名函数如何被调用时，它会自动为参数设置类型：

```typescript
// 数组的map方法：TypeScript知道names数组元素是string类型
const names = ["张三", "李四", "王五"];
names.map((name) => {
  // name自动被推断为string类型
  return name.toUpperCase();
});
```

## 对象类型

TypeScript 中，对象可以通过形状来描述：

```typescript
// 内联对象类型注解
function printCoord(point: { x: number; y: number }) {
  console.log(`坐标是 (${point.x}, ${point.y})`);
}

// 使用时必须提供所有属性
printCoord({ x: 10, y: 20 });
```

### 可选属性

对象类型中可以使用`?`标记某个属性为可选：

```typescript
interface User {
  id: string;
  name: string;
  email?: string; // 可选属性
  phone?: string; // 可选属性
}

function createUser(user: User) {
  // ...
}

// 可以不提供可选属性
createUser({ id: "1", name: "张三" });
```

在 Vue 组件中：

```typescript
// User.vue中定义props
const props = defineProps<{
  user: User;
  showDetails?: boolean; // 可选属性
}>();
```

## 联合类型

联合类型表示一个值可以是几种类型之一：

```typescript
// ID可以是字符串或数字
function printId(id: string | number) {
  console.log(`ID: ${id}`);
}

printId(101); // 有效
printId("202"); // 有效
printId({ id: 123 }); // 错误: 对象类型不在联合类型中
```

### 使用联合类型

当使用联合类型时，你只能访问联合类型中共有的属性和方法：

```typescript
function getFirstChar(input: string | string[]) {
  // 错误: 属性'charAt'在类型'string[]'上不存在
  // return input.charAt(0);

  // 正确: 使用类型守卫判断
  if (typeof input === "string") {
    return input.charAt(0);
  } else {
    return input[0] || "";
  }
}
```

## 类型别名

类型别名用于给类型起一个新名字：

```typescript
// 定义类型别名
type Point = {
  x: number;
  y: number;
};

type ID = string | number;

// 使用类型别名
function printCoord(pt: Point) {
  console.log(`坐标是: ${pt.x}, ${pt.y}`);
}

function printId(id: ID) {
  console.log(`ID: ${id}`);
}
```

## 接口

接口是另一种定义对象类型的方式：

```typescript
interface User {
  id: string;
  name: string;
  age: number;
}

function getUserName(user: User): string {
  return user.name;
}
```

在 Vue 项目中：

```typescript
// 定义API响应接口
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// 使用接口
function fetchUserData() {
  return fetch("/api/user")
    .then((res) => res.json())
    .then((response: ApiResponse<User>) => {
      return response.data;
    });
}
```

### 类型别名和接口的区别

接口和类型别名非常相似，主要区别是：

1. 接口可以被扩展和实现，更适合定义公共 API
2. 类型别名不能重新开放以添加新属性，但接口可以
3. 接口只能描述对象类型，而类型别名可以为任何类型命名

```typescript
// 接口扩展
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// 接口合并
interface User {
  name: string;
}

interface User {
  age: number;
}
// User现在同时有name和age属性
```

## 类型断言

有时你会比 TypeScript 更了解某个值的类型，这时可以使用类型断言：

```typescript
// 使用as语法
const canvas = document.getElementById("main_canvas") as HTMLCanvasElement;

// 使用<>语法 (在JSX中不能使用)
const canvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

## 字面量类型

TypeScript 允许定义更具体的类型，而不仅仅是 string 或 number：

```typescript
// 限定只能使用特定的字符串值
type Direction = "north" | "south" | "east" | "west";
function move(direction: Direction) {
  // ...
}

move("north"); // 正确
move("northeast"); // 错误: "northeast"不在允许的值中
```

在 Vue 组件中特别有用：

```typescript
// 定义按钮大小选项
type ButtonSize = "small" | "medium" | "large";

// Button.vue
const props = defineProps<{
  size: ButtonSize;
  type?: "primary" | "success" | "warning" | "danger" | "default";
}>();
```

### 字面量推断

当初始化变量为对象字面量时，TypeScript 假定该对象属性的值在未来可能会改变：

```typescript
const req = { url: "https://example.com", method: "GET" };
// method被推断为string类型，而不是"GET"字面量类型

// 使用类型断言修正
const req = { url: "https://example.com", method: "GET" as "GET" };
// 或使用as const将整个对象转为只读
const req = { url: "https://example.com", method: "GET" } as const;
```

## null 和 undefined

TypeScript 中有两个对应的类型：`null`和`undefined`。它们的行为取决于`strictNullChecks`编译选项。

### strictNullChecks 开启时

当此选项开启时，你必须在使用可能为`null`或`undefined`的值前进行检查：

```typescript
function printName(name: string | null) {
  if (name === null) {
    console.log("无名氏");
  } else {
    console.log(name.toUpperCase());
  }
}
```

### 非空断言操作符(!)

当你确定某个值不会是`null`或`undefined`时，可以使用非空断言操作符：

```typescript
// id可能为null或undefined，但你确信它有值时
function getUserName(id?: string | null) {
  // 使用!断言id不为空
  const user = findUserById(id!);
  return user.name;
}
```

> 警告：仅在你确信该值不为空时使用!操作符，否则会导致运行时错误。

## 其他需要了解的类型

### 枚举(Enum)

TypeScript 提供了枚举类型，但在现代实践中，我们通常推荐使用联合类型和常量对象代替：

```typescript
// 不推荐使用枚举
enum Status {
  Active,
  Inactive,
  Pending,
}

// 推荐使用常量对象和类型
const HttpStatus = {
  OK: 200,
  NOT_FOUND: 404,
  SERVER_ERROR: 500,
} as const;

type HttpStatusCode = (typeof HttpStatus)[keyof typeof HttpStatus];
```

### 不常见的原始类型

1. **bigint**: 用于表示非常大的整数

   ```typescript
   const bigNumber: bigint = 100n;
   ```

2. **symbol**: 用于创建全局唯一的引用
   ```typescript
   const key: symbol = Symbol("key");
   ```

这些类型在日常开发中使用相对较少，但在特定场景下很有用。

## 在 Vue 项目中的应用示例

```typescript
// 定义通用组件props类型
interface PaginationProps {
  currentPage: number;
  pageSize: number;
  total: number;
  showSizeChanger?: boolean;
  pageSizes?: number[];
}

// Pagination.vue
const props = defineProps<PaginationProps>();

const emit = defineEmits<{
  (e: "change", page: number): void;
  (e: "sizeChange", size: number): void;
}>();

// 使用组合式函数处理分页逻辑
function usePagination(props: PaginationProps, emit: any) {
  const currentPage = ref(props.currentPage);

  function changePage(page: number) {
    currentPage.value = page;
    emit("change", page);
  }

  return {
    currentPage,
    changePage,
  };
}
```

# TypeScript 类型收窄

类型收窄(Type Narrowing)是 TypeScript 中非常重要的概念，它允许 TypeScript 在特定的代码区域将变量的类型由一个较为宽泛的类型转换为更具体的类型。

## 什么是类型收窄？

当我们使用联合类型时，需要处理可能的每种类型。通过类型收窄，TypeScript 可以知道在代码的特定区域中变量的类型更具体。

```typescript
// 一个基本例子
function padLeft(padding: number | string, input: string): string {
  // 此时padding可能是number或string
  if (typeof padding === "number") {
    // 此代码块中，TypeScript知道padding一定是number类型
    return new Array(padding + 1).join(" ") + input;
  }
  // 此代码块中，TypeScript知道padding一定是string类型
  return padding + input;
}
```

## 类型保护(Type Guards)

类型保护是一些表达式，它们会在运行时检查以确保在某些范围内的类型。

### typeof 类型保护

`typeof`操作符是最常见的类型保护方式之一：

```typescript
function printValue(value: string | number | boolean) {
  if (typeof value === "string") {
    // 在这个块中，value的类型被收窄为string
    console.log(value.toUpperCase());
  } else if (typeof value === "number") {
    // 在这个块中，value的类型被收窄为number
    console.log(value.toFixed(2));
  } else {
    // 在这个块中，value的类型被收窄为boolean
    console.log(value ? "是" : "否");
  }
}
```

> 注意：`typeof null`返回`"object"`，这是 JavaScript 的一个历史问题。使用`typeof`来检查是否为对象类型时需要注意这一点。

```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    // 警告！此时strs可能是null
    for (const s of strs) {
      // 错误：Object is possibly 'null'
      console.log(s);
    }
  }
}
```

### 真值收窄(Truthiness Narrowing)

JavaScript 中的条件语句会自动将值转换为布尔值进行判断，我们可以利用这点来排除`null`、`undefined`等值：

```typescript
function printAll(strs: string | string[] | null) {
  // 先排除null和undefined
  if (strs) {
    if (typeof strs === "object") {
      // 此时strs只能是string[]
      for (const s of strs) {
        console.log(s);
      }
    } else {
      // 此时strs只能是string
      console.log(strs);
    }
  }
}
```

在 Vue 组件中经常使用这种模式：

```typescript
interface UserInfo {
  id: string;
  name: string;
  avatar?: string;
}

// 在模板中使用
// <img v-if="user && user.avatar" :src="user.avatar" />

// 在脚本中使用
function getUserDisplayName(user: UserInfo | null) {
  return user ? user.name : "游客";
}
```

### 等值收窄(Equality Narrowing)

TypeScript 也使用`===`、`!==`、`==`、`!=`这样的操作符来收窄类型：

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // 在这个块中，x和y的类型都被收窄为string
    // 因为这是x和y唯一可能相等的类型
    console.log(x.toUpperCase());
    console.log(y.toUpperCase());
  } else {
    // 这里x可能是string或number，y可能是string或boolean
    console.log(x);
    console.log(y);
  }
}
```

这种技术在检查特殊值时特别有用：

```typescript
function printId(id: string | number | undefined) {
  if (id === undefined) {
    console.log("ID未提供");
    return;
  }

  // 这里id只能是string或number
  console.log(`ID: ${id}`);
}
```

### in 操作符收窄

JavaScript 的`in`操作符用于判断对象是否有特定属性，TypeScript 将其用作类型保护：

```typescript
interface Admin {
  id: string;
  role: string;
}

interface User {
  id: string;
  email: string;
}

function redirect(user: Admin | User) {
  if ("role" in user) {
    // 这里user被收窄为Admin类型
    console.log(`管理员${user.role}已登录`);
    return "/admin-dashboard";
  } else {
    // 这里user被收窄为User类型
    console.log(`用户${user.email}已登录`);
    return "/user-home";
  }
}
```

### instanceof 收窄

`instanceof`操作符是检查一个值是否是某个类的实例的另一种方式：

```typescript
function formatDate(date: string | Date) {
  if (date instanceof Date) {
    // 在这个块中，date是Date类型
    return date.toLocaleDateString("zh-CN");
  } else {
    // 在这个块中，date是string类型
    return new Date(date).toLocaleDateString("zh-CN");
  }
}
```

在 Vue 应用中处理错误时很有用：

```typescript
function handleError(error: unknown) {
  if (error instanceof Error) {
    // 错误对象
    console.error(`错误: ${error.message}`);
    console.error(`堆栈: ${error.stack}`);
  } else {
    // 其他类型的错误
    console.error(`未知错误: ${String(error)}`);
  }
}

// 在Vue组件的错误处理中
try {
  // 某些操作
} catch (err: unknown) {
  handleError(err);
}
```

### 赋值语句(Assignments)

当我们给变量赋值时，TypeScript 会根据赋值的右侧内容收窄左侧变量的类型：

```typescript
let x = Math.random() < 0.5 ? 10 : "hello";
// x的类型是string | number

x = 1;
// 现在x的类型被收窄为number

x = "goodbye";
// 现在x的类型被收窄为string

x = true;
// 错误！boolean不能赋值给string | number
```

### 控制流分析(Control Flow Analysis)

TypeScript 使用控制流分析来确定变量在程序的特定位置上最可能的类型：

```typescript
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;
  // 此时x是boolean类型

  if (Math.random() < 0.5) {
    x = "hello";
    // 此时x是string类型
  } else {
    x = 100;
    // 此时x是number类型
  }

  // 此时x的类型又回到string | number
  return x;
}
```

### 类型判断式(Type Predicates)

有时我们需要自定义类型保护函数，这时可以使用类型判断式：

```typescript
interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

// 返回类型是类型判断式
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    // 在这个块中，pet的类型被收窄为Fish
    pet.swim();
  } else {
    // 在这个块中，pet的类型被收窄为Bird
    pet.fly();
  }
}
```

在 Vue 项目中，类型判断式可以用于确定 API 返回的数据类型：

```typescript
interface SuccessResponse<T> {
  status: "success";
  data: T;
}

interface ErrorResponse {
  status: "error";
  message: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccess<T>(
  response: ApiResponse<T>
): response is SuccessResponse<T> {
  return response.status === "success";
}

// 在组件中使用
async function fetchUser() {
  const response: ApiResponse<UserInfo> = await api.get("/user");

  if (isSuccess(response)) {
    // response.data 类型为 UserInfo
    return response.data;
  } else {
    // 处理错误
    console.error(response.message);
    return null;
  }
}
```

## 可辨别联合(Discriminated Unions)

可辨别联合是 TypeScript 中一种特殊的联合类型，每个成员都有一个共同的单例类型属性：

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      // 此时shape的类型被收窄为Circle
      return Math.PI * shape.radius ** 2;
    case "square":
      // 此时shape的类型被收窄为Square
      return shape.sideLength ** 2;
    case "triangle":
      // 此时shape的类型被收窄为Triangle
      return 0.5 * shape.base * shape.height;
  }
}
```

在 Vue 项目中，可辨别联合常用于处理组件状态：

```typescript
interface LoadingState {
  status: "loading";
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  error: string;
}

type RequestState<T> = LoadingState | SuccessState<T> | ErrorState;

// 在组件中
const state = ref<RequestState<UserInfo>>({ status: "loading" });

// 更新状态
function fetchData() {
  state.value = { status: "loading" };

  api
    .get("/user")
    .then((data) => {
      state.value = { status: "success", data };
    })
    .catch((error) => {
      state.value = { status: "error", error: error.message };
    });
}

// 在模板中使用
// <div v-if="state.status === 'loading'">加载中...</div>
// <div v-else-if="state.status === 'error'">错误: {{ state.error }}</div>
// <div v-else>用户名: {{ state.data.name }}</div>
```

## never 类型与穷尽检查(Exhaustiveness Checking)

`never`类型表示一个值永远不会发生的类型。在类型收窄时，如果 TypeScript 确定某个分支永远不会运行，变量的类型会变为`never`。

我们可以利用这个特性进行穷尽检查：

```typescript
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      // 如果Shape联合类型新增了成员，但这里没有处理，会报错
      return assertNever(shape);
  }
}
```

当我们向`Shape`联合类型添加新成员（如 Triangle）但忘记在`getArea`中处理它时，TypeScript 会给出错误，因为`Triangle`类型不能赋值给`never`类型。

这种技术在确保处理了所有可能情况时非常有用，比如在状态管理中处理不同的 action 类型：

```typescript
type Action =
  | { type: "INCREMENT"; amount: number }
  | { type: "DECREMENT"; amount: number }
  | { type: "RESET" };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case "INCREMENT":
      return state + action.amount;
    case "DECREMENT":
      return state - action.amount;
    case "RESET":
      return 0;
    default:
      // 穷尽检查：如果添加了新的Action类型但忘记处理
      return assertNever(action);
  }
}
```

## 实际应用总结

类型收窄是 TypeScript 最强大的特性之一，它允许我们：

1. 安全地处理联合类型
2. 减少对类型断言的依赖
3. 让代码更加健壮和自文档化
4. 在编译时捕获可能的错误

在 Vue 项目中，类型收窄尤其有用于：

1. 处理 API 响应数据
2. 管理组件状态
3. 处理用户输入
4. 实现条件渲染逻辑

掌握类型收窄技术，能够让你的 TypeScript 代码更加类型安全，同时保持代码的简洁和可读性。
