##Proyecto Guía de Programación Web - Gestión de Tareas

Este proyecto es una aplicación de gestión de tareas que utiliza Vue.js para el frontend y Node.js con Express y MySQL para el backend.

## Índice

- [Requisitos Previos](#requisitos-previos)
- [Configuración del Proyecto Backend (Node.js, Express, MySQL)](#configuración-del-proyecto-backend-nodejs-express-mysql)
- [Crear la base de datos y la tabla](#crear-la-base-de-datos-y-la-tabla)
- [Configuración del Proyecto Frontend (Vue.js)](#configuración-del-proyecto-frontend-vuejs)
- [Ejecución de la Aplicación](#ejecución-de-la-aplicación)
- [Compilar el Proyecto Vue.js e Integrarlo en Node.js](#compilar-el-proyecto-vuejs-e-integrarlo-en-nodejs)
- [Configurar Express para Servir la Aplicación Vue.js](#configurar-express-para-servir-la-aplicación-vue.js)
- [Estructura Final del Proyecto](#estructura-final-del-proyecto)
## Requisitos Previos

Antes de comenzar, asegúrate de tener instalados:

1. **Node.js**
2. **MySQL**
3. **Conocimientos básicos de JavaScript, Vue.js, Node.js, Express y MySQL**

## Configuración del Proyecto Backend (Node.js, Express, MySQL)

**Crear el proyecto backend**
```bash
mkdir task-manager
cd task-manager
npm init -y
```
**Instalar dependencias necesarias**
```bash
npm install express mysql2 body-parser cors
```
**Estructura de carpetas:**
```bash
mkdir server
cd server
mkdir routes models controllers
```
**Configurar la conexión a la base de datos**

Crear un archivo "db.js" en la carpeta server: 
```bash
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
```
**Crear el servidor Express**

Crear un archivo "index.js" en la carpeta server:
```bash
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
```
**Definir las rutas y controladores**

Crear un archivo "tasks.js" en la carpeta routes:
```bash
const express = require('express');
const router = express.Router();
const taskController = require('../controllers/taskController');

router.get('/', taskController.getAllTasks);
router.post('/', taskController.createTask);
router.put('/:id', taskController.updateTask);
router.delete('/:id', taskController.deleteTask);

module.exports = router;
```
**Crear controllador**

Crea un archivo taskController.js en la carpeta controllers:
```bash
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
```
## Crear la base de datos y la tabla

Script de la Base de Datos:
```bash
CREATE DATABASE task_manager;
USE task_manager;

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT
);
```
## Configuración del Proyecto Frontend (Vue.js)

Crear el proyecto Vue.js
```bash
cd ..
npx @vue/cli create client
```
**Instalar Axios para las peticiones HTTP**
```bash
npm install axios
```
**Configurar Axios**

Crea un archivo src/services/api.js:
```bash
import axios from 'axios';

const api = axios.create({
    baseURL: 'http://localhost:3000/api'
});

export default api;
```
**Crear componentes Vue para la gestión de tareas**

Crear src/components/TaskList.vue: 
```bash
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
```

Crear src/components/AddTask.vue:
```bash
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
```

**Integrar los componentes en la aplicación principal**

En src/App.vue:
```bash
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
```

## Ejecución de la Aplicación

**Iniciar el servidor backend**
```bash
cd server
node index.js
```
**Iniciar la aplicación Vue.js**
```bash
cd ../client
npm run serve
```
**Verificar la aplicación**

Abre tu navegador y navega a http://localhost:8080. Deberías ver la aplicación de gestión de tareas funcionando, donde puedes añadir, ver y eliminar tareas.
Despues compilar en proyecto de vue para agregar en la carpeta de node (views).
## Compilar el Proyecto Vue.js e Integrarlo en Node.js

Después de desarrollar la aplicación en Vue.js, compílala y luego integra los archivos compilados en el proyecto Node.js con Express:

**Configurar la salida del build:** 

En tu proyecto Vue.js, abre el archivo vue.config.js (créalo si no existe): 
```bash
module.exports = { 
outputDir: '../server/public', 
devServer: { 
proxy: 'http://localhost:3000' 
} 
};
```
**Compilar el proyecto Vue.js**

Esto generará los archivos compilados de la aplicación Vue.js en la carpeta server/public.
```bash
cd client
npm run build
```

## Configurar Express para Servir la Aplicación Vue.js

**Modificar el servidor Express para servir archivos estáticos**

Abre index.js en la carpeta server y añade el middleware para servir archivos estáticos:
```bash
const bodyParser = require('body-parser'); 
const cors = require('cors'); 
const path = require('path'); 
const taskRoutes = require('./routes/tasks'); 
const app = express(); 
app.use(cors()); 
app.use(bodyParser.json()); 
// Configurar la carpeta pública para servir archivos estáticos 
app.use(express.static(path.join(__dirname, 'public'))); 
app.use('/api/tasks', taskRoutes); 
// Configurar el enrutamiento de Vue.js 
app.get('*', (req, res) => { 
res.sendFile(path.join(__dirname, 'public', 'index.html')); 
}); 
const PORT = process.env.PORT || 3000; 
app.listen(PORT, () => { 
console.log(`Server is running on port ${PORT}`); 
});
```
**Verificar la Integración**

Iniciar el servidor backend:
```bash
cd server 
node index.js
```
**Acceder a la aplicación:**

Abre tu navegador y navega a http://localhost:3000. Deberías ver la aplicación Vue.js 
sirviendo desde tu servidor Express.

## Estructura Final del Proyecto

La estructura final de tu proyecto debería verse algo así:

![image](https://github.com/JorellStudent/ProyectoGuiaDeProgramacionWeb/assets/167504858/ab8d9fa0-a620-4e87-ab1a-c26064b0b1a1)

