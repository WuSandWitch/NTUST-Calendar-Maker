<script setup>
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'
import { ref } from 'vue';

const startDate = ref();
const endDate = ref();

const curriculumFile = ref();

function parseCourses(htmlContent) {
  const parser = new DOMParser();
  const doc = parser.parseFromString(htmlContent, "text/html");

  const courses = [];
  const courseRows = doc.querySelectorAll("table:nth-child(2) tbody tr");

  courseRows.forEach((row, index) => {
    if (index > 0) { // 跳过标题行
      const cells = row.querySelectorAll("td");
      if (cells.length > 5) {
        const courseCode = cells[0].textContent.trim();
        const courseName = cells[1].textContent.trim();
        const courseCredits = cells[2].textContent.trim();
        const courseType = cells[3].textContent.trim();
        const courseTeacher = cells[4].textContent.trim();
        // 根据需要扩展其他字段

        courses.push({ courseCode, courseName, courseCredits, courseType, courseTeacher });
      }
    }
  });
  console.log(courses);
  return courses;
}

function downloadFile() {
  if (!startDate.value || !endDate.value) {
    alert('請選擇起始日期和結束日期！');
    return;
  }
  if (!curriculumFile.value || !curriculumFile.value.files[0]) {
    alert('請上傳課表網頁檔案！');
    return;
  }

  const fileReader = new FileReader();
  fileReader.onload = function(event) {
    const htmlContent = event.target.result;
    const courses = parseCourses(htmlContent);

    // TODO: 在这里解析HTML内容并提取课程信息
    // 这个示例中跳过了解析步骤

    // 假设从HTML中提取到了课程信息，并已经转换成ICS格式的字符串
    const icsData = "BEGIN:VCALENDAR\nVERSION:2.0\nPRODID:-//hacksw/handcal//NONSGML v1.0//EN\n...";
    // 添加更多ICS内容...

    // 触发下载ICS文件
    const blob = new Blob([icsData], { type: 'text/calendar' });
    const downloadUrl = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = downloadUrl;
    a.download = 'schedule.ics'; 
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  };

  fileReader.readAsText(curriculumFile.value.files[0]);
}
</script>

<template>
  <img alt="Logo" src="./assets/ntust_so_hard.png" class="logo" />
  <h1 class="title">NTUST Calendar Maker</h1>
  <p>製作 .ics 檔案以匯入通用日曆軟體！</p>
  <VueDatePicker class="datePicker" v-model="startDate" :enable-time-picker="false" placeholder="起始日期（開學）" dark />
  <VueDatePicker class="datePicker" v-model="endDate" :enable-time-picker="false" placeholder="結束日期（結業）" dark />
  
  <label class="fileUpload" for="curriculumFile">
    <span>上傳課表網頁檔案（html檔）</span>
    <input type="file" ref="curriculumFile" id="curriculumFile"/>
  </label>
  <button id="buttonDownload" @click="downloadFile">下載課表</button>
</template>

<style scoped>
.datePicker {
  margin: 1em 0;
}
#buttonDownload {
  margin: 1em;
  padding: 1em;
  font-size: 1.5em;
  border-radius: 1em;
  background-color: #d15dcb;
  color: white;
  cursor: pointer;
  transition: background-color 300ms;
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
