# vue-calendar-ui
这是一个vue日历组件
## 组件使用示例:
#### template代码:
```
<Calendar @change="handlerCalenDate" />
```
#### script代码:
>```import Calendar from './calendar''``` 
```
methods: {
  // 日历日期改变
  handlerCalenDate(date) {
    console.log(date)
  }
}
```
![Image text]('./sketch.png')
