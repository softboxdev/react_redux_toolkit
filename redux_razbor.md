# Что такое Redux — подробный разбор 

## Простое объяснение

**Redux** — это библиотека для управления состоянием (данными) в JavaScript-приложениях, особенно в React. Она помогает хранить **все данные приложения в одном месте** и **предсказуемо их изменять**.

**Аналогия из жизни:** Представьте, что Redux — это **единый банк** для всего вашего приложения.

- Без Redux: каждый компонент хранит свои деньги в разных карманах, кошельках, копилках. Трудно отследить, у кого сколько денег.
- С Redux: все деньги лежат в одном банке. Чтобы взять или положить деньги, нужно заполнить заявку (экшен). Банк (store) обрабатывает заявку по строгим правилам (редюсер) и выдаёт новый баланс.

---


## Когда нужен Redux?

### ✅ Когда стоит использовать Redux:

1. **Большое приложение** (много компонентов, страниц)
2. **Много общего состояния** (данные используются во многих местах)
3. **Сложная логика обновления** (много способов изменить данные)
4. **Команда разработчиков** (единый подход к управлению состоянием)
5. **Нужен time-travel debugging** (возможность откатывать изменения)

### ❌ Когда НЕ нужен Redux:

1. **Маленькое приложение** (3-5 компонентов)
2. **Данные только локальные** (не используются в разных компонентах)
3. **Простая логика** (пользователь вводит данные в форму и отправляет)
4. **Вы только учитесь** (начните с React useState/useContext)

### Альтернативы Redux:

| Инструмент | Когда использовать |
|-----------|-------------------|
| `useState` | Локальное состояние компонента |
| `useContext` | Несколько компонентов, редкие изменения |
| `Zustand` | Простой Redux, меньше кода |
| `MobX` | Объектно-ориентированный подход |
| `Jotai` | Атомарное состояние, как Recoil |

---


## Проблема, которую решает Redux

### Проблема: Сложность управления состоянием в больших приложениях

В маленьком приложении с 2-3 компонентами можно использовать локальное состояние React (`useState`). Но когда приложение растёт:

```javascript
// ❌ Проблема: состояние размазано по всему приложению
function Grandparent() {
  const [user, setUser] = useState({ name: 'Анна' });
  return <Parent user={user} setUser={setUser} />;
}

function Parent({ user, setUser }) {
  return <Child user={user} setUser={setUser} />;
}

function Child({ user, setUser }) {
  // Чтобы изменить данные, нужно передавать setUser через 10 компонентов!
  return <button onClick={() => setUser({ name: 'Новое имя' })}>
    Сменить имя
  </button>;
}
```

**Проблемы такого подхода:**
- **Prop drilling** — передача данных через много компонентов, которые их не используют
- **Сложность отслеживания** — непонятно, где и как меняются данные
- **Трудно отлаживать** — когда что-то сломалось, сложно понять, кто это сделал
- **Нет единого источника истины** — данные могут дублироваться и расходиться

### Решение: Redux

```javascript
// ✅ Redux: все данные в одном месте
// Любой компонент может читать данные и отправлять команды на их изменение

function Grandparent() {
  const user = useSelector(state => state.user); // читаем
  return <div>{user.name}</div>;
}

function Child() {
  const dispatch = useDispatch();
  // Не нужно передавать через все компоненты!
  return <button onClick={() => dispatch(setName('Новое имя'))}>
    Сменить имя
  </button>;
}
```

---

## Три главных принципа Redux

### 1. Единый источник истины (Single Source of Truth)

**Всё состояние приложения хранится в одном объекте (store).**

```javascript
// Весь state приложения — это один большой объект
const state = {
  user: { id: 1, name: 'Анна', age: 25 },
  posts: [
    { id: 1, title: 'Первый пост', text: '...' },
    { id: 2, title: 'Второй пост', text: '...' }
  ],
  cart: ['яблоко', 'банан', 'апельсин'],
  ui: { theme: 'dark', sidebarOpen: true }
}
```

**Преимущества:**
- Легко найти любые данные (всё в одном месте)
- Просто отлаживать (видно всё состояние)
- Можно сохранять/восстанавливать состояние

### 2. Состояние только для чтения (State is Read-Only)

**Нельзя изменить состояние напрямую. Только через отправку экшенов.**

```javascript
// ❌ НЕЛЬЗЯ (напрямую)
store.user.name = 'Новое имя';

// ✅ МОЖНО (через экшен)
store.dispatch({ type: 'user/setName', payload: 'Новое имя' });
```

**Почему так:** Это делает изменения предсказуемыми. Вы всегда знаете, кто, когда и почему изменил данные.

### 3. Изменения делаются чистыми функциями (Reducers)

**Редюсеры — это чистые функции, которые берут текущее состояние и экшен, и возвращают новое состояние.**

```javascript
// Чистая функция: одинаковый вход → одинаковый выход
function counterReducer(state = 0, action) {
  if (action.type === 'increment') {
    return state + 1;  // возвращаем НОВОЕ состояние
  }
  if (action.type === 'decrement') {
    return state - 1;
  }
  return state;  // по умолчанию возвращаем текущее состояние
}
```

**Чистая функция означает:**
- Не меняет входные параметры
- Не делает запросов к API
- Не использует случайные значения (Date.now(), Math.random())
- Всегда возвращает новое значение, а не меняет старое

---

## Основные части Redux

### 1. Store (Хранилище)

**Store** — это объект, который хранит всё состояние приложения.

```javascript
import { createStore } from 'redux';

const store = createStore(counterReducer);

// Методы store:
store.getState();     // получить текущее состояние
store.dispatch(action); // отправить экшен
store.subscribe(() => {}); // подписаться на изменения
```

### 2. Action (Экшен)

**Action** — это простой объект, который описывает, что произошло. У него обязательно есть поле `type`.

```javascript
// Экшен — это объект
const incrementAction = {
  type: 'counter/increment'  // тип (обязательно)
};

const setNameAction = {
  type: 'user/setName',
  payload: 'Анна'  // данные (опционально)
};

// Action creator — функция, которая создаёт экшен
function increment() {
  return { type: 'counter/increment' };
}

function setName(name) {
  return { type: 'user/setName', payload: name };
}
```

### 3. Reducer (Редюсер)

**Reducer** — это чистая функция, которая принимает текущее состояние и экшен, и возвращает новое состояние.

```javascript
const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'counter/increment':
      return { ...state, count: state.count + 1 };
    
    case 'counter/decrement':
      return { ...state, count: state.count - 1 };
    
    default:
      return state;  // всегда возвращаем state по умолчанию
  }
}
```

### 4. Dispatch (Диспатч)

**Dispatch** — это функция, которая отправляет экшен в store.

```javascript
// Отправляем экшен
store.dispatch(increment());  // экшен: { type: 'counter/increment' }
store.dispatch(setName('Анна')); // экшен: { type: 'user/setName', payload: 'Анна' }
```

---

## Полный пример работы Redux

### Пример счётчика на чистом Redux (без React)

```javascript
// 1. Начальное состояние
const initialState = { count: 0 };

// 2. Редюсер (описывает, как меняется состояние)
function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    
    default:
      return state;
  }
}

// 3. Создаём store
const store = createStore(counterReducer);

// 4. Подписываемся на изменения
store.subscribe(() => {
  console.log('Новое состояние:', store.getState());
});

// 5. Отправляем экшены
store.dispatch({ type: 'INCREMENT' }); // Новое состояние: { count: 1 }
store.dispatch({ type: 'INCREMENT' }); // Новое состояние: { count: 2 }
store.dispatch({ type: 'DECREMENT' }); // Новое состояние: { count: 1 }

console.log(store.getState()); // { count: 1 }
```

### Схема работы Redux

```
┌─────────────┐
│  Компонент  │
│  (React)    │
└──────┬──────┘
       │
       │ dispatch(action)
       ▼
┌─────────────┐
│   Store     │
│  (Redux)    │
└──────┬──────┘
       │
       │ передаёт action
       ▼
┌─────────────┐
│  Reducer    │
│ (чистая    │
│  функция)   │
└──────┬──────┘
       │
       │ возвращает новое состояние
       ▼
┌─────────────┐
│   Store     │
│ обновляет   │
│ состояние   │
└──────┬──────┘
       │
       │ уведомляет
       ▼
┌─────────────┐
│  Компонент  │
│ обновляется │
└─────────────┘
```

---

## Redux + React

Для связи Redux с React используется библиотека `react-redux`, которая предоставляет хуки.

### Полный пример с React

```javascript
// ========== store.js ==========
import { createStore } from 'redux';

const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}

export const store = createStore(counterReducer);

// ========== App.jsx ==========
import { Provider, useSelector, useDispatch } from 'react-redux';
import { store } from './store';

// Компонент для отображения
function CounterDisplay() {
  const count = useSelector(state => state.count);
  return <h1>Счётчик: {count}</h1>;
}

// Компонент для управления
function CounterButtons() {
  const dispatch = useDispatch();
  
  return (
    <div>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
    </div>
  );
}

// Главный компонент
function App() {
  return (
    <Provider store={store}>
      <CounterDisplay />
      <CounterButtons />
    </Provider>
  );
}
```

---

## Современный Redux: Redux Toolkit

**Redux Toolkit (RTK)** — это официальный инструмент, который упрощает написание Redux-кода. Сегодня **всегда** используют RTK, а не чистый Redux.

### Сравнение: чистый Redux vs Redux Toolkit

**Чистый Redux (много кода):**
```javascript
// actionTypes.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';

// actions.js
export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });

// reducer.js
const initialState = 0;
export default function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return state + 1;
    case DECREMENT:
      return state - 1;
    default:
      return state;
  }
}

// store.js
import { createStore } from 'redux';
import counterReducer from './reducer';
export const store = createStore(counterReducer);
```

**Redux Toolkit (мало кода):**
```javascript
// counterSlice.js — ВСЁ В ОДНОМ ФАЙЛЕ!
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1
  }
});

export const { increment, decrement } = counterSlice.actions;
export const store = configureStore({
  reducer: { counter: counterSlice.reducer }
});
```

## Асинхронность в Redux

Redux по умолчанию синхронный. Для асинхронных операций (запросы к API) используют **middleware**:

### Redux Thunk (самый простой)

```javascript
// Асинхронный action creator
const fetchUser = (id) => async (dispatch) => {
  dispatch({ type: 'FETCH_USER_START' });
  
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    dispatch({ type: 'FETCH_USER_ERROR', error });
  }
};

// Использование
dispatch(fetchUser(123));
```

### Redux Toolkit createAsyncThunk (проще)

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (id) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
);

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

## Redux DevTools

Одно из главных преимуществ Redux — инструменты для отладки.

**Что можно делать с DevTools:**
- Смотреть все экшены, которые происходят в приложении
- Видеть состояние ДО и ПОСЛЕ каждого экшена
- "Путешествовать во времени" — отменять и повторять действия
- Экспортировать и импортировать состояние
- Отслеживать производительность

![Redux DevTools (в уме)](https://example.com/redux-devtools)

---


```
┌─────────────────────────────────────────────────────────────┐
│                         REDUX                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Store     = место, где хранятся все данные                 │
│  Action    = объект, описывающий, что произошло             │
│  Reducer   = функция, которая решает, как изменить данные   │
│  Dispatch  = функция, которая отправляет action             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ПРИНЦИПЫ:                                                  │
│  1. Единый источник истины (один store)                     │
│  2. Состояние только для чтения (immutable)                 │
│  3. Изменения через чистые функции (reducers)               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  СОВРЕМЕННЫЙ ПОДХОД:                                        │
│  • Redux Toolkit (не чистый Redux)                          │
│  • createSlice вместо switch                                │
│  • configureStore вместо createStore                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
