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





