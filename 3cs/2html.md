# 目录

[TOC]




- div 把一些标签内容嵌套进一个 box 里面
- span 类似 div，但是内嵌 box,
-


- 箭头函数`()=>(xxx)`等效于`()=> {return (xxx)}`，是隐式返回的函数，甚至可以简化`()=>xxx`

```JSX
const Button = ({ handleClick, text }) => (
  <button onClick={handleClick}>{text}</button>
)


const Button = ({ handleClick, text }) => {
  return (<button onClick={handleClick}>{text}</button>)
}


```
