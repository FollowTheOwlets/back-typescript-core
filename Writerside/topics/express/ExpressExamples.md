# Примеры проектов

Примеры pet-проектов. Простые варианты использования express

## MVC Rest
MVC (Model-View-Controller) — это шаблон проектирования, который используется для построения структуры веб-приложений.

- **Модель (Model)**: Отвечает за данные и бизнес-логику. В данном примере, модель представлена классом `User`, который описывает сущность пользователя.

- **Представление (View)**: В контексте Express, представление обычно генерируется с использованием шаблонизаторов, но в этом примере мы ограничимся RESTful API, и представлением будут JSON-ответы.

- **Контроллер (Controller)**: Принимает запросы от клиента, взаимодействует с моделью для получения/изменения данных и возвращает результат клиенту. В этом примере, контроллер `userController` обрабатывает запросы для пользователей.

- **Сервис (Service)**: Предоставляет слой абстракции для бизнес-логики, чтобы контроллеры оставались тонкими. Здесь `userService` предоставляет методы для взаимодействия с данными пользователей.

- **Репозиторий (Repository)**: Отвечает за взаимодействие с данными. Здесь `userRepository` работает с файловой системой для хранения данных о пользователях.

Важно отметить, что структура и организация зависят от конкретных требований проекта, и приведенный пример можно адаптировать под нужды вашего приложения.

### 1. Создание структуры проекта {id="1_1"}

```text
/project
  /controllers
    userController.js
  /models
    user.js
  /routes
    userRoutes.js
  /services
    userService.js
  /repositories
    userRepository.js
  app.js
  package.json
  .env
```

### 2. Реализация модели

```javascript
// models/user.js
class User {
  constructor(id, username, email) {
    this.id = id;
    this.username = username;
    this.email = email;
  }
}

module.exports = User;
```

### 3. Реализация репозитория

```javascript
// repositories/userRepository.js
const fs = require('fs');
const path = require('path');

const dataFilePath = path.join(__dirname, '../data/users.json');

class UserRepository {
  static getAllUsers() {
    const rawData = fs.readFileSync(dataFilePath);
    let result = [];
    try {
      result = JSON.parse(rawData);
    } catch (e) { }

    return result;
  }

  static getUserById(userId) {
    const allUsers = this.getAllUsers();
    return allUsers.find(user => user.id === userId);
  }

  static getUserByEmail(email) {
    const allUsers = this.getAllUsers();
    return allUsers.find(user => user.email === email);
  }

  static addUser(user) {
    const allUsers = this.getAllUsers();
    user.id = allUsers.length > 0 ? Math.max(...allUsers.map(u => u.id)) + 1 : 1;
    allUsers.push(user);
    fs.writeFileSync(dataFilePath, JSON.stringify(allUsers, null, 2));
    return user;
  }

  static updateUser(userId, updatedUserData) {
    const allUsers = this.getAllUsers();
    const userIndex = allUsers.findIndex(user => user.id === userId);

    if (userIndex !== -1) {
      allUsers[userIndex] = { ...allUsers[userIndex], ...updatedUserData };
      fs.writeFileSync(dataFilePath, JSON.stringify(allUsers, null, 2));
      return allUsers[userIndex];
    }

    return null;
  }

  static deleteUser(userId) {
    const allUsers = this.getAllUsers();
    const updatedUsers = allUsers.filter(user => user.id !== userId);
    fs.writeFileSync(dataFilePath, JSON.stringify(updatedUsers, null, 2));
  }
}

module.exports = UserRepository;
```

`UserRepository` поддерживает следующие операции:

- `getAllUsers`: Возвращает массив всех пользователей.
- `getUserById`: Возвращает пользователя по идентификатору.
- `getUserByEmail`: Возвращает пользователя по email.
- `addUser`: Добавляет нового пользователя.
- `updateUser`: Обновляет информацию о пользователе по идентификатору.
- `deleteUser`: Удаляет пользователя по идентификатору.

Пример использования:

```javascript
// Пример использования UserRepository
const UserRepository = require('../repositories/userRepository');

// Получение всех пользователей
const allUsers = UserRepository.getAllUsers();
console.log(allUsers);

// Получение пользователя по идентификатору
const userById = UserRepository.getUserById(1);
console.log(userById);

// Добавление нового пользователя
const newUser = UserRepository.addUser({ username: 'john_doe', email: 'john@example.com' });
console.log(newUser);

// Обновление информации о пользователе
const updatedUser = UserRepository.updateUser(1, { email: 'new_email@example.com' });
console.log(updatedUser);

// Удаление пользователя
UserRepository.deleteUser(1);
```

Эти методы позволяют вам взаимодействовать с данными о пользователях, используя CRUD операции. Помните, что в реальном приложении, особенно в production, вы, вероятно, захотите добавить больше обработки ошибок и валидации данных.

### 4. Реализация сервиса

```javascript
// services/userService.js
const UserRepository = require('../repositories/userRepository');

class UserService {
  static getAllUsers() {
    return UserRepository.getAllUsers();
  }

  static getUserById(userId) {
    return UserRepository.getUserById(userId);
  }

  static getUserByEmail(email) {
    return UserRepository.getUserByEmail(email);
  }

  static addUser(userData) {
    // Внутренняя логика, например, валидация данных
    if (!userData.username || !userData.email) {
      throw new Error('Username and email are required');
    }

    return UserRepository.addUser(userData);
  }

  static updateUser(userId, updatedUserData) {
    // Внутренняя логика, например, проверка наличия пользователя
    const existingUser = UserRepository.getUserById(userId);
    if (!existingUser) {
      throw new Error('User not found');
    }

    return UserRepository.updateUser(userId, updatedUserData);
  }

  static deleteUser(userId) {
    // Внутренняя логика, например, проверка наличия пользователя перед удалением
    const existingUser = UserRepository.getUserById(userId);
    if (!existingUser) {
      throw new Error('User not found');
    }

    UserRepository.deleteUser(userId);
  }

  // Дополнительные методы с внутренней логикой
  static findUsersByEmail(domain) {
    const allUsers = UserRepository.getAllUsers();
    return allUsers.filter(user => user.email.endsWith(`@${domain}`));
  }

  static generateRandomUsername() {
    const randomSuffix = Math.floor(Math.random() * 1000);
    return `user_${randomSuffix}`;
  }
}

module.exports = UserService;
```

Пример использования:

```javascript
// Пример использования UserService
const UserService = require('../services/userService');

// Получение всех пользователей
const allUsers = UserService.getAllUsers();
console.log(allUsers);

// Получение пользователя по идентификатору
const userById = UserService.getUserById(1);
console.log(userById);

// Добавление нового пользователя
try {
  const newUser = UserService.addUser({ username: 'john_doe', email: 'john@example.com' });
  console.log(newUser);
} catch (error) {
  console.error(error.message);
}

// Обновление информации о пользователе
try {
  const updatedUser = UserService.updateUser(1, { email: 'new_email@example.com' });
  console.log(updatedUser);
} catch (error) {
  console.error(error.message);
}

// Удаление пользователя
try {
  UserService.deleteUser(1);
} catch (error) {
  console.error(error.message);
}

// Дополнительные методы
const usersWithDomain = UserService.findUsersByEmailDomain('example.com');
console.log(usersWithDomain);

const randomUsername = UserService.generateRandomUsername();
console.log(randomUsername);
```

Эти методы демонстрируют, как вы можете расширить функциональность `UserService`, добавляя внутреннюю логику и дополнительные операции.

### 5. Реализация контроллера

```javascript
// controllers/userController.js
const express = require('express');
const router = express.Router();
const UserService = require('../services/userService');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

router.use(express.json());

// Пример middleware для проверки наличия JWT в заголовке Authorization
const authenticateToken = (req, res, next) => {
  const token = req.header('Authorization');
  if (!token) return res.sendStatus(401);

  jwt.verify(token, process.env.SECRET_KEY, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};

// Регистрация пользователя и выдача JWT
router.post('/register', async (req, res) => {
  try {
    const { username, email, password } = req.body;

    // Валидация данных
    if (!username || !email || !password) {
      return res.status(400).json({ error: 'All fields are required' });
    }

    // Проверка на уникальность email
    const existingUser = UserService.getUserByEmail(email);

    if (existingUser) {
      return res.status(400).json({ error: 'Email is already registered' });
    }

    // Хеширование пароля
    const hashedPassword = await bcrypt.hash(password, 10);

    // Создание нового пользователя
    const newUser = UserService.addUser({ username, email, password: hashedPassword });

    // Создание JWT
    const token = jwt.sign({ id: newUser.id, username: newUser.username }, process.env.SECRET_KEY);

    res.json({ user: newUser, token });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// Авторизация пользователя и выдача JWT
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Поиск пользователя по email
    const user = UserService.getUserByEmail(email);

    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Проверка пароля
    const passwordMatch = await bcrypt.compare(password, user.password);

    if (!passwordMatch) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Создание JWT
    const token = jwt.sign({ id: user.id, username: user.username }, process.env.SECRET_KEY);

    res.json({ user, token });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

router.get('/', authenticateToken, (req, res) => {
  const users = UserService.getAllUsers();
  res.json(users);
});

router.post('/', (req, res) => {
  const { username, email } = req.body;

  try {
    const newUser = UserService.addUser({ username, email });
    res.json(newUser);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

router.get('/:userId', authenticateToken, (req, res) => {
  const userId = parseInt(req.params.userId);
  const user = UserService.getUserById(userId);

  if (user) {
    res.json(user);
  } else {
    res.status(404).json({ error: 'User not found' });
  }
});

router.put('/:userId', authenticateToken, (req, res) => {
  const userId = parseInt(req.params.userId);
  const { email } = req.body;

  try {
    const updatedUser = UserService.updateUser(userId, { email });
    if (updatedUser) {
      res.json(updatedUser);
    } else {
      res.status(404).json({ error: 'User not found' });
    }
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

router.delete('/:userId', authenticateToken, (req, res) => {
  const userId = parseInt(req.params.userId);

  try {
    UserService.deleteUser(userId);
    res.sendStatus(204);
  } catch (error) {
    res.status(404).json({ error: 'User not found' });
  }
});

module.exports = router;
```

Этот `userController`:

- Использует `express.json()` для обработки JSON-тела запроса.
- Включает middleware `authenticateToken`, который проверяет наличие и валидность JWT в заголовке `Authorization`.
- Использует простое хеширование пароля с помощью `bcrypt`` и сохранение данных в памяти.

Пример использования JWT:

```javascript
// Пример создания JWT при успешной аутентификации
const user = { username: 'john_doe', id: 1 };
const token = jwt.sign(user, secretKey);
```

Теперь каждый запрос, который использует middleware `authenticateToken`, должен предоставлять в заголовке `Authorization` валидный JWT.
Это пример, и в реальном приложении вам, возможно, захочется использовать более сложные механизмы аутентификации и управления JWT.

### 6. Роутинг

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.use('/users', userController);

module.exports = router;
```

### 7. Интеграция роутов в приложение

```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/userRoutes');
require('dotenv').config(); // Загрузка переменных среды

const app = express();
const port = process.env.PORT || 3000;

// Пример middleware для логгирования запросов
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use('/api', userRoutes);

// Обработка ошибок 404
app.use((req, res, next) => {
  res.status(404).json({ error: 'Not Found' });
});

// Обработка ошибок 500
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

Обратите внимание на следующее в `app.js`:

- Добавили простой middleware для логгирования запросов.
- Подключили роуты для пользователей с префиксом `/api/users`.
- Добавили middleware для обработки ошибок 404 и 500.

Теперь вы можете запустить приложение с помощью `node app.js` и оно будет слушать запросы на `http://localhost:3000`.

В этом примере у нас есть базовая структура Express-приложения с обработкой запросов пользователей, аутентификацией с использованием JWT, обработкой ошибок и middleware для логгирования запросов. Эту структуру можно дополнять и расширять в зависимости от конкретных требований вашего приложения.

### 8. Переменные среды

Файл `.env` (Environment) — это файл конфигурации, который содержит переменные среды вашего приложения. В контексте Express.js, файл `.env` часто используется для хранения конфиденциальной информации, такой как секретные ключи, URL базы данных и другие параметры, которые не должны попасть в репозиторий.

Вот как это работает:

#### Шаг 1: Установка пакета `dotenv`

```bash
npm install dotenv
```

#### Шаг 2: Создание файла `.env`

В корне вашего проекта создайте файл с именем `.env` и добавьте в него переменные среды в формате `KEY=VALUE`. Пример:

```text
PORT=3000
DB_URL=mongodb://localhost:27017/mydatabase
SECRET_KEY=mysecretkey
```

#### Шаг 3: Загрузка переменных среды в Express приложение

```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/userRoutes');
require('dotenv').config(); // Загрузка переменных среды

const app = express();
const port = process.env.PORT || 3000;

// ...

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

Теперь переменные из файла `.env` будут доступны в приложении через `process.env`.

#### Пример использования переменных среды в коде:

```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/userRoutes');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 3000;
const secretKey = process.env.SECRET_KEY || 'mysecretkey';

// ...

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

#### Важные замечания:

1. **Не добавляйте `.env` в репозиторий.** Этот файл должен быть включен в файл `.gitignore`, чтобы избежать случайного попадания конфиденциальной информации в репозиторий.
2. **Не храните в `.env` секретные ключи или конфиденциальную информацию в открытом доступе.** Предоставляйте шаблон `.env.example` с образцами переменных, и оставляйте только необходимые значения в `.env`.
3. **Перезапустите приложение после изменения `.env`.** Изменения в файле `.env` не вступают в силу, пока вы не перезапустите сервер.
4. **Используйте `.env` только для переменных среды.** Не храните в `.env` какие-либо другие файлы или конфигурации.

[исходный код rest сервиса](https://github.com/FollowTheOwlets/nodejs-options/rest%20api%20mvc/)

## MVC Web Application

Раберем простой пример - разработку приложения `To-Do-List` с использование [PostgreSQL](https://www.postgresql.org) в качестве хранилища.

### 1: Настройка проекта

1.1 **Создание нового проекта:**

```bash
# Инициализируем проект npm
npm init -y
```

1.2 **Установка зависимостей:**

```bash
# Установка Express для создания веб-приложения
npm i express pug sequelize pg sequelize-cli dotenv bootstrap
```

1.3 **Настройка базы данных:**

Файл `.sequelizerc` - это файл конфигурации, используемый Sequelize CLI (Command Line Interface), чтобы определить структуру проекта и пути к различным компонентам Sequelize, таким как модели, миграции и сидеры. Он позволяет настраивать организацию вашего проекта на основе Sequelize.

```javascript
// .sequelizerc

const path = require('path');

module.exports = {
  'config': path.resolve('config', 'database.js'),
  'models-path': path.resolve('models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations')
};
```

- **`config`**: Эта строка указывает путь к файлу конфигурации Sequelize. В данном примере она ссылается на `config/database.js`, где хранятся настройки вашей базы данных.

- **`models-path`**: Здесь определен каталог, в котором будут храниться файлы моделей Sequelize. В примере он установлен в `models`, так что файлы моделей следует помещать в каталог `models`.

- **`seeders-path`**: Эта строка устанавливает каталог для файлов сидеров. Сидеры используются для заполнения вашей базы данных начальными данными. В примере используется каталог `seeders`.

- **`migrations-path`**: Здесь указан каталог для файлов миграций. Миграции используются для изменения схемы базы данных со временем. В примере используется каталог `migrations`.

При выполнении команд Sequelize CLI просматривает этот файл конфигурации, чтобы определить, где искать и хранить различные типы файлов.

Убедитесь, что пути соответствуют фактической структуре вашего проекта. Если вам нужно впоследствии внести изменения в структуру, вы можете соответствующим образом обновить пути в файле `.sequelizerc`.
Теперь создайте каталог `config` и файл `database.js` внутри него:

Теперь перейдем к настройкам соединения с БД, обратите внимание, что большая часть данных берется из переменных окружения.
```javascript
// config/database.js
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres'
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres'
  },
  production: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres'
  }
};
```

Не забудьте создать файл `.env` в корне проекта и добавить настройки для вашей базы данных:

```shell
DB_USER=ваше_имя_пользователя
DB_PASSWORD=ваш_пароль
DB_NAME=ваша_база_данных
DB_HOST=localhost
```

### 2: Создание модели с использованием Sequelize

2.1 **Создание модели Task:**

Создадим каталог `models` и внутри него файл `task.js`:

```javascript
// models/task.js
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Task = sequelize.define('Task', {
  title: {
    type: DataTypes.STRING,
    allowNull: false
  },
  completed: {
    type: DataTypes.BOOLEAN,
    defaultValue: false
  }
});

module.exports = Task;
```

2.2 **Синхронизация с базой данных:**

Файл `sync-db.js` в корне проекта используется для синхронизации модели с базой данных:

```javascript
// sync-db.js
const sequelize = require('./config/database');
const Task = require('./models/task');

async function syncDatabase() {
  try {
    await sequelize.sync({ force: true });
    console.log('Database synchronized successfully.');
  } catch (error) {
    console.error('Error synchronizing database:', error);
  } finally {
    process.exit();
  }
}

syncDatabase();
```

2.3 **Запуск синхронизации базы данных:**

```bash
node sync-db.js
```

Теперь базовая настройка проекта завершена и модель Task, связанная с базой данных, сконфигурирована.

### 3: TaskService

Сервис, предоставляющий интерфейс для взаимодействия с репозиторием Task, предоставленным sequelize выглядит следующим образом. Да, в данном кейсе модель содержит в себе API репозитория.

### `services/taskService.js`

```javascript
const Task = require('../models/task');

class TaskService {
  // Получение всех задач
  static async getAllTasks() {
    try {
      return await Task.findAll();
    } catch (error) {
      throw error;
    }
  }

  // Добавление новой задачи
  static async addTask(title) {
    try {
      return await Task.create({ title });
    } catch (error) {
      throw error;
    }
  }

  // Переключение статуса выполнения задачи
  static async toggleTaskCompletion(taskId) {
    try {
      const task = await Task.findByPk(taskId);
      if (task) {
        task.completed = !task.completed;
        await task.save();
      }
    } catch (error) {
      throw error;
    }
  }

  // Удаление задачи по идентификатору
  static async deleteTask(taskId) {
    try {
      await Task.destroy({ where: { id: taskId } });
    } catch (error) {
      throw error;
    }
  }

  // Получение задачи по идентификатору
  static async getTaskById(taskId) {
    try {
      return await Task.findByPk(taskId);
    } catch (error) {
      throw error;
    }
  }

  // Обновление задачи по идентификатору
  static async updateTask(taskId, title) {
    try {
      const task = await Task.findByPk(taskId);
      if (task) {
        task.title = title;
        await task.save();
      }
    } catch (error) {
      throw error;
    }
  }
}

module.exports = TaskService;
```

### 4: TaskController

Контроллер, отвечающий за обработку запросов:

```javascript
const express = require('express');
const TaskService = require('../services/taskService');

const router = express.Router();

router.get('/', async (req, res) => {
  try {
    // Получение всех задач и отображение на главной странице
    const tasks = await TaskService.getAllTasks();
    res.render('index', { tasks });
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

router.post('/add', async (req, res) => {
  try {
    // Добавление новой задачи и перенаправление на главную страницу
    const { title } = req.body;
    await TaskService.addTask(title);
    res.redirect('/');
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

router.get('/complete/:taskId', async (req, res) => {
  try {
    // Переключение статуса выполнения задачи и перенаправление на главную страницу
    const taskId = req.params.taskId;
    await TaskService.toggleTaskCompletion(taskId);
    res.redirect('/');
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

router.get('/delete/:taskId', async (req, res) => {
  try {
    // Удаление задачи и перенаправление на главную страницу
    const taskId = req.params.taskId;
    await TaskService.deleteTask(taskId);
    res.redirect('/');
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

router.get('/edit/:taskId', async (req, res) => {
  try {
    // Получение задачи для редактирования и отображение формы
    const taskId = req.params.taskId;
    const task = await TaskService.getTaskById(taskId);
    if (task) {
      res.render('edit', { task });
    } else {
      res.render('not_found');
    }
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

router.post('/edit/:taskId', async (req, res) => {
  try {
    // Обновление задачи и перенаправление на главную страницу
    const taskId = req.params.taskId;
    const { title } = req.body;
    await TaskService.updateTask(taskId, title);
    res.redirect('/');
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

module.exports = router;
```

### 5. Express приложение

```javascript
// Подключение необходимых модулей и файлов
const express = require('express'); // Express.js для работы с веб-приложением
const path = require('path'); // Модуль для работы с путями файловой системы
const taskController = require('./controllers/taskController'); // Контроллер для обработки маршрутов
const sequelize = require('./config/database'); // Подключение к базе данных с использованием Sequelize

// Создание экземпляра приложения Express
const app = express();

// Установка пути для представлений и шаблонизатора Pug
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

// Настройка обработки данных из формы
app.use(express.urlencoded({ extended: true }));

// Настройка обработки статических файлов (CSS, изображения и др.)
app.use(express.static(path.join(__dirname, 'public')));

// Использование контроллера для маршрутов, начинающихся с корневого пути '/'
app.use('/', taskController);

// Синхронизация с базой данных и запуск сервера после успешной синхронизации
sequelize.sync().then(() => {
    // Запуск сервера на указанном порту
    const port = process.env.PORT || 3000;
    app.listen(port, () => {
        console.log(`Server is running on port ${port}`);
    });
});

```

### 6. Views
Для работы приложения понадобится 3 шаблона pug (inde, edit, not_found) и 1 css файл:

```text
// views/index.pug

// Определение типа документа и языка
doctype html
html(lang="en")
  head
    // Мета-теги для правильной кодировки и масштабирования страницы
    meta(charset="UTF-8")
    meta(name="viewport", content="width=device-width, initial-scale=1.0")
    // Заголовок страницы
    title To-Do List
    // Подключение стилей Bootstrap и Material Icons, а также пользовательского стиля
    link(rel="stylesheet", href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css")
    link(rel="stylesheet", href="https://fonts.googleapis.com/icon?family=Material+Icons")
    link(rel="stylesheet", href="/style.css")

  body
    // Контейнер для выравнивания и отступа от верхней части страницы
    .container.mt-5
      // Строка с колонкой с отступом и шириной 6/12
      .row
        .col-md-6.offset-md-3
          // Карточка Bootstrap
          .card
            // Заголовок карточки
            .card-header
              h1.text-center To-Do List
            // Тело карточки
            .card-body
              // Форма для добавления новой задачи
              form(action="/add" method="post" class="mb-3")
                // Группа элементов ввода для стилизации с Bootstrap
                .input-group
                  // Поле ввода с именем "title" для новой задачи
                  input(type="text" name="title" placeholder="Add a new task..." required class="form-control")
                  // Кнопка для отправки формы
                  .input-group-append
                    button(type="submit" class="btn btn-primary") Add Task
              // Список задач
              ul.list-group
                // Цикл для вывода каждой задачи в виде элемента списка
                each task in tasks
                  li.list-group-item.d-flex.align-items-center.py-2
                    // Див для отображения названия задачи
                    div( class=`task`) #{task.title}
                    // Ссылка для пометки задачи как выполненной
                    a.btn.btn-secondary.btn-sm.ml-auto(href=`/complete/${task.id}`)
                        // Иконка в зависимости от статуса выполнения
                        if (task.completed)
                            i.material-icons.md-18.done done
                        else
                            i.material-icons.md-18.cls done
                    // Ссылка для редактирования задачи
                    a.btn.btn-warning.btn-sm.ml-2(href=`/edit/${task.id}`)
                      i.material-icons.md-18 edit
                    // Ссылка для удаления задачи
                    a.btn.btn-danger.btn-sm.ml-2(href=`/delete/${task.id}`)
                      i.material-icons.md-18 delete
```

```text
// views/edit.pug
doctype html
html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport", content="width=device-width, initial-scale=1.0")
    title Edit Task
    link(rel="stylesheet", href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css")
    link(rel="stylesheet", href="/style.css")
  body
    .container.mt-5
      .row
        .col-md-6.offset-md-3
          .card
            .card-header
              h1.text-center Edit Task
            .card-body
              form(action=`/edit/${task.id}` method="post" class="mb-3")
                .input-group
                  input(type="text" name="title" value=task.title required class="form-control")
                  .input-group-append
                    button(type="submit" class="btn btn-primary") Save Changes
              a.btn.btn-danger.btn-sm(href="/") Cancel

```

```text
// views/not_found.pug
doctype html
html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport", content="width=device-width, initial-scale=1.0")
    title 404 Not Found
    link(rel="stylesheet", href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css")

  body
    .container.mt-5
      .row
        .col-md-6.offset-md-3
          .text-center
            h1 404
            p Task not found
            a.btn.btn-primary(href="/") home page
```

```css
/* public/style.css */
body {
    font-family: Arial, sans-serif;
}

h1 {
    color: #333;
}

form {
    margin-bottom: 20px;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
}

.material-icons {
    font-size: 14px;
}

.btn {
    display: flex;
    align-items: center;
    justify-content: center;
}

.btn:has(.done) {
    background-color: darkgreen;
}

.btn:has(.cls) {
    background-color: white;
    color: #333;
}

.btn-group-sm>.btn,
.btn-sm {
    padding: 0.5rem 0.5rem;
    border-radius: 10rem;
}

.task {
    max-width: 60%;
}
```

В этом примере была разобрана разработка монолитного Express-приложения с использованием шаблонизатора Pug. Эту структуру можно дополнять и расширять в зависимости от конкретных требований вашего приложения.

[исходный код](https://github.com/FollowTheOwlets/nodejs-options/web%20application%20mvc/)