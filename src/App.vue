<script setup>
import DatePicker from 'primevue/datepicker';
import '@vuepic/vue-datepicker/dist/main.css'
import { ref } from 'vue';
import * as ics from 'ics';
import { Toaster, toast } from 'vue-sonner';


const startDate = ref();
const endDate = ref();

const fileName = ref("上傳課表網頁檔案（html檔）");
const curriculumFile = ref();

const scheduleTime = [
  { startTime: "08:10", endTime: "09:00" },
  { startTime: "09:10", endTime: "10:00" },
  { startTime: "10:20", endTime: "11:10" },
  { startTime: "11:20", endTime: "12:10" },
  { startTime: "12:20", endTime: "13:10" },
  { startTime: "13:20", endTime: "14:10" },
  { startTime: "14:20", endTime: "15:10" },
  { startTime: "15:30", endTime: "16:20" },
  { startTime: "16:30", endTime: "17:20" },
  { startTime: "17:30", endTime: "18:20" },
  { startTime: "18:25", endTime: "19:15" },
  { startTime: "19:20", endTime: "20:10" },
  { startTime: "20:15", endTime: "21:05" },
  { startTime: "21:10", endTime: "22:00" },
];

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file) {
    fileName.value = "檔案上傳成功！";
  } else {
    fileName.value = "上傳失敗，請重試。";
  }
}

function parseCourses(htmlContent) {
  const parser = new DOMParser();
  const doc = parser.parseFromString(htmlContent, "text/html");

  let courses = [];
  const courseRows = doc.querySelectorAll("table:nth-child(2) tbody tr");

  courseRows.forEach((row, index) => {
    if (index > 0) {
      const cells = row.querySelectorAll("td");
      if (cells.length > 5) {
        const courseCode = cells[0].textContent.trim();
        const courseName = cells[1].textContent.trim();
        const courseCredits = cells[2].textContent.trim();
        const courseType = cells[3].textContent.trim();
        const courseTeacher = cells[4].textContent.trim();

        courses.push({
          courseCode,
          courseName,
          courseCredits,
          courseType,
          courseTeacher,
        });
      }
    }
  });
  return courses;
}

function parseClassRoom(htmlContent) {
  const parser = new DOMParser();
  const doc = parser.parseFromString(htmlContent, "text/html");

  let classRooms = {
    星期一: [],
    星期二: [],
    星期三: [],
    星期四: [],
    星期五: [],
    星期六: [],
    星期日: [],
  };
  const curriculumRows = doc.querySelectorAll("table:nth-child(5) tbody tr");

  curriculumRows.forEach((row, index) => {
    if (index > 0) {
      const cells = row.querySelectorAll("td");
      if (cells.length > 5) {
        const days = [
          "星期一",
          "星期二",
          "星期三",
          "星期四",
          "星期五",
          "星期六",
          "星期日",
        ];
        for (let i = 2; i < 9; i++) {
          let courseClassRoom = "";
          if (
            cells[i].textContent.trim().length != 0 &&
            cells[i].textContent.trim().includes("\n")
          ) {
            courseClassRoom = cells[i].textContent.trim().split("\n")[1].trim();
          }

          classRooms[days[i - 2]].push(courseClassRoom);
        }
      }
    }
  });
  return classRooms;
}

function parseCurriculum(htmlContent) {
  const parser = new DOMParser();
  const doc = parser.parseFromString(htmlContent, "text/html");

  let curriculum = {
    星期一: [],
    星期二: [],
    星期三: [],
    星期四: [],
    星期五: [],
    星期六: [],
    星期日: [],
  };
  const curriculumRows = doc.querySelectorAll("table:nth-child(5) tbody tr");
  curriculumRows.forEach((row, index) => {
    if (index > 0) {
      const cells = row.querySelectorAll("td");
      if (cells.length > 5) {
        curriculum["星期一"].push(
          cells[2].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期二"].push(
          cells[3].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期三"].push(
          cells[4].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期四"].push(
          cells[5].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期五"].push(
          cells[6].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期六"].push(
          cells[7].textContent.trim().split("\n")[0].trim(),
        );
        curriculum["星期日"].push(
          cells[8].textContent.trim().split("\n")[0].trim(),
        );
      }
    }
  });
  return curriculum;
}

function createEvents(courses, curriculum, classRooms) {
  let events = {
    星期一: [],
    星期二: [],
    星期三: [],
    星期四: [],
    星期五: [],
    星期六: [],
    星期日: [],
  };

  for (const [day, classes] of Object.entries(curriculum)) {
    let currentClassIndex = 0;
    let currentStartTime = scheduleTime[0].startTime;
    let currentEndTime = scheduleTime[0].endTime;

    while (currentClassIndex < classes.length) {
      // blank class
      while (
        classes[currentClassIndex] == "" &&
        currentClassIndex + 1 < classes.length
      ) {
        currentClassIndex++;
        currentStartTime = scheduleTime[currentClassIndex].startTime;
        currentEndTime = scheduleTime[currentClassIndex].endTime;
      }
      if (
        currentClassIndex + 1 == classes.length &&
        classes[currentClassIndex] == ""
      ) {
        break;
      }
      while (
        classes[currentClassIndex] == classes[currentClassIndex + 1] &&
        currentClassIndex + 1 < classes.length
      ) {
        currentClassIndex++;
        currentEndTime = scheduleTime[currentClassIndex].endTime;
      }
      const courseInfo = courses.find(
        (c) => c.courseName == classes[currentClassIndex],
      );
      const classRoom = classRooms[day][currentClassIndex];

      events[day].push({
        courseCode: courseInfo.courseCode,
        courseName: courseInfo.courseName,
        courseClassRoom: classRoom,
        courseTeacher: courseInfo.courseTeacher,
        courseCredits: courseInfo.courseCredits,
        courseType: courseInfo.courseType,
        startTime: currentStartTime,
        endTime: currentEndTime,
      });
      currentClassIndex++;
      currentStartTime = scheduleTime[currentClassIndex].startTime;
      currentEndTime = scheduleTime[currentClassIndex].endTime;
    }
  }
  return events;
}

function downloadFile() {
  if (!startDate.value || !endDate.value) {
    toast.error('請選擇起始日期和結束日期！');
    return;
  }
  if(new Date(startDate.value) > new Date(endDate.value)) {
    toast('結束日期需大於起始日期！');
    return;
  }
  if (!curriculumFile.value || !curriculumFile.value.files[0]) {
    toast('請上傳課表網頁檔案！');
    return;
  }

  const fileReader = new FileReader();
  fileReader.onload = function (event) {
    const htmlContent = event.target.result;

    const courses = parseCourses(htmlContent);
    const curriculum = parseCurriculum(htmlContent);
    const classRooms = parseClassRoom(htmlContent);

    console.log(classRooms);

    const events = createEvents(courses, curriculum, classRooms);

    const icsEvents = [];

    let currentDate = new Date(startDate.value);
    while (currentDate <= new Date(endDate.value)) {
      const day = currentDate.getDay();
      const dayString = [
        "星期日",
        "星期一",
        "星期二",
        "星期三",
        "星期四",
        "星期五",
        "星期六",
      ][day];
      events[dayString].forEach((event) => {
        const eventStartTime = new Date(currentDate);
        eventStartTime.setHours(event.startTime.split(":")[0]);
        eventStartTime.setMinutes(event.startTime.split(":")[1]);

        const eventEndTime = new Date(currentDate);
        eventEndTime.setHours(event.endTime.split(":")[0]);
        eventEndTime.setMinutes(event.endTime.split(":")[1]);

        icsEvents.push({
          start: eventStartTime.getTime(),
          end: eventEndTime.getTime(),
          title: event["courseName"],
          description: `導師：${event["courseTeacher"]}\n課程代碼：${event["courseCode"]}\n學分數：${event["courseCredits"]}\n類型：${event["courseType"]}`,
          location: `臺科大 ${event["courseClassRoom"]} 教室`,
          status: "CONFIRMED",
          busyStatus: "BUSY",
        });
      });

      currentDate.setDate(currentDate.getDate() + 1);
    }
    console.log(icsEvents);
    const { error, value } = ics.createEvents(icsEvents);
    if (error) {
      console.log(error);
      return;
    }
    console.log(value);
    const blob = new Blob([value], { type: "text/calendar" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "ntust_calendar.ics";
    a.click();
  };

  fileReader.readAsText(curriculumFile.value.files[0]);
}
</script>

<template>
  <Toaster richColors />
  <img alt="Logo" src="./assets/ntust_so_hard.png" class="logo" />
  <h1 class="title">NTUST Calendar Maker</h1>
  <p>製作 .ics 檔案以匯入通用日曆軟體！</p>
  <p>
    <a href="https://github.com/Zudo-Developent/NTUST-Calendar-Maker"
      >使用教學 | Github</a
    >
  </p>
  <div class="datePickerContainer">
    <DatePicker
      class="datePicker"
      v-model="startDate"
      placeholder="起始日期（開學）"
      showIcon
      fluid
      iconDisplay="input"
    />
    <DatePicker
      class="datePicker"
      v-model="endDate"
      placeholder="結束日期（結業）"
      showIcon
      fluid
      iconDisplay="input"
    />
  </div>

  <label class="fileUpload" for="curriculumFile">
    <span v-text="fileName"></span>
    <input
      type="file"
      ref="curriculumFile"
      @change="handleFileUpload"
      id="curriculumFile"
      accept=".html"
    />
  </label>
  <button id="buttonDownload" @click="downloadFile" raised>下載課表</button>
</template>

<style scoped>
.datePickerContainer {
  display: flex;
  gap: 1em;
  padding: 1em;
}

.title {
  font-size: 2em;
  font-weight: bold;
  text-align: center;
  background: linear-gradient(90deg, #d479d0, #a785eb);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 2px 4px 6px rgba(0, 0, 0, 0.3);
  margin: 0.5em 0;
  font-family: "Poppins", sans-serif;
}

#buttonDownload {
  margin: 1em;
  padding: 0.8em 2em;
  font-size: 1.25em;
  border-radius: 0.5em;
  background: linear-gradient(135deg, #d15dcb, #8c52ff);
  color: white;
  border: none;
  cursor: pointer;
  box-shadow:
    0 4px 6px rgba(0, 0, 0, 0.1),
    0 1px 3px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
}

#buttonDownload:hover {
  background: linear-gradient(135deg, #b84db6, #7345d6);
  transform: scale(1.05);
  box-shadow:
    0 6px 8px rgba(0, 0, 0, 0.15),
    0 2px 4px rgba(0, 0, 0, 0.1);
}

#buttonDownload:active {
  transform: scale(0.98);
  box-shadow:
    0 3px 5px rgba(0, 0, 0, 0.2),
    0 1px 3px rgba(0, 0, 0, 0.1);
}

.logo {
  height: 10em;
  border-radius: 1em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #d15dcb);
}

.fileUpload {
  display: flex;
  flex-direction: column;
  align-items: space-between;
  gap: 20px;
  cursor: pointer;
  align-items: center;
  justify-content: center;
  border: 2px dashed #e8e8e8;
  background-color: #212121;
  padding: 1.5rem;
  border-radius: 10px;
  margin: 1em 0;
}

.fileUpload input {
  display: none;
}
</style>
