# Основное приложение на Redux Toolkit: Todo List (Список дел)

Мы создадим простое приложение "Список дел", которое поможет понять основы Redux Toolkit. Приложение будет уметь:
- Добавлять новую задачу
- Отмечать задачу как выполненную
- Удалять задачу

### Объяснение компонентов

```tsx
const dispatch = useAppDispatch();
```
**Что делает:** Возвращает функцию `dispatch`, которая отправляет экшены в Redux store.

---

```tsx
dispatch(addTodo(inputValue));
```
**Что происходит:**
1. Создаётся экшен: `{ type: 'todos/addTodo', payload: 'Купить хлеб' }`
2. Экшен отправляется в store
3. Store вызывает редюсер `addTodo` из слайса
4. Редюсер добавляет задачу в `state.items`
5. Компоненты, которые читают данные (через `useSelector`), автоматически обновляются

---

```tsx
const todos = useAppSelector(state => state.todos.items);
```
**Что делает:**
- Берёт из store состояние `state.todos.items`
- Подписывается на изменения
- Когда `items` меняется, компонент автоматически перерендеривается

---

## Визуализация потока данных

```
Пользователь вводит текст и нажимает "Добавить"
                │
                ▼
         ┌─────────────┐
         │  AddTodo    │
         │  dispatch   │
         │ (addTodo)   │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │    Store    │
         │  (Redux)    │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │   Reducer   │
         │  addTodo    │
         │  state      │
         │  .items     │
         │   .push()   │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  Store      │
         │ обновляет   │
         │ состояние   │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  TodoList   │
         │  useSelector│
         │  получает   │
         │  новые      │
         │  данные     │
         └─────────────┘
```

1. **Слайс (slice)** — объединяет имя, начальное состояние и редюсеры
2. **Редюсеры (reducers)** — функции, которые описывают, как менять состояние
3. **Экшены (actions)** — объекты, которые описывают, что произошло
4. **Dispatch** — функция для отправки экшенов
5. **useSelector** — хук для чтения данных из store
6. **Provider** — компонент, который делает store доступным для всего приложения


## Полный код приложения

### Шаг 1: Установка зависимостей

```bash
# Создаём новое React-приложение
npx create-react-app my-redux-todo --template typescript

# Переходим в папку проекта
cd my-redux-todo

# Устанавливаем Redux Toolkit и React Redux
npm install @reduxjs/toolkit react-redux
```

### Шаг 2: Создаём слайс (todosSlice.ts)

```typescript
// src/features/todos/todosSlice.ts

import { createSlice, PayloadAction } from '@reduxjs/toolkit';

// ========== 1. ОПРЕДЕЛЯЕМ ТИПЫ ==========
// Тип для одной задачи
interface Todo {
  id: number;        // уникальный идентификатор
  text: string;      // текст задачи
  completed: boolean; // выполнена ли задача
}

// Тип для состояния всего слайса
interface TodosState {
  items: Todo[];     // массив всех задач
}

// ========== 2. НАЧАЛЬНОЕ СОСТОЯНИЕ ==========
const initialState: TodosState = {
  items: [],  // начинаем с пустого списка задач
};

// ========== 3. СОЗДАЁМ СЛАЙС ==========
// createSlice - это главная функция Redux Toolkit для создания слайса
export const todosSlice = createSlice({
  name: 'todos',        // имя слайса (будет использоваться в типах экшенов)
  initialState,         // начальное состояние
  reducers: {           // объект с функциями-редюсерами
    // ========== РЕДЮСЕР №1: ДОБАВЛЕНИЕ ЗАДАЧИ ==========
    addTodo: (state, action: PayloadAction<string>) => {
      // action.payload - это текст новой задачи (тип string)
      // В Redux Toolkit МОЖНО "мутировать" состояние благодаря Immer
      state.items.push({
        id: Date.now(),           // используем timestamp как id
        text: action.payload,     // текст из action
        completed: false,         // новая задача не выполнена
      });
    },
    
    // ========== РЕДЮСЕР №2: ПЕРЕКЛЮЧЕНИЕ СТАТУСА ЗАДАЧИ ==========
    toggleTodo: (state, action: PayloadAction<number>) => {
      // action.payload - это id задачи, которую нужно переключить
      const todo = state.items.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed; // переключаем статус
      }
    },
    
    // ========== РЕДЮСЕР №3: УДАЛЕНИЕ ЗАДАЧИ ==========
    deleteTodo: (state, action: PayloadAction<number>) => {
      // action.payload - это id задачи, которую нужно удалить
      state.items = state.items.filter(todo => todo.id !== action.payload);
    },
  },
});

// ========== 4. ЭКСПОРТИРУЕМ ЭКШЕНЫ И РЕДЮСЕР ==========
// Экспортируем action creators для использования в компонентах
export const { addTodo, toggleTodo, deleteTodo } = todosSlice.actions;

// Экспортируем reducer для store
export default todosSlice.reducer;
```

### Шаг 3: Создаём хранилище (store.ts)

```typescript
// src/app/store.ts

import { configureStore } from '@reduxjs/toolkit';
// Импортируем редюсер из нашего слайса
import todosReducer from '../features/todos/todosSlice';

// ========== 1. СОЗДАЁМ STORE ==========
// configureStore - главная функция Redux Toolkit для создания store
export const store = configureStore({
  reducer: {
    // Здесь мы собираем все редюсеры из разных слайсов
    // Ключ 'todos' будет использоваться для доступа к этому состоянию
    todos: todosReducer,
  },
});

// ========== 2. ЭКСПОРТИРУЕМ ТИПЫ ДЛЯ TypeScript ==========
// Тип всего состояния (используется в useSelector)
export type RootState = ReturnType<typeof store.getState>;

// Тип dispatch (используется в useDispatch)
export type AppDispatch = typeof store.dispatch;
```

### Шаг 4: Создаём типизированные хуки (hooks.ts)

```typescript
// src/app/hooks.ts

import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// ========== 1. ТИПИЗИРОВАННЫЙ ХУК useDispatch ==========
// Этот хук знает тип AppDispatch и будет правильно типизировать dispatch
export const useAppDispatch = () => useDispatch<AppDispatch>();

// ========== 2. ТИПИЗИРОВАННЫЙ ХУК useSelector ==========
// Этот хук знает тип RootState и будет правильно типизировать state
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Шаг 5: Создаём компоненты

#### Компонент для добавления задачи (AddTodo.tsx)

```tsx
// src/components/AddTodo.tsx

import React, { useState } from 'react';
import { useAppDispatch } from '../app/hooks';
import { addTodo } from '../features/todos/todosSlice';

export const AddTodo: React.FC = () => {
  // ========== 1. ЛОКАЛЬНОЕ СОСТОЯНИЕ ДЛЯ ПОЛЯ ВВОДА ==========
  // Это НЕ Redux! Это обычный useState React для управления формой
  const [inputValue, setInputValue] = useState('');
  
  // ========== 2. ПОЛУЧАЕМ DISPATCH ==========
  // useAppDispatch - наш типизированный хук для отправки экшенов
  const dispatch = useAppDispatch();
  
  // ========== 3. ОБРАБОТЧИК ОТПРАВКИ ФОРМЫ ==========
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault(); // предотвращаем перезагрузку страницы
    
    // Проверяем, что поле не пустое
    if (inputValue.trim()) {
      // Отправляем экшен addTodo с текстом задачи
      dispatch(addTodo(inputValue));
      
      // Очищаем поле ввода
      setInputValue('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Введите новую задачу..."
      />
      <button type="submit">➕ Добавить</button>
    </form>
  );
};
```

#### Компонент для отображения одной задачи (TodoItem.tsx)

```tsx
// src/components/TodoItem.tsx

import React from 'react';
import { useAppDispatch } from '../app/hooks';
import { toggleTodo, deleteTodo } from '../features/todos/todosSlice';

// Тип пропсов компонента
interface TodoItemProps {
  id: number;        // id задачи
  text: string;      // текст задачи
  completed: boolean; // выполнена ли задача
}

export const TodoItem: React.FC<TodoItemProps> = ({ id, text, completed }) => {
  // Получаем dispatch для отправки экшенов
  const dispatch = useAppDispatch();
  
  return (
    <li style={{ 
      textDecoration: completed ? 'line-through' : 'none', // зачёркиваем выполненную
      opacity: completed ? 0.7 : 1,
    }}>
      {/* Кнопка переключения статуса */}
      <button onClick={() => dispatch(toggleTodo(id))}>
        {completed ? 'OK' : 'NOT'}
      </button>
      
      {/* Текст задачи */}
      <span style={{ margin: '0 10px' }}>{text}</span>
      
      {/* Кнопка удаления */}
      <button onClick={() => dispatch(deleteTodo(id))}>
        ❌
      </button>
    </li>
  );
};
```

#### Компонент для отображения списка задач (TodoList.tsx)

```tsx
// src/components/TodoList.tsx

import React from 'react';
import { useAppSelector } from '../app/hooks';
import { TodoItem } from './TodoItem';

export const TodoList: React.FC = () => {
  // ========== ЧИТАЕМ ДАННЫЕ ИЗ STORE ==========
  // useAppSelector позволяет получить данные из Redux store
  // Мы достаём массив задач из state.todos.items
  const todos = useAppSelector(state => state.todos.items);
  
  // Считаем статистику
  const totalCount = todos.length;
  const completedCount = todos.filter(todo => todo.completed).length;
  
  return (
    <div>
      {/* Статистика */}
      <div style={{ marginBottom: '20px', padding: '10px', background: '#f0f0f0' }}>
        <span>Всего задач: {totalCount} | </span>
        <span>Выполнено: {completedCount} | </span>
        <span>Осталось: {totalCount - completedCount}</span>
      </div>
      
      {/* Список задач */}
      {todos.length === 0 ? (
        <p style={{ color: '#999' }}>Нет задач. Добавьте первую!</p>
      ) : (
        <ul style={{ listStyle: 'none', padding: 0 }}>
          {todos.map(todo => (
            <TodoItem
              key={todo.id}  // React требует уникальный key
              id={todo.id}
              text={todo.text}
              completed={todo.completed}
            />
          ))}
        </ul>
      )}
    </div>
  );
};
```

### Шаг 6: Собираем всё в главном компоненте (App.tsx)

```tsx
// src/App.tsx

import React from 'react';
import { Provider } from 'react-redux';
import { store } from './app/store';
import { AddTodo } from './components/AddTodo';
import { TodoList } from './components/TodoList';

function App() {
  return (
    // ========== PROVIDER ОБЁРТЫВАЕТ ВСЁ ПРИЛОЖЕНИЕ ==========
    // Provider делает store доступным для всех компонентов
    <Provider store={store}>
      <div style={{ 
        maxWidth: '500px', 
        margin: '50px auto', 
        padding: '20px',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1>Список дел (Todo List)</h1>
        
        {/* Компонент для добавления задач */}
        <AddTodo />
        
        {/* Компонент для отображения списка задач */}
        <TodoList />
      </div>
    </Provider>
  );
}

export default App;
```

### Шаг 7: Запуск приложения

```bash
npm start
```

---

## Подробное объяснение каждой строчки

### Объяснение слайса (todosSlice.ts)

```typescript
// Интерфейс для одной задачи
interface Todo {
  id: number;        // число, уникальный идентификатор
  text: string;      // строка, текст задачи
  completed: boolean; // true/false, выполнена или нет
}
```
**Зачем:** TypeScript проверяет, что каждая задача имеет именно эти поля и правильные типы.

---

```typescript
const initialState: TodosState = {
  items: [],
};
```
**Зачем:** Определяем, как выглядят данные до любых действий. У нас пустой список задач.

---

```typescript
export const todosSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: { ... }
});
```
**Зачем:** `createSlice` — это фабрика, которая автоматически генерирует:
- Action creators (функции, создающие экшены)
- Reducer (функцию, обрабатывающую экшены)
- Типы экшенов (например, `'todos/addTodo'`)

---

```typescript
addTodo: (state, action: PayloadAction<string>) => {
  state.items.push({
    id: Date.now(),
    text: action.payload,
    completed: false,
  });
},
```
**Разбор:**
- `state` — текущее состояние слайса
- `action` — объект экшена, пришедший из компонента
- `PayloadAction<string>` — TypeScript говорит, что в `action.payload` будет строка
- `state.items.push(...)` — в классическом Redux так нельзя, но в RTK можно благодаря Immer (он под капотом создаёт копию)
- `Date.now()` — возвращает количество миллисекунд, используем как уникальный id

---

```typescript
toggleTodo: (state, action: PayloadAction<number>) => {
  const todo = state.items.find(todo => todo.id === action.payload);
  if (todo) {
    todo.completed = !todo.completed;
  }
},
```
**Разбор:**
- `PayloadAction<number>` — ожидаем число (id задачи)
- `find` — ищем задачу с нужным id
- `!todo.completed` — переворачиваем значение (true→false, false→true)

---

```typescript
deleteTodo: (state, action: PayloadAction<number>) => {
  state.items = state.items.filter(todo => todo.id !== action.payload);
},
```
**Разбор:**
- `filter` — создаёт новый массив, исключая задачу с указанным id
- `state.items = ...` — заменяем старый массив новым

---

### Объяснение store (store.ts)

```typescript
export const store = configureStore({
  reducer: {
    todos: todosReducer,
  },
});
```
**Зачем:** `configureStore` создаёт Redux store, который:
- Хранит всё состояние приложения
- Автоматически подключает thunk (для асинхронности)
- Автоматически подключает DevTools (для отладки)

**Важно:** Ключ `todos` будет использоваться в селекторах: `state.todos.items`

---

```typescript
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```
**Зачем:** Эти типы нужны для TypeScript, чтобы хуки `useSelector` и `useDispatch` знали структуру store.

---
