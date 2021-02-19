<template>
  <div>
    <ul class="calendar">
      <template v-for="(weekday, key) in weekdays">
        <li class="weekday" :key="'weekday' + key">
          <span>{{ weekday }}</span>
        </li>
      </template>
      <template v-for="(day, key) in startWeekday">
        <li class="day" :key="'null' + key">
          <!-- <span class="nullBlock"></span> -->
        </li>
      </template>
      <template v-for="(day, key) in daysCount">
        <li class="day" :key="'day' + key">
          <span
            v-if="!singleDate"
            :class="getDayStyle(day)"
            @click="updateSelectingDay(day)"
          >
            {{ day }}
          </span>

          <span
            v-if="singleDate"
            :class="getDayStyle(day)"
            @click="updateSelectingSingleDay(day)"
          >
            {{ day }}
          </span>
        </li>
      </template>
    </ul>
  </div>
</template>

<script>
import utils from "../../lib/date";
import {addZeroToString} from "../../lib/common"

const isToday = otherDay => {
  const today = new Date();
  return utils.isSameDay(today, otherDay);
};

const isBetweenDays = (smallDay, bigDay, currentDay) => {
  if (currentDay < bigDay && smallDay < currentDay) return true;
  return false;
};

export default {
  name: "Calender",
  props: {
    year: Number,
    month: Number,
    startDate: Date,
    endDate: Date,
    onChange: Function,
    singleDate: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    startWeekday: function() {
      const {currentYYMMString} = this
      return utils.getWeekday(
        new Date(`${currentYYMMString}-01`).getTime()
      );
    },

    daysCount: function() {
      return utils.daysInMonth(this.year, this.month);
    },

    currentYYMMString: function() {
      const _m = addZeroToString(`${this.month + 1}`)
      return `${this.year}-${_m}`
    }
  },
  methods: {
    callOnChange: function(returnData) {
      if (this.$listeners.onChange) {
        return this.$emit("onChange", { ...returnData });
      }

      if (this.onChange) {
        return this.onChange({ ...returnData });
      }
    },
    updateSelectingSingleDay: function(day) {
      if (!day) return;

      const { currentYYMMString, innerStartDate, innerEndDate } = this;
      const currentDay = new Date(`${currentYYMMString}-${addZeroToString(day)}`);

      this.innerStartDate = currentDay;
      this.innerEndDate = currentDay;
      this.selectedDay = day;

      return this.callOnChange({
        selectedDay: currentDay,
        startDate: this.innerStartDate
      });
    },
    updateSelectingDay: function(day) {
      if (!day) return;

      const {
        currentYYMMString,
        innerStartDate,
        innerEndDate,
        isSelectingStartDay
      } = this;

      const currentDay = new Date(`${currentYYMMString}-${addZeroToString(day)}`);

      // reset
      if (
        isSelectingStartDay ||
        (!isSelectingStartDay && currentDay < innerStartDate)
      ) {
        this.innerStartDate = currentDay;
        this.isSelectingStartDay = false;
      } else {
        this.isSelectingStartDay = true;
      }

      this.innerEndDate = currentDay;
      this.selectedDay = day;

      return this.callOnChange({
        selectedDay: currentDay,
        startDate: this.innerStartDate,
        endDate: this.innerEndDate
      });
    },

    getDayStyle: function(day) {
      const { innerStartDate, innerEndDate, currentYYMMString } = this;
      const currentDay = new Date(`${currentYYMMString}-${addZeroToString(day)}`);

      if (utils.isSameDay(currentDay, innerStartDate)) return "innerStartDate";
      if (utils.isSameDay(currentDay, innerEndDate)) return "innerEndDate";
      if (isBetweenDays(innerStartDate, innerEndDate, currentDay))
        return "between";
      if (isToday(currentDay)) return "today";

      return "";
    }
  },
  data() {
    const { startDate, endDate, singleDate } = this;
    return {
      selectedDay: null,
      isSelectingStartDay: true, // either startDay or endDay
      weekdays: utils.weekDayShortConfig,
      innerStartDate: startDate,
      innerEndDate: singleDate ? startDate : endDate
    };
  }
};
</script>

<style lang="scss" scoped>
@import "../../style/main.scss";

ul.calendar {
  width: 364px;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  background: #fff;

  li {
    display: inline-block;
    width: 52px;
  }
  li.weekday {
    font-size: 14px;
    color: $silver;
    font-weight: 600;
    margin-bottom: 8px;
    text-align: center;
  }
  li.day {
    span {
      width: 100%;
      height: 40px;
      display: inline-block;
      text-align: center;
      line-height: 40px;
      font-size: 15px;
      font-weight: 600;
      color: $slate-grey;
      background: #fff;

      &:hover {
        cursor: pointer;
        color: #fff;
        background: $secondary-01;
        transition-duration: 0.3s;
      }
      &.today {
        box-shadow: inset 0 0 0 2px $secondary-01;
      }
      &.innerStartDate {
        background: $secondary-01;
        color: #fff;
      }
      &.innerEndDate {
        background: $secondary-01;
        color: #fff;
      }
      &.between {
        background: #eaf0fd;
      }
    }
  }
}
@media only screen and (max-width: 700px) {
  ul.calendar {
    width: 100%;
    li {
      width: calc(100% / 7);
    }
  }
}
</style>
