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

   