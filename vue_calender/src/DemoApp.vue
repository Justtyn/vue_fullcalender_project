<script>
import { ref } from "vue";
import axios from "axios";
import { defineComponent } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import { INITIAL_EVENTS, createEventId } from "./event-utils";
//当前本地json配置
import scheduleData from "../src/output.json"; // 根据您的文件路径调整
import listPlugin from "@fullcalendar/list";
import { Calendar } from "@fullcalendar/core";
import allLocales from "@fullcalendar/core/locales-all";

export default defineComponent({
  components: {
    FullCalendar,
  },
  data() {
    return {
      calendarOptions: {
        plugins: [
          listPlugin,
          dayGridPlugin,
          timeGridPlugin,
          interactionPlugin, // needed for dateClick
        ],
        firstDay: 1,
        headerToolbar: {
          left: "",
          center: "title",
          right: "",
        },
        footerToolbar: {
          left: "today",
          center: "prev,next",
          right: "listWeek,timeGridWeek,timeGridDay",
        },
        initialView: "timeGridWeek",
        initialEvents: this.parseScheduleData(), // alternatively, use the `events` setting to fetch from a feed
        editable: true,
        selectable: true,
        selectMirror: true,
        dayMaxEvents: true,
        weekends: true,
        select: this.handleDateSelect,
        // eventClick: this.handleEventClick,
        // eventsSet: this.handleEvents,
        nowIndicator: true,
        locales: allLocales,
        locale: "zh-CN",
        slotMinTime: "08:00:00",
        slotMaxTime: "22:00:00",
        height: "95%",
        handleWindowResize: true,
        allDaySlot: false,
        expandRows: true,
        eventMinHeight: 30,
        weekNumbers: true,
        weekNumberCalculation: "ISO",
        selectable: false,
        selectInfo: false,
        stickyFooterScrollbar: false,
        // aspectRatio:2,
        /* you can update a remote database when these fire:
        eventAdd:
        eventChange:
        eventRemove:
        */
      },
      currentEvents: [],
    };
  },
  methods: {
    parseScheduleData() {
      const events = [];

      //定义固定的颜色映射
      const courseColors = {
        // 数据库系统原理: "#FF9800",
        // Web前端开发技术: "#2196F3",
        // Java程序设计: "#4CAF50",
        // 计算机网络: "#9C27B0",
        // 软件工程导论: "#E91E63",
        // Linux操作系统应用: "#795548",
        // 大学英语: "#607D8B",
        // 中国近现代史纲要: "#FF5722",
        // 形势与政策: "#009688",
      };

      scheduleData.forEach((weekData) => {
        Object.entries(weekData.per_day_lessons).forEach(([date, lessons]) => {
          lessons.forEach((lesson) => {
            const startDateTime = `${date}T${lesson.start_time}:00`;
            const endDateTime = `${date}T${lesson.end_time}:00`;

            // 找出课程的基础名称（不包含班级等信息）
            let courseBase = "";
            Object.keys(courseColors).forEach((key) => {
              if (lesson.lesson_name.includes(key)) {
                courseBase = key;
              }
            });

            events.push({
              id: createEventId(),
              title: `${lesson.lesson_name}\n${lesson.classroom}\n${lesson.teacher_name}`,
              start: startDateTime,
              end: endDateTime,
              backgroundColor: courseColors[courseBase] || "#3788d8",
              extendedProps: {
                teacher: lesson.teacher_name,
                classroom: lesson.classroom,
              },
            });
          });
        });
      });

      return events;
    },
    handleWeekendsToggle() {
      this.calendarOptions.weekends = !this.calendarOptions.weekends; // update a property
    },
    handleDateSelect(selectInfo) {
      let title = prompt("Please enter a new title for your event");
      let calendarApi = selectInfo.view.calendar;

      calendarApi.unselect(); // clear date selection

      if (title) {
        calendarApi.addEvent({
          id: createEventId(),
          title,
          start: selectInfo.startStr,
          end: selectInfo.endStr,
          allDay: selectInfo.allDay,
        });
      }
    },
    handleEventClick(clickInfo) {
      if (
        confirm(
          `Are you sure you want to delete the event '${clickInfo.event.title}'`
        )
      ) {
        clickInfo.event.remove();
      }
    },
    handleEvents(events) {
      this.currentEvents = events;
    },
  },
  // 更新按钮方法
  name: "UpdateButton",
  setup() {
    const storedData = ref(localStorage.getItem("data") || "暂无数据"); // 初始化从localStorage读取的数据

    const updateData = async () => {
      try {
        // 发送API请求
        const response = await axios.get("https://api.example.com/data");

        // 假设API返回的数据在response.data中
        const newData = response.data;

        // 更新localStorage
        localStorage.setItem("data", newData);

        // 更新页面显示
        storedData.value = newData;

        alert("数据已更新！");
      } catch (error) {
        console.error("请求数据失败:", error);
        alert("更新失败，请稍后再试。");
      }
    };

    return {
      storedData,
      updateData,
    };
  },
});
</script>

<template>
  <div class="demo-app">
    <div class="demo-app-main">
      <FullCalendar class="demo-app-calendar" :options="calendarOptions">
        <template v-slot:eventContent="arg">
          <b>{{ arg.timeText }}</b>
          <i>{{ arg.event.title }}</i>
        </template>
      </FullCalendar>

      <div class="demo-app-sidebar-section">
        <label>
          <input
            type="checkbox"
            :checked="calendarOptions.weekends"
            @change="handleWeekendsToggle"
          />
          显示周末
        </label>

        <button @click="updateData">更新数据</button>
      </div>
    </div>
  </div>
</template>

<style lang="css">
h2 {
  margin: 0;
  font-size: 16px;
}

ul {
  margin: 0;
  padding: 0 0 0 1.5em;
}

li {
  margin: 1.5em 0;
  padding: 0;
}

b {
  /* used for event dates/times */
  margin-right: 3px;
}

.demo-app {
  overflow: hidden;
  display: flex;
  min-height: 100%;
  font-family: Arial, Helvetica Neue, Helvetica, sans-serif;
  font-size: 14px;
}

.demo-app-sidebar {
  width: 300px;
  line-height: 1.5;
  background: #eaf9ff;
  border-right: 1px solid #d3e2e8;
}

.demo-app-sidebar-section {
  padding: 2em;
}

.demo-app-main {
  flex-grow: 1;
  padding: 3em;
}

.fc {
  /* the calendar root */
  max-width: 1100px;
  margin: 0 auto;
}

button {
  padding: 10px 20px;
  font-size: 12px;
  cursor: pointer;
  background-color: #000000;
  color: white;
  border: none;
  border-radius: 5px;
}

button:hover {
  background-color: #1e5eff;
}
</style>
