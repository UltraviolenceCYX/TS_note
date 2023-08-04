## 基本类型

1. 自动判断类型：

   首次声明变量会自动绑定类型，对于普通函数和匿名函数，都存在更具上下文的自动类型判断。

   ``` ts
   let name = "Alice" // name: string
   
   const names = ["Alice", "Bob", "Eve"];
    
   names.forEach((s) => {
     console.log(s.toUpperCase());
   }); //s: string
   ```

2. 自定义对象类型：

   可以自定义一个对象作为类型，接收参数会严格遵守类型定义，不可包含其他属性

   ``` tsx
   function printCoord(pt: { x: number; y: number }) {
     console.log("The coordinate's x value is " + pt.x);
     console.log("The coordinate's y value is " + pt.y);
   }
   ```

3. 可选参数

   可以用`？`表示可选参数，可选参数使用时需要进行类型检查，且必须要放在确定的参数之后

   ```ts
   function bar(x:number, y?: string) {
       if(y !== undefined)
           console.log(y.toUpperCase())
   }
   ```

4. 联合类型

   `number | string`可以表示两种中任意一种类型，除了进行所有类型的共有操作，其他操作都需要进一步做类型确定

   ```ts
   function printId(id: number | string) {
     if (typeof id === "string") {
       // In this branch, id is of type 'string'
       console.log(id.toUpperCase());
     } else {
       // Here, id is of type 'number'
       console.log(id);
     }
   }
   ```

5. type与interface

   两者都可以用来定义类型的别名，区别在于扩展新类型的方式和是否可添加新属性

   ```ts
   //interface使用extends, type使用&
   interface Animal {
     name: string;
   }
   interface Bear extends Animal {
     honey: boolean;
   }
   type Animal = {
     name: string;
   }
   type Bear = Animal & { 
     honey: boolean;
   }
   //interface可以重复声明来合并属性，而type不可以重复声明
   interface W {
     title: string;
   }
   interface W {
     ts: number;
   }
   ```


6. 字面量类型

   可以使用字面量类型配合联合类型来限定可接收的值，as const来固定属性为字面量类型

   ```ts
   function printText(s: string, alignment: "left" | "right" | "center") {}
   function compare(a: string, b: string): -1 | 0 | 1 {
     return a === b ? 0 : a > b ? 1 : -1;
   }
   
   //当使用const定义一个对象时，其属性不会定义为字面量
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   const req = { url: "https://example.com", method: "GET" };
   handleRequest(req.url, req.method); //error Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
   
   ```

//使用as const可以解决上面的问题，将所有属性设定为字面量类型
   const req = { url: "https://example.com", method: "GET" } as const
```
   

7. 排除null和undefined

   当一个值可能为null或undefined时，可以使用！断言来排除

   ```ts
   function liveDangerously(x?: number | null) {
     // No error
     console.log(x!.toFixed());
   }
```

   

## Narrowing

1. 可以使用`typeof`来做基本类型的类型守卫，但null除外，这时可以用`js`的真值判断来进一步判断（false: 0, `NaN`, "", null, undefined）

   ```ts
   function printAll(strs: string | string[] | null) {
     if (typeof strs === "object") {
       for (const s of strs) { //s: string[] | null
         console.log(s);
       }
     } else if (typeof strs === "string") {
       console.log(strs);
     } else {
       // do nothing
     }
   }
   
   function printAll(strs: string | string[] | null) {
     if (strs && typeof strs === "object") {
       for (const s of strs) { // s: string[]
         console.log(s);
       }
     } else if (typeof strs === "string") {
       console.log(strs);
     }
   }
   //使用xx != null 可以同时过滤掉null和undefined !==则只能过滤null
   
   ```

2. 使用in，判断是否有特有属性也可以做narrowing

   ```ts
   type Fish = { swim: () => void };
   type Bird = { fly: () => void };
   function move(animal: Fish | Bird) {
     if ("swim" in animal) {
       return animal.swim(); //animal: Fish
     }
     return animal.fly();
   }
   ```

3. 使用`instanceof`可以通过原型链判断类型

4. 可辨识联合（Discriminated Unions）

   通过联合类型中共有的属性可以用来narrowing

   ```ts
   //下面的例子无法使不同kind与特有属性结合
   interface Shape {
     kind: "circle" | "square";
     radius?: number;
     sideLength?: number;
   }
   function getArea(shape: Shape) {
     return Math.PI * shape.radius ** 2; // error! 'shape.radius' is possibly 'undefined'.
   }
   
   //修改成如下方式，其中kind就是Shape类型的可辨识属性（共有属性）
   interface Circle {
     kind: "circle";
     radius: number;
   } 
   interface Square {
     kind: "square";
     sideLength: number;
   }
   type Shape = Circle | Square;
   
   function getArea(shape: Shape) {
     if (shape.kind === "circle") {
       return Math.PI * shape.radius ** 2; //shape: Circle
     }
   }
   ```

## 函数类型

1. 函数的类型签名

   ```ts
   type GreetFunction = (a: string) => void //普通形式，没法定义具有属性的可调用对象
   
   type DescribableFunction = {    //可以同时定义属性和可调用
     description: string;
     (someArg: number): boolean;
   };
   
   type SomeConstructor = {      //定义使用new的构造器函数
     new (s: string): SomeObject;
   };
   ```

2. **泛型函数**

   ```tsx
   //当需要表示函数参数之间的关系时，就需要用到泛型，相当于会接受参数类型作为额外的参数，会自动根据上下文获取泛型
   function firstElement<Type>(arr: Type[]): Type | undefined {
     return arr[0];
   }
   const s = firstElement(["a", "b", "c"]); //Type为string
   const n = firstElement([1, 2, 3]); //Type为number
   
   //泛型定义时还可以用到extend语句来做限制，下面这个只有包含length属性的参数类型才会被接受，同时函数内部可以直接访问length属性
   function longest<Type extends { length: number }>(a: Type, b: Type) {
     if (a.length >= b.length) {
       return a;
     } else {
       return b;
     }
   }
   /*
   建议: 
   1. When possible, use the type parameter itself rather than constraining it
   2. Always use as few type parameters as possible
   3. Remember, type parameters are for relating the types of multiple values. If a type parameter is    only used once in the function signature, it’s not relating anything. 
   */
   ```

3. 函数重载

   ```ts
   //将重载的所有签名用;连接，最后接上函数体
   function makeDate(timestamp: number): Date;
   function makeDate(m: number, d: number, y: number): Date;
   function makeDate(mOrTimestamp: number, d?: number, y?: number): Date { //实现的函数声明需要兼容上面所有的重载函数声明，共有属性不一样类型就any，部分有的属性就用?: 返回类型也需要兼容
     if (d !== undefined && y !== undefined) {
       return new Date(y, mOrTimestamp, d);
     } else {
       return new Date(mOrTimestamp);
     }
   }
   /*
   建议:
   1. Always prefer parameters with union types instead of overloads when possible 因为某些输入参数可能是表达式的联合类型，而重载只能选择其中一种声明
   */
   ```

4. this类型的定义

   由于js中禁止使用this作为函数的参数，于是ts中可以使用this来指定函数体中this的指向类型

   ```ts
   interface DB {
     filterUsers(filter: (this: User) => boolean): User[];
   }
   const db = getDB();
   const admins = db.filterUsers(function (this: User) {
     return this.admin;
   });
   //对this指定的写法主要用在回调函数中，规范调用者的使用场景，箭头函数无法使用此语法
   ```

5. 函数类型的可赋值性

   ```ts
   //返回为void函数类型可以赋给任意返回类型的函数，但是其接收的变量类型还是void
   type func = (a: number) => void
   let f1: func = function(a: number): number {
       return a
   }
   let b = f1(1) //b: void
   
   //字面量定义的函数则不能适配void返回
   function f2(): void {
     // @ts-expect-error
     return true;
   }
   ```

## 对象类型

1. index签名

   对于只知道属性的值类型，但不知道其属性名的情况，可以使用index签名

   ```ts
   interface StringArray { //对于所有number的属性名，其值只能是string
     [index: number]: string;
   }
   
   interface ReadonlyStringArray { //还可以和readonly进行组合，使number的属性都无法修改
     readonly [index: number]: string;
   }
   ```

2. 严格类型检测

   ```ts
   interface SquareConfig {
     color?: string;
     width?: number;
   }
   function createSquare(config: SquareConfig): { color: string; area: number } {
     return {
       color: config.color || "red",
       area: config.width ? config.width * config.width : 20,
     };
   }
   let mySquare = createSquare({ colour: "red", width: 100 });//通过字面量创建的对象会进行严格类型检查，如果有任何一个属性不存在目标类型中就会报错
   /*
   解决方法1：
       interface SquareConfig {
         color?: string;
         width?: number;
         [propName: string]: any;  //使这个类型可以被添加其他string类型的属性
       }
   解决方法2：
       let squareOptions = { colour: "red", width: 100 };
       let mySquare = createSquare(squareOptions);  //严格类型检查只会对字面量对象进行检测，对变量不会检	查，只要传入的变量和目标类型有至少一个属性相交就不会报错
   */
   ```

3. 类型的继承与并集

   ```ts
   interface Colorful {
     color: string;
   }
   interface Circle {
     radius: number;
   }
   interface ColorfulCircle extends Colorful, Circle {} //extends可以继承多种类型
   const cc: ColorfulCircle = {
     color: "red",
     radius: 42,
   };
   
   interface Colorful {
     color: string;
   }
   interface Circle {
     radius: number;
   }
   type ColorfulCircle = Colorful & Circle; // &之后，新类型拥有两个类型的所有属性
   ```

4. 泛型类型

   除了在函数中可以声明泛型，类型定义中也可以声明泛型	

   ```ts
   interface Box<Type> {
     contents: Type;
   }
   
   type OneOrMany<Type> = Type | Type[]; //type也可以用泛型，且可以组合更多类型
   ```

5. 元组tuple

   ```ts
   type StringNumberPair = [string, number];//元组就像一种指定长度和类型的array
   
   //可以使用...的语法
   type StringNumberBooleans = [string, number, ...boolean[]];
   type StringBooleansNumber = [string, ...boolean[], number];
   type BooleansStringNumber = [...boolean[], string, number];
   
   function doSomething(pair: readonly [string, number]) { //可以使用readonly语法与as const声明自变量类型语法
     // ...
   }
   ```

## 类型操作

通过已有的类型创建新的类型

1. 两种不同的泛型interface

   ```ts
   interface GenericIdentityFn{ //可以通过这种方式来定义泛型函数类型, 类似定义带有属性的方法类型的语法
     <Type>(arg: Type): Type; 
   }
   function identity<Type>(arg: Type): Type {
     return arg;
   }
   let myIdentity: GenericIdentityFn = identity; //这种情况下泛型对外部是不可见的，不需要赋值，其会自动决定
   
   interface GenericIdentityFn<Type> { //还可以把泛型放到外面
     (arg: Type): Type;
   }
   function identity<Type>(arg: Type): Type {
     return arg;
   }
   let myIdentity: GenericIdentityFn<number> = identity;//此时必须明确传入泛型变量，此处是number
   ```

2. 泛型class

   ```ts
   class GenericNumber<NumType> { //在类名后面跟泛型，且不能对static成员上用泛型
     zeroValue: NumType;
     add: (x: NumType, y: NumType) => NumType;
   }
   let myGenericNumber = new GenericNumber<number>();
   myGenericNumber.zeroValue = 0;
   myGenericNumber.add = function (x, y) {
     return x + y;
   };
   ```


3. keyof操作符

   `keyof` 以对象作为参数，会生成其键的string或者number的字面联合类型

   ```ts
   type Point = { x: number; y: number };
   type P = keyof Point; // type P = "x" | "y"
   ```

4. typeof关键字

   用在变量上

   ```ts
   function f() {
     return { x: 10, y: 3 };
   }
   type P = ReturnType<f>; //此处会报错，因为f是一个value，而不是一个类型值，这时就需要使用typeof来获取函数的类型
   ```

5. 使用泛型参数作为其他泛型的约束

   关键字：`keyof` `in`

   ```ts
   //keyof
   function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
     return obj[key];
   }
   let x = { a: 1, b: 2, c: 3, d: 4 };
   getProperty(x, "a");
   getProperty(x, "m"); //报错 m不存在与x的键中
   
   //in 切记不要用于interface, 否则会报错
   type name = 'firstname' | 'lastname'
   type TName = {
     [key in name]: string
   }
   // TName = { firstname: string, lastname: string }
   
   //使用案例
   function getValue(o:object, key: string){
     return o[key]
   }
   const obj1 = { name: '张三', age: 18 }
   const values = getValue(obj1, 'name') //无法确定返回类型， 且key也无法限定
   //改进
   function getValue<T extends Object, K extends keyof T>(o: T, key: K):T[K]{
     return o[key]
   }
   const obj1 = { name: '张三', age: 18 }
   const values = getValue(obj1, 'name')
   ```

6. 通过键名获取类型

   主要是通过`[]`来获取, **且只能用在type上**

   ```ts
   type Person = { age: number; name: string; alive: boolean };
   type Age = Person["age"]; // age = number
   
   type I1 = Person["age" | "name"];  //还可以和| keyof联合使用
   type I2 = Person[keyof Person];
   type AliveOrName = "alive" | "name";
   type I3 = Person[AliveOrName];
   
   //对于数组类型，还可以使用number来获取所有元素的类型
   const MyArray = [
     { name: "Alice", age: 15 },
     { name: "Bob", age: 23 },
     { name: "Eve", age: 38 },
   ];
   type Person = typeof MyArray[number]; //Person = {name: string, age: number}
   type Age = typeof MyArray[number]["age"]; //Person = number
   ```

7. 条件类型

   ```ts
   type NameOrId<T extends number | string> = T extends number //可以使用三元表达式
     ? IdLabel
     : NameLabel;
   
   type ToArray<Type> = Type extends any ? Type[] : never; 
   type StrArrOrNumArr = ToArray<string | number>;  //三元表达式会对union中的每一个类型进行判断
   //StrArrOrNumArr = string[] | number[]
   ```

8. Mapped类型

   ```ts
   //1. mapped类型通常使用in keyof来组成联合类型，遍历所有的键组成新的类型
   type OptionsFlags<Type> = { 
     [Property in keyof Type]: boolean;
   };
   
   //2. 在mapping的时候还可以通过+，-修改readonly和？（可选）属性，不写+-默认为+
   type CreateMutable<Type> = {
     -readonly [Property in keyof Type]: Type[Property]; //去除readonly属性
   };
   type LockedAccount = {
     readonly id: string;
     readonly name: string;
   };
   type UnlockedAccount = CreateMutable<LockedAccount>;
   //去除可选属性
   type Concrete<Type> = {
     [Property in keyof Type]-?: Type[Property];
   };
   type MaybeUser = {
     id: string;
     name?: string;
     age?: number;
   };
   type User = Concrete<MaybeUser>;
   
   //3. 还可以使用as关键字，对键名重命名
   /*
   type Getters<Type> = {
       [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
   };
   interface Person {
       name: string;
       age: number;
       location: string;
   }
    
   type LazyPerson = Getters<Person>;
   //type LazyPerson = {
       getName: () => string;
       getAge: () => number;
       getLocation: () => string;
   }
   */
   
   //4. 去除某些键
   type RemoveKindField<Type> = {
       [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
   };
   interface Circle {
       kind: "circle";
       radius: number;
   }
   ```

9. 内置操作String的类型

   ```ts
   //1. Uppercase<StringType>
   //2. Lowercase<StringType>
   //3. Capitalize<StringType>
   //4. Uncapitalize<StringType>
   type Greeting = "Hello, world"
   type ShoutyGreeting = Uppercase<Greeting> //ShoutyGreeting = "HELLO, WORLD"
   ```

   