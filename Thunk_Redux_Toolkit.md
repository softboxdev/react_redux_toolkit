# Асинхронные экшены и Thunk в Redux Toolkit — Полное руководство

## Что такое асинхронность и зачем она нужна?

### Проблема синхронного Redux

Классический Redux работает только с **синхронными** действиями. Это значит:

```javascript
// ✅ Синхронный экшен (работает мгновенно)
dispatch(increment()); // сразу меняет состояние

// ❌ Асинхронная операция (НЕ РАБОТАЕТ)
dispatch(fetchUserFromApi()); // Redux не понимает, что делать с Promise
```

**В реальном приложении почти всегда нужно:**
- Загружать данные с сервера (`fetch`, `axios`)
- Читать файлы
- Работать с таймерами (`setTimeout`)
- Получать доступ к базе данных

**Redux не умеет делать это сам!** Поэтому нужен инструмент для асинхронности — **Thunk**.

---

## Что такое Thunk простыми словами?

**Thunk** — это middleware (промежуточный слой), который позволяет экшенам быть **функциями**, а не только объектами.

### Простая аналогия

Представьте, что Redux — это **почтальон**, который разносит письма (экшены).

```
Обычный экшен (объект):
┌─────────────────────────────────────────────────────────────────┐
│ "Почтальон, возьми этот конверт { type: 'INCREMENT' } и отнеси" │
│ Почтальон: "Понял, несу!" → сразу доставляет                    │
└─────────────────────────────────────────────────────────────────┘

Асинхронный экшен (функция с Thunk):
┌─────────────────────────────────────────────────────────────────┐
│ "Почтальон, вот тебе инструкция (функция). Сначала сходи в     │
│  библиотеку (API), возьми книгу, а потом отнеси её получателю"  │
│ Почтальон: "Понял, иду в библиотеку, жду ответа..."             │
│ через 2 секунды: "Книгу получил, доставляю!"                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Диаграмма: Синхронный vs Асинхронный экшен

```
СИНХРОННЫЙ ЭКШЕН (БЕЗ THUNK)
═══════════════════════════════════════════════════════════════════

Компонент                Store                  Reducer
    │                      │                        │
    │ dispatch(increment())│                        │
    ├─────────────────────►│                        │
    │                      │  { type: 'INCREMENT' } │
    │                      ├───────────────────────►│
    │                      │                        │ return state + 1
    │                      │◄───────────────────────┤
    │                      │                        │
    │                      │  state обновлён        │
    │◄─────────────────────┤                        │
    │                      │                        │
    ▼                      ▼                        ▼


АСИНХРОННЫЙ ЭКШЕН (С THUNK)
═══════════════════════════════════════════════════════════════════

Компонент         Thunk Middleware              API Server           Reducer
    │                   │                            │                  │
    │ dispatch(fetchUser())                        │                  │
    ├──────────────────►│                           │                  │
    │                   │                           │                  │
    │                   │ "Это функция, запускаю"   │                  │
    │                   │ dispatch({ type: 'pending' })              │
    │                   ├───────────────────────────────────────────►│
    │                   │                           │                  │
    │                   │  fetch('/api/user/123')   │                  │
    │                   ├──────────────────────────►│                  │
    │                   │                           │                  │
    │                   │      (ждём ответа...)     │                  │
    │                   │                           │                  │
    │                   │◄──────────────────────────┤                  │
    │                   │     { data: user }        │                  │
    │                   │                           │                  │
    │                   │ dispatch({ type: 'fulfilled', payload: user })
    │                   ├───────────────────────────────────────────►│
    │                   │                           │   обновляем state│
    │                   │                           │                  │
    │  компонент        │                           │                  │
    │  перерендеривается│                           │                  │
    │◄──────────────────┤                           │                  │
    │                   │                           │                  │
    ▼                   ▼                           ▼                  ▼
```

---

## Как работает Thunk внутри (упрощённая схема)

```javascript
// Упрощённая реализация Thunk (как это работает под капотом)
const thunkMiddleware = (store) => (next) => (action) => {
  // Если action — это функция, а не объект
  if (typeof action === 'function') {
    // Вызываем эту функцию и передаём ей dispatch и getState
    return action(store.dispatch, store.getState);
  }
  
  // Если action — это объект, передаём дальше
  return next(action);
};
```

**Что происходит:**
1. Thunk проверяет: `action` — это функция или объект?
2. Если **функция** — вызывает её и даёт ей доступ к `dispatch`
3. Если **объект** — передаёт дальше в редюсер

---

## Практический пример: загрузка пользователя

### Без Thunk (НЕ РАБОТАЕТ ❌)

```javascript
// ❌ Так НЕЛЬЗЯ! Redux не понимает Promise
function fetchUser(id) {
  return async (dispatch) => {
    // Этот код НЕ СРАБОТАЕТ без Thunk
    const user = await api.getUser(id);
    dispatch({ type: 'user/set', payload: user });
  };
}
```

### С Thunk (РАБОТАЕТ ✅)

```javascript
// ✅ Так МОЖНО! Thunk обрабатывает функцию
import { configureStore } from '@reduxjs/toolkit';

// Thunk уже ВНУТРИ configureStore, ничего подключать не нужно!
function fetchUser(id) {
  return async (dispatch) => {
    const user = await api.getUser(id);
    dispatch({ type: 'user/set', payload: user });
  };
}

// Использование
dispatch(fetchUser(123)); // Thunk обработает функцию
```

---

## Диаграмма: Жизненный цикл асинхронного экшена

```
ВРЕМЕННАЯ ШКАЛА
═══════════════════════════════════════════════════════════════════

t=0ms: Пользователь нажимает кнопку "Загрузить профиль"
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│ dispatch(fetchUser(123))                                       │
│                                                                 │
│ Thunk видит: "Это функция!" → запускает её                      │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=1ms: ┌─────────────────────────────────────────────────────────┐
│ dispatch({ type: 'user/fetch/pending' })                       │
│                                                                 │
│ Редюсер: state.loading = true, state.error = null              │
│ Компонент показывает спиннер "Загрузка..."                      │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=2ms: ┌─────────────────────────────────────────────────────────┐
│ await fetch('/api/users/123')                                  │
│                                                                 │
│ 🔄 Идёт запрос на сервер... (100-500 мс)                       │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=300ms: ┌───────────────────────────────────────────────────────┐
│ Сервер вернул ответ: { id: 123, name: "Анна" }                 │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=301ms: ┌───────────────────────────────────────────────────────┐
│ dispatch({ type: 'user/fetch/fulfilled', payload: user })      │
│                                                                 │
│ Редюсер: state.loading = false, state.data = user              │
│ Компонент показывает данные пользователя                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Redux Toolkit: createAsyncThunk

`createAsyncThunk` — это функция, которая автоматически создаёт три типа экшенов:
- `pending` — когда запрос начался
- `fulfilled` — когда запрос успешно завершился
- `rejected` — когда произошла ошибка

### Базовый синтаксис

```javascript
import { createAsyncThunk } from '@reduxjs/toolkit';

// Создаём асинхронный thunk
export const fetchUser = createAsyncThunk(
  'user/fetch',      // 1. Уникальное имя (префикс для экшенов)
  async (userId) => { // 2. Асинхронная функция (payload creator)
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();
    return data; // то, что попадёт в action.payload
  }
);
```

### Что создаёт createAsyncThunk?

```javascript
// createAsyncThunk автоматически генерирует 3 экшена:

fetchUser.pending   // тип: 'user/fetch/pending'
fetchUser.fulfilled // тип: 'user/fetch/fulfilled'  
fetchUser.rejected  // тип: 'user/fetch/rejected'
```

---

## Полный пример: приложение с загрузкой пользователей

### Шаг 1: Создаём слайс с createAsyncThunk

```javascript
// features/users/usersSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// 1. СОЗДАЁМ АСИНХРОННЫЙ THUNK
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',                    // уникальное имя
  async (_, { rejectWithValue }) => {    // payload creator
    try {
      const response = await axios.get('https://jsonplaceholder.typicode.com/users');
      return response.data; // данные попадут в action.payload
    } catch (error) {
      // Если ошибка, возвращаем её через rejectWithValue
      return rejectWithValue(error.message);
    }
  }
);

// 2. СОЗДАЁМ СЛАЙС
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [],        // массив пользователей
    status: 'idle',   // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null       // сообщение об ошибке
  },
  reducers: {
    // синхронные редюсеры (если нужны)
    clearUsers: (state) => {
      state.items = [];
      state.status = 'idle';
    }
  },
  // 3. ОБРАБАТЫВАЕМ АСИНХРОННЫЕ ЭКШЕНЫ
  extraReducers: (builder) => {
    builder
      // Когда запрос начался
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
        state.error = null;
      })
      // Когда запрос успешно завершился
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload; // данные пришли в action.payload
      })
      // Когда произошла ошибка
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload || action.error.message;
      });
  }
});

export const { clearUsers } = usersSlice.actions;
export default usersSlice.reducer;
```

### Шаг 2: Используем в компоненте

```jsx
// components/UsersList.jsx
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsers, clearUsers } from '../features/users/usersSlice';

export function UsersList() {
  const dispatch = useDispatch();
  const { items, status, error } = useSelector(state => state.users);
  
  // Загружаем пользователей при монтировании компонента
  useEffect(() => {
    dispatch(fetchUsers());
    
    // Очистка при размонтировании (опционально)
    return () => {
      dispatch(clearUsers());
    };
  }, [dispatch]);
  
  // Отображаем состояние загрузки
  if (status === 'loading') {
    return (
      <div className="loading">
        <div className="spinner"></div>
        <p>Загрузка пользователей...</p>
      </div>
    );
  }
  
  // Отображаем ошибку
  if (status === 'failed') {
    return (
      <div className="error">
        <h3>Ошибка!</h3>
        <p>{error}</p>
        <button onClick={() => dispatch(fetchUsers())}>
          Попробовать снова
        </button>
      </div>
    );
  }
  
  // Отображаем список пользователей
  if (status === 'succeeded') {
    return (
      <div className="users-list">
        <h2>Пользователи ({items.length})</h2>
        <button onClick={() => dispatch(fetchUsers())}>
          Обновить
        </button>
        <ul>
          {items.map(user => (
            <li key={user.id}>
              <strong>{user.name}</strong>
              <br />
              Email: {user.email}
              <br />
              Телефон: {user.phone}
            </li>
          ))}
        </ul>
      </div>
    );
  }
  
  // Начальное состояние (ещё не загружали)
  return (
    <div className="initial">
      <button onClick={() => dispatch(fetchUsers())}>
        Загрузить пользователей
      </button>
    </div>
  );
}
```

---

## Диаграмма: Полный цикл createAsyncThunk

```
КОМПОНЕНТ                  THUNK                  API                  SLICE
    │                        │                     │                     │
    │  dispatch(fetchUsers())│                     │                     │
    ├───────────────────────►│                     │                     │
    │                        │                     │                     │
    │                        │  dispatch(pending)  │                     │
    │                        ├─────────────────────────────────────────►│
    │                        │                     │   state.status='loading'
    │◄───────────────────────┤                     │                     │
    │                        │                     │                     │
    │  (спиннер)             │                     │                     │
    │                        │                     │                     │
    │                        │  fetch('/api/users')│                     │
    │                        ├────────────────────►│                     │
    │                        │                     │                     │
    │                        │      (ждём)         │                     │
    │                        │                     │                     │
    │                        │◄────────────────────┤                     │
    │                        │   response.data     │                     │
    │                        │                     │                     │
    │                        │  dispatch(fulfilled)│                     │
    │                        ├─────────────────────────────────────────►│
    │                        │                     │   state.items=data │
    │◄───────────────────────┤                     │   state.status='succeeded'
    │                        │                     │                     │
    │  (рендер списка)       │                     │                     │
    │                        │                     │                     │
    
    Если ошибка:
    │                        │                     │                     │
    │                        │  dispatch(rejected) │                     │
    │                        ├─────────────────────────────────────────►│
    │                        │                     │   state.error=...   │
    │                        │                     │   state.status='failed'
    │◄───────────────────────┤                     │                     │
    │                        │                     │                     │
    │  (показываем ошибку)   │                     │                     │
```

---

## Работа с параметрами в createAsyncThunk

```javascript
// Thunk с параметром
export const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId, { rejectWithValue, getState, dispatch }) => {
    // userId - первый аргумент, переданный при вызове
    // getState() - позволяет получить текущее состояние store
    // dispatch() - для отправки других экшенов
    // rejectWithValue - для возврата кастомной ошибки
    
    const state = getState();
    const existingUser = state.users.items.find(u => u.id === userId);
    
    if (existingUser) {
      return existingUser; // если уже есть, возвращаем из кэша
    }
    
    try {
      const response = await axios.get(`/api/users/${userId}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(`Не удалось загрузить пользователя ${userId}`);
    }
  }
);

// Использование в компоненте
function UserProfile() {
  const { id } = useParams();
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUserById(id)); // передаём параметр id
  }, [id, dispatch]);
}
```

---

## Диаграмма: Передача параметров в Thunk

```
КОМПОНЕНТ                           THUNK                           API
    │                                 │                              │
    │  dispatch(fetchUserById(123))  │                              │
    ├────────────────────────────────►│                              │
    │                                 │                              │
    │                                 │  userId = 123               │
    │                                 │                              │
    │                                 │  fetch('/api/users/123')    │
    │                                 ├─────────────────────────────►│
    │                                 │                              │
    │                                 │◄─────────────────────────────┤
    │                                 │     { id: 123, name: ... }   │
    │                                 │                              │
    │  dispatch(fulfilled)            │                              │
    │◄────────────────────────────────┤                              │
    │                                 │                              │
```

---

## Дополнительные возможности createAsyncThunk

### 1. Отмена запроса (cancel)

```javascript
export const searchProducts = createAsyncThunk(
  'products/search',
  async (query, { signal }) => {
    // signal позволяет отменить запрос
    const response = await fetch(`/api/search?q=${query}`, { signal });
    return response.json();
  }
);

// В компоненте
function SearchComponent() {
  const [query, setQuery] = useState('');
  const dispatch = useDispatch();
  
  useEffect(() => {
    // Создаём promise для отмены
    const promise = dispatch(searchProducts(query));
    
    // Отменяем запрос при размонтировании или изменении query
    return () => {
      promise.abort();
    };
  }, [query, dispatch]);
}
```

### 2. Условное выполнение

```javascript
export const loadMorePosts = createAsyncThunk(
  'posts/loadMore',
  async (_, { getState, rejectWithValue }) => {
    const state = getState();
    const { items, hasMore, loading } = state.posts;
    
    // Не загружаем, если уже грузим или нет больше данных
    if (loading || !hasMore) {
      return rejectWithValue('skip');
    }
    
    const response = await fetch(`/api/posts?offset=${items.length}`);
    return response.json();
  }
);
```

### 3. Обработка в компоненте с unwrap()

```javascript
function SubmitButton() {
  const dispatch = useDispatch();
  const [error, setError] = useState(null);
  
  const handleSubmit = async () => {
    try {
      // unwrap() возвращает результат или выбрасывает ошибку
      const result = await dispatch(saveData(formData)).unwrap();
      console.log('Успешно сохранено:', result);
      showSuccessToast('Данные сохранены!');
    } catch (err) {
      console.error('Ошибка:', err);
      setError(err.message);
    }
  };
  
  return <button onClick={handleSubmit}>Сохранить</button>;
}
```

---

## Сравнение: createAsyncThunk vs ручной Thunk

| Аспект | createAsyncThunk | Ручной Thunk |
|--------|------------------|--------------|
| **Код** | Минимальный | Много шаблонного кода |
| **Создание экшенов** | Автоматически (pending, fulfilled, rejected) | Вручную (3 экшена) |
| **Обработка ошибок** | Встроенная | Нужно писать вручную |
| **Статусы загрузки** | Автоматически | Нужно отслеживать вручную |
| **TypeScript** | Отличная поддержка | Сложнее типизировать |
| **Отмена запросов** | Встроенная | Нужно реализовывать самому |

### Пример с ручным Thunk (без createAsyncThunk)

```javascript
// ❌ Много кода!
const fetchUsersPending = () => ({ type: 'users/fetch/pending' });
const fetchUsersSuccess = (users) => ({ type: 'users/fetch/success', payload: users });
const fetchUsersError = (error) => ({ type: 'users/fetch/error', error });

const fetchUsers = () => async (dispatch) => {
  dispatch(fetchUsersPending());
  try {
    const response = await axios.get('/api/users');
    dispatch(fetchUsersSuccess(response.data));
  } catch (error) {
    dispatch(fetchUsersError(error.message));
  }
};

// И в редюсере нужно обрабатывать три экшена вручную
```

### Против createAsyncThunk

```javascript
// ✅ Коротко и понятно!
export const fetchUsers = createAsyncThunk('users/fetch', async () => {
  const response = await axios.get('/api/users');
  return response.data;
});

// Редюсер обрабатывает через extraReducers
```

## Что даёт Thunk? (преимущества)

| Без Thunk | С Thunk |
|-----------|---------|
| ❌ Нельзя делать запросы к серверу | ✅ Можно делать асинхронные запросы |
| ❌ Нельзя использовать таймеры | ✅ Можно использовать setTimeout |
| ❌ Весь код синхронный | ✅ Можно писать асинхронный код |
| ❌ Нужно подключать дополнительные библиотеки | ✅ Встроен в Redux Toolkit |
| ❌ Сложно обрабатывать состояния загрузки | ✅ Легко отслеживать статус |

---

**Thunk** — это специальная функция-посредник, которая позволяет Redux работать с асинхронными операциями (запросами к серверу, таймерами, чтением файлов).

**Аналогия из жизни:** Представьте, что Redux — это ресторан, а Thunk — это официант.

```
БЕЗ THUNK (только синхронные заказы):
┌─────────────────────────────────────────────────────────────────┐
│ Вы: "Мне пиццу!"                                                │
│ Redux: "Понял, несу пиццу" → сразу даёт результат              │
│ (можно заказать только то, что уже есть на кухне)              │
└─────────────────────────────────────────────────────────────────┘

С THUNK (можно заказать то, чего нет в наличии):
┌─────────────────────────────────────────────────────────────────┐
│ Вы: "Мне пиццу!"                                                │
│ Thunk (официант): "Схожу на кухню, закажу приготовить"         │
│          → идёт на кухню, ждёт 10 минут                         │
│          → получает готовую пиццу                               │
│          → приносит её вам                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Проблема, которую решает Thunk

### Обычный Redux (без Thunk)

```javascript
// ❌ Redux НЕ УМЕЕТ работать с асинхронным кодом
function loadUser() {
  // Эта функция НЕ СРАБОТАЕТ!
  return async (dispatch) => {
    const user = await fetch('/api/user');
    dispatch({ type: 'SET_USER', payload: user });
  };
}

dispatch(loadUser()); // Ошибка: Redux ожидает объект, а получил функцию
```

**Почему не работает:** Redux ожидает, что `dispatch` получит **простой объект** (action), а не функцию.

### С Thunk (всё работает)

```javascript
// ✅ С Thunk можно!
function loadUser() {
  return async (dispatch) => {
    const user = await fetch('/api/user');
    dispatch({ type: 'SET_USER', payload: user });
  };
}

dispatch(loadUser()); // Thunk обработает функцию
```

---

## Что такое Thunk простыми словами?

**Thunk** — это middleware (промежуточный слой), который:
1. Проверяет: что отправили в `dispatch`?
2. Если **объект** → передаёт дальше в редюсер (как обычно)
3. Если **функция** → запускает её и даёт ей доступ к `dispatch`

### Визуализация работы Thunk

```
dispatch(обычный_объект)
═══════════════════════════════════════════════════════════════════

dispatch({ type: 'INCREMENT' })
         │
         ▼
    ┌─────────┐
    │  Thunk  │ → "Это объект? Да, отправляю в редюсер"
    └────┬────┘
         │
         ▼
      Reducer


dispatch(функция)
═══════════════════════════════════════════════════════════════════

dispatch(async (dispatch) => {
  const data = await fetch(...);
  dispatch({ type: 'SET', payload: data });
})
         │
         ▼
    ┌─────────┐
    │  Thunk  │ → "Это функция? Запускаю её и даю ей dispatch"
    └────┬────┘
         │
         ▼
    Выполняется функция:
    ┌─────────────────────────────────────┐
    │ 1. Ждём ответ от сервера...         │
    │ 2. Получили данные                  │
    │ 3. Вызываем dispatch с объектом     │
    │ 4. Объект идёт в редюсер            │
    └─────────────────────────────────────┘
```

---

## Как Thunk выглядит в коде

### Простейший Thunk

```javascript
// Это Thunk - функция, которая возвращает другую функцию
function simpleThunk() {
  // Возвращаем функцию, которая получит dispatch от Redux
  return function(dispatch) {
    // Здесь можно делать что угодно
    console.log('Thunk выполняется!');
    dispatch({ type: 'ACTION' });
  };
}

// Использование
dispatch(simpleThunk());
```

### Thunk с асинхронным кодом

```javascript
// Thunk для загрузки данных с сервера
function fetchUserThunk(userId) {
  // Возвращаем асинхронную функцию
  return async function(dispatch) {
    // 1. Сообщаем, что начали загрузку
    dispatch({ type: 'FETCH_USER_START' });
    
    try {
      // 2. Ждём ответ от сервера (асинхронно)
      const response = await fetch(`/api/users/${userId}`);
      const user = await response.json();
      
      // 3. Когда данные пришли, отправляем их в редюсер
      dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
    } catch (error) {
      // 4. Если ошибка, отправляем ошибку
      dispatch({ type: 'FETCH_USER_ERROR', error: error.message });
    }
  };
}

// Использование в компоненте
dispatch(fetchUserThunk(123));
```

---

## Диаграмма: Полный цикл работы Thunk

```
КОМПОНЕНТ                     THUNK                     API СЕРВЕР
    │                           │                            │
    │  dispatch(fetchUser(123)) │                            │
    ├──────────────────────────►│                            │
    │                           │                            │
    │                           │  dispatch({ type: 'PENDING' })
    │                           ├────────────┐               │
    │                           │            │               │
    │                           │◄───────────┘               │
    │                           │                            │
    │   (компонент показывает   │                            │
    │    спиннер загрузки)      │                            │
    │                           │                            │
    │                           │  await fetch(...)          │
    │                           ├───────────────────────────►│
    │                           │                            │
    │                           │     (ждём ответ... ⏰)      │
    │                           │                            │
    │                           │◄───────────────────────────┤
    │                           │       user data            │
    │                           │                            │
    │                           │  dispatch({ type: 'SUCCESS', payload })
    │                           ├────────────┐               │
    │                           │            │               │
    │                           │◄───────────┘               │
    │                           │                            │
    │   (компонент показывает   │                            │
    │    данные пользователя)   │                            │
    │◄──────────────────────────┤                            │
    │                           │                            │
```

---

## Redux Toolkit и Thunk

В Redux Toolkit Thunk уже **встроен**! Не нужно ничего устанавливать или подключать.

### createAsyncThunk (упрощённая версия Thunk)

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

// createAsyncThunk сам создаёт Thunk!
export const fetchUser = createAsyncThunk(
  'user/fetch',           // имя
  async (userId) => {     // асинхронная функция
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

// То, что создал createAsyncThunk:
// fetchUser.pending   - экшен для начала загрузки
// fetchUser.fulfilled - экшен для успешной загрузки  
// fetchUser.rejected  - экшен для ошибки

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, status: 'idle' },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state) => {
        state.status = 'failed';
      });
  }
});
```

---

## Сравнение: ручной Thunk vs createAsyncThunk

### Ручной Thunk (без Redux Toolkit)

```javascript
// ❌ Много ручного кода
function fetchUser(id) {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_USER_START' });
    
    try {
      const response = await fetch(`/api/users/${id}`);
      const user = await response.json();
      dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'FETCH_USER_ERROR', error: error.message });
    }
  };
}

// В редюсере нужно вручную обрабатывать три экшена
```

### createAsyncThunk (Redux Toolkit)

```javascript
// ✅ Коротко и понятно
export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (id) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
);
// Три экшена создаются автоматически!
```

---


## Реальный пример: загрузка списка товаров

```jsx
// productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// 1. Создаём Thunk для загрузки товаров
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async () => {
    const response = await axios.get('https://fakestoreapi.com/products');
    return response.data; // то, что попадёт в action.payload
  }
);

// 2. Создаём слайс
const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    status: 'idle',     // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.status = 'loading';  // началась загрузка
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded'; // загрузка успешна
        state.items = action.payload; // сохраняем товары
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed';  // ошибка
        state.error = action.error.message;
      });
  }
});

export default productsSlice.reducer;

// ProductsList.jsx
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchProducts } from './productsSlice';

function ProductsList() {
  const dispatch = useDispatch();
  const { items, status, error } = useSelector(state => state.products);
  
  // Загружаем товары при монтировании компонента
  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchProducts()); // ← вызываем Thunk!
    }
  }, [status, dispatch]);
  
  if (status === 'loading') return <div>Загрузка товаров...</div>;
  if (status === 'failed') return <div>Ошибка: {error}</div>;
  
  return (
    <ul>
      {items.map(product => (
        <li key={product.id}>{product.title} - ${product.price}</li>
      ))}
    </ul>
  );
}
```

---

## Что происходит под капотом?

```javascript
// Упрощённая реализация Thunk middleware
const thunkMiddleware = (store) => (next) => (action) => {
  // Если action — это функция
  if (typeof action === 'function') {
    // Вызываем её и передаём dispatch и getState
    return action(store.dispatch, store.getState);
  }
  
  // Если action — это объект, передаём дальше
  return next(action);
};

// Как это работает:
// 1. Вы вызываете dispatch(fetchProducts())
// 2. fetchProducts() возвращает функцию
// 3. Thunk видит: "это функция"
// 4. Thunk запускает эту функцию
// 5. Функция может делать асинхронные операции
// 6. Когда данные готовы, функция вызывает dispatch с объектом
// 7. Объект идёт в редюсер
```

---

## Thunk с параметрами

```javascript
// Thunk, который принимает параметры
export const fetchUserPosts = createAsyncThunk(
  'posts/fetchUserPosts',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}/posts`);
      return await response.json();
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

// В компоненте
function UserPosts({ userId }) {
  const dispatch = useDispatch();
  const { posts, status } = useSelector(state => state.posts);
  
  useEffect(() => {
    dispatch(fetchUserPosts(userId)); // ← передаём параметр
  }, [userId, dispatch]);
  
  // ...
}
```

---

## Thunk с доступом к состоянию (getState)

```javascript
export const addToCart = createAsyncThunk(
  'cart/addToCart',
  async (productId, { getState, rejectWithValue }) => {
    // Получаем текущее состояние
    const state = getState();
    const product = state.products.items.find(p => p.id === productId);
    
    if (!product) {
      return rejectWithValue('Товар не найден');
    }
    
    if (product.stock === 0) {
      return rejectWithValue('Товар закончился');
    }
    
    // Добавляем в корзину
    return { id: productId, name: product.name, price: product.price };
  }
);
```

---

## Частые ошибки с Thunk

### Ошибка 1: Забыли импортировать Thunk

```javascript
// ❌ ОШИБКА: Redux Toolkit уже включает Thunk, ничего подключать не нужно!
import thunk from 'redux-thunk'; // лишнее
const store = configureStore({
  reducer: rootReducer,
  middleware: [thunk] // лишнее
});

// ✅ ПРАВИЛЬНО: Thunk уже внутри configureStore
const store = configureStore({
  reducer: rootReducer
});
```

### Ошибка 2: Неправильный return в createAsyncThunk

```javascript
// ❌ ОШИБКА: забыли вернуть данные
export const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  const response = await fetch(`/api/users/${id}`);
  // нет return → action.payload = undefined
});

// ✅ ПРАВИЛЬНО: возвращаем данные
export const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data; // ← важно!
});
```

### Ошибка 3: Вызов Thunk вне useEffect

```javascript
// ❌ ОШИБКА: бесконечный цикл
function Component() {
  const dispatch = useDispatch();
  dispatch(fetchProducts()); // каждый рендер вызывает Thunk!
}

// ✅ ПРАВИЛЬНО: в useEffect
function Component() {
  const dispatch = useDispatch();
  useEffect(() => {
    dispatch(fetchProducts()); // один раз при монтировании
  }, [dispatch]);
}
```

---

## Шпаргалка по Thunk

```
┌─────────────────────────────────────────────────────────────────┐
│                          THUNK                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ЧТО ЭТО: Функция, которая возвращает другую функцию           │
│                                                                 │
│  ЗАЧЕМ: Позволяет Redux работать с асинхронным кодом           │
│                                                                 │
│  КАК РАБОТАЕТ:                                                  │
│  1. Вы вызываете dispatch(thunkFunction())                      │
│  2. Thunk middleware проверяет: "это функция?"                  │
│  3. Запускает функцию и даёт ей dispatch                        │
│  4. Функция делает асинхронную операцию                         │
│  5. Функция вызывает dispatch с обычным объектом                │
│  6. Объект идёт в редюсер                                       │
│                                                                 │
│  В REDUX TOOLKIT:                                               │
│  - Thunk уже встроен в configureStore                           │
│  - Используйте createAsyncThunk для создания Thunk              │
│  - Не нужно подключать вручную!                                 │
│                                                                 │
│  ПРИМЕР:                                                        │
│  const fetchUser = createAsyncThunk('user/fetch', async (id) =>│
│    (await fetch(`/api/users/${id}`)).json()                    │
│  );                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Золотое правило:** Если вам нужно сделать запрос к серверу, подождать, использовать таймер или любую другую асинхронную операцию — используйте Thunk (через `createAsyncThunk` в Redux Toolkit). Это стандартный и самый простой способ работать с асинхронностью в Redux! 🚀