##### 使用DocumentFragment优化多次append #####

添加多个dom元素时，先将元素append到DocumentFragment中，最后统一将DocumentFragment添加到页面。该做法可以减少页面渲染dom元素的次数。经IE和FX下测试，在append1000个元素时，效率能提高10%-30%，FX下提升较为明显。

前：
```javascript
for (var i = 0; i < 1000; i++) {
    var el = document.createElement('p');
    el.innerHTML = i;
    document.body.appendChild(el);
}
```
后：
```javascript
var frag = document.createDocumentFragment();
for (var i = 0; i < 1000; i++) {
    var el = document.createElement('p');
    el.innerHTML = i;
    frag.appendChild(el);
}
document.body.appendChild(frag);
```

##### 通过模板元素clone，替代createElement #####

通过一个模板dom对象cloneNode，效率比直接创建element高。性能提高不明显，约为10%左右。在低于100个元素create和append操作时，没有优势。

```javascript
var frag = document.createDocumentFragment();
for (var i = 0; i < 1000; i++) {
    var el = document.createElement('p');
    el.innerHTML = i;
    frag.appendChild(el);
}
document.body.appendChild(frag);    
```

```javascript
var frag = document.createDocumentFragment();
var pEl = document.getElementsByTagName('p')[0];
for (var i = 0; i < 1000; i++) {
    var el = pEl.cloneNode(false);
    el.innerHTML = i;
    frag.appendChild(el);
}
document.body.appendChild(frag);
```

##### 使用一次innerHTML赋值代替构建dom元素 #####

根据数据构建列表样式的时候，使用设置列表容器innerHTML的方式，比构建dom元素并append到页面中的方式，效率有数量级上的提高。

```javascript
var frag = document.createDocumentFragment();
for (var i = 0; i < 1000; i++) {
    var el = document.createElement('p');
    el.innerHTML = i;
    frag.appendChild(el);
}
document.body.appendChild(frag);
```

```javascript
var html = [];
for (var i = 0; i < 1000; i++) {
    html.push('<p>' + i + '</p>');
}
document.body.innerHTML = html.join('');
```

##### 使用firstChild和nextSibling代替childNodes遍历dom元素 #####

约能获得30%-50%的性能提高。逆向遍历时使用lastChild和previousSibling。

```javascript
var nodes = element.childNodes;
for (var i = 0, l = nodes.length; i < l; i++) {
var node = nodes[i];
//……
}
```

```javascript
var node = element.firstChild;
while (node) {
//……
node = node.nextSibling;
}
```

##### 使用Array做为StringBuffer，代替字符串拼接的操作 #####

IE在对字符串拼接的时候，会创建临时的String对象；经测试，在IE下，当拼接的字符串越来越大时，运行效率会急剧下降。Fx和Opera都对字符串拼接操作进行了优化；经测试，在Fx下，使用Array的join方式执行时间约为直接字符串拼接的1.4倍。

```javascript
var now = new Date();
var str = '';
for (var i = 0; i < 10000; i++) {
    str += '123456789123456789';
}
alert(new Date() - now);
```

```javascript
var now = new Date();
var strBuffer = [];
for (var i = 0; i < 10000; i++) {
    strBuffer.push('123456789123456789');
}
var str = strBuffer.join('');
alert(new Date() - now);
```

##### 将循环控制量保存到局部变量 #####

对数组和列表对象的遍历时，提前将length保存到局部变量中，避免在循环的每一步重复取值。

```javascript
var list = document.getElementsByTagName('p');
for (var i = 0; i < list.length; i++) {
    //……
}
```

```javascript
var list = document.getElementsByTagName('p');
for (var i = 0, l = list.length; i < l; i++) {
    //……
}
```

##### 顺序无关的遍历时，用while替代for #####

该方法可以减少局部变量的使用。比起效率优化，更能直接看到的是字符数量的优化。该做法有程序员强迫症的嫌疑。

```javascript
var arr = [1,2,3,4,5,6,7];
var sum = 0;
for (var i = 0, l = arr.length; i < l; i++) {
    sum += arr[i];
}    
```

```javascript
var arr = [1,2,3,4,5,6,7];
var sum = 0, l = arr.length;
while (l--) {
    sum += arr[l];
}
```

##### 使用三目运算符替代条件分支 #####

```javascript
if (a > b) {
num = a;
} else {
num = b;
}
```

```javascript
num = a > b ? a : b;
```

##### 需要不断执行的时候，优先考虑使用setInterval #####

setTimeout每一次都会初始化一个定时器。setInterval只会在开始的时候初始化一个定时器。

```javascript
var timeoutTimes = 0;
function timeout () {
    timeoutTimes++;
    if (timeoutTimes < 10) {
        setTimeout(timeout, 10);
    }
}
timeout();
```

```javascript
var intervalTimes = 0;
function interval () {
    intervalTimes++;
    if (intervalTimes >= 10) {
        clearInterval(interv);
    }
}
var interv = setInterval(interval, 10);
```

##### 使用function而不是string #####

如果把字符串作为setTimeout和setInterval的参数，浏览器会先用这个字符串构建一个function。

```javascript
var num = 0;
setTimeout('num++', 10);
```

```javascript
var num = 0;
function addNum () {
    num++;
}
setTimeout(addNum, 10);
```

##### 重复使用的调用结果，事先保存到局部变量 #####

避免多次取值的调用开销。

```javascript
var h1 = element1.clientHeight + num1;
var h2 = element1.clientHeight + num2;
```

```javascript
var eleHeight = element1.clientHeight;
var h1 = eleHeight + num1;
var h2 = eleHeight + num2;
```

##### 使用直接量 #####

```javascript
var a = new Array(param,param,...) -> var a = []
var foo = new Object() -> var foo = {}
var reg = new RegExp() -> var reg = /.../
```

##### 避免使用with #####

with虽然可以缩短代码量，但是会在运行时构造一个新的scope。OperaDev上还有这样的解释，使用with语句会使得解释器无法在语法解析阶段对代码进行优化。对此说法，无法验证。

```javascript
with (a.b.c.d) {
property1 = 1;
property2 = 2;
}
```

```javascript
var obj = a.b.c.d;
obj.property1 = 1;
obj.property2 = 2;
```

##### 巧用||和&&布尔运算符 #####

```javascript
function eventHandler (e) {
if(!e) e = window.event;
}
```

```javascript
function eventHandler (e) {
e = e || window.event;
}
```

##### 还有下面的例子： #####

```javascript
if (myobj) {
doSomething(myobj);
}
```

```javascript
myobj && doSomething(myobj);
```

##### 类型转换 #####

数字转换成字符串，应用"" + 1，性能上：("" +) > String() > .toString() > new String()；

浮点数转换成整型，不使用parseInt()， parseInt()是用于将字符串转换成数字，而不是浮点数和整型之间的转换，建议使用Math.floor()或者Math.round()。

对于自定义的对象，推荐显式调用toString()。内部操作在尝试所有可能性之后，会尝试对象的toString()方法尝试能否转化为String。

##### 循环引用 #####

如果循环引用中包含DOM对象或者ActiveX对象，那么就会发生内存泄露。内存泄露的后果是在浏览器关闭前，即使是刷新页面，这部分内存不会被浏览器释放。

简单的循环引用：

```javascript
var el = document.getElementById('MyElement');
var func = function () {…}
el.func = func;
func.element = el; 
```

但是通常不会出现这种情况。通常循环引用发生在为dom元素添加闭包作为expendo的时候。

```javascript
function init() {
    var el = document.getElementById('MyElement');
el.onclick = function () {……}
}
init();
```

init在执行的时候，当前上下文我们叫做context。这个时候，context引用了el，el引用了function，function引用了context。这时候形成了一个循环引用。

##### 置空dom对象 #####

```javascript
function init() {
var el = document.getElementById('MyElement');
el.onclick = function () {……}
}
init();
```

```javascript
function init() {
var el = document.getElementById('MyElement');
el.onclick = function () {……}
el = null;
}
init();
```

将el置空，context中不包含对dom对象的引用，从而打断循环应用。如果我们需要将dom对象返回，可以用如下方法：

```javascript
function init() {
    var el = document.getElementById('MyElement');
    el.onclick = function () {……}
    return el;
}
init();
```

```javascript
function init() {
var el = document.getElementById('MyElement');
el.onclick = function () {……}
try{
return el;
} finally {
    el = null;
}
}
init();
```

构造新的context：

```javascript
function init() {
    var el = document.getElementById('MyElement');
    el.onclick = function () {……}
}
init();
```

```javascript
function elClickHandler() {……}
function init() {
    var el = document.getElementById('MyElement');
    el.onclick = elClickHandler;
}
init();
```

把function抽到新的context中，这样，function的context就不包含对el的引用，从而打断循环引用。

##### 通过javascript创建的dom对象，必须append到页面中 #####

IE下，脚本创建的dom对象，如果没有append到页面中，刷新页面，这部分内存是不会回收的！

```javascript
function create () {
       var gc = document.getElementById('GC');
       for (var i = 0; i < 5000 ; i++)
       {
           var el = document.createElement('div');
           el.innerHTML = "test";
           //下面这句可以注释掉，看看浏览器在任务管理器中，点击按钮然后刷新后的内存变化
           gc.appendChild(el);
       }
   }
```

##### 释放dom元素占用的内存 #####

将dom元素的innerHTML设置为空字符串，可以释放其子元素占用的内存。

在rich应用中，用户也许会在一个页面上停留很长时间，可以使用该方法释放积累得越来越多的dom元素使用的内存。

##### 释放javascript对象 #####

在rich应用中，随着实例化对象数量的增加，内存消耗会越来越大。所以应当及时释放对对象的引用，让GC能够回收这些内存控件。

对象：obj = null，对象属性：delete obj.myproperty。

数组item：使用数组的splice方法释放数组中不用的item。

##### 避免string的隐式装箱 #####

对string的方法调用，比如'xxx'.length，浏览器会进行一个隐式的装箱操作，将字符串先转换成一个String对象。推荐对声明有可能使用String实例方法的字符串时，采用如下写法：

```javascript
var myString = new String('Hello World');
```
