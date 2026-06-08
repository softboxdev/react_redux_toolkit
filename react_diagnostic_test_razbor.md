
---

## Часть 1. Redux Toolkit (база) 

**1. Какой метод в Redux Toolkit создаёт store с автоматической настройкой middleware (включая thunk)?**
- A) `configureStore()`
- B) `createStore()`
- C) `buildStore()`
- D) `initStore()`

**2. Что возвращает `createSlice()`?**
- A) Только reducer
- B) Только actions
- C) Объект с полями `reducer`, `actions`, `name`
- D) Готовый store

**3. Как правильно типизировать `useSelector` и `useDispatch` в TypeScript для RTK?**
- A) `useSelector` и `useDispatch`
- B) Создать хуки `useAppDispatch` и `useAppSelector` с предустановленными типами RootState и AppDispatch
- C) Использовать `useStore` вместо них
- D) Типизировать каждый вызов отдельно через generics

**4. Какой хук позволяет получить доступ к dispatch в компоненте?**
- A) `useReducer`
- B) `useDispatch`
- C) `useThunk`
- D) `useStore`

**5. Что из перечисленного **не** является преимуществом Redux Toolkit перед классическим Redux?**
- A) Меньше шаблонного кода
- B) Автоматическое использование Immer для иммутабельности
- C) Обязательное использование классов компонентов
- D) Встроенный thunk middleware

---

## Часть 2. React Router v6 (база) 

**6. Какой компонент используется для определения маршрутов в React Router v6?**
- A) `<Route>`
- B) `<Router>`
- C) `<Path>`
- D) `<Link>`

**7. Как получить доступ к параметрам URL (например, `/user/:id`) в компоненте?**
- A) `useParams()`
- B) `useRouteMatch()`
- C) `useLocation()`
- D) `useHistory()`

**8. Как программно перейти на другой маршрут в React Router v6?**
- A) `history.push('/about')`
- B) `useNavigate()` → `navigate('/about')`
- C) `useHistory().push('/about')`
- D) `redirect('/about')`

**9. Что делает компонент `<Outlet />`?**
- A) Показывает все маршруты сразу
- B) Рендерит дочерние маршруты внутри родительского layout
- C) Перенаправляет на 404
- D) Обрабатывает ошибки

**10. Как создать активную ссылку с классом "active", когда URL совпадает?**
- A) `<NavLink to="/" activeClassName="active">`
- B) `<Link to="/" active="active">`
- C) `<NavLink to="/" className={({ isActive }) => isActive ? "active" : ""}>`
- D) `<ActiveLink to="/" className="active">`

---

## Часть 3. Интеграция RTK + React Router 

**11. Вам нужно при переходе на страницу пользователя загрузить его данные через асинхронный thunk. Где лучше вызвать `dispatch(fetchUser(id))`?**
- A) В `useEffect` с зависимостью от `id` из `useParams`
- B) Непосредственно в теле компонента без условий
- C) В редюсере слайса
- D) В `useMemo`

**12. Какой паттерн позволяет избежать повторной загрузки данных при возврате на страницу (кэширование в Redux)?**
- A) Использовать `useSelector` с проверкой наличия данных перед dispatch
- B) Всегда загружать заново при монтировании
- C) Использовать локальный `useState` вместо Redux
- D) Вызвать dispatch в `useLayoutEffect`

**13. Что произойдёт, если в компоненте написать такой код:**
```jsx
const { id } = useParams();
const dispatch = useDispatch();
dispatch(fetchItem(id)); // вне useEffect
```
- A) Всё будет работать отлично
- B) React выдаст ошибку "Cannot update a component while rendering"
- C) Данные загрузятся, но каждый рендер будет отправлять новый запрос
- D) Redux Toolkit автоматически оптимизирует вызов

**14. Как организовать защищённый маршрут (PrivateRoute), который перенаправляет на логин, если пользователь не авторизован (используя Redux store)?**
- A) Использовать `<Route element={isAuth ? <Outlet /> : <Navigate to="/login" />}>`
- B) Запретить маршрут через middleware Redux
- C) Удалить маршрут из роутера
- D) Использовать `useHistory` в момент монтирования

**15. Как получить текущий pathname (например, `/dashboard/settings`) в компоненте?**
- A) `useLocation().pathname`
- B) `usePath()`
- C) `getCurrentPath()`
- D) `useRoute().path`

---

## Часть 4. Практическое задание 

**16. Напишите Redux Toolkit slice для управления постами и страницу списка постов с переходом на детальную страницу.**

Требования:
1. Создать `postsSlice` с:
   - initialState: `{ items: [], status: 'idle', error: null }`
   - асинхронный thunk `fetchPosts`
   - редюсеры для обработки загрузки/успеха/ошибки

2. Компонент `PostsList`:
   - Диспатчит `fetchPosts` при монтировании (один раз)
   - Отображает список постов (заголовки)
   - Каждый пост – ссылка на `/posts/:id`

3. Компонент `PostDetails`:
   - Получает `id` из URL
   - Показывает полный пост (можно заглушку, логика отображения из состояния)

4. Настройте роутинг: главная `/` → список постов, `/posts/:id` → детали

*Напишите ключевые части кода (slice, компоненты, роутинг).*

---

## Часть 5. Диагностика ошибок 

**17. Почему этот код вызывает бесконечный рендер? Как исправить?**
```jsx
function UserProfile() {
  const dispatch = useDispatch();
  const user = useSelector(state => state.user.data);
  
  dispatch(fetchUser()); // ← здесь
  
  return <div>{user?.name}</div>;
}
```

**18. В чём ошибка в настройке роутера?**
```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" component={Home} />
    <Route path="/about" element={<About />} />
    <Route path="/contact" element={<Contact />} />
  </Routes>
</BrowserRouter>
```

---

## Ответы и критерии

### Ответы:

**Часть 1:** 1-A, 2-C, 3-B, 4-B, 5-C  
**Часть 2:** 6-A, 7-A, 8-B, 9-B, 10-C  
**Часть 3:** 11-A, 12-A, 13-C, 14-A, 15-A  


Ниже представлен **подробный разбор ответов** на Часть 1 теста по Redux Toolkit.  
Для каждого вопроса объясняется: почему правильный ответ верен, почему остальные не подходят, и даётся дополнительный контекст для углублённого понимания.

---

## Вопрос 1
**Какой метод в Redux Toolkit создаёт store с автоматической настройкой middleware (включая thunk)?**  
**Правильный ответ: A) `configureStore()`**

### Почему это правильно
`configureStore()` — это главный API Redux Toolkit для создания store. Он:
- Автоматически подключает `redux-thunk` (для асинхронных экшенов)
- Включает middleware для проверки иммутабельности и сериализуемости (в development)
- Автоматически настраивает Redux DevTools
- Упрощает добавление кастомных middleware

### Почему не подходят другие варианты
- **B) `createStore()`** — это оригинальный метод из классического Redux. Он не настраивает middleware автоматически, требует ручного подключения `applyMiddleware(thunk)`, не включает проверки.
- **C) `buildStore()`** — такого метода в Redux Toolkit нет. Возможно, путаница с `buildCreateSlice` или кастомными билдерами.
- **D) `initStore()`** — не существует в Redux Toolkit. Иногда используется в других библиотеках (например, в некоторых стейт-менеджерах).

### Дополнительный контекст
```javascript
// Классический Redux (без RTK):
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
const store = createStore(rootReducer, applyMiddleware(thunk));

// Redux Toolkit:
import { configureStore } from '@reduxjs/toolkit';
const store = configureStore({ reducer: rootReducer }); // thunk уже внутри

// BUG (mutation - изменение состояния напрямую - опасно)
state.users.push(newUser);

// Good practice
// Нельзя изменять состяние напрямую, нужно создавать копию
return { ...state, users: [...state.users, newUser] };


```

---

## Вопрос 2
**Что возвращает `createSlice()`?**  
**Правильный ответ: C) Объект с полями `reducer`, `actions`, `name`**

### Почему это правильно
`createSlice()` принимает объект с `name`, `initialState`, `reducers` и возвращает объект, содержащий:
- `reducer` — готовый reducer, который можно передать в `configureStore`
- `actions` — объект со всеми action creators (сгенерированными из поля `reducers`)
- `name` — имя слайса (используется внутренне)
- (опционально) `caseReducers` — оригинальные редюсеры

### Почему не подходят другие варианты
- **A) Только reducer** — неполно. Хотя reducer — самое важное, слайс возвращает и actions тоже.
- **B) Только actions** — тоже неполно. Без reducer слайс бесполезен.
- **D) Готовый store** — абсолютно неверно. Слайс — это часть store (кусочек состояния и логика), а не сам store.

### Дополнительный контекст
```javascript
const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1
  }
});

console.log(counterSlice.reducer);    // function
console.log(counterSlice.actions);    // { increment: fn, decrement: fn }
console.log(counterSlice.name);       // 'counter'

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```
createSlice({ name: 'users', initialState: [], reducers: {...} })
                           │
                           ▼
              ┌─────────────────────────┐
              │      ОБЪЕКТ SLICE       │
              ├─────────────────────────┤
              │ name: "users"           │
              │                         │
              │ reducer: fn()           │──────→ для configureStore()
              │                         │
              │ actions: {              │
              │   addTodo: fn()         │──────→ для dispatch() в компонентах
              │   toggleTodo: fn()      │
              │   deleteTodo: fn()      │
              │ }                       │
              │                         │
              │ caseReducers: {...}     │──────→ (редко нужно 1% использования)
              └─────────────────────────┘

---

## Вопрос 3
**Как правильно типизировать `useSelector` и `useDispatch` в TypeScript для RTK?**  
**Правильный ответ: B) Создать хуки `useAppDispatch` и `useAppSelector` с предустановленными типами `RootState` и `AppDispatch`**

### Почему это правильно
Официальная документация Redux Toolkit рекомендует создавать типизированные версии хуков:
- `useAppDispatch` — предустановлен с типом `AppDispatch` (который возвращает `configureStore`)
- `useAppSelector` — предустановлен с типом `RootState` (тип всего состояния)

Это избавляет от необходимости указывать типы в каждом компоненте и предотвращает ошибки.

### Почему не подходят другие варианты
- **A) Использовать `useSelector` и `useDispatch` напрямую** — они не знают о типах вашего store. Придётся каждый раз писать `useSelector<RootState>(...)`.
- **C) Использовать `useStore` вместо них** — `useStore` даёт доступ ко всему store, но это менее удобно и не рекомендуемый подход для чтения данных или dispatch.
- **D) Типизировать каждый вызов отдельно через generics** — это работает (`useSelector<RootState>(...)`), но создаёт дублирование кода. Лучше один раз создать типизированные хуки.

### Дополнительный контекст
```typescript
// app/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// В компоненте:
const dispatch = useAppDispatch();
const user = useAppSelector(state => state.user);

```
┌─────────────────────────────────────────────────────────┐
│  ХУКИ — это функции, которые:                           │
│  • Начинаются с "use" (useState, useEffect, useContext..│
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

---

## Вопрос 4
**Какой хук позволяет получить доступ к dispatch в компоненте?**  
**Правильный ответ: B) `useDispatch`**

### Почему это правильно
`useDispatch()` — это хук из `react-redux`, который возвращает ссылку на функцию `dispatch` из Redux store. Без него нельзя отправить экшен.

### Почему не подходят другие варианты
- **A) `useReducer`** — это встроенный хук React для локального состояния, не связанный с Redux.
- **C) `useThunk`** — такого хука нет в React или Redux. Thunk — это middleware, а не хук.
- **D) `useStore`** — возвращает сам объект store, но не `dispatch`. Можно вызвать `useStore().dispatch()`, но это неудобно и не рекомендуется.

### Дополнительный контекст
```javascript
import { useDispatch } from 'react-redux';
import { increment } from './counterSlice';

function Button() {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment())}>+1</button>;
}
```

---

## Вопрос 5
**Что из перечисленного **не** является преимуществом Redux Toolkit перед классическим Redux?**  
**Правильный ответ: C) Обязательное использование классов компонентов**

### Почему это правильно
Redux Toolkit **не требует** классовых компонентов. Он прекрасно работает и с функциональными компонентами (и даже предпочитает их через хуки `useDispatch`/`useSelector`). Обязательное использование классов — это недостаток, а не преимущество, и RTK его не вводит.

### Почему остальные варианты — преимущества
- **A) Меньше шаблонного кода** — да, RTK сокращает boilerplate (action types, action creators, switch в reducer).
- **B) Автоматическое использование Immer** — да, можно "мутировать" состояние в редюсерах, Immer создаёт неизменяемые обновления под капотом.
- **D) Встроенный thunk middleware** — да, `configureStore` подключает `redux-thunk` автоматически, не нужно настраивать вручную.

### Дополнительный контекст
```javascript
// Классический Redux (много кода):
const INCREMENT = 'INCREMENT';
const increment = () => ({ type: INCREMENT });
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case INCREMENT: return state + 1;
    default: return state;
  }
};

// Redux Toolkit (меньше кода):
const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1  // Immer позволяет так писать
  }
});
```

---

## Итог по Части 1

| Вопрос | Правильный ответ | Ключевая идея |
|--------|------------------|----------------|
| 1 | A (`configureStore`) | Автоматическая настройка store с thunk |
| 2 | C (объект с reducer, actions, name) | Слайс возвращает и reducer, и actions |
| 3 | B (кастомные хуки `useAppDispatch`/`useAppSelector`) | Типизация один раз для всего проекта |
| 4 | B (`useDispatch`) | Хук для отправки экшенов |
| 5 | C (обязательное использование классов) | Это ложное утверждение, RTK с хуками работает отлично |


---

## Вопрос 6
**Какой middleware чаще всего используют для работы с асинхронными экшенами в Redux?**  
**Правильный ответ: A) Redux Thunk**

### Объяснение

**Что такое middleware простыми словами?**  
Middleware — это как "перехватчик" (или помощник), который сидит между тем, как мы отправили экшен (action), и тем, как он попадёт в редюсер. Он может:
- Остановить экшен
- Изменить экшен
- Отправить другой экшен
- Дождаться асинхронной операции (запроса на сервер)

**Что такое асинхронный экшен?**  
Это действие, которое требует времени. Например: запрос на сервер за списком пользователей. Redux без middleware умеет работать только с синхронными объектами.

**Redux Thunk (вариант А)** — самый простой и популярный middleware. Он позволяет писать экшены-функции вместо экшенов-объектов. Пример:
```javascript
// Обычный экшен-объект (синхронный)
{ type: 'INCREMENT' }

// Асинхронный экшен с Thunk (функция)
function fetchUser(id) {
  return async function(dispatch) {
    dispatch({ type: 'LOADING_START' });
    const user = await api.getUser(id);  // ждём ответ от сервера
    dispatch({ type: 'LOADING_SUCCESS', payload: user });
  }
}
```
95% проектов начинают с Thunk, потому что он простой и встроен в Redux Toolkit.

**Почему не подходят остальные варианты:**

- **B) Redux Logger** — этот middleware не для асинхронности, а для логирования. Он просто выводит в консоль каждый экшен и новое состояние, чтобы легче отлаживать код. Он не умеет ждать промисы.

- **C) Redux Persist** — это middleware для сохранения состояния в localStorage (чтобы после перезагрузки страницы данные не терялись). К асинхронным экшенам он отношения не имеет.

- **D) Redux Saga** — тоже умеет работать с асинхронностью, но это **более сложный** инструмент. Он использует генераторы (function* и yield) и подходит для очень сложных сценариев (например, отмена запросов, паузы, гонки запросов). Новички обычно начинают с Thunk, поэтому в вопросе "чаще всего используют" — ответ Thunk.

**Запоминалка:**  
- Thunk — Тяжёлая асинхронность для начинающих  
- Logger — Логи в консоль  
- Persist — Пережить перезагрузку  
- Saga — Сложная, как сага (эпическое приключение)

---

## Вопрос 7
**Что делает `useSelector`?**  
**Правильный ответ: B) Позволяет извлечь данные из Redux store и подписаться на их обновления**

### Объяснение 

**useSelector** — это хук (функция-помощник) из библиотеки `react-redux`. Он делает две важные вещи:

1. **Извлекает данные** из store. Вы пишете функцию, которая говорит: "Дай мне кусочек состояния". Например:
```javascript
const userName = useSelector(state => state.user.name);
```

2. **Подписывается на изменения**. Если данные в store изменятся (другой компонент отправит экшен), React автоматически обновит ваш компонент с новыми данными.

**Почему это важно:** Без `useSelector` вам пришлось бы вручную подписываться на store через `store.subscribe()` и самим следить за обновлениями — это было бы очень муторно.

**Разбор ошибочных ответов:**

- **A) Отправляет экшен** — это работа `useDispatch()`, а не `useSelector`. Это частая путаница у новичков.

```javascript
const dispatch = useDispatch();  // для отправки
dispatch(increment());           // отправляем экшен

const value = useSelector(state => state.value); // для чтения
```

- **C) Создаёт новый store** — нет, store создаётся один раз функцией `configureStore()` (или `createStore` в классическом Redux). `useSelector` только читает из уже существующего store.

- **D) Комбинирует редюсеры** — это делает `combineReducers()`. Редюсеры — это чистые функции, которые говорят, как состояние должно измениться в ответ на экшен. `combineReducers` склеивает несколько таких функций в одну большую.

**Запоминалка :**  
- `useSelector` = **SELECT** (выбрать данные из store)  
- `useDispatch` = **DISPATCH** (отправить экшен)  
- `combineReducers` = **COMBINE** (объединить редюсеры)

---

## Вопрос 8
**Какая функция объединяет несколько редюсеров в один?**  
**Правильный ответ: A) `combineReducers`**

### Объяснение

**Зачем объединять редюсеры?**  
В реальном приложении у вас много разных данных: пользователи, посты, корзина товаров, тема оформления. Каждую часть логично обрабатывать в своём редюсере. Например:

```javascript
// Редюсер для пользователя
function userReducer(state = {}, action) { ... }

// Редюсер для постов
function postsReducer(state = [], action) { ... }

// Редюсер для корзины
function cartReducer(state = [], action) { ... }
```

Но Redux store может принять только **один** корневой редюсер. Поэтому мы **объединяем** их в один большой объект с помощью `combineReducers`:

```javascript
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  user: userReducer,
  posts: postsReducer,
  cart: cartReducer
});

// Теперь состояние store выглядит так:
// { user: {...}, posts: [...], cart: [...] }
```

**Разбор ошибочных ответов:**

- **B) `mergeReducers`** — такой функции нет в Redux. Возможно, путаница с `combineReducers` или с функцией `merge` из других библиотек.

- **C) `applyMiddleware`** — эта функция подключает middleware (например, Thunk). Она не объединяет редюсеры, а добавляет "прослойки" между dispatch и редюсером.

- **D) `createStore`** — эта функция создаёт сам store, принимая корневой редюсер (часто результат `combineReducers`). Она не объединяет редюсеры, а использует уже объединённый.

**Аналогия из жизни:**  
- `combineReducers` как папка-скоросшиватель: собирает отдельные листы (редюсеры) в одну общую папку (корневой редюсер).  
- `createStore` как открытие офиса: берёт эту папку и создаёт хранилище (store).  
- `applyMiddleware` как установка охраны на входе.

---

## Вопрос 9
**Как исправить ошибку «Cannot update a component while rendering a different component» при использовании `dispatch` внутри `useSelector`?**  
**Правильный ответ: A) Перенести `dispatch` в `useEffect`**

### Объяснение 

**Что это за ошибка?**  
Это очень частая ошибка . React говорит: "Ты сейчас находишься в процессе рендера (отрисовки) компонента А, а пытаешься обновить компонент Б через dispatch — так нельзя!"

**Почему возникает ошибка?**  
Посмотрите на неправильный код:
```javascript
function MyComponent() {
  const dispatch = useDispatch();
  const data = useSelector(state => state.data);
  
  dispatch(fetchData());  // ❌ Ошибка! dispatch прямо во время рендера
  
  return <div>{data}</div>;
}
```

**Что здесь происходит:**  
1. React начинает рендерить компонент  
2. Встречает `dispatch(fetchData())` — это запускает асинхронный экшен  
3. Когда запрос завершится, Redux изменит состояние  
4. Изменение состояния вызовет перерендер компонента  
5. Но React ещё не закончил текущий рендер → ошибка

**Как это исправить (вариант А — правильный):**  
Нужно сказать React: "Не запускай dispatch сразу во время рендера, а сделай это **после** того, как компонент отрендерится в первый раз". Для этого используется `useEffect`:

```javascript
import { useEffect } from 'react';

function MyComponent() {
  const dispatch = useDispatch();
  const data = useSelector(state => state.data);
  
  useEffect(() => {
    dispatch(fetchData());  // ✅ Запускается после рендера
  }, []);  // Пустой массив = запустить один раз при монтировании
  
  return <div>{data}</div>;
}
```

**Разбор ошибочных ответов (почему они не помогут):**

- **B) Использовать `useCallback`** — `useCallback` запоминает функцию между рендерами, но он не меняет момент вызова. Dispatch всё равно выполнится во время рендера, просто функция будет "закеширована". Ошибка останется.

- **C) Вызвать dispatch сразу после useSelector без условий** — это как раз то, что вызывает ошибку. Без `useEffect` это гарантированно приведёт к той же проблеме.

- **D) Использовать `React.memo`** — этот инструмент оптимизирует рендеры, предотвращая лишние перерисовки компонента, если пропсы не изменились. Он никак не влияет на момент вызова dispatch. Ошибка никуда не денется.

**Золотое правило новичка:**  
> Если dispatch должен произойти в ответ на событие (клик, загрузка страницы, таймер) — используйте `useEffect` или обработчики событий (`onClick`), но никогда не вызывайте dispatch напрямую в теле компонента.

---

## Вопрос 10
**В Redux Toolkit функция `createSlice` возвращает:**  
**Правильный ответ: B) Объект с `reducer` и `actions`**

### Объяснение

**Что такое slice (кусочек)?**  
Slice — это способ собрать в одном месте:
- название части состояния (например, `'user'`)
- начальное состояние (например, `{ name: '', age: 0 }`)
- редюсеры (функции, которые изменяют состояние)
- экшен-креаторы (функции, создающие экшены)

Раньше в классическом Redux это всё нужно было писать отдельно и вручную связывать. `createSlice` делает это автоматически.

**Что именно возвращает `createSlice`?**  
Давайте посмотрим на пример:

```javascript
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1
  }
});

console.log(counterSlice);
// Вывод в консоль (упрощённо):
// {
//   name: 'counter',
//   reducer: function(state, action) {...},
//   actions: {
//     increment: function() {...},
//     decrement: function() {...}
//   },
//   caseReducers: {...} // внутренняя техническая деталь
// }
```

**Зачем нужны эти поля:**
- `reducer` — передаётся в `configureStore` при создании store
- `actions` — используется в компонентах для отправки экшенов
- `name` — используется внутри для генерации типов экшенов

**Правильный ответ B** наиболее полный, потому что для разработчика важнее всего именно `reducer` и `actions`. `name` и `caseReducers` — это внутренности.

**Разбор ошибочных ответов:**

- **A) Только редюсер** — неверно, потому что в реальном коде вы всегда импортируете и reducer, и actions:
```javascript
export default counterSlice.reducer;        // редюсер
export const { increment, decrement } = counterSlice.actions; // экшен-креаторы
```
Если бы возвращался только редюсер, откуда бы взялись `counterSlice.actions`?

- **C) Новый store** — абсолютно неверно. Store создаётся функцией `configureStore()` и может включать много слайсов. Один слайс — это только часть store.

- **D) Middleware конфигурацию** — путаница с `configureStore`, который может принимать middleware. Slice к middleware не имеет отношения.

**Аналогия из жизни:**  
Представьте, что Redux store — это большой холодильник.  
- `createSlice` — это создание **контейнера с едой**: у него есть название (name), содержимое (initialState), инструкция по изменению содержимого (reducers). Контейнер возвращает:  
  * **Крышку с кнопками** (actions) — нажимая на кнопки, вы меняете еду  
  * **Сам контейнер с едой** (reducer) — вы ставите его в холодильник  
- `configureStore` — это сам холодильник, куда вы ставите контейнеры.

**Частая ошибка :**  
Некоторые думают, что `createSlice` нужно вызывать внутри `configureStore`. Нет, сначала создаёте слайсы, потом передаёте их редюсеры в `configureStore`:

```javascript
// Правильный порядок
const userSlice = createSlice(...);
const postsSlice = createSlice(...);

const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    posts: postsSlice.reducer
  }
});
```

---

## Итоговая таблица для запоминания (Часть 2)

| Вопрос | Правильный ответ | Простое правило для запоминания |
|--------|------------------|--------------------------------|
| 6 | Redux Thunk | Для асинхронности — Thunk (Saga для профи) |
| 7 | useSelector | Читает и подписывается (SELECT) |
| 8 | combineReducers | Собирает редюсеры в один объект |
| 9 | useEffect + dispatch | Не вызывай dispatch при рендере, только после |
| 10 | Объект с reducer и actions | Slice = reducer + actions в одной упаковке |

