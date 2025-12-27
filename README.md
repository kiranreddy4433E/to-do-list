<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Task Manager - Date & Week Day Views</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #1e1e2f, #5c5cde, #f06565);
    margin: 0; padding: 0; display: flex; flex-direction: column; align-items: center;
    min-height: 100vh;
    color: #fff;
  }
  h1 {
    margin-top: 20px;
    text-shadow: 2px 2px 5px rgba(0,0,0,0.4);
  }
  #cnt {
    width: 90vw; max-width: 650px;
    background: rgba(255,255,255,0.1);
    border-radius: 20px;
    padding: 20px;
    box-sizing: border-box;
    margin: 20px 0;
    box-shadow: 0 0 15px rgba(0,0,0,0.4);
    backdrop-filter: blur(10px);
  }
  button {
    background: #70ff70;
    border: none;
    padding: 8px 16px;
    font-weight: bold;
    cursor: pointer;
    border-radius: 10px;
    margin: 8px 0;
    transition: transform 0.2s, opacity 0.3s;
  }
  button:hover {
    transform: scale(1.05);
    opacity: 0.85;
  }
  input[type=text], input[type=date], input[type=time], select {
    width: 100%;
    padding: 8px 12px;
    margin: 6px 0;
    border-radius: 8px;
    border: none;
    font-size: 1rem;
    box-sizing: border-box;
    outline: none;
  }
  .task {
    background: #fff;
    border-radius: 10px;
    color: #000;
    margin: 10px 0;
    padding: 12px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    display: flex;
    justify-content: space-between;
    align-items: center;
    animation: fadeIn 0.3s ease;
    transition: background 0.3s;
  }
  .task.completed span.task-text {
    text-decoration: line-through;
    color: gray;
  }
  .task.overdue {
    background: #ffd5d5;
    border-left: 6px solid #e63946;
  }
  .task small {
    font-size: 0.8rem;
    color: #444;
    display: block;
    margin-top: 4px;
  }
  .task-actions button {
    margin-left: 6px;
    font-size: 0.85rem;
  }
  .day-section {
    background: #4a4a9f;
    border-radius: 12px;
    padding: 15px;
    margin-top: 20px;
    color: white;
    box-shadow: inset 0 0 8px rgba(0,0,0,0.3);
  }
  .day-section h3 {
    margin-top: 0;
    text-shadow: 1px 1px 3px rgba(0,0,0,0.5);
  }
  /* Animation */
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  /* Modal */
  #alarmModal {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.6);
    display: none;
    justify-content: center;
    align-items: center;
  }
  #alarmBox {
    background: #fff;
    color: #000;
    padding: 20px;
    border-radius: 15px;
    max-width: 400px;
    text-align: center;
    animation: fadeIn 0.3s ease;
  }
</style>
</head>
<body>

<h1>✨ Task Manager</h1>
<div id="cnt">
  <button id="toggleViewBtn">Switch to Week View</button>

  <!-- Date-wise view controls -->
  <div id="dateView">
    <input type="date" id="dateSelector" />
    <input type="text" id="taskText" placeholder="Task description" />
    <input type="time" id="taskTime" />
    <button id="addBtnDate">Add Task to Date</button>
    <div id="tasksContainerDate"></div>
  </div>

  <!-- Week view controls - hidden initially -->
  <div id="weekView" style="display:none;">
    <select id="weekDaySelect">
      <option value="Sunday">Sunday</option>
      <option value="Monday">Monday</option>
      <option value="Tuesday">Tuesday</option>
      <option value="Wednesday">Wednesday</option>
      <option value="Thursday">Thursday</option>
      <option value="Friday">Friday</option>
      <option value="Saturday">Saturday</option>
    </select>
    <input type="text" id="taskTextWeek" placeholder="Task description" />
    <input type="time" id="taskTimeWeek" />
    <button id="addBtnWeek">Add Task to Selected Weekday</button>
    <div id="tasksContainerWeek"></div>
  </div>
</div>

<audio id="alarmSound" src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" preload="auto" playsinline></audio>

<!-- Custom Alarm Modal -->
<div id="alarmModal">
  <div id="alarmBox">
    <h2>⏰ Reminder</h2>
    <p id="alarmText"></p>
    <button onclick="dismissAlarm()">Dismiss</button>
  </div>
</div>

<script>
  let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
  let isDateView = true;

  const alarmSound = document.getElementById('alarmSound');
  const alarmModal = document.getElementById('alarmModal');
  const alarmText = document.getElementById('alarmText');

  function saveTasks(){ localStorage.setItem('tasks', JSON.stringify(tasks)); }

  function getWeekDayDate(weekday){
    const now = new Date();
    const dayIdx = now.getDay();
    const targetDayIdx = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'].indexOf(weekday);
    const diff = targetDayIdx - dayIdx;
    const targetDate = new Date(now);
    targetDate.setDate(now.getDate() + diff);
    return targetDate.toISOString().slice(0,10);
  }

  function getTasksForDate(date){ return tasks.filter(t => t.date === date); }

  function renderTasksDate(){
    const dateSelector = document.getElementById('dateSelector');
    const tasksContainerDate = document.getElementById('tasksContainerDate');
    const selectedDate = dateSelector.value;
    tasksContainerDate.innerHTML = '';
    if(!selectedDate){
      tasksContainerDate.innerHTML = '<p style="color:lightgray;">Please select a date.</p>';
      return;
    }
    const filtered = getTasksForDate(selectedDate);
    if(filtered.length === 0){
      tasksContainerDate.innerHTML = '<p style="color:lightgray;">No tasks for this date.</p>';
      return;
    }
    filtered.forEach((task, idx) => {
      const now = new Date();
      const taskTime = task.time ? new Date(task.date + "T" + task.time) : null;
      const overdue = taskTime && taskTime < now && !task.completed;
      const div = document.createElement('div');
      div.className = 'task' + (task.completed ? ' completed' : '') + (overdue ? ' overdue' : '');
      div.innerHTML = `
        <div>
          <span class="task-text">${task.text}</span>
          <small>Reminder Time: ${task.time || 'None'}</small>
        </div>
        <div class="task-actions">
          <button onclick="toggleComplete('${task.date}', ${idx})">${task.completed ? 'Undo' : 'Complete'}</button>
          <button onclick="editTask('${task.date}', ${idx})">Edit</button>
          <button onclick="deleteTask('${task.date}', ${idx})">Delete</button>
        </div>
      `;
      tasksContainerDate.appendChild(div);
    });
  }

  function renderTasksWeek(){
    const tasksContainerWeek = document.getElementById('tasksContainerWeek');
    tasksContainerWeek.innerHTML = '';
    ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'].forEach(day => {
      const dayDiv = document.createElement('div');
      dayDiv.className = 'day-section';
      const dateOfDay = getWeekDayDate(day);
      dayDiv.innerHTML = `<h3>${day} (${dateOfDay})</h3>`;
      const dayTasks = getTasksForDate(dateOfDay);
      if(dayTasks.length === 0){
        dayDiv.innerHTML += '<p style="color:#ccc;">No tasks.</p>';
      } else {
        dayTasks.forEach((task, idx) => {
          const now = new Date();
          const taskTime = task.time ? new Date(task.date + "T" + task.time) : null;
          const overdue = taskTime && taskTime < now && !task.completed;
          const taskDiv = document.createElement('div');
          taskDiv.className = 'task' + (task.completed ? ' completed' : '') + (overdue ? ' overdue' : '');
          taskDiv.innerHTML = `
            <div>
              <span class="task-text">${task.text}</span>
              <small>Reminder Time: ${task.time || 'None'}</small>
            </div>
            <div class="task-actions">
              <button onclick="toggleComplete('${task.date}', ${idx})">${task.completed ? 'Undo' : 'Complete'}</button>
              <button onclick="editTask('${task.date}', ${idx})">Edit</button>
              <button onclick="deleteTask('${task.date}', ${idx})">Delete</button>
            </div>
          `;
          dayDiv.appendChild(taskDiv);
        });
      }
      tasksContainerWeek.appendChild(dayDiv);
    });
  }

  window.toggleComplete = function(date, idx){
    const filtered = getTasksForDate(date);
    const task = filtered[idx];
    const realIndex = tasks.findIndex(t => t === task);
    if(realIndex > -1){
      tasks[realIndex].completed = !tasks[realIndex].completed;
      saveTasks();
      isDateView ? renderTasksDate() : renderTasksWeek();
    }
  }

  window.editTask = function(date, idx){
    const filtered = getTasksForDate(date);
    const task = filtered[idx];
    const realIndex = tasks.findIndex(t => t === task);
    if(realIndex === -1) return;
    const newText = prompt('Edit task description:', task.text);
    if(newText === null) return;
    const newTime = prompt('Edit reminder time (HH:MM):', task.time || '');
    if(newTime === null) return;
    tasks[realIndex].text = newText.trim() || tasks[realIndex].text;
    tasks[realIndex].time = newTime;
    saveTasks();
    isDateView ? renderTasksDate() : renderTasksWeek();
  }

  window.deleteTask = function(date, idx){
    const filtered = getTasksForDate(date);
    const task = filtered[idx];
    const realIndex = tasks.findIndex(t => t === task);
    if(realIndex === -1) return;
    if(confirm('Delete this task?')){
      tasks.splice(realIndex, 1);
      saveTasks();
      isDateView ? renderTasksDate() : renderTasksWeek();
    }
  }

  document.getElementById('toggleViewBtn').addEventListener('click', () => {
    isDateView = !isDateView;
    document.getElementById('dateView').style.display = isDateView ? 'block' : 'none';
    document.getElementById('weekView').style.display = isDateView ? 'none' : 'block';
    document.getElementById('toggleViewBtn').textContent = isDateView ? 'Switch to Week View' : 'Switch to Date View';
    isDateView ? renderTasksDate() : renderTasksWeek();
  });

  document.getElementById('addBtnDate').addEventListener('click', () => {
    const dateSelector = document.getElementById('dateSelector');
    const taskText = document.getElementById('taskText');
    const taskTime = document.getElementById('taskTime');
    if(!dateSelector.value){ alert('Select a date first'); return; }
    if(!taskText.value.trim()){ alert('Enter task description'); return; }
    tasks.push({ text: taskText.value.trim(), time: taskTime.value, date: dateSelector.value, completed: false });
    saveTasks();
    renderTasksDate();
    taskText.value = ''; taskTime.value = '';
  });

  document.getElementById('addBtnWeek').addEventListener('click', () => {
    const weekDaySelect = document.getElementById('weekDaySelect');
    const taskTextWeek = document.getElementById('taskTextWeek');
    const taskTimeWeek = document.getElementById('taskTimeWeek');
    if(!taskTextWeek.value.trim()){ alert('Enter task description'); return; }
    const date = getWeekDayDate(weekDaySelect.value);
    tasks.push({ text: taskTextWeek.value.trim(), time: taskTimeWeek.value, date, completed: false });
    saveTasks();
    renderTasksWeek();
    taskTextWeek.value = ''; taskTimeWeek.value = '';
  });

  // Alarm Check
  setInterval(() => {
    const now = new Date();
    const currentDate = now.toISOString().slice(0,10);
    const currentTime = now.toTimeString().slice(0,5);
    tasks.forEach(task => {
      if(!task.completed && task.date === currentDate && task.time === currentTime){
        alarmSound.loop = true;
        alarmSound.play().catch(e => console.warn('Audio blocked:', e));
        alarmText.textContent = `Task "${task.text}" is due now!`;
        alarmModal.style.display = "flex";
      }
    });
  }, 15000);

  window.dismissAlarm = function(){
    alarmSound.pause();
    alarmSound.currentTime = 0;
    alarmModal.style.display = "none";
  }

  if(!document.getElementById('dateSelector').value){
    document.getElementById('dateSelector').value = new Date().toISOString().slice(0,10);
  }
  renderTasksDate();
</script>

</body>
</html>
