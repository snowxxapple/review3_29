# 2016.3.29复习内容

## callee caller call apply的用法与区别

##### 同时涉及到了`arguments`,函数内部`this`指向谁,自执行函数
---

### `callee`属性

`callee`属性表示当前的函数体

**示例:**

``` javascript
function fn(){
	console.log(arguments.callee);//函数本身
}
fn();
```

函数都有一个接受参数的数组arguments[]

### `caller`属性

`caller`属性表示调用当前函数的函数,并不是调用该函数的对象(并不是`this`指向)

**示例:**

``` javascript
function a(){
		console.log(a.caller,'调用a的函数');//null 自执行函数不能用caller
	}
	a();

	function b(){
		console.log(b.caller,'调用b的函数');//函数c()
	}
	function c(){
		b();
	}
	c();
```

### `call`方法

通过`call`方法调用的函数，可以更改函数体内部this的指向问题，即改变调用当前函数的对象；同时可以传参,`call(obj,para1,para2,....)`

**示例1:**

未指定m函数内部的`this`,则`this`为调用m函数的对象，此处为`window`对象

```javascript
function m(a,b){
	console.log(this,'未指定this');//window
	console.log(a,b,'m(2,3)');
}
m.call(null,2,3);
```

**示例2:**

通过`call`函数指定函数内部this的指向

```javascript
function n(a,b){
		console.log(arguments,'arguments数组');//[3,4]
		console.log(this,'指定this');//document
		console.log(a,b,'n(document,3,4)');//(3,4)
	}
	n.call(document,3,4);

```

### `apply`方法

通过`apply`方法调用的函数，可以改变函数内部`this`的指向，即改变调用当前函数的对象；同时可以传数组形式的参数，会自动将`[para1,para2,....]`转化为`(para1,para2,....)`这个特性非常重要，使得apply有其他巧妙的用处。`apply(obj,[para1,para2,....])`

**示例:**

```javascript
function x(args){ 
		console.log(this,'apply没改this');//window this指向的是调用当前函数体的对象
		console.log(args,'apply参数列表');
	}
	x.apply(null,[1,2,3,4]);//调用x函数的是window对象

	function y(a,b,c,d){
		console.log(this,'apply改变this');//document
		console.log(a,b,c,d,'apply参数列表');
	}
	y.apply(document,[6,7,8,9]);
```

** `apply()`和`call()`函数在使用的时候，如果不改变函数内部`this`对指向，要设为`null`，不可以不设置 **

### `apply()`方法的妙用

`apply`方法可以劫持调用`apply()`方法的对象的方法，继承该对象的属性。

+ **继承属性(面向对象编程中常用的)**

**示例:**

``` javascript
function Person(name,age){
	this.name=name;
	this.age=age;
}
function Student(name,age,grade){
	Person.apply(Student,arguments);
	this.grade=grade;
}
var student=new Student('xx','12','一年级');
console.log("name:"+student.name+';年龄:'+student.age+';年级:'+student.grade);
```

调用`apply`方法的是`Person`函数，改变`Person`函数内部`this`的指向为`Student`,并传入了`arguments['xx','12','一年级']`参数，但是只用了`name`,`age`两个属性值，那么`Student.name=name;` `Student.age=age;`因此`Student`构造函数继承了父类`Person`的两个属性值，除此之外`Student`构造函数还给`Student`对象添加了`grade`新属性。

+ **劫持方法**

**示例1:**

`Math.min()`,`Math.min()`的参数都只能是`para1,para2,... `参数列表，而不能是数组，而现在要在数组中找到最大值和最小值，则可以用`apply()`方法，把参数数组转化成参数列表。

```javascript
var arr=[1,2,5,6,3,8,22,44];
var result=Math.min.apply(null,arr);
console.log(result);//1
```

没有对象去调用`Math.min()`方法，只是返回一个结果，因此保持原有的调用对象，令`obj`为`null`就可以，直接调用`apply(null,arr)`就可以。

** 示例2: **

假设要连接两个数组，(尽管`arr.concat()`可以直接完成)

```javascript
var arr1=[1,2,3];
var arr2=[4,5,6];
arr1.push.apply(arr1,arr2);
或者：Array.prototype.push.apply(arr1,arr2);
console.log(arr1);//[1,2,3,4,5,6]
```

`push()`方法提供了向数组尾部插入数据`para1,para2,...`的功能，要利用`apply`将数组`arr2`转化成参数列表。由于是要`arr1`调用`push`方法，因此`apply`方法内应该指明调用`push`方法的对象是`arr1`,虽然`arr1`本身就有`push`方法，但是它只是`Array`的一个实例，`push`方法中原本的`this`是`Array`，所以`apply`内第一个参数一定是`arr1`,而不是`null`。

### 自执行函数

函数声明并立即执行

**示例1:**

```javascript
(function(){
	console.log('1');
})(接收的参数);

or:

~function(){
	
}(接收的参数);

or:

+function(){
	
}(接收的参数);

```

**示例2:**

利用自执行函数来做选项卡，可以不用给按钮添加索引，直接用接收到的形参即可。

```javascript

var oIput=document.getElementsByTagName('li');
var oP=document.getElementsByClassName('display');
	for(var i=0;i<oIput.length;i++){
		(function(index){//传入参数i
			oIput[index].onclick=function(){
				for(var j=0;j<oIput.length;j++){
					oIput[j].style.background='';
					oP[j].style.display='none';
					}
				oIput[index].style.background='yellow';
				oP[index].style.display='block';
				}
			})(i);//参数i传入立即执行函数
		}

```
