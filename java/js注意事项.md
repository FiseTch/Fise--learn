### `js`注意事项

#### 变量的赋值

-------------



>对于求值获取的变量，需要**兜底**

```js
const MIN_NAME_LENGTH = 8;

let lastName = fullName[1]; //这样代码存在一个问题，当fullName长度为1时，数组越界，lastName取不到值，因此lastName会被定义为 undefined 

if(lastName.length > MIN_NAME_LENGTH) { //当lastName为undefined时，会造成代码出错
    ....
}

//正确的代码为
const MIN_NAME_LENGTH = 8;

let lastName = fullName[1] || '';

if(lastName.length > MIN_NAME_LENGTH) {
    ....
}
其实在项目中有很多求值变量，对于每个求值变量都需要做好兜底。
let propertyValue = Object.attr || 0; // 因为Object.attr有可能为空，所以需要兜底。


```



##### `js`中逻辑运算符的使用

##### 数值类型的逻辑运算

----------

```js
	console.log( 5 && 4 );//当结果为真时，返回第二个为真的值4 
    console.log( 0 && 4 );//当结果为假时，返回第一个为假的值0 
	console.log( 5 && 0 );//当结果为假时，返回第一个为假的值0 
	console.log("---------------------")
    console.log( 5 || 4 );//当结果为真时，返回第一个为真的值5 
	console.log( 0 || 4 );//当结果为真时，返回第一个为真的值4 
    console.log( 0 || 0 );//当结果为假时，返回第二个为假的值0 
	console.log("---------------------")
    console.log((3||2)&&(5||0));//5 
    console.log(!5);//false 
```

结果

```js
4
0
0
------------------------
5
4
0
-------------------------
5
false
```

总结·

**进行与运算时，同真则真，返回第二个值，一假则假，返回其中第一个假的值（若只有一个假值，则返回那个假值，无须在意其位置）**

**进行非运算时，同假则假，一真则真，当结果为真时，返回第一个真的值（若只有一个真值，则返回那个真值，无须在意其位置），当结果为假时，返回第二个假值**



##### 表达式类型的逻辑运算

---

>运算逻辑与数字型逻辑一致，但是对于对象有额外的判断true和false的逻辑
>
>具体判断逻辑为
>
>转换规则:
>
>==对象为true；==
>==非零数字为true；==
>
>==零为false;==
>
>==非空字符串为true；==
>==空字符串为false;==
>==其他为false；==







