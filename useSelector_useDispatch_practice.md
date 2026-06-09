
## Что такое хуки (Hooks) вообще?

**Хуки** — это специальные функции в React, которые позволяют "подключаться" к возможностям React из функциональных компонентов.

**Простая аналогия:** Представьте, что React — это большой дом. Хуки — это специальные розетки и выключатели, через которые вы можете:
- Включить свет (использовать состояние)
- Открыть окно (получить доступ к контексту)
- Подключить телефон (использовать эффекты)

### Самые известные хуки React:
- `useState` — для хранения данных внутри компонента
- `useEffect` — для выполнения действий после рендера
- `useContext` — для доступа к глобальным данным

### Хуки из React Redux:
- `useSelector` — для чтения данных из Redux store
- `useDispatch` — для отправки экшенов в Redux store

---



### Проблема

Обычные хуки `useSelector` и `useDispatch` **не знают** какие данные хранятся в вашем Redux store:

```typescript
// Обычные хуки (из react-redux)
import { useSelector, useDispatch } from 'react-redux';

function MyComponent() {
  // Проблема: useSelector не знает структуру вашего store
  const user = useSelector(state => state.user); // ❌ TypeScript ругается: 
  // Параметр 'state' имеет неявный тип 'any'
  
  const dispatch = useDispatch(); // dispatch не знает типы ваших экшенов
}
```

### Решение

Мы **создаём свои собственные хуки** на основе существующих, но с предустановленными типами:

```typescript
// Создаём НОВЫЕ хуки с типами
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// Это НОВЫЙ хук, основанный на useDispatch, но с типом AppDispatch
export const useAppDispatch = () => useDispatch<AppDispatch>();

// Это НОВЫЙ хук, основанный на useSelector, но с типом RootState
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

**Что мы сделали?** Мы создали **обёртки** (свои хуки) вокруг стандартных хуков, добавив в них типы.

---

## Подробный разбор: что такое хук `useAppDispatch`?

Давайте разберём эту строчку кода по косточкам:

```typescript
export const useAppDispatch = () => useDispatch<AppDispatch>();
```

### Что здесь происходит:

1. **`export const useAppDispatch`** — мы создаём и экспортируем новую константу, которая называется `useAppDispatch`

2. **`= () =>`** — это стрелочная функция (лямбда), которая не принимает аргументов

3. **`useDispatch<AppDispatch>()`** — внутри мы вызываем оригинальный хук `useDispatch` и передаём ему тип `AppDispatch`

### Визуальное представление:

```
Оригинальный хук:     useDispatch (без типов)
                           │
                           ▼
Наша обёртка:         useAppDispatch = () => useDispatch<AppDispatch>()
                           │
                           ▼
Результат:            функция, которая возвращает dispatch с правильным типом
```

### Как это используется:

```typescript
// Вместо этого (без типов):
const dispatch = useDispatch(); // dispatch имеет тип 'any'

// Мы пишем это (с нашими хуками):
const dispatch = useAppDispatch(); // dispatch имеет тип AppDispatch ✅
```

---

## Разбор хука `useAppSelector`

```typescript
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Что здесь происходит:

1. **`useAppSelector: TypedUseSelectorHook<RootState>`** — мы говорим TypeScript, что наша новая переменная `useAppSelector` имеет тип `TypedUseSelectorHook<RootState>`

2. **`= useSelector`** — мы присваиваем этой переменной оригинальный хук `useSelector`

**Что такое `TypedUseSelectorHook`?** Это специальный тип из библиотеки `react-redux`, который помогает правильно типизировать `useSelector`.

### Визуальное представление:

```
Оригинальный хук:     useSelector (не типизирован)
                           │
                           ▼
Тип из TypeScript:    TypedUseSelectorHook<RootState>
                           │
                           ▼
Наш хук:              useAppSelector (теперь знает тип RootState)
```

### Как это используется:

```typescript
// Вместо этого:
const user = useSelector((state: RootState) => state.user); // нужно каждый раз указывать тип

// Мы пишем это:
const user = useAppSelector(state => state.user); // тип state уже известен! ✅
```

---

## Почему это называется "хуки"?

Потому что **результат** нашей работы — это новые функции, которые:

1. **Являются хуками** (они используют внутри себя оригинальные хуки React Redux)
2. **Следуют правилам хуков** (начинаются с `use`, могут вызываться только внутри компонентов)
3. **Предоставляют ту же функциональность**, но с добавленными типами

### Правила хуков (напоминание):

```typescript
// ✅ Правильно: хуки вызываются на верхнем уровне компонента
function MyComponent() {
  const dispatch = useAppDispatch();
  const user = useAppSelector(state => state.user);
}

// ❌ Неправильно: хуки нельзя вызывать внутри условий
function MyComponent() {
  if (something) {
    const dispatch = useAppDispatch(); // ОШИБКА!
  }
}

// ❌ Неправильно: хуки нельзя вызывать в циклах
function MyComponent() {
  for (let i = 0; i < 3; i++) {
    const user = useAppSelector(state => state.user); // ОШИБКА!
  }
}
```

---

## Аналогия для лучшего понимания

### Аналогия с пультом управления

Представьте, что у вас есть универсальный пульт (оригинальный `useDispatch`):
- Он может управлять любым телевизором
- Но он не знает кнопки вашего конкретного телевизора

Вы создаёте **свой пульт** (`useAppDispatch`):
- Вы берёте универсальный пульт и программируете его под свой телевизор
- Теперь он знает все кнопки именно вашего телевизора
- Вы даёте ему новое имя

```typescript
// Универсальный пульт (не знает ваш телевизор)
const remote = useDispatch(); // может всё, но без подсказок

// Ваш программированный пульт (знает ваш телевизор)
const myRemote = useAppDispatch(); // знает только ваши кнопки (экшены)
```

### Аналогия с калькулятором

Обычный `useSelector` — как калькулятор без памяти:
```typescript
// Каждый раз нужно говорить "возьми число из ячейки A1"
const value = useSelector((state: RootState) => state.counter.value);
```

Ваш `useAppSelector` — как калькулятор с памятью:
```typescript
// Калькулятор уже знает, где лежат данные
const value = useAppSelector(state => state.counter.value);
// ↑ TypeScript сам понимает, что state.counter.value — это число
```

---


---

## Частые вопросы 

### Вопрос 1: Это магия? Как TypeScript понимает, что нужно подставить?

**Ответ:** Не магия, а дженерики (generics). `useDispatch` — это функция-дженерик:

```typescript
// Упрощённый тип useDispatch
function useDispatch<AppDispatch>() {
  // возвращает dispatch с типом AppDispatch
}
```

Когда мы пишем `useDispatch<AppDispatch>()`, мы говорим TypeScript: "Используй тип `AppDispatch` для этого вызова".

### Вопрос 2: Можно ли не создавать свои хуки, а просто писать тип каждый раз?

**Ответ:** Можно, но не нужно. Это приведёт к дублированию кода:

```typescript
// Без своих хуков (плохо):
function Component1() {
  const dispatch = useDispatch<AppDispatch>();
  const user = useSelector((state: RootState) => state.user);
}

function Component2() {
  const dispatch = useDispatch<AppDispatch>(); // снова пишем
  const posts = useSelector((state: RootState) => state.posts); // снова пишем
}

// Со своими хуками (хорошо):
function Component1() {
  const dispatch = useAppDispatch();
  const user = useAppSelector(state => state.user);
}

function Component2() {
  const dispatch = useAppDispatch();
  const posts = useAppSelector(state => state.posts);
}
```

### Вопрос 3: Нарушаю ли я правила хуков, создавая свои?

**Ответ:** Нет! Создание своих хуков — это **рекомендуемый паттерн**. Это называется "композиция хуков" (composing hooks). Главное правило: ваш хук должен начинаться с `use`.

### Вопрос 4: Где хранить эти хуки?

**Ответ:** Обычно в файле `app/hooks.ts` или `store/hooks.ts`:

```
src/
├── app/
│   ├── store.ts       # store и типы
│   └── hooks.ts       # useAppDispatch, useAppSelector
├── features/
│   └── counter/
│       └── Counter.tsx
└── App.tsx
```

---

## Шпаргалка

```
┌─────────────────────────────────────────────────────────┐
│  ХУКИ — это функции, которые:                          │
│  • Начинаются с "use" (useState, useEffect и т.д.)     │
│  • Дают доступ к возможностям React                     │
│  • Можно создавать свои (кастомные хуки)                │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  В НАШЕМ СЛУЧАЕ:                                       │
│                                                         │
│  Оригинальные хуки:  useSelector, useDispatch          │
│         ↓                                               │
│  Добавляем типы:   useAppSelector, useAppDispatch      │
│         ↓                                               │
│  Это НАШИ кастомные хуки с предустановленными типами    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  ЗАПОМНИ:                                               │
│  • Хуки создаются один раз в hooks.ts                   │
│  • Хуки используются во всех компонентах                │
│  • Хуки делают код чище и безопаснее                    │
└─────────────────────────────────────────────────────────┘
```

**Итог:** В контексте этого вопроса "хуки" — это функции `useAppDispatch` и `useAppSelector`, которые мы создали сами на основе стандартных хуков Redux, добавив в них типы TypeScript. Это **кастомные хуки**, и они являются правильным решением для типизации в Redux Toolkit.

## Задание 1: выполните следующий пример или напишите свой на основе этого разбора

## Полный процесс создания своих хуков (пошагово)

### Шаг 1: Создаём store с типами

```typescript
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});

// 👇 Экспортируем типы для хуков
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Шаг 2: Создаём свои хуки

```typescript
// hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// Создаём хук useAppDispatch
export const useAppDispatch = () => useDispatch<AppDispatch>();

// Создаём хук useAppSelector
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Шаг 3: Используем в приложении

```typescript
// App.tsx
import { useAppDispatch, useAppSelector } from './hooks';

function Counter() {
  const dispatch = useAppDispatch();  // ← наш хук
  const count = useAppSelector(state => state.counter.value); // ← наш хук
  
  return (
    <button onClick={() => dispatch(increment())}>
      Счётчик: {count}
    </button>
  );
}
```

---

## Что происходит "под капотом"?

Когда вы вызываете `useAppDispatch()`:

```typescript
// Вы пишете:
const dispatch = useAppDispatch();

// На самом деле вызывается:
const dispatch = (() => useDispatch<AppDispatch>())();

// useDispatch<AppDispatch>() возвращает функцию dispatch,
// которая уже знает типы всех ваших экшенов
```

Когда вы вызываете `useAppSelector(state => state.counter)`:

```typescript
// Вы пишете:
const value = useAppSelector(state => state.counter);

// useAppSelector внутри знает, что state имеет тип RootState,
// и может проверить, что свойство counter существует
```



# Что делает `useDispatch` — подробный разбор 

## Простое объяснение

**`useDispatch`** — это хук из библиотеки `react-redux`, который даёт вам возможность **отправлять (диспатчить) экшены** в Redux store.

**Аналогия из жизни:** Представьте, что Redux store — это штаб компании. `useDispatch` — это ваш служебный телефон, по которому вы можете звонить в штаб и говорить: "Сделайте то-то". Когда вы звоните, вы отправляете сообщение (экшен), а штаб (store) обрабатывает его и меняет своё состояние.

---

## Зачем нужен `useDispatch`?

В Redux данные нельзя изменить напрямую. Единственный способ изменить данные в store — **отправить экшен**. А `useDispatch` даёт вам функцию, с помощью которой вы можете эти экшены отправлять.

### Проблема без `useDispatch`:

```javascript
// ❌ Нельзя просто так взять и изменить store
store.user.name = "Анна"; // ТАК НЕЛЬЗЯ!
```

### Решение с `useDispatch`:

```javascript
// ✅ Правильно: отправляем экшен, который говорит "как изменить"
const dispatch = useDispatch();
dispatch(setName("Анна")); // отправляем команду
```

---

## Как использовать `useDispatch`
## Золотые правила использования `useDispatch`

1. **Всегда получайте dispatch через `useDispatch()`** — не используйте `store.dispatch` напрямую

2. **Не вызывайте dispatch в теле компонента** — только в обработчиках или useEffect

3. **Импортируйте action creators** из слайса, не пишите объекты вручную

4. **В TypeScript создайте `useAppDispatch`** для типизации

5. **Dispatch стабилен** — его не нужно добавлять в зависимости useEffect (если он там не используется как зависимость)

**Запомните:** `useDispatch` — это ваш канал связи с Redux store. Только через него вы можете сказать Redux: "Измени данные вот так". Без dispatch вы можете только читать данные (через `useSelector`), но не изменять их.

### Базовый пример

```javascript
import { useDispatch } from 'react-redux';
import { increment, decrement } from './counterSlice';

function CounterButtons() {
  // 1. Получаем функцию dispatch
  const dispatch = useDispatch();
  
  // 2. Используем dispatch для отправки экшенов
  return (
    <div>
      <button onClick={() => dispatch(increment())}>
        +1
      </button>
      <button onClick={() => dispatch(decrement())}>
        -1
      </button>
    </div>
  );
}
```

**Что здесь происходит по шагам:**

1. **`useDispatch()`** — вызываем хук, он возвращает функцию `dispatch`
2. **`dispatch(increment())`** — вызываем `dispatch` и передаём ей экшен, который создаёт `increment()`
3. Redux получает этот экшен и вызывает соответствующий редюсер
4. Редюсер меняет состояние
5. Все компоненты, которые используют `useSelector`, автоматически обновляются

---

## Что такое "экшен" (action)?

Экшен — это обычный объект JavaScript, у которого обязательно есть поле `type`. Обычно есть ещё поле `payload` (данные).

```javascript
// Экшен — это объект
const incrementAction = {
  type: 'counter/increment'  // тип экшена
};

const setNameAction = {
  type: 'user/setName',
  payload: 'Анна'            // данные
};
```

### Как создаются экшены?

С помощью **action creator** (функции, которая создаёт экшен):

```javascript
// Action creator (из слайса Redux Toolkit)
const increment = () => ({ type: 'counter/increment' });

// Использование:
dispatch(increment()); // dispatch получает объект { type: 'counter/increment' }
```

---


## Важные особенности `useDispatch`

### 1. Dispatch всегда один и тот же

```javascript
function MyComponent() {
  const dispatch = useDispatch();
  // dispatch — это стабильная ссылка (она не меняется между рендерами)
  // Поэтому её НЕ нужно добавлять в зависимости useEffect (если она не используется по-особенному)
}
```

### 2. Dispatch можно вызывать где угодно

```javascript
function MyComponent() {
  const dispatch = useDispatch();
  
  // ✅ В обработчиках событий
  const handleClick = () => {
    dispatch(increment());
  };
  
  // ✅ В useEffect
  useEffect(() => {
    dispatch(fetchData());
  }, []);
  
  // ❌ НЕЛЬЗЯ в теле компонента (вызовет бесконечный цикл)
  dispatch(increment()); // ОШИБКА!
  
  return <button onClick={handleClick}>Клик</button>;
}
```

### 3. Dispatch можно передавать в дочерние компоненты

```javascript
function Parent() {
  const dispatch = useDispatch();
  return <Child onAction={() => dispatch(increment())} />;
}

function Child({ onAction }) {
  return <button onClick={onAction}>+1</button>;
}
```

---

## Распространённые ошибки с `useDispatch`

### Ошибка 1: Вызов dispatch в теле компонента

```javascript
// ❌ ОШИБКА
function BadComponent() {
  const dispatch = useDispatch();
  dispatch(increment()); // Не делайте так!
  return <div>...</div>;
}
```

**Что произойдёт:** Бесконечный цикл рендеринга (dispatch → обновление состояния → ререндер → dispatch → ...)

**Как исправить:** Использовать useEffect или обработчики событий.

```javascript
// ✅ ПРАВИЛЬНО
function GoodComponent() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(increment()); // Только один раз при монтировании
  }, []);
  
  return <div>...</div>;
}
```

### Ошибка 2: Забыли импортировать action creator

```javascript
// ❌ ОШИБКА
import { useDispatch } from 'react-redux';
// забыли импортировать increment

function Counter() {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment())}>+</button>;
  // Ошибка: increment is not defined
}
```

```javascript
// ✅ ПРАВИЛЬНО
import { useDispatch } from 'react-redux';
import { increment } from './counterSlice'; // добавили импорт

function Counter() {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment())}>+</button>;
}
```

### Ошибка 3: Вызов dispatch без action creator

```javascript
// ❌ ОШИБКА (неправильный синтаксис)
dispatch(increment); // забыли вызвать increment()

// ✅ ПРАВИЛЬНО
dispatch(increment()); // вызываем функцию increment(), она возвращает экшен
```

### Ошибка 4: Неправильная передача payload

```javascript
// ❌ ОШИБКА
dispatch(incrementByAmount); // не передали значение
dispatch(incrementByAmount(5)); // так правильно

// ❌ ОШИБКА: ждёт число, передали строку
dispatch(incrementByAmount("пять")); // TypeScript подсветит ошибку
```

---

## `useDispatch` с TypeScript

В TypeScript нужно типизировать `useDispatch`, чтобы он знал типы ваших экшенов.

### Без типизации (плохо):

```typescript
const dispatch = useDispatch(); // тип: any (нет подсказок)
dispatch(incorrectAction()); // ошибка не подсветится
```

### С типизацией (хорошо):

```typescript
// hooks.ts
import { useDispatch } from 'react-redux';
import type { AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();

// Counter.tsx
import { useAppDispatch } from './hooks';

const dispatch = useAppDispatch(); // тип: AppDispatch (есть подсказки!)
dispatch(increment()); // ✅ работает с подсказками
```

---

## Сравнение: с `useDispatch` и без него

### Без Redux (локальный state):

```javascript
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

### С Redux (через dispatch):

```javascript
import { useDispatch, useSelector } from 'react-redux';
import { increment } from './counterSlice';

function Counter() {
  const dispatch = useDispatch();
  const count = useSelector(state => state.counter.value);
  
  return (
    <button onClick={() => dispatch(increment())}>
      {count}
    </button>
  );
}
```

**Разница:** В локальном состоянии вы меняете данные напрямую (`setCount`). В Redux вы отправляете команду (`dispatch`), а Redux сам решает, как изменить данные.

---

## Как работает `useDispatch` внутри (упрощённо)

Упрощённая версия того, как мог бы выглядеть `useDispatch`:

```javascript
// Псевдокод, упрощённая реализация
function useDispatch() {
  // Получаем store из контекста React Redux
  const store = useContext(ReduxContext);
  
  // Возвращаем функцию dispatch из store
  return store.dispatch;
}
```

Когда вы вызываете `dispatch(action)`:

```javascript
dispatch(increment());

// ↓ внутри происходит примерно так:

// 1. Экшен попадает в store
// 2. Store прогоняет экшен через все редюсеры
// 3. Редюсеры вычисляют новое состояние
// 4. Store обновляет состояние
// 5. Все подписанные компоненты получают уведомление
// 6. React перерендеривает эти компоненты
```

---

## Шпаргалка по `useDispatch`

```
┌─────────────────────────────────────────────────────────────┐
│  useDispatch — это хук для отправки экшенов в Redux store  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Что возвращает:    функцию dispatch                        │
│  Что делает:        отправляет экшены в store               │
│  Где использовать:  в компонентах React                     │
│  Когда вызывать:    в обработчиках событий или в useEffect  │
│  Нельзя вызывать:   в теле компонента (бесконечный цикл)    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ПРИМЕР:                                                    │
│                                                             │
│  const dispatch = useDispatch();                           │
│  dispatch(increment());                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```



## Задание 2: выполните следующий пример или напишите свой на основе этого разбора

Давайте создадим простое приложение счётчика, чтобы увидеть `useDispatch` в действии.

### Шаг 1: Создаём slice

```javascript
// counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: (state) => state + 1,
    decrement: (state) => state - 1,
    incrementByAmount: (state, action) => state + action.payload
  }
});

// Экспортируем action creators
export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

### Шаг 2: Создаём store

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

### Шаг 3: Используем dispatch в компоненте

```javascript
// Counter.jsx
import { useDispatch } from 'react-redux';
import { useSelector } from 'react-redux';
import { increment, decrement, incrementByAmount } from './counterSlice';

function Counter() {
  const dispatch = useDispatch();     // получаем dispatch
  const count = useSelector(state => state.counter.value); // читаем значение
  
  return (
    <div>
      <h1>Счётчик: {count}</h1>
      
      {/* Отправляем экшены по клику */}
      <button onClick={() => dispatch(increment())}>
        +1
      </button>
      
      <button onClick={() => dispatch(decrement())}>
        -1
      </button>
      
      <button onClick={() => dispatch(incrementByAmount(5))}>
        +5
      </button>
    </div>
  );
}
```

---

## Что возвращает `useDispatch`?

`useDispatch()` возвращает функцию `dispatch`, которая умеет принимать:

1. **Объект-экшен** (редко, но можно)
2. **Функцию** (если используется thunk middleware для асинхронности)

```javascript
const dispatch = useDispatch();

// 1. Отправка объекта-экшена
dispatch({ type: 'counter/increment' });

// 2. Отправка экшена через action creator
dispatch(increment());

// 3. Отправка асинхронного экшена (thunk)
dispatch(fetchUserData(123));
```

---

