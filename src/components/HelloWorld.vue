<template>
  <div id="app" class="container">
    <div class="clock">
      <h2>地球时间: {{ formatTime(earthTime) }}</h2>
      <h2>外星时间: {{ formatAlienTime(alienTime) }}</h2>
    </div>

    <div class="input-group">
      <label for="alienTime">设置外星时间 (YYYY-MM-DD HH:MM:SS):</label>
      <input id="alienTime" type="text" v-model="alienTimeInput" class="input-field">
      <button @click="setAlienTime" class="btn">设置</button>
    </div>

    <div class="input-group">
      <label for="alarm">设置闹钟 (HH:MM:SS):</label>
      <input id="alarm" type="text" v-model="alarmInput" class="input-field">
      <button @click="addAlarm" class="btn">添加</button>
    </div>

    <div class="alarm-list">
      <h3>闹钟列表</h3>
      <div v-for="(alarm, index) in alarms" :key="index" class="alarm-item">
        {{ formatAlarmTime(alarm) }}
        <span class="delete-alarm" @click="deleteAlarm(index)">删除</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  el: '#app',
  data() {
    return {
      alienTime: {
        year: 2804,
        month: 18,
        day: 31,
        hour: 2,
        minute: 2,
        second: 88
      },
      earthTime: new Date(),
      alienTimeInput: '',
      alarmInput: '',
      alarms: []
    }
  },
  methods: {
    updateEarthTime() {
      this.earthTime = new Date();
      this.updateAlienTimeFromEarthTime();
      this.checkAlarms();
    },

    alienSecondsToAlienTime(secondsToAdd) {
      const initialDateStr = "2804-18-31 02:02:88";
      // 定义时间单位
      const secondsPerMinute = 90;
      const minutesPerHour = 90;
      const hoursPerDay = 36;
      const monthsPerYear = 18;
      const daysInMonths = [44, 42, 48, 40, 48, 44, 40, 44, 42, 40, 40, 42, 44, 48, 42, 40, 44, 38];

      // 将初始日期拆分为年、月、日、小时、分钟和秒
      let [year, month, day, hour, minute, second] = initialDateStr.split(/[- :]/).map(Number);

      // 增加秒数
      second += secondsToAdd;

      // 处理秒进位到分钟
      minute += Math.floor(second / secondsPerMinute);
      second %= secondsPerMinute;
      second = Math.floor(second);

      // 处理分钟进位到小时
      hour += Math.floor(minute / minutesPerHour);
      minute %= minutesPerHour;

      // 处理小时进位到天
      day += Math.floor(hour / hoursPerDay);
      hour %= hoursPerDay;

      // 处理天进位到月份
      while (day > daysInMonths[month - 1]) {
        day -= daysInMonths[month - 1];
        month++;
        if (month > monthsPerYear) {
          month = 1;
          year++;
        }
      }

      this.alienTime.second = second;
      this.alienTime.minute = minute;
      this.alienTime.hour = hour;
      this.alienTime.day = day;
      this.alienTime.month = month;
      this.alienTime.year = year;
    },

    updateAlienTimeFromEarthTime() {
      const earthSeconds = (this.earthTime - new Date(1970, 0, 1, 0, 0, 0)) / 1000;
      const alienSeconds = earthSeconds * 2; // 1外星秒 = 0.5地球秒

      this.alienSecondsToAlienTime(alienSeconds);
    },
    formatTime(time) {
      return `${time.getFullYear()}-${String(time.getMonth() + 1).padStart(2, '0')}-${String(time.getDate()).padStart(2, '0')} ${String(time.getHours()).padStart(2, '0')}:${String(time.getMinutes()).padStart(2, '0')}:${String(time.getSeconds()).padStart(2, '0')}`;
    },
    formatAlienTime(alienTime) {
      return `${alienTime.year}-${String(alienTime.month).padStart(2, '0')}-${String(alienTime.day).padStart(2, '0')} ${String(alienTime.hour).padStart(2, '0')}:${String(alienTime.minute).padStart(2, '0')}:${String(alienTime.second).padStart(2, '0')}`;
    },
    setAlienTime() {
      const [year, month, day, hour, minute, second] = this.alienTimeInput.split(/[- :]/).map(Number);
      if (year && month && day && hour && minute && second &&
        month >= 1 && month <= 18 &&
        day >= 1 && day <= alienMonthDays[month - 1] &&
        hour >= 0 && hour < 36 &&
        minute >= 0 && minute < 90 &&
        second >= 0 && second < 90) {
        this.alienTime = { year, month, day, hour, minute, second };
        this.updateEarthTimeFromAlienTime();
        alert(`对应地球时间: ${this.formatTime(this.earthTime)}`);
      } else {
        alert('无效的外星时间格式。请使用YYYY-MM-DD HH:MM:SS，并确保时间符合外星时间规则。');
      }
    },
    updateEarthTimeFromAlienTime() {
      const daysInMonths = [44, 42, 48, 40, 48, 44, 40, 44, 42, 40, 40, 42, 44, 48, 42, 40, 44, 38];
      let sum = 0;
      for (let i = 0; i <= this.alienTime.month - 2; i++) {
        sum += daysInMonths[i];
      }

      //2804-18-31 02:02:88
      const totalDaysInYear = this.getTotalDaysInYear();
      const alienSeconds = this.alienTime.second +
        this.alienTime.minute * 90 +
        this.alienTime.hour * 90 * 90 +
        (this.alienTime.day - 1) * 90 * 90 * 36 +
        sum * 90 * 90 * 36 +
        (this.alienTime.year * totalDaysInYear) * 90 * 90 * 36 -629809943668;


      console.log(totalDaysInYear)
      const earthSeconds = alienSeconds / 2;
      console.log(alienSeconds)
      this.earthTime = new Date(1970, 0, 1, 0, 0, 0);
      this.earthTime.setSeconds(this.earthTime.getSeconds() + earthSeconds);
    },
    addAlarm() {
      const [hours, minutes, seconds] = this.alarmInput.split(':').map(Number);
      if (hours >= 0 && hours < 24 && minutes >= 0 && minutes < 60 && seconds >= 0 && seconds < 60) {
        this.alarms.push({ hours, minutes, seconds });
      } else {
        alert('无效的闹钟时间格式。请使用HH:MM:SS。');
      }
    },
    checkAlarms() {
      const currentTime = this.earthTime.getHours() * 3600 + this.earthTime.getMinutes() * 60 + this.earthTime.getSeconds();
      this.alarms = this.alarms.filter(alarm => {
        const alarmTime = alarm.hours * 3600 + alarm.minutes * 60 + alarm.seconds;
        if (alarmTime === currentTime) {
          alert('闹钟响了!');
          return false;
        }
        return true;
      });
    },
    deleteAlarm(index) {
      this.alarms.splice(index, 1);
    },
    formatAlarmTime(alarm) {
      return `${String(alarm.hours).padStart(2, '0')}:${String(alarm.minutes).padStart(2, '0')}:${String(alarm.seconds).padStart(2, '0')}`;
    },
    getTotalDaysInYear() {
      return alienMonthDays.reduce((acc, days) => acc + days, 0);
    }
  },
  mounted() {
    setInterval(() => {
      this.updateEarthTime();
    }, 1000 * alienTimeRatio);
  }
}

const alienMonthDays = [44, 42, 48, 40, 48, 44, 40, 44, 42, 40, 40, 42, 44, 48, 42, 40, 44, 38];
const alienTimeRatio = 0.5; // 1外星秒 = 0.5地球秒
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  font-family: 'Roboto', sans-serif;
  background-color: #f8f9fa;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.clock {
  text-align: center;
  margin-bottom: 30px;
}

h2 {
  font-size: 1.5rem;
  margin: 10px 0;
  color: #343a40;
}

.input-group {
  margin: 20px 0;
  text-align: center;
}

.input-group label {
  display: block;
  font-size: 1rem;
  margin-bottom: 5px;
  color: #495057;
}

.input-with-button {
  display: flex;
  align-items: center;
}

.input-field {
  flex-grow: 1;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ced4da;
  border-radius: 5px 0 0 5px;
  margin-right: -1px;
}

.btn {
  padding: 10px 15px;
  font-size: 1rem;
  color: #ffffff;
  background-color: #007bff;
  border: none;
  border-radius: 0 5px 5px 0;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.btn:hover {
  background-color: #0056b3;
}

.alarm-list {
  margin-top: 30px;
}

.alarm-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-align: center;
  padding: 10px;
  background-color: #e9ecef;
  border-radius: 5px;
  margin-bottom: 10px;
  margin-left: 30%;
  margin-right: 30%;
}

.delete-alarm {
  cursor: pointer;
  color: #dc3545;
  font-weight: bold;
  transition: color 0.3s ease;
}

.delete-alarm:hover {
  color: #c82333;
}
</style>
