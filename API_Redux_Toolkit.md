# Упражнения по Redux Toolkit

## 🎯 Упражнение 1: extraReducers (реакция на чужие экшены)

### Задание: Счётчик, который реагирует на logout

**Цель:** Понять, как один слайс может реагировать на экшены другого слайса.

### Начальный код

```javascript
// authSlice.js
import { createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
  name: 'auth',
  initialState: { isLoggedIn: true, userName: 'Анна' },
  reducers: {
    login: (state) => { state.isLoggedIn = true; },
    logout: (state) => { state.isLoggedIn = false; }
  }
});

export const { login, logout } = authSlice.actions;
export default authSlice.reducer;
```

### 🟢 Уровень 1: Простое задание

Допишите `counterSlice.js`, чтобы он **обнулял счётчик** при logout:

```javascript
// counterSlice.js (нужно дополнить)
import { createSlice } from '@reduxjs/toolkit';
import { logout } from './authSlice'; // ← экшен из другого слайса

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 10 },
  reducers: {
    increment: (state) => { state.value++; },
    decrement: (state) => { state.value--; },
    reset: (state) => { state.value = 0; }
  },
  extraReducers: (builder) => {
    // ✍️ Напишите здесь: при logout обнулять value
    // Подсказка: builder.addCase(logout, (state) => { ... })
  }
});

export const { increment, decrement, reset } = counterSlice.actions;
export default counterSlice.reducer;
```

**Ожидаемый результат:** При вызове `dispatch(logout())` счётчик должен стать 0.

<details>
<summary>📖 Показать решение</summary>

```javascript
extraReducers: (builder) => {
  builder.addCase(logout, (state) => {
    state.value = 0;
  });
}
```

</details>

---

### 🟡 Уровень 2: Сложнее

Сделайте так, чтобы при logout:
1. Счётчик обнулялся
2. Добавлялось поле `lastReset: Date.now()`

```javascript
// Подсказка: добавьте новое поле в initialState
initialState: { 
  value: 10, 
  lastReset: null 
}
```

<details>
<summary>📖 Показать решение</summary>

```javascript
initialState: { 
  value: 10, 
  lastReset: null 
},

extraReducers: (builder) => {
  builder.addCase(logout, (state) => {
    state.value = 0;
    state.lastReset = Date.now();
  });
}
```

</details>

---

## 🎯 Упражнение 2: createAsyncThunk (асинхронные операции)

### Задание: Загрузка шуток с API

**Цель:** Научиться использовать `createAsyncThunk` для загрузки данных.

### Бесплатное API для тестов: `https://api.chucknorris.io/jokes/random`

### 🟢 Уровень 1: Простая загрузка

Создайте слайс для загрузки случайной шутки:

```javascript
// jokesSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// ✍️ Напишите createAsyncThunk для загрузки шутки
// API: https://api.chucknorris.io/jokes/random
// Нужно вернуть data.value (текст шутки)

export const fetchJoke = createAsyncThunk(
  'jokes/fetchJoke',
  async () => {
    // ✍️ Напишите fetch запрос здесь
    // Подсказка: const response = await fetch('https://api.chucknorris.io/jokes/random');
    // const data = await response.json();
    // return data.value;
  }
);

const jokesSlice = createSlice({
  name: 'jokes',
  initialState: {
    joke: 'Нажмите кнопку, чтобы получить шутку',
    status: 'idle', // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    // ✍️ Обработайте три состояния: pending, fulfilled, rejected
    // pending: status = 'loading'
    // fulfilled: status = 'succeeded', joke = action.payload
    // rejected: status = 'failed', error = action.error.message
  }
});

export default jokesSlice.reducer;
```

<details>
<summary>📖 Показать решение</summary>

```javascript
export const fetchJoke = createAsyncThunk(
  'jokes/fetchJoke',
  async () => {
    const response = await fetch('https://api.chucknorris.io/jokes/random');
    const data = await response.json();
    return data.value;
  }
);

const jokesSlice = createSlice({
  name: 'jokes',
  initialState: {
    joke: 'Нажмите кнопку, чтобы получить шутку',
    status: 'idle',
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchJoke.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchJoke.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.joke = action.payload;
      })
      .addCase(fetchJoke.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  }
});
```

</details>

---


### 🟡 Уровень 2: С обработкой ошибок

Добавьте кнопку "Обновить" в компонент, которая показывает статус загрузки:

```jsx
// JokesComponent.jsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchJoke } from './jokesSlice';

export const JokesComponent = () => {
  const dispatch = useDispatch();
  const { joke, status, error } = useSelector(state => state.jokes);
  
  // ✍️ Напишите JSX:
  // - Если status === 'loading', показать "Загрузка..."
  // - Если status === 'failed', показать ошибку и кнопку "Попробовать снова"
  // - Показать шутку и кнопку "Новая шутка"

  return (
    <div>
      {/* Напишите код здесь */}
    </div>
  );
};
```

<details>
<summary>📖 Показать решение</summary>

```jsx
export const JokesComponent = () => {
  const dispatch = useDispatch();
  const { joke, status, error } = useSelector(state => state.jokes);
  
  if (status === 'loading') {
    return <div>⏳ Загрузка шутки...</div>;
  }

  if (status === 'failed') {
    return (
      <div>
        <p>❌ Ошибка: {error}</p>
        <button onClick={() => dispatch(fetchJoke())}>
          Попробовать снова
        </button>
      </div>
    );
  }
  
  return (
    <div>
      <p>📢 {joke}</p>
      <button onClick={() => dispatch(fetchJoke())}>
        🃏 Новая шутка
      </button>
    </div>
  );
};
```

</details>

---

### 🔴 Уровень 3: С параметром

Создайте Thunk, который принимает **категорию** шутки:

```javascript
// API: https://api.chucknorris.io/jokes/random?category={category}
// Категории: animal, career, celebrity, dev, explicit, fashion, food, history, money, movie, music, political, religion, science, sport, travel

export const fetchJokeByCategory = createAsyncThunk(
  'jokes/fetchJokeByCategory',
  async (category) => {
    // ✍️ Напишите запрос с параметром category
    // Подсказка: `https://api.chucknorris.io/jokes/random?category=${category}`
  }
);
```

<details>
<summary>📖 Показать решение</summary>

```javascript
export const fetchJokeByCategory = createAsyncThunk(
  'jokes/fetchJokeByCategory',
  async (category) => {
    const response = await fetch(`https://api.chucknorris.io/jokes/random?category=${category}`);
    const data = await response.json();
    return data.value;
  }
);
```

</details>

---

## 🎯 Упражнение 3: Сохранение в localStorage

### Задание: Счётчик, который запоминает значение

### 🟢 Уровень 1: Сохранение при изменении

```javascript
// counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const STORAGE_KEY = 'counter_value';

// ✍️ Напишите функцию loadFromStorage()
// Должна возвращать число из localStorage или 0

const loadFromStorage = () => {
  // ✍️ Ваш код здесь
};

// ✍️ Напишите функцию saveToStorage(value)
// Должна сохранять число в localStorage

const saveToStorage = (value) => {
  // ✍️ Ваш код здесь
};

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: loadFromStorage() // ← загружаем сохранённое значение
  },
  reducers: {
    increment: (state) => {
      state.value++;
      saveToStorage(state.value); // ← сохраняем
    },
    decrement: (state) => {
      state.value--;
      saveToStorage(state.value); // ← сохраняем
    },
    reset: (state) => {
      state.value = 0;
      saveToStorage(0); // ← сохраняем
    }
  }
});

export const { increment, decrement, reset } = counterSlice.actions;
export default counterSlice.reducer;
```

<details>
<summary>📖 Показать решение</summary>

```javascript
const loadFromStorage = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : 0;
  } catch {
    return 0;
  }
};

const saveToStorage = (value) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value));
  } catch (error) {
    console.error('Ошибка сохранения:', error);
  }
};
```

</details>

---

### 🟡 Уровень 2: Сохранение всего состояния

Сохраняйте не только значение, но и другие поля (например, шаг изменения):

```javascript
const initialState = {
  value: 0,
  step: 1,      // на сколько увеличивать
  maxHistory: 10 // максимальное количество сохранённых значений
};

// ✍️ Сохраняйте и загружайте всё состояние целиком
```

<details>
<summary>📖 Показать решение</summary>

```javascript
const STORAGE_KEY = 'counter_full_state';

const loadFromStorage = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      return JSON.parse(saved);
    }
  } catch (error) {
    console.error('Ошибка загрузки:', error);
  }
  return { value: 0, step: 1, maxHistory: 10 };
};

const saveToStorage = (state) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  } catch (error) {
    console.error('Ошибка сохранения:', error);
  }
};

// В каждом редюсере:
saveToStorage(state);
```

</details>

---

### 🔴 Уровень 3: История изменений

Сохраняйте **историю последних 5 значений**:

```javascript
initialState: {
  current: 0,
  history: []  // массив последних значений
}

// При каждом изменении добавляем текущее значение в историю
// history.push(state.current)
// Если history.length > 5, удаляем первый элемент
```

<details>
<summary>📖 Показать решение</summary>

```javascript
const STORAGE_KEY = 'counter_with_history';

const loadFromStorage = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      return JSON.parse(saved);
    }
  } catch (error) {
    console.error('Ошибка загрузки:', error);
  }
  return { current: 0, history: [] };
};

const saveToStorage = (state) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify({
      current: state.current,
      history: state.history
    }));
  } catch (error) {
    console.error('Ошибка сохранения:', error);
  }
};

const counterSlice = createSlice({
  name: 'counter',
  initialState: loadFromStorage(),
  reducers: {
    increment: (state) => {
      state.history.push(state.current);
      if (state.history.length > 5) state.history.shift();
      state.current++;
      saveToStorage(state);
    },
    decrement: (state) => {
      state.history.push(state.current);
      if (state.history.length > 5) state.history.shift();
      state.current--;
      saveToStorage(state);
    }
  }
});
```

</details>

---

## 🎯 Упражнение 4: Комбинация (Thunk + localStorage)

### Задание: Загрузка и кэширование шуток

**Цель:** Сохранять загруженные шутки в localStorage, чтобы при перезагрузке они не исчезали.

### 🟢 Уровень 1: Кэширование последней шутки

```javascript
// jokesSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

const STORAGE_KEY = 'last_joke';

// ✍️ Загружаем последнюю шутку из localStorage
const loadLastJoke = () => {
  // Вернуть сохранённую шутку или null
};

// ✍️ Сохраняем шутку в localStorage
const saveLastJoke = (joke) => {
  // Сохранить joke в localStorage
};

export const fetchJoke = createAsyncThunk(
  'jokes/fetchJoke',
  async () => {
    const response = await fetch('https://api.chucknorris.io/jokes/random');
    const data = await response.json();
    return data.value;
  }
);

const jokesSlice = createSlice({
  name: 'jokes',
  initialState: {
    joke: loadLastJoke() || 'Нажмите кнопку',
    status: 'idle'
  },
  reducers: {},
  extraReducers: (builder) => {
    builder.addCase(fetchJoke.fulfilled, (state, action) => {
      state.joke = action.payload;
      saveLastJoke(action.payload); // ← сохраняем при успешной загрузке
    });
  }
});
```

<details>
<summary>📖 Показать решение</summary>

```javascript
const loadLastJoke = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : null;
  } catch {
    return null;
  }
};

const saveLastJoke = (joke) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(joke));
  } catch (error) {
    console.error('Ошибка сохранения:', error);
  }
};
```

</details>

---

### 🟡 Уровень 2: Кэширование нескольких шуток

Сохраняйте **историю из 5 последних шуток**:

```javascript
initialState: {
  jokes: [],        // массив шуток
  status: 'idle'
}

// При загрузке новой шутки добавляем её в начало массива
// Если шуток больше 5, удаляем последнюю
// Сохраняем весь массив в localStorage
```

<details>
<summary>📖 Показать решение</summary>

```javascript
const STORAGE_KEY = 'jokes_history';

const loadJokesHistory = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : [];
  } catch {
    return [];
  }
};

const saveJokesHistory = (jokes) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(jokes));
  } catch (error) {
    console.error('Ошибка сохранения:', error);
  }
};

const jokesSlice = createSlice({
  name: 'jokes',
  initialState: {
    jokes: loadJokesHistory(),
    status: 'idle'
  },
  extraReducers: (builder) => {
    builder.addCase(fetchJoke.fulfilled, (state, action) => {
      // Добавляем новую шутку в начало
      state.jokes.unshift(action.payload);
      // Оставляем только 5 последних
      if (state.jokes.length > 5) {
        state.jokes.pop();
      }
      saveJokesHistory(state.jokes);
    });
  }
});
```

</details>


## 🚀 Как выполнять упражнения

1. **Создайте новый React проект:**
```bash
npx create-react-app redux-practice
cd redux-practice
npm install @reduxjs/toolkit react-redux
```

2. **Для каждого упражнения создавайте отдельный слайс** в папке `src/features/`

3. **Проверяйте работу в компоненте** и через Redux DevTools

4. **Для localStorage** открывайте DevTools браузера (F12 → Application → Local Storage)

---