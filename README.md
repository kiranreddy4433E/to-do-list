<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Task Manager with Reminder and Sound</title>
    <style>
        html, body {
            height: 100%;
            margin: 0;
            padding: 0;
        }
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            background-image: linear-gradient(30deg, rgb(222, 189, 189), rgb(100, 100, 235));
            background-repeat: no-repeat;
            background-position: center;
            background-size: cover;
        }
        #cnt {
            width: 35vw;
            background-color: slateblue;
            text-align: center;
            border-radius: 20px;
            padding: 20px;
            box-sizing: border-box;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            margin-top: 30px;
        }
        input, button {
            width: 20vw;
            height: 5vh;
            border-radius: 6px;
            box-sizing: border-box;
            margin-top: 10px;
            outline: none;
        }
        #reminderTime{
            border: none;
            text-decoration: none;
        }
        input[type="text"] {
            border: 1px solid rgb(118, 160, 118);
            background-color: rgb(118, 160, 118);
            box-shadow: 0 0 5px rgb(118, 160, 118);
        }
        input[type="time"] {
            border-radius: 6px;
            margin-top: 10px;
            height: 5vh;
        }
        button {
            width: auto;
            padding: 0 20px;
            background-color: lawngreen;
            border: 1px solid lawngreen;
            box-shadow: 0 0 5px lawngreen;
        }
        button:hover {
            background-color: rgb(127, 250, 5);
            opacity: 0.7;
        }
        .task {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background-color: white;
            padding: 10px;
            margin: 10px 0;
            border-radius: 10px;
            box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
        }
        .task.completed {
            text-decoration: line-through;
            background-color: lightgray;
        }
        .task button {
            width: auto;
            height: auto;
            padding: 5px 10px;
            margin-left: 10px;
        }
        small {
            font-size: 0.8rem;
            color: #555;
        }
        
    </style>
</head>
<body>
    <h1>NOTE YOUR TASKS</h1>
    <div id="cnt">
        <input type="text" id="text" placeholder="Enter your task here" />
        <input type="time" id="reminderTime"><br><br>
        <button id="btn" onclick="addTask()">ADD</button>
        <div id="tasks"></div>
    </div>

    <audio id="alarmSound" src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" preload="auto"></audio>

    <script>
        const input = document.getElementById('text');
        const reminderInput = document.getElementById('reminderTime');
        const tasksContainer = document.getElementById('tasks');
        const alarmSound = document.getElementById('alarmSound');

        function formatTime(date) {
            return date.toTimeString().slice(0, 5);
        }

        function addTask() {
            const taskText = input.value.trim();
            const reminderTime = reminderInput.value;

            if (taskText !== "") {
                const task = document.createElement('div');
                task.classList.add('task');
                task.dataset.reminderTime = reminderTime || "";
                task.innerHTML = `
                    <span>${taskText} ${reminderTime ? `<small>(Remind at ${reminderTime})</small>` : ''}</span>
                    <div>
                        <button onclick="editTask(this)">Edit</button>
                        <button onclick="deleteTask(this)">Delete</button>
                        <button onclick="toggleComplete(this)">Complete</button>
                    </div>
                `;
                tasksContainer.appendChild(task);
                input.value = "";
                reminderInput.value = "";
                saveTasks();
            }
        }

        function editTask(button) {
            const task = button.parentElement.parentElement;
            const taskSpan = task.querySelector('span');
            let taskText = taskSpan.childNodes[0].nodeValue.trim(); // Get text excluding small tag
            const newTaskText = prompt('Edit your task:', taskText);
            if (newTaskText !== null && newTaskText.trim() !== "") {
                taskSpan.childNodes[0].nodeValue = newTaskText + ' ';
                saveTasks();
            }
        }

        function deleteTask(button) {
            const task = button.parentElement.parentElement;
            tasksContainer.removeChild(task);
            saveTasks();
        }

        function toggleComplete(button) {
            const task = button.parentElement.parentElement;
            task.classList.toggle('completed');
            saveTasks();
        }

        function saveTasks() {
            const tasks = [];
            tasksContainer.querySelectorAll('.task').forEach(task => {
                const textNode = task.querySelector('span').childNodes[0];
                const text = textNode ? textNode.nodeValue.trim() : "";
                tasks.push({
                    text: text,
                    completed: task.classList.contains('completed'),
                    reminderTime: task.dataset.reminderTime || ""
                });
            });
            localStorage.setItem('tasks', JSON.stringify(tasks));
        }

        function loadTasks() {
            const tasks = JSON.parse(localStorage.getItem('tasks')) || [];
            tasks.forEach(taskData => {
                const task = document.createElement('div');
                task.classList.add('task');
                if (taskData.completed) task.classList.add('completed');
                task.dataset.reminderTime = taskData.reminderTime || "";
                task.innerHTML = `
                    <span>${taskData.text} ${taskData.reminderTime ? `<small>(Remind at ${taskData.reminderTime})</small>` : ''}</span>
                    <div>
                        <button onclick="editTask(this)">Edit</button>
                        <button onclick="deleteTask(this)">Delete</button>
                        <button onclick="toggleComplete(this)">Complete</button>
                    </div>
                `;
                tasksContainer.appendChild(task);
            });
        }

        // Check reminders every 30 seconds
        setInterval(() => {
            const now = new Date();
            const currentTime = formatTime(now);

            tasksContainer.querySelectorAll('.task').forEach(task => {
                const reminder = task.dataset.reminderTime;
                if (reminder === currentTime && !task.classList.contains('completed')) {
                    alert(`Reminder: Task "${task.querySelector('span').childNodes[0].nodeValue.trim()}" is due now!`);
                    alarmSound.play().catch(e => {
                        console.warn('Audio play prevented:', e);
                    });
                }
            });
        }, 30000);

        window.onload = loadTasks;
    </script>
</body>
</html>
