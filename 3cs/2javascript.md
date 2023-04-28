# 目录

[TOC]



# 基础
- console.log("bird");
- pop() pop out the last element
- push() push somthing to the end of the array
- map(..)
  Creates a new array by applying the callback function to every element of the starting array.
- filter()
  const staffNames = ["Nick", "", "Jacky", "", "Philena", "Leo"];
  const validNames = staffNames.filter(name => name !== "");
  // validNames = ["Nick", "Jacky", "Philena", "Leo"]
- [...]symbol,can be used to copy array.
  '[...array]' will unpacked the array elements into the new array.
  array = [1, 2, 3];
  let copiedArray2Times = [...array, ...array,2];//[1,2,3,1,2,3,2]

- === judge equal elements of the same type
 == judge with force type transformation
 (2== '2' is ture)

- anonymous function
  Syntax: (parameters) => output;
  modify(array, x => x+2)

- class


```javascript
class Rectangle {
  constuctor(width, height) {
    this.width = width;
    this.height = height;
  }

  area = () => {
    return this.width * this.height;
  };
}

const rect1 = new Rectangle(3, 4);
console.log(rect1.area());

```



# 脚本
## 自动统计脚本

### up主投稿视频时长
1. 使用方法：https://www.bilibili.com/video/BV1LV4y1g77U  （代码已更新为全自动一键统计，如有错误，欢迎大家指正。）
2. 如果网络不佳建议把第20行 `setInterval()` 的数字调大。该行是点击下一页与统计下一页数据的时间间隔，默认值为400毫秒，网络不佳可能导致计算时网页还未成功刷新，计算出错。
3. 改编自 https://www.52pojie.cn/thread-1517520-1-1.html 在此表示感谢。

#### 全自动版 

```javascript
javascript:(
    function () {
        var hour = 0;
        var minute = 0;
        var second = 0;
        var sum_of_hours = 0;
        var sum_of_minutes = 0;
        var sum_of_seconds = 0;
        var i = 0;
        var num = 0;
        var run = 0;
        var txt = document.getElementsByClassName('count')[0].innerHTML;
        var num_of_videos = txt.match(/\d+/g)[0];
        function f0(){
            if(num_of_videos > 30){
                var j = 0;
                run = setInterval(function(){
                    f1();
                    $('li:contains(下一页)')[0].click();
                },400);
            }
            else{
                f1();
                clearInterval(run);
            }
        }
        function f1() {
            if(num_of_videos <= 30){
                if(document.getElementById('divChild')){
                    d.removeChild(document.getElementById('divChild'));
                }
                for(i = 0;i <= (num_of_videos-1);i++){
                    var time  =  document.getElementsByClassName('length')[i].innerHTML;
                    var t = time.match(/\d+/g);
                    if(t.length == 3){
                        var h = t[0];
                        var m = t[1];
                        var s = t[2];
                    }
                    else{
                        var h = 0;
                        var m = t[0];
                        var s = t[1];
                    }
                    sum_of_hours += parseInt(h);
                    sum_of_minutes += parseInt(m);
                    sum_of_seconds += parseInt(s);
                }
                num += i;
            }
            else{
                if(document.getElementById('divChild')){
                    d.removeChild(document.getElementById('divChild'));
                }
                for(var i = 0;i <= ((num_of_videos - num) < 30 ? (num_of_videos - num) : 30) - 1;i++){
                    var time  =  document.getElementsByClassName('length')[i].innerHTML;
                    var t = time.match(/\d+/g);
                    if(t.length == 3){
                        var h = t[0];
                        var m = t[1];
                        var s = t[2];
                    }
                    else{
                        var h = 0;
                        var m = t[0];
                        var s = t[1];
                    }
                    sum_of_hours += parseInt(h);
                    sum_of_minutes += parseInt(m);
                    sum_of_seconds += parseInt(s);
                }
                num += i;
            }
            second = (sum_of_seconds % 60);
            minute = sum_of_minutes + parseInt(sum_of_seconds/60);
            if (minute > 60){
                hour = sum_of_hours + parseInt(minute/60);
                minute = (minute % 60);
            }
            var d1 = document.createElement('div');
            d1.style.cssText = "margin-top:1px";
            d1.setAttribute("id","divChild");
            d.appendChild(d1);
            var t1 = document.createTextNode("已统计"+num+"个视频:"+hour+"时"+minute+"分"+parseInt(second)+"秒");
            d1.appendChild(t1);
        }
        function over() {
            btn.style.backgroundColor = "#E4E4E4";
        }
        function out() {
            btn.style.backgroundColor = "#F4F4F4";
        }
        var body = document.body;
        var d = document.createElement("div");
        d.style.cssText = "padding-top:1px;width:320px;height:55px;background-color:#F4F4F4;position:fixed;left:5px;bottom:5px;border:1px solid #00A1D6;color:#00A1D6;z-index:999;text-align:center;font-size:16px";
        body.appendChild(d);
        var d2 = document.createElement("div");
        var btn = document.createElement('input');
        btn.setAttribute("type","button");
        btn.setAttribute("value","计算");
        btn.style.cssText = "width:50px;margin-top:1px;border: 1px solid #00A1D6;font-size:18px;cursor:pointer";
        d.appendChild(btn);
        btn.onclick = f0;
        btn.onmouseover = over;
        btn.onmouseout = out;
    })();

```

#### 半自动版
>无需移动鼠标位置，手动控制刷新速度。每一次点击确定键即可同时完成一页视频的数据统计和刷新下一页。（点击过快同样会出现数据不准的情况。）

```javascript
javascript:(
    function () {
        var hour = 0;
        var minute = 0;
        var second = 0;
        var sum_of_hours = 0;
        var sum_of_minutes = 0;
        var sum_of_seconds = 0;
        var i = 0;
        var num = 0;
        var txt = document.getElementsByClassName('count')[0].innerHTML;
        var num_of_videos = txt.match(/\d+/g)[0];
        function f1() {
            if(num_of_videos <= 30){
                if(document.getElementById('divChild')){
                    d.removeChild(document.getElementById('divChild'));
                }
                for(i = 0;i <= (num_of_videos-1);i++){
                    var time  =  document.getElementsByClassName('length')[i].innerHTML;
                    var t = time.match(/\d+/g);
                    if(t.length == 3){
                        var h = t[0];
                        var m = t[1];
                        var s = t[2];
                    }
                    else{
                        var h = 0;
                        var m = t[0];
                        var s = t[1];
                    }
                    sum_of_hours += parseInt(h);
                    sum_of_minutes += parseInt(m);
                    sum_of_seconds += parseInt(s);
                }
                num += i;
            }
            else{
                if(document.getElementById('divChild')){
                    d.removeChild(document.getElementById('divChild'));
                }
                for(var i = 0;i <= ((num_of_videos - num) < 30 ? (num_of_videos - num) : 30) - 1;i++){
                    var time  =  document.getElementsByClassName('length')[i].innerHTML;
                    var t = time.match(/\d+/g);
                    if(t.length == 3){
                        var h = t[0];
                        var m = t[1];
                        var s = t[2];
                    }
                    else{
                        var h = 0;
                        var m = t[0];
                        var s = t[1];
                    }
                    sum_of_hours += parseInt(h);
                    sum_of_minutes += parseInt(m);
                    sum_of_seconds += parseInt(s);
                }
                num += i;
                window.setTimeout(function(){$('li:contains(下一页)')[0].click()},500);
            }
            second = (sum_of_seconds % 60);
            minute = sum_of_minutes + parseInt(sum_of_seconds/60);
            if (minute > 60){
                hour = sum_of_hours + parseInt(minute/60);
                minute = (minute % 60);
            }
            var d1 = document.createElement('div');
            d1.style.cssText = "margin-top:1px";
            d1.setAttribute("id","divChild");
            d.appendChild(d1);
            var t1 = document.createTextNode("已统计"+num+"个视频:"+hour+"时"+minute+"分"+parseInt(second)+"秒");
            d1.appendChild(t1);
        }
        function over() {
            btn.style.backgroundColor = "#E4E4E4";
        }
        function out() {
            btn.style.backgroundColor = "#F4F4F4";
        }
        var body = document.body;
        var d = document.createElement("div");
        d.style.cssText = "padding-top:1px;width:320px;height:55px;background-color:#F4F4F4;position:fixed;left:5px;bottom:5px;border:1px solid #00A1D6;color:#00A1D6;z-index:999;text-align:center;font-size:16px";
        body.appendChild(d);
        var d2 = document.createElement("div");
        var btn = document.createElement('input');
        btn.setAttribute("type","button");
        btn.setAttribute("value","计算");
        btn.style.cssText = "width:50px;margin-top:1px;border: 1px solid #00A1D6;font-size:18px;cursor:pointer";
        d.appendChild(btn);
        btn.onclick = f1;
        btn.onmouseover = over;
        btn.onmouseout = out;
    })();

```

### 视频合集时长（支持任意集数之间，支持倍速计算） 
>使用方法：B站分集视频教程时长统计脚本_哔哩哔哩_bilibili
转载自 https://www.52pojie.cn/thread-1517520-1-1.html
，在此表示感谢。


```javascript
javascript: (function() {
    var hour = 0;
    var minute = 0;
    var second = 0;
    var txt = document.getElementsByClassName('cur-page')[0].innerHTML;
    var page = txt.match(/\/(\d+)/)[1];
    function f1() {
        hour = 0;
        minute = 0;
        second = 0;
        var min = 0;
        var sec = 0;
        if (document.getElementById('divChild')) {
            d.removeChild(document.getElementById('divChild'));
        }
        if (parseInt(input1.value) >= 1 && parseInt(input1.value) <= page && parseInt(input2.value) <= page && parseInt(input1.value) <= parseInt(input2.value)) {
            for (var i = parseInt(input1.value) - 1; i < parseInt(input2.value); i++) {
                var time = document.getElementsByClassName('duration')[i].innerHTML;
                var t = time.match(/\d+/g);
                if (t.length == 3) {
                    var h = t[0];
                    var m = t[1];
                    var s = t[2];
                } else {
                    var h = 0;
                    var m = t[0];
                    var s = t[1];
                }
                hour += parseInt(h);
                min += parseInt(m);
                sec += parseInt(s);
            }
            hour += parseInt(min / 60);
            minute = min - parseInt(min / 60) * 60 + parseInt(sec / 60);
            second = sec - parseInt(sec / 60) * 60;
            if (minute >= 60) {
                hour += parseInt(minute / 60);
                minute = minute - parseInt(minute / 60) * 60;
            }
            if (second >= 60) {
                minute = parseInt(second / 60);
                second = second - parseInt(second / 60) * 60;
            }
            if (input3.value != 1) {
                var total = hour * 3600 + minute * 60 + second;
                total = total / input3.value;
                hour = parseInt(total / 3600);
                minute = parseInt((total % 3600) / 60);
                second = total % 60;
            }
            var d1 = document.createElement('div');
            d1.style.cssText = "margin-top:15px";
            d1.setAttribute("id", "divChild");
            d.appendChild(d1);
            var t1 = document.createTextNode("全" + (parseInt(input2.value) - parseInt(input1.value) + 1) + "集:" + hour + "时" + minute + "分" + parseInt(second) + "秒");
            d1.appendChild(t1);
        } else {
            var d1 = document.createElement('div');
            d1.style.cssText = "margin-top:15px";
            d1.setAttribute("id", "divChild");
            d.appendChild(d1);
            var t1 = document.createTextNode("输入与实际集数不符");
            d1.appendChild(t1);
        }
    }
    function over() {
        btn.style.backgroundColor = "#E4E4E4";
    }
    function out() {
        btn.style.backgroundColor = "#F4F4F4";
    }
    var body = document.body;
    var d = document.createElement("div");
    d.style.cssText = "padding-top:15px;width:145px;height:135px;background-color:#F4F4F4;position:absolute;right:55px;top:218px;border:1px solid #00A1D6;color:#00A1D6;z-index:999;text-align:center;font-size:14px";
    body.appendChild(d);
    var d2 = document.createElement("div");
    d.appendChild(d2);
    var t2 = document.createTextNode("第");
    d2.appendChild(t2);
    var input1 = document.createElement('input');
    input1.setAttribute("type", "number");
    input1.style.cssText = "border: 1px solid deepskyblue;width:40px";
    d2.appendChild(input1);
    var t3 = document.createTextNode("集到");
    d2.appendChild(t3);
    var input2 = document.createElement('input');
    input2.setAttribute("type", "number");
    input2.style.cssText = "border: 1px solid deepskyblue;width:40px";
    d2.appendChild(input2);
    var t4 = document.createTextNode("集");
    d2.appendChild(t4);
    var btn = document.createElement('input');
    btn.setAttribute("type", "button");
    btn.setAttribute("value", "计算");
    btn.style.cssText = "width:50px;margin-top:15px;border: 1px solid #00A1D6;cursor:pointer";
    d.appendChild(btn);
    btn.onclick = f1;
    btn.onmouseover = over;
    btn.onmouseout = out;
    var t5 = document.createTextNode("倍速：");
    d2.appendChild(t5);
    var input3 = document.createElement('input');
    input3.setAttribute("type", "number");
    input3.style.cssText = "border: 1px solid deepskyblue;width:50px;margin-top:15px;margin-right:10px";
    input3.value = 1;
    d2.appendChild(input3);
    var t6 = document.createTextNode("倍");
    d2.appendChild(t6);
})();
​
```
