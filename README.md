# to-do-list
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 30px;
        }
        #cnt {
            width: 35vw;
            background-color: slateblue;
            text-align: center;
            border-radius: 20px;
            padding: 20px;
            box-sizing: border-box;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        input, button {
            width: 20vw;
            height: 5vh;
            border-radius: 6px;
            box-sizing: border-box;
            margin-top: 10px;
        }
        input {
            border: 1px solid rgb(118, 160, 118);
            background-color: rgb(118, 160, 118);
            box-shadow: 0 0 5px rgb(118, 160, 118);
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
    </style>
</head>
<body>
    <h1>NOTE YOUR TASKS</h1>
    <div id="cnt">
        <input type="text" id="text" placeholder="Enter your task here">
        <button id="btn" onclick="addTask()">ADD</button>
        <div id="tasks"></div>
    </div>
    <script>
        const input = document.getElementById('text');
        const tasksContainer = document.getElementById('tasks');

        function addTask() {
            const taskText = input.value.trim();
            if (taskText !== "") {
                const task = document.createElement('div');
                task.classList.add('task');
                task.innerHTML = `
                    <span>${taskText}</span>
                    <div>
                        <button onclick="editTask(this)">Edit</button>
                        <button onclick="deleteTask(this)">Delete</button>
                        <button onclick="toggleComplete(this)">Complete</button>
                    </div>
                `;
                tasksContainer.appendChild(task);
                input.value = "";
                saveTasks();
            }
        }

        function editTask(button) {
            const task = button.parentElement.parentElement;
            const taskText = task.querySelector('span').innerText;
            const newTaskText = prompt('Edit your task:', taskText);
            if (newTaskText !== null && newTaskText.trim() !== "") {
                task.querySelector('span').innerText = newTaskText;
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
                tasks.push({
                    text: task.querySelector('span').innerText,
                    completed: task.classList.contains('completed')
                });
            });
            localStorage.setItem('tasks', JSON.stringify(tasks));
        }

        function loadTasks() {
            const tasks = JSON.parse(localStorage.getItem('tasks')) || [];
            tasks.forEach(taskData => {
                const task = document.createElement('div');
                task.classList.add('task');
                if (taskData.completed) {
                    task.classList.add('completed');
                }
                task.innerHTML = `
                    <span>${taskData.text}</span>
                    <div>
                        <button onclick="editTask(this)">Edit</button>
                        <button onclick="deleteTask(this)">Delete</button>
                        <button onclick="toggleComplete(this)">Complete</button>
                    </div>
                `;
                tasksContainer.appendChild(task);
            });
        }

        // Load tasks from localStorage on page load
        window.onload = loadTasks;
    </script>
</body>
</html>
