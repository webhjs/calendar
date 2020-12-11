<template>
  <div class="sign-flag">
    <div id="calendar">
      <span class="flag">{{ currentMonth }}</span>
      <!-- 年份 月份 -->
      <div class="month">
        <ul>
          <!--点击会触发pickpre函数，重新刷新当前日期 @click(vue v-on:click缩写) -->
          <li class="arrow" @click="pickPre(currentYear,currentMonth)">
            <span class="arrow-left" />
          </li>
          <li class="year-month">
            <span class="choose-year">{{ currentYear }}年</span>
            <span class="choose-month">{{ currentMonth }}月</span>
          </li>
          <li class="arrow" @click="pickNext(currentYear,currentMonth)">
            <span class="arrow-right" />
          </li>
        </ul>
      </div>
      <!-- 星期 -->
      <ul class="weekdays">
        <li>日</li>
        <li>一</li>
        <li>二</li>
        <li>三</li>
        <li>四</li>
        <li>五</li>
        <li>六</li>
      </ul>
      <!-- 日期 -->
      <ul class="days">
        <!-- 核心 v-for循环 每一次循环用<li>标签创建一天 -->
        <li v-for="(dayobject,i) in days" :key="i">
          <!--本月-->
          <!--如果不是本月  改变类名加灰色-->
          <span v-if="dayobject.day.getMonth()+1 != currentMonth" class="other-month" @click="getDayTime(dayobject.day)">{{ dayobject.day.getDate() }}</span>
          <!--如果是本月  还需要判断是不是这一天-->
          <span v-else>
            <!--今天  同年同月同日-->
            <span v-if="dayobject.day.getFullYear() === selectObject.currentYear && dayobject.day.getMonth() + 1 === selectObject.currentMonth && dayobject.day.getDate() === selectObject.currentDay" class="active" @click="getDayTime(dayobject.day)">{{ comTody(dayobject.day) ? '今' : dayobject.day.getDate() }}</span>
            <span v-else @click="getDayTime(dayobject.day)">{{ comTody(dayobject.day) ? '今' : dayobject.day.getDate() }}</span>
          </span>
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      currentDay: new Date().getDate(),
      currentMonth: new Date().getMonth() + 1,
      currentYear: new Date().getFullYear(),
      currentWeek: new Date().getDay(),
      days: [],
      selectObject: {
        currentDay: new Date().getDate(),
        currentMonth: new Date().getMonth() + 1,
        currentYear: new Date().getFullYear(),
        currentWeek: new Date().getDay()
      }
    }
  },
  created: function() {
    // 在vue初始化时调用
    this.initData(null)
  },
  methods: {
    comTody(param) {
      return String(param) === String(new Date(new Date().setHours(8, 0, 0, 0)))
    },
    initData: function(cur) {
      // var leftcount = 0 // 存放剩余数量
      var date
      if (cur) {
        date = new Date(cur)
      } else {
        var now = new Date()
        var d = new Date(this.formatDate(now.getFullYear(), now.getMonth(), 1))
        d.setDate(41)
        date = new Date(this.formatDate(d.getFullYear(), d.getMonth() + 1, 1))
      }
      this.currentDay = date.getDate()
      this.currentYear = date.getFullYear()
      this.currentMonth = date.getMonth() + 1
      this.currentWeek = date.getDay() // 1...6,0
      if (this.currentWeek === 0) {
        this.currentWeek = 7
      }
      var str = this.formatDate(
        date.getFullYear(),
        date.getMonth() + 1,
        date.getDate()
      )
      this.days.length = 0
      // 今天是周日，放在第一行第7个位置，前面6个
      // 初始化本周
      for (var i = this.currentWeek; i >= 0; i--) {
        var d2 = new Date(str)
        d2.setDate(d2.getDate() - i)
        var dayobjectSelf = {} // 用一个对象包装Date对象  以便为以后预定功能添加属性
        dayobjectSelf.day = d2
        this.days.push(dayobjectSelf) // 将日期放入data 中的days数组 供页面渲染使用
      }
      // 其他周
      for (var j = 1; j <= 41 - this.currentWeek; j++) {
        var d3 = new Date(str)
        d3.setDate(d3.getDate() + j)
        var dayobjectOther = {}
        dayobjectOther.day = d3
        this.days.push(dayobjectOther)
      }
    },
    getDayTime(el) {
      // 选择日期的年月日
      const currentWeek = new Date(el).getDay()
      const currentYear = new Date(el).getFullYear()
      const currentMonth = new Date(el).getMonth() + 1
      const currentDay = new Date(el).getDate()
      // 判断选择日期是上月下月
      let nextMonth = ''
      if (currentYear > this.currentYear) {
        nextMonth = 'next'
      } else if (currentYear === this.currentYear) {
        if (currentMonth > this.currentMonth) {
          nextMonth = 'next'
        } else if (currentMonth < this.currentMonth) {
          nextMonth = 'prev'
        }
      } else {
        nextMonth = 'prev'
      }
      // 定义日期执行函数map
      const operateMap = {
        prev: (() => {
          let tempYear, tempMonth
          if (currentMonth + 1 === 13) {
            tempYear = currentYear + 1
            tempMonth = 1
          } else {
            tempYear = currentYear
            tempMonth = currentMonth + 1
          }
          return this.pickPre.bind(this, tempYear, tempMonth)
        })(),
        next: (() => {
          let tempYear, tempMonth
          if (currentMonth - 1 === 0) {
            tempYear = currentYear - 1
            tempMonth = 12
          } else {
            tempYear = currentYear
            tempMonth = currentMonth - 1
          }
          return this.pickNext.bind(this, tempYear, tempMonth)
        })()
      }
      if (nextMonth) {
        operateMap[nextMonth]()
      }
      // 当前选择日期给active样式赋值
      this.selectObject = {
        currentDay,
        currentMonth,
        currentYear,
        currentWeek
      }
      this.$emit('change', el)
    },
    pickPre: function(year, month) {
      // setDate(0); 上月最后一天
      // setDate(-1); 上月倒数第二天
      // setDate(dx) 参数dx为 上月最后一天的前后dx天
      var d = new Date(this.formatDate(year, month, 1))
      d.setDate(0)
      this.initData(this.formatDate(d.getFullYear(), d.getMonth() + 1, 1))
    },
    pickNext: function(year, month) {
      var d = new Date(this.formatDate(year, month, 1))
      d.setDate(41)
      this.initData(this.formatDate(d.getFullYear(), d.getMonth() + 1, 1))
    },
    // 返回 类似 2016-01-02 格式的字符串
    formatDate: function(year, month, day) {
      var y = year
      var m = month
      if (m < 10) m = '0' + m
      var d = day
      if (d < 10) d = '0' + d
      return y + '-' + m + '-' + d
    }
  }
}
</script>
<style lang="scss" scoped>
ul,li{
  list-style: none;
}
.sign-flag{
  position: relative;
  width: 100%;
  height: 100%;
  &::before{
    position: absolute;
    top: 0;
    left: 35px;
    transform: translateY(-50%);
    content: ' ';
    display: inline-block;
    width: 13px;
    height: 35px;
    background: #FF5500;
    box-shadow: 2px 2px 5px 0px rgba(19, 217, 108, 0.47);
    border-radius: 7px;
    z-index: 3;
  }
  &::after{
    position: absolute;
    top: 0;
    right: 35px;
    transform: translateY(-50%);
    content: ' ';
    display: inline-block;
    width: 13px;
    height: 35px;
    background: #FF5500;
    box-shadow: 2px 2px 5px 0px rgba(19, 217, 108, 0.47);
    border-radius: 7px;
    z-index: 3;
  }
}
#calendar {
  height: 100%;
  font-size: 12px;
  width: 100%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  border-radius: 32px;
  box-shadow: 0 2px 2px 0 #C7F8CC, 0 3px 1px -2px #C7F8CC, 0 1px 5px 0 #C7F8CC;
  overflow: hidden;
  background: #f9fef9;
  position: relative;
  .flag{
    position: absolute;
    display: inline-block;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    font-size: 234px;
    color: #EFFBEE;
    z-index: 0;
    top: 55px;
  }
}
.month {
  width: 100%;
  color: #333333;
  z-index: 0;
}
.month ul {
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 65px;
  background-color: #C7F8CC;
}
.month ul .year-month{
  width: 160px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #2EA438;
  font-size: 0;
  .choose-year,.choose-month{
    font-size: 25px;
  }
}
.choose-month {
  text-align: center;
  font-size: 12px;
}
.arrow{
  display: flex;
  justify-content: center;
  align-content: center;
  padding: 20px;
}
.arrow-left, .arrow-right{
  cursor: pointer;
  display: inline-block;
  width: 10px;
  height: 10px;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
}
.arrow-left {
  border-left: 10px solid transparent;
  border-right: 10px solid #2EA438;
}
.arrow-right {
  border-left: 10px solid #2EA438;
  border-right: 10px solid transparent;
}
.month ul li {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 3px;
  line-height: 45px;
}
.weekdays {
  margin: 0;
  padding: 10px;
  display: flex;
  flex-wrap: wrap;
  color: #36363A;
  line-height: 75px;
  justify-content: space-around;
  font-size: 25px;
  z-index: 0;
  &:after{
    display: inline-block;
    content: ' ';
    width: 100%;
    height: 1px;
    background-color: #2EA438;
  }
}
.weekdays li {
  display: inline-block;
  width: 13.6%;
  text-align: center;
}
.days {
  flex: 1;
  padding: 10px;
  margin: 0;
  display: flex;
  flex-wrap: wrap;
}
.days li {
  list-style-type: none;
  display: inline-block;
  width: 14.2%;
  text-align: center;
  padding-bottom: 4px;
  padding-top: 10px;
  font-size: 24px;
  color: #000;
  cursor: pointer;
  position: relative;
}
.days li .active {
  position: absolute;
  left: 17px;
  top: 6px;
  display: inline-block;
  width: 38px;
  height: 38px;
  line-height: 38px;
  border-radius: 50%;
  background: #2EA438;
  color: #fff;
}
.days li .other-month {
  padding: 5px;
  color: gainsboro;
}
.days li:hover>span>span {
  position: absolute;
  left: 17px;
  top: 6px;
  display: inline-block;
  width: 38px;
  height: 38px;
  line-height: 38px;
  border-radius: 50%;
  background: #e1e1e1;
  color: #fff;
}
</style>
