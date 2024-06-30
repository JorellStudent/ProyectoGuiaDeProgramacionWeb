Proyecto Guía de Programación Web - Gestión de Tareas
Este proyecto es una aplicación de gestión de tareas que utiliza Vue.js para el frontend y Node.js con Express y MySQL para el backend.

Índice

- [Requisitos Previos](#requisitos-previos)
- [Configuración del Proyecto Backend (Node.js, Express, MySQL)](#configuración-del-proyecto-backend-nodejs-express-mysql)
  - [Crear el proyecto backend](#crear-el-proyecto-backend)
  - [Instalar dependencias necesarias](#instalar-dependencias-necesarias)
  - [Estructura de carpetas](#estructura-de-carpetas)
  - [Configurar la conexión a la base de datos](#configurar-la-conexión-a-la-base-de-datos)
  - [Crear el servidor Express](#crear-el-servidor-express)
  - [Definir las rutas y controladores](#definir-las-rutas-y-controladores)
  - [Crear la base de datos y la tabla](#crear-la-base-de-datos-y-la-tabla)
- [Configuración del Proyecto Frontend (Vue.js)](#configuración-del-proyecto-frontend-vuejs)
  - [Crear el proyecto Vue.js](#crear-el-proyecto-vuejs)
  - [Instalar Axios para las peticiones HTTP](#instalar-axios-para-las-peticiones-http)
  - [Configurar Axios](#configurar-axios)
  - [Crear componentes Vue para la gestión de tareas](#crear-componentes-vue-para-la-gestión-de-tareas)
  - [Integrar los componentes en la aplicación principal](#integrar-los-componentes-en-la-aplicación-principal)
- [Ejecución de la Aplicación](#ejecución-de-la-aplicación)
- [Compilar el Proyecto Vue.js e Integrarlo en Node.js](#compilar-el-proyecto-vuejs-e-integrarlo-en-nodejs)

Requisitos Previos
Antes de comenzar, asegúrate de tener instalados:

Node.js
MySQL
Conocimientos básicos de JavaScript, Vue.js, Node.js, Express y MySQL

Configuración del Proyecto Backend (Node.js, Express, MySQL)

Crear el proyecto backend
bash
Copiar código
mkdir task-manager
cd task-manager
npm init -y

Instalar dependencias necesarias
bash
Copiar código
npm install express mysql2 body-parser cors

Estructura de carpetas
bash
Copiar código
mkdir server
cd server
mkdir routes models controllers

Configurar la conexión a la base de datos
javascript
Copiar código
// server/db.js
const mysql = require('mysql2');

const connection = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'task_manager'
});

connection.connect((err) => {
    if (err) {
        console.error('Error connecting to the database:', err);
        return;
    }
    console.log('Connected to the MySQL database.');
});

module.exports = connection;

Crear el servidor Express
javascript
Copiar código
// server/index.js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const taskRoutes = require('./routes/tasks');

const app = express();
app.use(cors());
app.use(bodyParser.json());

app.use('/api/tasks', taskRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});

Definir las rutas y controladores
javascript
Copiar código
// server/routes/tasks.js
const express = require('express');
const router = express.Router();
const taskController = require('../controllers/taskController');

router.get('/', taskController.getAllTasks);
router.post('/', taskController.createTask);
router.put('/:id', taskController.updateTask);
router.delete('/:id', taskController.deleteTask);

module.exports = router;

javascript
Copiar código
// server/controllers/taskController.js
const db = require('../db');

exports.getAllTasks = (req, res) => {
    db.query('SELECT * FROM tasks', (err, results) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.json(results);
    });
};

exports.createTask = (req, res) => {
    const { title, description } = req.body;
    db.query('INSERT INTO tasks (title, description) VALUES (?, ?)', [title, description], (err, results) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.status(201).json({ id: results.insertId, title, description });
    });
};

exports.updateTask = (req, res) => {
    const { id } = req.params;
    const { title, description } = req.body;
    db.query('UPDATE tasks SET title = ?, description = ? WHERE id = ?', [title, description, id], (err, results) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.json({ id, title, description });
    });
};

exports.deleteTask = (req, res) => {
    const { id } = req.params;
    db.query('DELETE FROM tasks WHERE id = ?', [id], (err, results) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        res.status(204).send();
    });
};

Crear la base de datos y la tabla
sql
Copiar código
-- Crear la base de datos y la tabla
mysql -u root -p

CREATE DATABASE task_manager;
USE task_manager;

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT
);

Configuración del Proyecto Frontend (Vue.js)

Crear el proyecto Vue.js
bash
Copiar código
cd ..
npx @vue/cli create client

Instalar Axios para las peticiones HTTP
bash
Copiar código
npm install axios

Configurar Axios
javascript
Copiar código
// client/src/services/api.js
import axios from 'axios';

const api = axios.create({
    baseURL: 'http://localhost:3000/api'
});

export default api;

Crear componentes Vue para la gestión de tareas
vue
Copiar código
<!-- client/src/components/TaskList.vue -->
<template>
    <div>
        <h1>Task List</h1>
        <ul>
            <li v-for="task in tasks" :key="task.id">
                {{ task.title }} - {{ task.description }}
                <button @click="deleteTask(task.id)">Delete</button>
            </li>
        </ul>
    </div>
</template>

<script>
import api from '../services/api';

export default {
    data() {
        return {
            tasks: []
        };
    },
    created() {
        this.fetchTasks();
    },
    methods: {
        async fetchTasks() {
            const response = await api.get('/tasks');
            this.tasks = response.data;
        },
        async deleteTask(id) {
            await api.delete(`/tasks/${id}`);
            this.fetchTasks();
        }
    }
};
</script>

vue
Copiar código
<!-- client/src/components/AddTask.vue -->
<template>
    <div>
        <h1>Add Task</h1>
        <form @submit.prevent="addTask">
            <input v-model="title" placeholder="Title" required />
            <textarea v-model="description" placeholder="Description"></textarea>
            <button type="submit">Add Task</button>
        </form>
    </div>
</template>

<script>
import api from '../services/api';

export default {
    data() {
        return {
            title: '',
            description: ''
        };
    },
    methods: {
        async addTask() {
            await api.post('/tasks', {
                title: this.title,
                description: this.description
            });
            this.title = '';
            this.description = '';
            this.$emit('taskAdded');
        }
    }
};
</script>

Integrar los componentes en la aplicación principal
vue
Copiar código
<!-- client/src/App.vue -->
<template>
    <div id="app">
        <AddTask @taskAdded="fetchTasks"/>
        <TaskList ref="taskList"/>
    </div>
</template>

<script>
import AddTask from './components/AddTask.vue';
import TaskList from './components/TaskList.vue';

export default {
    name: 'App',
    components: {
        AddTask,
        TaskList
    },
    methods: {
        fetchTasks() {
            this.$refs.taskList.fetchTasks();
        }
    }
};
</script>

Ejecución de la Aplicación

Iniciar el servidor backend
bash
Copiar código
cd server
node index.js

Iniciar la aplicación Vue.js
bash
Copiar código
cd ../client
npm run serve

Verificar la aplicación

Abre tu navegador y navega a http://localhost:8080. Deberías ver la aplicación de gestión de tareas funcionando, donde puedes añadir, ver y eliminar tareas.

Compilar el Proyecto Vue.js e Integrarlo en Node.js

Después de desarrollar la aplicación en Vue.js, compílala y luego integra los archivos compilados en el proyecto Node.js con Express:

Compilar el proyecto Vue.js
bash
Copiar código
cd client
npm run build

Mover los archivos compilados al proyecto Node.js
bash
Copiar código
cp -r dist/* ../server/public/

Configurar el servidor Express para servir los archivos compilados
javascript
Copiar código
// server/index.js
const express = require('express');
const path = require('path');

const app = express();
app.use(express.static('public'));

app.get('*', (req, res) => {
    res.sendFile(path.resolve(__dirname, 'public', 'index.html'));
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
