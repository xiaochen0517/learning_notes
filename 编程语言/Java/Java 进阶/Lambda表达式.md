# Lambda 表达式

`actuator` 执行器

`target` 目标方法

```js
function Hand(func){
    func();
}

function Hammer(){
	console.log("use hammer");
}

Hand(Hammer);

Hand(function BigHammer(){
	console.log("this is big hammer");
});

Hand(()=>{
	console.log("this is my code");
});
```





```java
```

