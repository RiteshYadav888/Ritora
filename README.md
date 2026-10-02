// ========================================
// AI STUDY PLANNER
// MAIN JAVASCRIPT
// ========================================


// ========================================
// ELEMENTS
// ========================================

const taskModal = document.getElementById("taskModal");

const taskForm = document.getElementById("taskForm");

const closeModal = document.getElementById("closeModal");

const cancelTask = document.getElementById("cancelTask");

const addTaskButtons =
    document.querySelectorAll(".add-task-btn");

const taskList =
    document.getElementById("taskList");

const emptyState =
    document.getElementById("emptyState");

const todayTasks =
    document.getElementById("todayTasks");

const completedTasks =
    document.getElementById("completedTasks");

const studyHours =
    document.getElementById("studyHours");

const progressPercentage =
    document.getElementById("progressPercentage");

const progressText =
    document.getElementById("progressText");

const progressFill =
    document.getElementById("progressFill");

const recommendationText =
    document.getElementById("recommendationText");


// ========================================
// TASK DATA
// ========================================

let tasks =
    JSON.parse(
        localStorage.getItem("studyTasks")
    ) || [];


// ========================================
// OPEN MODAL
// ========================================

addTaskButtons.forEach(button => {

    button.addEventListener("click", () => {

        taskModal.classList.add("show");

        document
            .getElementById("subject")
            .focus();

    });

});


// ========================================
// CLOSE MODAL
// ========================================

function closeTaskModal() {

    taskModal.classList.remove("show");

    taskForm.reset();

}


closeModal.addEventListener(
    "click",
    closeTaskModal
);


cancelTask.addEventListener(
    "click",
    closeTaskModal
);


// ========================================
// CLOSE WHEN CLICKING OUTSIDE
// ========================================

taskModal.addEventListener(
    "click",
    (event) => {

        if (event.target === taskModal) {

            closeTaskModal();

        }

    }
);


// ========================================
// ESC KEY
// ========================================

document.addEventListener(
    "keydown",
    (event) => {

        if (
            event.key === "Escape" &&
            taskModal.classList.contains("show")
        ) {

            closeTaskModal();

        }

    }
);


// ========================================
// ADD NEW TASK
// ========================================

taskForm.addEventListener(
    "submit",
    (event) => {

        event.preventDefault();


        const subject =
            document
                .getElementById("subject")
                .value
                .trim();


        const topic =
            document
                .getElementById("topic")
                .value
                .trim();


        const date =
            document
                .getElementById("taskDate")
                .value;


        const studyTime =
            Number(
                document
                    .getElementById("studyTime")
                    .value
            );


        const priority =
            document
                .getElementById("priority")
                .value;


        const newTask = {

            id: Date.now(),

            subject: subject,

            topic: topic,

            date: date,

            studyTime: studyTime,

            priority: priority,

            completed: false

        };


        tasks.push(newTask);


        saveTasks();

        renderTasks();

        updateDashboard();

        updateRecommendation();

        closeTaskModal();

    }
);


// ========================================
// SAVE TASKS
// ========================================

function saveTasks() {

    localStorage.setItem(
        "studyTasks",
        JSON.stringify(tasks)
    );

}


// ========================================
// RENDER TASKS
// ========================================

function renderTasks() {

    taskList.innerHTML = "";


    if (tasks.length === 0) {

        emptyState.style.display = "block";

        return;

    }


    emptyState.style.display = "none";


    tasks.forEach(task => {

        const taskElement =
            document.createElement("div");


        taskElement.className =
            "task-item";


        if (task.completed) {

            taskElement.classList.add(
                "completed"
            );

        }


        taskElement.innerHTML = `

            <button
                class="task-check"
                data-id="${task.id}">
            </button>


            <div class="task-content">

                <h4>
                    ${escapeHTML(task.topic)}
                </h4>

                <p>
                    ${escapeHTML(task.subject)}
                    •
                    ${task.studyTime} min
                    •
                    ${formatDate(task.date)}
                </p>

            </div>


            <span class="priority ${task.priority}">
                ${task.priority}
            </span>


            <button
                class="delete-task"
                data-id="${task.id}">
                ×
            </button>

        `;


        taskList.appendChild(taskElement);

    });

}


// ========================================
// TASK CLICK EVENTS
// ========================================

taskList.addEventListener(
    "click",
    (event) => {

        const id =
            Number(
                event.target.dataset.id
            );


        if (!id) return;


        // Complete task

        if (
            event.target.classList.contains(
                "task-check"
            )
        ) {

            toggleTask(id);

        }


        // Delete task

        if (
            event.target.classList.contains(
                "delete-task"
            )
        ) {

            deleteTask(id);

        }

    }
);


// ========================================
// COMPLETE TASK
// ========================================

function toggleTask(id) {

    tasks =
        tasks.map(task => {

            if (task.id === id) {

                return {
                    ...task,
                    completed: !task.completed
                };

            }

            return task;

        });


    saveTasks();

    renderTasks();

    updateDashboard();

    updateRecommendation();

}


// ========================================
// DELETE TASK
// ========================================

function deleteTask(id) {

    tasks =
        tasks.filter(
            task => task.id !== id
        );


    saveTasks();

    renderTasks();

    updateDashboard();

    updateRecommendation();

}


// ========================================
// DASHBOARD UPDATE
// ========================================

function updateDashboard() {

    const totalTasks =
        tasks.length;


    const completed =
        tasks.filter(
            task => task.completed
        ).length;


    const totalMinutes =
        tasks.reduce(
            (total, task) =>
                total + task.studyTime,
            0
        );


    const progress =
        totalTasks === 0
            ? 0
            : Math.round(
                (completed / totalTasks) * 100
            );


    todayTasks.textContent =
        totalTasks;


    completedTasks.textContent =
        completed;


    studyHours.textContent =
        formatHours(totalMinutes);


    progressPercentage.textContent =
        `${progress}%`;


    progressText.textContent =
        `${progress}%`;


    progressFill.style.width =
        `${progress}%`;

}


// ========================================
// AI RECOMMENDATION
// ========================================

function updateRecommendation() {

    if (tasks.length === 0) {

        recommendationText.textContent =
            "Add some study tasks and your personalized recommendation will appear here.";

        return;

    }


    const incompleteTasks =
        tasks.filter(
            task => !task.completed
        );


    if (incompleteTasks.length === 0) {

        recommendationText.textContent =
            "Excellent work! You have completed all your current study tasks. 🎉";

        return;

    }


    const highPriority =
        incompleteTasks.filter(
            task => task.priority === "High"
        );


    if (highPriority.length > 0) {

        const task =
            highPriority[0];


        recommendationText.textContent =
            `Focus on "${task.topic}" first. It is marked as a high-priority task and needs your attention.`;

        return;

    }


    const longestTask =
        [...incompleteTasks].sort(
            (a, b) =>
                b.studyTime - a.studyTime
        )[0];


    recommendationText.textContent =
        `Consider starting with "${longestTask.topic}" for ${longestTask.studyTime} minutes to make good progress today.`;

}


// ========================================
// DATE FORMAT
// ========================================

function formatDate(dateString) {

    if (!dateString) return "";

    const date =
        new Date(dateString + "T00:00:00");


    return date.toLocaleDateString(
        "en-IN",
        {
            day: "2-digit",
            month: "short"
        }
    );

}


// ========================================
// HOURS FORMAT
// ========================================

function formatHours(minutes) {

    if (minutes < 60) {

        return `${minutes}m`;

    }


    const hours =
        Math.floor(minutes / 60);


    const remainingMinutes =
        minutes % 60;


    if (remainingMinutes === 0) {

        return `${hours}h`;

    }


    return `${hours}h ${remainingMinutes}m`;

}


// ========================================
// SECURITY HELPER
// ========================================

function escapeHTML(text) {

    const div =
        document.createElement("div");

    div.textContent = text;

    return div.innerHTML;

}


// ========================================
// INITIAL LOAD
// ========================================

renderTasks();

updateDashboard();

updateRecommendation();
