<template>
<!-- 组件根元素 -->
<div id="timer">
  <!-- 倒计时面板 -->
  <div id="timer-dash">
    <!-- 倒计时 -->
    <div id="count-down">
      <!-- 倒计时圆环 -->
      <svg
          :width="size"
          :height="size">
        <circle
            :cx="circleOffset"
            :cy="circleOffset"
            :r="radius"
            fill="transparent"
            stroke="#282828"
            :stroke-width="strokeWidth">
        </circle>
        <circle
            id="timer-ring"
            :cx="circleOffset"
            :cy="circleOffset"
            :r="radius"
            fill="transparent"
            stroke="#ffa000"
            stroke-linecap="round"
            :stroke-width="strokeWidth"
            :stroke-dasharray="circumference"
            :stroke-dashoffset="progress">
        </circle>
      </svg> 
      <!-- 倒计时剩余时间 -->
      <span id="left-time" :style="{fontSize: digitFontSize}">
        <span class="digits" :class="{hidden}">{{ leftHours }}</span>
        <span class="colon" :class="{hidden}">:</span>
        <span class="digits"> {{ leftMinutes }} </span>
        <span class="colon">:</span>
        <span class="digits">{{ leftSeconds }}</span>
      </span>
      <!-- 倒计时结束之间 -->
      <span id="alarm-time" :class="{paused}">
        <span>
          <font-awesome-icon icon="bell" />
        </span>
        <span>{{ alarmDayNight }}</span>
        <span class="digits">{{ alarmHours }}</span>
        <span class="colon">:</span>
        <span class="digits">{{ alarmMinutes }}</span>
      </span>
    </div>
    <!-- 计时时间设置 -->
    <div id="time-setting" :class="{hidden: timer}">
      <!-- 轮盘组合容器 -->
      <div id="wheels-container">
        <!-- 各个轮盘 -->
        <div 
            :id="`${part.name}-digits-container`"
            class="wheel-part"
            v-for="part in wheels"
            :key="part.label">
          <!-- 轮盘上各个数码 -->
          <div
              class="wheel-digits"
              :class="{
                'moved-before': part.isMoving.before,
                'moved-after': part.isMoving.after,
              }"
              v-for="digit in part.shownDigits"
              :key="digit">
            {{ digit }}
          </div>
        </div>
      </div>
      <!-- 选中时间窗口 -->
      <div id="wheel-window">
        <!-- 各个选中时间 -->
        <div 
            class="window-part"
            v-for="part in wheels"
            :key="part.label">
          <!-- 选中时间数码 -->
          <div class="part-digits-container">
            <div
                class="part-digits"
                :class="{
                  'moved-before': part.isMoving.before,
                  'moved-after': part.isMoving.after,
                }"
                :ref="part.name + 'Window'"
                :data-before="part.shownDigits[3]"
                :data-after="part.shownDigits[5]">
              {{ part.shownDigits[4] }}
            </div>
          </div>
          <!-- 选中时间单位标签 -->
          <div class="part-label">
            <span>
              {{ part.label }}
            </span>
          </div>
        </div>
      </div>
      <!-- 轮盘拨动控制器，接收鼠标滚轮事件 -->
      <div id="controlers-container">
        <div class="controler" ref="hoursCtl"></div>
        <div class="controler" ref="minutesCtl"></div>
        <div class="controler" ref="secondsCtl"></div>
      </div>
    </div>
  </div>
  
  <!-- 倒计时按钮 -->
  <div id="timer-buttons">
    <!-- 取消按钮 -->
    <button @click="cancel" :disabled="cancelBtnDisabled">
      <span>取消</span>
    </button>
    <!-- 控制按钮 -->
    <button @click="control" :class="[controlBtnState]">
      <span>{{ controlBtnTag }}</span>
    </button>
  </div>
  
  <!-- 计时触发消息 -->
  <div id="timer-message" :class="{triggered}" @click="alarmOff">
    <!-- 消息头，显示应用及弹出时间 -->
    <div class="timer-msg-head">
      <!-- 消息所属 App -->
      <div class="app-info">
        <img src="../assets/images/clock-icon.png" alt="icon">
        <span>时钟</span>
      </div>
      <!-- 消息弹出时间 -->
      <div class="trigger-time">
        <span>{{ alarmDayNight }}</span>
        <span class="digits">{{ alarmHours }}</span>
        <span class="colon">:</span>
        <span class="digits">{{ alarmMinutes }}</span>
      </div>
    </div>
    <!-- 消息主体，显示消息内容 -->
    <div class="timer-msg-body">
      <span>计时器</span>
    </div>
  </div>
</div>
</template>


<script>
export default {
  data() {
    return {
      // 基础尺寸
      size: 300,
      strokeWidth: 6,
      // 倒计时器进程
      timer: null,
      leftMilliSeconds: 0,
      alarmTime: new Date(),
      // 倒计时器控制
      paused: false,
      btnTags: new Map([["stopped", "开始计时"], ["running", "暂停"], ["paused", "继续"]]),
      // 计时器触发
      triggered: false,
      // 倒计时时间设置轮盘
      wheels: {
        hoursWheel: {
          name: "hours",
          label: "小时",
          maxDigit: 23,
          shownDigits: [20, 21, 22, 23, 0, 1, 2, 3, 4],
          isMoving: {
            before: false,
            after: false,
          },
        },
        minutesWheel: {
          name: "minutes",
          label: "分钟",
          maxDigit: 59,
          shownDigits: [56, 57, 58, 59, 0, 1, 2, 3, 4],
          isMoving: {
            before: false,
            after: false,
          },
        },
        secondsWheel: {
          name: "seconds",
          label: "秒",
          maxDigit: 59,
          shownDigits: [56, 57, 58, 59, 0, 1, 2, 3, 4],
          isMoving: {
            before: false,
            after: false,
          },
        },
      },
    }
  },
  computed: {
    // 基础尺寸计算值
    radius() {
      return (this.size - this.strokeWidth) / 2;
    },
    circleOffset() {
      return this.size / 2;
    },
    circumference() {
      return 2 * this.radius * Math.PI;
    },
    //设置的倒计时时间
    hours() {
      return this.wheels.hoursWheel.shownDigits[4];
    },
    minutes() {
      return this.wheels.minutesWheel.shownDigits[4];
    },
    seconds() {
      return this.wheels.secondsWheel.shownDigits[4];
    },
    // 倒计时器进程计算值
    totalMilliSeconds() {
      return (this.hours * 3600 + this.minutes * 60 + this.seconds) * 1000;
    },
    progress() {
      return this.circumference * (1 - this.leftMilliSeconds / this.totalMilliSeconds);
    },
    leftHours() {
      return Math.floor(this.leftMilliSeconds / 3600000);
    },
    leftMinutes() {
      return Math.floor((this.leftMilliSeconds % 3600000) / 60000).toString().padStart(2, 0);
    },
    leftSeconds() {
      return Math.floor((this.leftMilliSeconds % 60000) / 1000).toString().padStart(2, 0);
    },
    // 倒计时面板数字显示控制计算值，根据小时设置，改变小时的可见行和整体字体大小
    hidden() {
      return this.leftHours === 0;
    },
    digitFontSize() {
      if (this.leftHours === 0) {
        return "80px";
      } else if (~~(this.leftHours / 10) === 0) {
        return "70px";
      } else {
        return "60px";
      }
    },
    alarmDayNight() {
      const hours = this.alarmTime.getHours();
      return hours >= 12 ? "下午" : "上午";
    },
    alarmHours() {
      const hours = this.alarmTime.getHours();
      return hours - (hours > 12 ? 12 : 0);
    },
    alarmMinutes() {
      const minutes = this.alarmTime.getMinutes();
      return minutes.toString().padStart(2, 0);
    },
    // 倒计时器控制计算值
    cancelBtnDisabled() {
      return this.timer === null;
    },
    controlBtnState() {
      if (this.timer === null) {
        return "stopped";
      } else {
        return this.paused ? "paused" : "running"
      }
    },
    controlBtnTag() {
      return this.btnTags.get(this.controlBtnState);
    },
    // 倒计时时间设置计算值
    hoursMoving() {
      const states = this.wheels.hoursWheel.isMoving;
      return states.before || states.after;
    },
    minutesMoving() {
      const states = this.wheels.minutesWheel.isMoving;
      return states.before || states.after;
    },
    secondsMoving() {
      const states = this.wheels.secondsWheel.isMoving;
      return states.before || states.after;
    },
  },
  methods: {
    // 左侧“取消”按钮点击处理器
    cancel() {
      clearInterval(this.timer);  // 清除计时器
      this.timer = null;  // 进入 stopped 状态
      this.paused = false;  // (保险) 退出 paused 状态
      this.leftMilliSeconds = this.totalMilliSeconds;  // (保险) 重置剩余时间
    },
    // 右侧控制按钮点击处理器
    control() {
      const deltaTime = 10;
      const timerOn = function () {
        this.alarmTime = new Date();  // 重新获取当前时间
        this.alarmTime.setTime(this.alarmTime.getTime() + this.leftMilliSeconds);  // 计算计时器触发时间点
        this.timer = setInterval(() => {  // 进入 running 状态
          this.leftMilliSeconds -= deltaTime;  // 更新剩余时间
          if (this.leftMilliSeconds <= 0) {
            clearInterval(this.timer);  // 清除计时器
            this.timer = null;  // 进入 stopped 状态
            this.triggered = true;  // 触发计时器 triggered 状态
            this.leftMilliSeconds = this.totalMilliSeconds;  // (保险) 重置剩余时间
          }
        }, deltaTime);
      }
      // 点击按钮开始计时
      if (this.controlBtnState === "stopped") {
        this.leftMilliSeconds = this.totalMilliSeconds;  // 设置剩余时间为设定值
        this.paused = false;  // (保险) 退出 paused 状态
        this.triggered = false;  // (保险) 解除计时器 triggered 状态
        timerOn.call(this);
      }
      // 点击按钮暂停计时
      else if (this.controlBtnState === "running") {
        this.paused = true;  // 进入 paused 状态
        clearInterval(this.timer);  // 清除计时器
      }
      // 点击按钮继续计时
      else {
        this.paused = false;  // 退出 paused 状态
        timerOn.call(this);
      }
    },
    // 点击计时器触发弹出消息以收回消息
    alarmOff() {
      this.triggered = false;
    },
  },
  mounted() {
    for (let wheel in this.wheels) {
      wheel = this.wheels[wheel]
      
      // 给倒计时时间设置轮盘控制器绑定鼠标滚轮事件
      this.$refs[wheel.name + 'Ctl'].addEventListener('wheel', event => {
        // 计时器开始计时或者轮盘正在转动时，不重复触发事件
        if (!this.timer && !this[this.name +'Moving']) {
          if (event.deltaY < 0) {
            wheel.isMoving.before = true;
          } else if (event.deltaY > 0) {
            wheel.isMoving.after = true;
          }
        }
      });

      // 给倒计时时间设置轮盘绑定过渡动画处理器
      this.$refs[wheel.name + 'Window'].addEventListener('transitionend', () => {
        if (wheel.isMoving.before) {
          wheel.shownDigits.pop();
          const first = wheel.shownDigits[0];
          wheel.shownDigits.unshift(first > 0 ? first - 1 : wheel.maxDigit);
          wheel.isMoving.before = false;
        }
        if (wheel.isMoving.after) {
          wheel.shownDigits.shift();
          const last = wheel.shownDigits[wheel.shownDigits.length - 1];
          wheel.shownDigits.push(last < wheel.maxDigit ? last + 1 : 0);
          wheel.isMoving.after = false;
        }
      });
    }
  },
}
</script>


<style lang="scss">
@use "sass:math";

@font-face {
  font-family: 'Uni Sans';
  src: url(../assets/fonts/uni-sans.thin-caps.otf);
}

$component-top: 30px;
$component-width: $component-top + 300px;

:root {
  font-family: Helvetica, sans-serif;
  font-size: 13px;  // 给rem单位设定基准
}

body {
  padding: 0;
  margin: 0;
  background-color: #000;
  min-width: $component-width;
  overflow: hidden;  // 阻止某些浏览器在滚动鼠标滚轮时尝试滚动页面
}

#timer {
  margin: 0 auto;
  padding-top: $component-top;
  width: $component-width;
  height: calc(100vh - #{$component-top});
  position: relative;
  color: #fff;

  #timer-dash {
    display: flex;
    justify-content: center;
  }

  #timer-buttons {
    display: flex;
    justify-content: space-between;
    position: relative;
    top: -30px;

    button {
      display: inline-block;
      position: relative;
      width: 70px;
      height: 70px;
      white-space: nowrap;  // 使得按钮内的文本不自动换行
      background-clip: padding-box;  // 背景色只沿伸到内边距框，从而使同色双线边框露出中间的透明
      border-radius: 50%;  // 按钮变成圆形
      border-style: double;
      border-width: 6px;
      cursor: pointer;
      font-size: 1rem;

      span {
        position: absolute;  // 绝对定位，隐试增加 display: inline-block 属性
        top: 50%;  // 将上边定位到容纳块（按钮）中间
        left: 50%;  // 将左边定位到容纳块（按钮）中间
        transform: translate(-50%, -50%);  // 将文本向上、左移动半个自身宽、长，继而将文本中点对齐按钮中点
      }

      @mixin buttonBgc($color) {
        background-color: $color;
        border-color: $color;
      }
      
      @mixin buttonColor($baseColor) {
        color: $baseColor;
        @include buttonBgc(rgba($baseColor, .15));

        // 鼠标事件，按钮文本不变色，按钮背景变色
        &:hover {
          @include buttonBgc(rgba($baseColor, .18));
        }

        &:active {
          @include buttonBgc(rgba($baseColor, 0.1));
        }

        // 按钮禁用，其文本变色，背景不变色
        &:disabled {
          color: rgba($baseColor, .3);
        }
      }

      // 给计时控制按钮传递不同控制状态的基色
      &.stopped, &.paused {
        @include buttonColor(#40cc64);
      }

      &.running {
        @include buttonColor(#ffa000);
      }

      // 给计时取消按钮传递基色
      &:first-child {
        @include buttonColor(#fff);
      }
    }
  }

  #timer-message {
    position: absolute;
    margin: 5px;
    padding: 10px;
    height: 40px;
    top: 0;
    left: 0;
    right: 0;
    border-radius: 10px;
    background-color: rgba(200, 200, 200, .95);
    transform-origin: 50% 0;
    transform: translateY(-70px);
    transition: transform .2s ease;  // 始态设置过渡，双向都有过渡
    cursor: pointer;

    .timer-msg-head {
      display: flex;
      justify-content: space-between;
      margin-bottom: 3px;
      color: #666;
      font-size: 0.8rem;
      line-height: 20px;
      
      .app-info {
        display: inline-block;

        img {
          margin-right: .5rem;
          height: 20px;
          vertical-align: -0.5rem;
        }
      }

      .trigger-time {
        .digits {
          font-family: Helvetica, sans-serif;
        }
        
        .colon {
          font-family: Times, serif;
          vertical-align: .1rem;
        }

        span:first-child {
          margin-right: .3rem;
        }
      }
    }

    .timer-msg-body {
      color: #000;
      font-size: 0.85em;
    }

    &.triggered {
      transform: translateY(0);
    }
  }
}

#count-down {
  display: inline-block;
  position: relative;

  span {
    font-weight: 200;

    .colon {
      vertical-align: .1em;
    }
  }

  #timer-ring {
    // 将计时器圆环逆时针旋转90度，使得起始终止位置为12点方向，符合表盘逻辑
    transform: rotate(-90deg);
    transform-origin: 50% 50%;
  }

  #left-time {
    position: absolute;
    top: 45%;
    left: 50%;
    transform: translate(-50%, -50%);

    .digits {
      font-family: 'Helvetica Neue', Helvetica, sans-serif;
    }

    .colon {
      font-family: "Uni Sans", sans-serif;
    }

    .hidden {
      display: none;  // 要使小时数字不显示且不占据空间，需要取消其渲染
    }
  }

  #alarm-time {
    position: absolute;
    top: 65%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 1.2rem;
    color: #909090;
    // 将 Emoji 改为纯色图标，但是 FontAwesome 不需要
    // color: transparent;
    // text-shadow: 0 0 0 #909090;

    .digits {
      font-family: Helvetica, sans-serif;
    }
    
    .colon {
      font-family: Times, serif;
    }

    span:nth-child(2) {
      font-weight: 400;
      margin: 0 .4rem;
    }

    &.paused {
      color: #282828;
      // text-shadow: 0 0 0 #282828;
    }
  }
}

#time-setting {
  position: absolute;
  left: 0;
  right: 0;
  height: 300px;
  background: #000;
  opacity: 1;
  transition: opacity .2s ease-in;

  $wheel-height: 13rem;
  $digits-width: 2rem;
  
  #wheels-container {
    position: absolute;
    left: 45.2%;  // 将中间轮盘对准选中窗口的中间位
    top: 45%;
    transform: translate(-50%, -50%);
    box-sizing: border-box;
    height: $wheel-height;
    width: 14rem;
    display: flex;
    justify-content: space-between;
    align-items: center;

    .wheel-part {
      height: 2rem;
      width: $digits-width;
      color: #909090;
      font-family: 'Helvetica Neue', Helvetica, sans-serif;
      font-size: 1.2rem;
      font-weight: 300;
      line-height: 2rem;
      text-align: right;
      position: relative;  // 作为轮盘上各个数码的容纳块
      
      .wheel-digits {
        position: absolute;  // 各个数码绝对定位至中间，方便变形
        left: 0;
        right: 0;
      }

      @function computeTransform($normIdx, $base-x, $delta-x, $base-y, $delta-y, $base-s, $delta-s) {
        // 计算数码的累加变形值
        @if $normIdx == 0 {
          // 标准化索引为0，即正中间数码，不需要变形
          @return translate(0, 0) scale(1);
        } @else {
          // 标准化索引不为0，需要变形产生轮盘效果
          $symbol-y: 0 - math.div($normIdx, math.abs($normIdx));  // 取出 y 轴平移正负号，中间以上数码，标准化索引为正，但是上移正负号为负号
          $normIdx: math.abs($normIdx) - 1;  // 标准化索引减1，为变形指数级累加次数
          $computed-x: $base-x;
          $computed-y: $base-y * $symbol-y; // y 轴累加值每一项都得乘正负号
          $computed-s: $base-s;
          $value: null;
          @for $j from 0 through $normIdx {
            @if $j > 0 {
              $times: math.pow(2, ($j - 1));  // 累加 delta 的乘数以指数级增长
              $computed-x: $computed-x + ($delta-x * $times);
              $computed-y: $computed-y + ($delta-y * $times * $symbol-y);
              $computed-s: $computed-s + ($delta-s * $times);
            }
            $value: $value + translate(#{$computed-x}rem, #{$computed-y}rem) scaleY(#{$computed-s});  // 累加变形函数
          }
          @return $value;
        }
      }

      @mixin makeDigit($visiable, $animated) {
        // 数码过渡动画的设置
        opacity: if($visiable, 1, 0);
        @if $animated {
          transition:  
            transform .2s ease-in,
            opacity .2s ease-in;
        }
      }

      @mixin makeWheel($base-x, $delta-x, $base-y, $delta-y, $base-s, $delta-s) {
        @for $i from 1 through 9 {
          .wheel-digits:nth-child(#{$i}) {
            // 原始状态，标准化索引 0 位放在轮盘中间
            transform: computeTransform((5 - $i), $base-x, $delta-x, $base-y, $delta-y, $base-s, $delta-s);
            // 原始状态各数码都可见，且不设置过渡，这样过渡单向，恢复的时候没有过渡
            @include makeDigit(true, false);
          }

          .wheel-digits.moved-before:nth-child(#{$i}) {
            // 轮盘向下拨动，所有数码标准化索引减1，向下移到下一个位置
            transform: computeTransform((4 - $i), $base-x, $delta-x, $base-y, $delta-y, $base-s, $delta-s);
            @if $i == 9 {
              // 最下边的数码改为透明
              @include makeDigit(false, true);
            } @else {
              @include makeDigit(true, true);
            }
          }

          .wheel-digits.moved-after:nth-child(#{$i}) {
            // 轮盘向上拨动，所有数码标准化索引加1，向上移动到下一个位置
            transform: computeTransform((6 - $i), $base-x, $delta-x, $base-y, $delta-y, $base-s, $delta-s);
            @if $i == 1 {
              // 最上边的数码改为透明
              @include makeDigit(false, true);
            } @else {
              @include makeDigit(true, true);
            }
          }
        }
      }

      &#hours-digits-container {
        // 左侧轮盘，x 轴正值使上下两端向右侧收
        @include makeWheel(.01, .02, 1.8, 0.1, .9, -.1);
      }

      &#minutes-digits-container {
        // 中间轮盘，x 轴不做变形
        @include makeWheel(0, 0, 1.8, 0.1, .9, -.1);
      }

      &#seconds-digits-container {
        // 右侧轮盘，x 轴负值使上下两端向左侧收
        @include makeWheel(-.01, -.02, 1.8, 0.1, .9, -.1);
      }
    }

    &::after {
      // 空的 ::after 伪元素给轮盘上下增加渐变遮盖
      content: "";  // 想要让伪元素渲染，必须给内容，即使空内容也要显式声明
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: linear-gradient(180deg, rgba(0, 0, 0, .5) 0%, rgba(0, 0, 0, 0) 40%, rgba(0, 0, 0, 0) 60%, rgba(0, 0, 0, .5) 100%);
    }
  }

  #wheel-window {
    position: absolute;
    left: 50%;
    top: 45%;
    transform: translate(-50%, -50%);
    box-sizing: border-box;
    height: 2rem;
    width: $component-width - ($component-top * 3);
    padding-left: .4rem;
    padding-right: 1.6rem;
    background-color: #333;
    border-radius: .4rem;
    display: flex;
    justify-content: space-between;
    font-size: 1rem;
    line-height: 2rem;
    overflow: hidden;

    .window-part {
      display: flex;

      .part-digits-container {
        width: $digits-width;  // 固定宽度，防止数码位数改变导致 flex 排布改变
        margin-right: .3rem;
        text-align: right;
        font-family: 'Helvetica Neue', Helvetica, sans-serif;
        font-size: 1.2rem;
        font-weight: 300;

        .part-digits{
          transform: translateY(-2rem);  // 行高 2rem，算上 ::before、::after 伪元素，变形以居中本体

          &.moved-before {
            // 向下拨动轮盘，显示 ::before 伪元素
            transform: translateY(0);
            transition: transform .2s ease-in;
          }

          &.moved-after {
            // 向上拨动轮盘，显示 ::after 伪元素
            transform: translateY(-4rem);
            transition: transform .2s ease-in;
          }

          &::before {
            content: attr(data-before);  // 使用在 html 中的 data-before 属性动态传入的值作为内容
            display: block;  // 块级使其与本体竖向排列
          }

          &::after {
            content: attr(data-after);  // 使用在 html 中的 data-after 属性动态传入的值作为内容
            display: block;  // 块级使其与本体竖向排列
          }
        }
      }

      .part-label span {
        vertical-align: -.1rem;
      }
    }
  }

  #controlers-container {
    position: absolute;
    left: 50%;
    top: 45%;
    transform: translate(-50%, -50%);
    box-sizing: border-box;
    height: $wheel-height;
    width: $component-width - ($component-top * 3);
    display: flex;

    .controler {
      flex: 1 1 auto;  // 各轮盘的滚轮控制器均匀铺满整个 flex 容器，横向各占 1/3 容器宽度
    }
  }

  &.hidden {
    opacity: 0;  // 开始倒计时后，时间选择轮盘变为不可见，从而显示倒计时面板
  }
}
</style>
