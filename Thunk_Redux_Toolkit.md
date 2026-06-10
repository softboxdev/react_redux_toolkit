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
