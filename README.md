<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Lịch Long Hải</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(to right, #0f2027, #203a43, #2c5364);
      color: #fff;
    }
    .calendar {
      max-width: 700px;
      margin: 2rem auto;
      background-color: rgba(15, 15, 50, 0.9);
      border-radius: 10px;
      padding: 1rem;
      box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
    }
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1rem;
    }
    select, button {
      background-color: rgba(255, 255, 255, 0.1);
      color: white;
      border: none;
      padding: 5px;
      border-radius: 5px;
    }
    button:hover {
      background-color: rgba(255, 255, 255, 0.3);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background-color: rgba(0, 0, 0, 0.3);
    }
    th, td {
      width: 14.2%;
      padding: 10px;
      text-align: center;
      cursor: pointer;
      border: 1px solid rgba(255, 255, 255, 0.2);
      vertical-align: top;
    }
    td:hover {
      background-color: rgba(255, 255, 255, 0.2);
    }
    .selected {
      background-color: rgba(200, 100, 255, 0.6);
      font-weight: bold;
      border: 2px solid #fff;
    }
    .not-current-month {
      opacity: 0.3;
    }
    #taskInput {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      border-radius: 5px;
      border: none;
    }
    #saveBtn, #exportBtn {
      margin-top: 10px;
      width: 100%;
      padding: 10px;
      background-color: #9b59b6;
      color: white;
      border: none;
      border-radius: 5px;
      font-weight: bold;
    }
    .has-task {
      background-color: rgba(155, 89, 182, 0.6) !important;
      font-weight: bold;
      border: 2px solid #fff;
    }
    .task-note {
      display: block;
      font-size: 0.7rem;
      margin-top: 5px;
      color: #fff;
      white-space: normal;
    }
  </style>
</head>
<body>
  <div class="calendar">
    <div class="header">
      <button onclick="changeMonth(-1)">◀</button>
      <div>
        <select id="monthSelect" onchange="renderCalendar()"></select>
        <select id="yearSelect" onchange="renderCalendar()"></select>
      </div>
      <button onclick="changeMonth(1)">▶</button>
    </div>
    <table id="calendarTable">
      <thead>
        <tr>
          <th>CN</th><th>T2</th><th>T3</th><th>T4</th><th>T5</th><th>T6</th><th>T7</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
    <input type="text" id="taskInput" placeholder="Nhập công việc..." />
    <button id="saveBtn" onclick="saveTask()">Lưu công việc</button>
    <button id="exportBtn" onclick="exportToPDF()">Xuất PDF công việc</button>
  </div>

  <audio id="clickSound" src="https://www.soundjay.com/buttons/sounds/button-3.mp3"></audio>

  <script>
    const monthSelect = document.getElementById('monthSelect');
    const yearSelect = document.getElementById('yearSelect');
    const calendarTable = document.getElementById('calendarTable').getElementsByTagName('tbody')[0];
    const taskInput = document.getElementById('taskInput');
    const clickSound = document.getElementById('clickSound');

    let selectedDate = null;

    for (let m = 0; m < 12; m++) {
      const opt = document.createElement('option');
      opt.value = m;
      opt.text = new Date(2023, m, 1).toLocaleString('vi-VN', { month: 'long' });
      monthSelect.appendChild(opt);
    }

    for (let y = 2000; y <= 2100; y++) {
      const opt = document.createElement('option');
      opt.value = y;
      opt.text = y;
      yearSelect.appendChild(opt);
    }

    const today = new Date();
    monthSelect.value = today.getMonth();
    yearSelect.value = today.getFullYear();

    function playSound() {
      clickSound.currentTime = 0;
      clickSound.play();
    }

    function changeMonth(delta) {
      let month = parseInt(monthSelect.value);
      let year = parseInt(yearSelect.value);

      month += delta;
      if (month < 0) {
        month = 11;
        year--;
      } else if (month > 11) {
        month = 0;
        year++;
      }
      monthSelect.value = month;
      yearSelect.value = year;
      renderCalendar();
    }

    function saveTask() {
      if (!selectedDate) return;
      const key = selectedDate.toDateString();
      const task = taskInput.value.trim();
      if (task) {
        localStorage.setItem(key, task);
      } else {
        localStorage.removeItem(key);
      }
      renderCalendar();
      taskInput.value = '';
    }

    function renderCalendar() {
      const month = parseInt(monthSelect.value);
      const year = parseInt(yearSelect.value);
      const firstDay = new Date(year, month, 1);
      const lastDay = new Date(year, month + 1, 0);
      const startDay = firstDay.getDay();

      calendarTable.innerHTML = '';
      let date = 1 - startDay;
      for (let i = 0; i < 6; i++) {
        const row = document.createElement('tr');
        for (let j = 0; j < 7; j++) {
          const cell = document.createElement('td');
          const cellDate = new Date(year, month, date);

          cell.className = ''; // Clear all classes

          if (cellDate.getMonth() === month) {
            const dayDiv = document.createElement('div');
            dayDiv.textContent = cellDate.getDate();
            cell.appendChild(dayDiv);

            cell.onclick = () => {
              playSound();
              selectedDate = cellDate;
              document.querySelectorAll('td').forEach(td => td.classList.remove('selected'));
              cell.classList.add('selected');
              const key = selectedDate.toDateString();
              taskInput.value = localStorage.getItem(key) || '';
            };

            const key = cellDate.toDateString();
            const task = localStorage.getItem(key);
            if (task) {
              cell.classList.add('has-task');
              const taskSpan = document.createElement('span');
              taskSpan.textContent = task;
              taskSpan.className = 'task-note';
              cell.appendChild(taskSpan);
            }
          } else {
            cell.classList.add('not-current-month');
            cell.textContent = cellDate.getDate();
          }

          row.appendChild(cell);
          date++;
        }
        calendarTable.appendChild(row);
      }
    }

    function exportToPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      const month = parseInt(monthSelect.value);
      const year = parseInt(yearSelect.value);
      doc.setFontSize(14);
      doc.text(`Công việc tháng ${month + 1}/${year}`, 10, 10);
      let y = 20;

      for (let day = 1; day <= 31; day++) {
        const date = new Date(year, month, day);
        if (date.getMonth() !== month) break;
        const key = date.toDateString();
        const task = localStorage.getItem(key);
        if (task) {
          doc.text(`${day}/${month + 1}/${year}: ${task}`, 10, y);
          y += 10;
          if (y > 280) {
            doc.addPage();
            y = 10;
          }
        }
      }

      doc.save(`Lich_Cong_Viec_${month + 1}_${year}.pdf`);
    }

    renderCalendar();
  </script>
</body>
</html>

