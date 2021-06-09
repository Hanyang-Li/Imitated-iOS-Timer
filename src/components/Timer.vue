<template>
<div id="timer">
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
  </div>
  
  <!-- 控制按钮 -->
  <div id="timer-buttons">
    <button @click="cancel" :disabled="cancelBtnDisabled">
      <span>取消</span>
    </button>
    <button @click="control" :class="[controlBtnState]">
      <span>{{ controlBtnTag }}</span>
    </button>
  </div>
  
  <!-- 计时触发消息 -->
  <div id="timer-message" :class="{triggered}" @click="alarmOff">
    <div class="timer-msg-head">
      <div class="app-info">
        <img src="../assets/images/clock-icon.png" alt="icon">
        <span>时钟</span>
      </div>
      <div class="trigger-time">
        <span>{{ alarmDayNight }}</span>
        <span class="digits">{{ alarmHours }}</span>
        <span class="colon">:</span>
        <span class="digits">{{ alarmMinutes }}</span>
      </div>
    </div>
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
      // 设置的倒计时时间
      hours: 0,
      minutes: 0,
      seconds: 5,
      // 倒计时器进程
      timer: null,
      leftMilliSeconds: 0,
      alarmTime: new Date(),
      // 倒计时器控制
      paused: false,
      btnTags: new Map([["stopped", "开始计时"], ["running", "暂停"], ["paused", "继续"]]),
      // 计时器触发
      triggered: false,
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
  },
  methods: {
    cancel() {
      clearInterval(this.timer);  // 清除计时器
      this.timer = null;  // 进入 stopped 状态
      this.paused = false;  // (保险) 退出 paused 状态
      this.leftMilliSeconds = this.totalMilliSeconds;  // (保险) 重置剩余时间
    },
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
    alarmOff() {
      this.triggered = false;
    }
  }
}
</script>


<style lang="scss">
@font-face {
  font-family: 'Uni Sans';
  src: url(../assets/fonts/uni-sans.thin-caps.otf);
}

$component-top: 30px;
$component-width: $component-top + 300px;

:root {
  font-family: Helvetica, sans-serif;
  font-size: 13px;
}

body {
  padding: 0;
  margin: 0;
  background-color: #000;
  min-width: $component-width;
}

#timer {
  outline: 1px solid palevioletred;
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
      white-space: nowrap;
      background-clip: padding-box;
      border-radius: 50%;
      border-style: double;
      border-width: 6px;
      cursor: pointer;
      font-size: 1rem;

      span {
        display: block;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }

      @mixin buttonBgc($color) {
        background-color: $color;
        border-color: $color;
      }
      
      @mixin buttonColor($baseColor) {
        color: $baseColor;
        @include buttonBgc(rgba($baseColor, .15));

        &:hover {
          @include buttonBgc(rgba($baseColor, .18));
        }

        &:active {
          @include buttonBgc(rgba($baseColor, 0.1));
        }

        &:disabled {
          color: rgba($baseColor, .3);
        }
      }

      &.stopped, &.paused {
        @include buttonColor(#40cc64);
      }

      &.running {
        @include buttonColor(#ffa000);
      }

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
    transition: transform .2s ease;
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
    transform: rotate(-90deg);
    transform-origin: 50% 50%;
  }

  #left-time {
    position: absolute;
    top: 45%;
    left: 50%;
    transform: translate(-50%, -50%);

    .digits {
      font-family: 'HelveticaNeue-UltraLight', 'Helvetica Neue UltraLight', 'Helvetica Neue', Helvetica, sans-serif;
    }

    .colon {
      font-family: "Uni Sans", sans-serif;
    }

    .hidden {
      display: none;
    }
  }

  #alarm-time {
    position: absolute;
    top: 65%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: #909090;
    font-size: 1.2rem;

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
    }
  }
}
</style>
