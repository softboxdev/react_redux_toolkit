# React Router v6 — Полный обзор + практика

## Что такое React Router?

**React Router** — это библиотека для навигации между разными страницами в React-приложении без перезагрузки страницы.

### Простая аналогия

Представьте, что ваше React-приложение — это **книга**:
- **Обычный сайт** (без React Router) — как книга, где для перехода к следующей главе нужно закрыть книгу и открыть новую (перезагрузка страницы)
- **React Router** — как книга с **закладками** и **оглавлением**, где вы можете мгновенно перепрыгивать между главами, не закрывая книгу

---

## Как это работает: общая схема

```
┌─────────────────────────────────────────────────────────────────┐
│                         БРАУЗЕР                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    URL: /about                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   React Router                            │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  Сравнивает URL с объявленными маршрутами (Routes) │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                          │                                  │  │
│  │                          ▼                                  │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  Находит совпадение: path="/about" → элемент About │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    React Component                        │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                   <About />                         │  │  │
│  │  │            (рендерится на экране)                   │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```
## Золотые правила React Router

1. **Всегда оборачивайте приложение в `<BrowserRouter>`** — иначе маршруты не будут работать

2. **Группируйте маршруты в `<Routes>`** — только один `Routes` обрабатывает URL

3. **Порядок маршрутов важен** — `<Routes>` рендерит первый совпадающий маршрут

4. **Используйте `element`, а не `component`** — в v6 синтаксис `element={<Component />}`

5. **Не забывайте про `index` маршруты** — для путей, совпадающих с родительским

6. **Redux store живёт отдельно** — смена маршрута НЕ очищает Redux состояние

**Запомните:** React Router делает ваше React-приложение похожим на многостраничный сайт, но с преимуществами SPA (быстрота, сохранение состояния, плавные переходы). Это неотъемлемая часть современной React-разработки!
---

## Основные компоненты React Router

### 1. BrowserRouter (обёртка)

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>  {/* ← Оборачиваем всё приложение */}
      {/* Здесь будут маршруты */}
    </BrowserRouter>
  );
}
```

**Что делает:** Слушает изменения URL в адресной строке и предоставляет контекст для всех дочерних компонентов.

### 2. Routes (контейнер маршрутов)

```jsx
import { Routes, Route } from 'react-router-dom';

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="/contact" element={<Contact />} />
</Routes>
```

**Что делает:** Группирует все маршруты и находит первый, который соответствует текущему URL.

### 3. Route (отдельный маршрут)

```jsx
<Route path="/users/:id" element={<UserProfile />} />
```

**Что делает:** Связывает URL-шаблон с компонентом, который нужно отрендерить.

### 4. Link (навигация)

```jsx
import { Link } from 'react-router-dom';

<Link to="/about">О нас</Link>
```

**Что делает:** Создаёт ссылку, которая меняет URL **без перезагрузки страницы**.

---

## Диаграмма: Последовательность обработки при клике на ссылку

```
Пользователь кликает на <Link to="/about">
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ШАГ 1: React Router перехватывает клик                          │
│ - Предотвращает стандартную перезагрузку страницы               │
│ - Изменяет URL в адресной строке (без перезагрузки)             │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ШАГ 2: Обновляется история браузера                             │
│ - window.history.pushState() добавляет новую запись             │
│ - URL меняется с "/" на "/about"                                │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ШАГ 3: Срабатывает слушатель изменений URL                      │
│ - BrowserRouter отслеживает событие popstate                    │
│ - Уведомляет все компоненты об изменении URL                    │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ШАГ 4: Routes пересчитывает совпадения                          │
│ - Берёт текущий URL: "/about"                                   │
│ - Перебирает все Route в поисках совпадения                     │
│ - Находит <Route path="/about" element={<About />}>            │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ШАГ 5: React перерендеривает компоненты                         │
│ - Старый компонент (<Home />) удаляется                         │
│ - Новый компонент (<About />) монтируется                       │
│ - DOM обновляется                                               │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ РЕЗУЛЬТАТ: Пользователь видит страницу About                    │
│ - Страница не перезагружалась!                                  │
│ - Состояние Redux сохранилось!                                  │
│ - Анимация плавная!                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Диаграмма: Рендер компонентов при смене маршрута

```
Временная шкала:
═══════════════════════════════════════════════════════════════════

Исходное состояние (URL: /)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│                        <App />                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    <Header />                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     <Routes />                          │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │  <Route path="/" element={<Home />}> → ✅ РЕНДЕРИТСЯ │  │   │
│  │  │  <Route path="/about" element={<About />}> → ❌     │  │   │
│  │  │  <Route path="/contact" element={<Contact />}> → ❌ │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    <Footer />                           │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Пользователь кликает на ссылку "/about"
═══════════════════════════════════════════════════════════════════

ШАГ 1: URL меняется на "/about"
───────────────────────────────────────────────────────────────────

ШАГ 2: Routes пересчитывает маршруты
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│                     <Routes />                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  <Route path="/" element={<Home />}> → ❌ (не совпадает)   │  │
│  │  <Route path="/about" element={<About />}> → ✅ СОВПАДАЕТ  │  │
│  │  <Route path="/contact" element={<Contact />}> → ❌        │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

ШАГ 3: React сравнивает старый и новый виртуальный DOM
───────────────────────────────────────────────────────────────────
Старый DOM:                    Новый DOM:
┌──────────────────┐          ┌──────────────────┐
│ <Home />         │          │ <About />        │
│  - Все state     │    →     │  - Новый state   │
│  - useEffect     │          │  - Новые эффекты │
└──────────────────┘          └──────────────────┘

ШАГ 4: React выполняет обновление
───────────────────────────────────────────────────────────────────
1. componentWillUnmount() у <Home /> (очистка эффектов)
2. componentDidMount() у <About /> (новые эффекты)
3. DOM обновляется

Результат (URL: /about)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│                        <App />                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    <Header />                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     <Routes />                          │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │  <Route path="/" element={<Home />}> → ❌         │  │   │
│  │  │  <Route path="/about" element={<About />}> → ✅   │  │   │
│  │  │  <Route path="/contact" element={<Contact />}> → ❌│  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    <Footer />                           │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Диаграмма: Взаимодействие с Redux (хранилищем памяти)

```
Важно: Redux store НЕ СБРАСЫВАЕТСЯ при смене маршрута!
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                         REDUX STORE                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  {                                                        │  │
│  │    user: { name: "Анна", isAuth: true },  ← ЖИВЁТ!        │  │
│  │    posts: { items: [...], status: "loaded" }, ← ЖИВЁТ!    │  │
│  │    cart: { items: [...] }                   ← ЖИВЁТ!       │  │
│  │  }                                                        │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐             │
│         │                    │                    │             │
│         ▼                    ▼                    ▼             │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐      │
│  │  <Home />   │      │ <About />   │      │ <Contact /> │      │
│  │             │      │             │      │             │      │
│  │ Читает:     │      │ Читает:     │      │ Читает:     │      │
│  │ - user      │      │ - user      │      │ - user      │      │
│  │ - posts     │      │ - posts     │      │ - cart      │      │
│  └─────────────┘      └─────────────┘      └─────────────┘      │
│         │                    │                    │             │
│         └────────────────────┼────────────────────┘             │
│                              │                                   │
│                    Все компоненты имеют доступ                   │
│                    к одним и тем же данным!                      │
└─────────────────────────────────────────────────────────────────┘

Пример: Пользователь залогинился на странице /login
═══════════════════════════════════════════════════════════════════
1. dispatch(login({ name: "Анна" }))
2. Redux store обновляется: user.isAuth = true
3. Пользователь переходит на /profile
4. <Profile /> читает user.isAuth = true через useSelector
5. Данные НЕ ПОТЕРЯЛИСЬ! (store живёт независимо от маршрутов)
```

---

## Диаграмма: Жизненный цикл компонента при навигации

```
Время →
────────────────────────────────────────────────────────────────────

Страница Home (URL: /)
┌─────────────────────────────────────────────────────────────────┐
│ <Home />                                                        │
│ 1. constructor()                                                │
│ 2. render() → "Домашняя страница"                               │
│ 3. componentDidMount() / useEffect(() => {}, [])                │
│    └─ dispatch(fetchPosts())                                    │
└─────────────────────────────────────────────────────────────────┘
                │
                │ Пользователь кликает <Link to="/about">
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ <Home /> (начинает процесс размонтирования)                     │
│ 4. componentWillUnmount() / cleanup функции                     │
│    └─ отмена запросов, очистка таймеров                         │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ <About /> (монтируется)                                         │
│ 1. constructor()                                                │
│ 2. render() → "О нас"                                           │
│ 3. componentDidMount() / useEffect(() => {}, [])                │
│    └─ dispatch(fetchAboutData())                                │
└─────────────────────────────────────────────────────────────────┘

ВАЖНО: Redux store между этими этапами НЕ ОЧИЩАЕТСЯ!
```

---

## Диаграмма: Вложенные маршруты (Nested Routes) и Outlet

```
Структура URL: /users/123/posts/456
═══════════════════════════════════════════════════════════════════

Компонентная иерархия:
┌─────────────────────────────────────────────────────────────────┐
│                         <App />                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      <Header />                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      <Routes />                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  <Route path="/users/:userId" element={<Users />}>  │  │  │
│  │  │                                                      │  │  │
│  │  │  ┌────────────────────────────────────────────────┐ │  │  │
│  │  │  │              <Users />                         │ │  │  │
│  │  │  │  ┌──────────────────────────────────────────┐  │ │  │  │
│  │  │  │  │         <UserSidebar />                  │  │ │  │  │
│  │  │  │  └──────────────────────────────────────────┘  │ │  │  │
│  │  │  │  ┌──────────────────────────────────────────┐  │ │  │  │
│  │  │  │  │              <Outlet />  ←───────────────┐│  │ │  │  │
│  │  │  │  └──────────────────────────────────────────┘│  │ │  │  │
│  │  │  │  ┌──────────────────────────────────────────┐│  │ │  │  │
│  │  │  │  │           <UserFooter />                 ││  │ │  │  │
│  │  │  │  └──────────────────────────────────────────┘│  │ │  │  │
│  │  │  └────────────────────────────────────────────────┘ │  │  │
│  │  │                                                      │  │  │
│  │  │  ┌────────────────────────────────────────────────┐ │  │  │
│  │  │  │  ВЛОЖЕННЫЕ МАРШРУТЫ (определены в <Users />)   │ │  │  │
│  │  │  │                                                │ │  │  │
│  │  │  │  <Route path="posts/:postId" element={<Post />}>│ │  │  │
│  │  │  │                      ↑                          │ │  │  │
│  │  │  │              рендерится в Outlet                │ │  │  │
│  │  │  └────────────────────────────────────────────────┘ │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      <Footer />                           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

Как это работает с URL /users/123/posts/456:
═══════════════════════════════════════════════════════════════════

1. BrowserRouter видит URL: /users/123/posts/456
2. Корневой Routes ищет совпадение:
   → находит <Route path="/users/:userId" element={<Users />}>
3. Рендерит <Users /> (вместе с его Sidebar, Outlet, Footer)
4. Внутри <Users /> есть свой Routes:
   → <Route path="posts/:postId" element={<Post />}>
5. Этот Route рендерит <Post /> ВНУТРИ <Outlet />
6. Итоговая страница:
   ┌─────────────────────────────────────┐
   │              Header                  │
   ├────────────┬────────────────────────┤
   │ UserSidebar│      Post              │
   │            │   (из Outlet)          │
   │            │                        │
   ├────────────┴────────────────────────┤
   │              Footer                  │
   └─────────────────────────────────────┘
```

---

## Диаграмма: useParams — получение параметров из URL

```
URL: /users/123/posts/456
                  │      │
                  │      └─ postId = "456"
                  └─ userId = "123"

Код компонента:
═══════════════════════════════════════════════════════════════════

function UserPost() {
  // useParams извлекает параметры из URL
  const { userId, postId } = useParams();
  // userId = "123"
  // postId = "456"
  
  const dispatch = useDispatch();
  
  useEffect(() => {
    // Используем параметры для загрузки данных
    dispatch(fetchPost({ userId, postId }));
  }, [userId, postId]); // ← если параметры меняются, запрос повторяется
  
  return (
    <div>
      <h1>Пост {postId} пользователя {userId}</h1>
    </div>
  );
}

// Настройка маршрута:
<Route path="/users/:userId/posts/:postId" element={<UserPost />} />
```

---

## Диаграмма: useNavigate — программная навигация

```
Ситуация: после успешного логина нужно перенаправить пользователя

┌─────────────────────────────────────────────────────────────────┐
│                      <LoginForm />                              │
│                                                                 │
│  const navigate = useNavigate();                                │
│                                                                 │
│  const handleSubmit = async (e) => {                           │
│    e.preventDefault();                                          │
│    try {                                                        │
│      await loginAPI(email, password);                           │
│      dispatch(setUser(userData));                               │
│                                                                 │
│      // ✅ Программный переход на другую страницу               │
│      navigate('/dashboard');                                    │
│                                                                 │
│    } catch (error) {                                            │
│      setError('Неверный логин или пароль');                     │
│    }                                                            │
│  };                                                             │
└─────────────────────────────────────────────────────────────────┘

Другие примеры использования navigate:
═══════════════════════════════════════════════════════════════════

navigate('/about');              // переход на /about
navigate(-1);                    // назад (как кнопка "Назад")
navigate(1);                     // вперёд
navigate('/profile', { state: { from: '/login' } }); // с передачей данных
navigate('/dashboard', { replace: true }); // заменить текущую страницу в истории
```

---

## Диаграмма: Защищённые маршруты (PrivateRoute)

```
Поток проверки авторизации:
═══════════════════════════════════════════════════════════════════

Пользователь пытается перейти на /dashboard
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    <PrivateRoute />                             │
│                                                                 │
│  const isAuth = useSelector(state => state.user.isAuth);       │
│                                                                 │
│  if (isAuth === undefined) {                                   │
│    return <LoadingSpinner />;  // ждём загрузки данных          │
│  }                                                              │
│                                                                 │
│  return isAuth ? <Outlet /> : <Navigate to="/login" />;        │
└─────────────────────────────────────────────────────────────────┘
                │
        ┌───────┴───────┐
        │               │
    isAuth=true     isAuth=false
        │               │
        ▼               ▼
┌───────────┐    ┌───────────┐
│ <Outlet />│    │<Navigate  │
│           │    │  to="/    │
│  Показываем│    │  login" />│
│  dashboard │    │           │
│           │    │Перенаправ-│
│           │    │ляем на    │
│           │    │логин      │
└───────────┘    └───────────┘

Использование в роутере:
═══════════════════════════════════════════════════════════════════

<Routes>
  {/* Публичные маршруты */}
  <Route path="/login" element={<Login />} />
  <Route path="/register" element={<Register />} />
  
  {/* Защищённые маршруты (группа) */}
  <Route element={<PrivateRoute />}>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile" element={<Profile />} />
    <Route path="/settings" element={<Settings />} />
  </Route>
</Routes>
```

---

## Диаграмма: useLocation — получение информации о текущем URL

```
Что содержит useLocation():
═══════════════════════════════════════════════════════════════════

const location = useLocation();

console.log(location);
{
  pathname: '/users/123',     // путь без параметров
  search: '?tab=posts&page=2', // GET-параметры
  hash: '#comments',           // якорь
  state: { from: '/home' },    // данные, переданные через navigate
  key: 'abc123'                // уникальный ключ
}

Как использовать:
═══════════════════════════════════════════════════════════════════

function Breadcrumbs() {
  const location = useLocation();
  const pathnames = location.pathname.split('/').filter(x => x);
  
  // /users/123/posts → ['users', '123', 'posts']
  
  return (
    <div>
      {pathnames.map((name, index) => (
        <span key={index}>
          {name}
          {index < pathnames.length - 1 && ' > '}
        </span>
      ))}
    </div>
  );
}

// Отслеживание переходов для аналитики:
useEffect(() => {
  // Отправляем в Google Analytics
  ga.send('pageview', location.pathname + location.search);
}, [location]);
```

---

## Полный пример приложения

```jsx
// App.jsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { Provider } from 'react-redux';
import { store } from './store';

// Компоненты
import { Layout } from './components/Layout';
import { Home } from './pages/Home';
import { About } from './pages/About';
import { UserProfile } from './pages/UserProfile';
import { PrivateRoute } from './components/PrivateRoute';
import { Login } from './pages/Login';

function App() {
  return (
    <Provider store={store}>
      <BrowserRouter>
        <Routes>
          {/* Публичные маршруты */}
          <Route path="/login" element={<Login />} />
          
          {/* Маршруты с общим Layout */}
          <Route element={<Layout />}>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/users/:id" element={<UserProfile />} />
            <Route path="products/*" element={<ProductsPage />}>
            
            {/* Защищённые маршруты */}
            <Route element={<PrivateRoute />}>
              <Route path="/dashboard" element={<Dashboard />} />
              <Route path="/settings" element={<Settings />} />
            </Route>
          </Route>
          
          {/* 404 - не найдено */}
          <Route path="*" element={<NotFound />} />
        </Routes>
      </BrowserRouter>
    </Provider>
  );
}

// Layout.jsx (общий для всех страниц)
import { Outlet, Link } from 'react-router-dom';

export function Layout() {
  return (
    <div>
      <nav>
        <Link to="/">Главная</Link>
        <Link to="/about">О нас</Link>
        <Link to="/dashboard">Личный кабинет</Link>
      </nav>
      
      <main>
        <Outlet />  {/* ← Здесь рендерятся текущие страницы */}
      </main>
      
      <footer>© 2024</footer>
    </div>
  );
}
```

---

## Шпаргалка по хукам React Router

| Хук | Назначение | Пример |
|-----|-----------|--------|
| `useParams()` | Получить параметры из URL | `const { id } = useParams()` |
| `useNavigate()` | Программная навигация | `const navigate = useNavigate(); navigate('/about')` |
| `useLocation()` | Информация о текущем URL | `const { pathname } = useLocation()` |
| `useSearchParams()` | Работа с GET-параметрами | `const [params, setParams] = useSearchParams()` |

---

# Полный пример приложения: Магазин с анимацией и подробным объяснением

## Структура проекта

```
my-shop/
├── src/
│   ├── app/
│   │   ├── store.js                 # Redux store
│   │   └── hooks.js                 # Типизированные хуки
│   ├── components/
│   │   ├── Layout.jsx               # Главный Layout
│   │   ├── PrivateRoute.jsx         # Защищённый маршрут
│   │   ├── ProductCard.jsx          # Карточка товара
│   │   └── LoadingSpinner.jsx       # Индикатор загрузки
│   ├── pages/
│   │   ├── Home.jsx                 # Главная страница
│   │   ├── About.jsx                # О нас
│   │   ├── Login.jsx                # Страница входа
│   │   ├── Dashboard.jsx            # Личный кабинет
│   │   ├── Settings.jsx             # Настройки
│   │   └── NotFound.jsx             # 404 страница
│   ├── modules/
│   │   └── products/
│   │       ├── ProductsPage.jsx     # Страница товаров
│   │       ├── ProductsList.jsx     # Список товаров
│   │       ├── ProductDetail.jsx    # Детали товара
│   │       └── productsSlice.js     # Redux слайс товаров
│   ├── features/
│   │   └── auth/
│   │       └── authSlice.js         # Redux слайс авторизации
│   ├── App.jsx
│   └── index.js
```

---

## Часть 1: Создаём Redux store

### app/store.js

```javascript
import { configureStore } from '@reduxjs/toolkit';
import authReducer from '../features/auth/authSlice';
import productsReducer from '../modules/products/productsSlice';

// Создаём store
export const store = configureStore({
  reducer: {
    auth: authReducer,      // управление авторизацией
    products: productsReducer // управление товарами
  }
});

// Типы для TypeScript (если используете)
export const RootState = store.getState;
export const AppDispatch = store.dispatch;
```

### app/hooks.js (для TypeScript)

```javascript
import { useDispatch, useSelector } from 'react-redux';

export const useAppDispatch = () => useDispatch();
export const useAppSelector = useSelector;
```

---

## Часть 2: Создаём слайсы

### features/auth/authSlice.js

```javascript
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  isAuthenticated: false,  // авторизован ли пользователь
  user: null,              // данные пользователя
  loading: false,
  error: null
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    loginStart: (state) => {
      state.loading = true;
      state.error = null;
    },
    loginSuccess: (state, action) => {
      state.isAuthenticated = true;
      state.user = action.payload;
      state.loading = false;
    },
    loginFailure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    },
    logout: (state) => {
      state.isAuthenticated = false;
      state.user = null;
    }
  }
});

export const { loginStart, loginSuccess, loginFailure, logout } = authSlice.actions;
export default authSlice.reducer;
```

### modules/products/productsSlice.js

```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Асинхронный thunk для загрузки товаров
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async () => {
    // Имитация API запроса
    const response = await fetch('https://jsonplaceholder.typicode.com/posts');
    const data = await response.json();
    return data.slice(0, 10); // берём первые 10 товаров
  }
);

const initialState = {
  items: [],
  status: 'idle', // 'idle' | 'loading' | 'succeeded' | 'failed'
  error: null
};

const productsSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  }
});

export default productsSlice.reducer;
```

---

## Часть 3: Компоненты

### components/Layout.jsx (с относительными и абсолютными ссылками)

```jsx
import { Outlet, Link, NavLink } from 'react-router-dom';
import { useSelector } from 'react-redux';

export function Layout() {
  const isAuthenticated = useSelector(state => state.auth.isAuthenticated);
  const userName = useSelector(state => state.auth.user?.name);
  
  return (
    <div>
      {/* Шапка сайта */}
      <header style={styles.header}>
        <div style={styles.logo}>
          <Link to="/">🛍️ Мой Магазин</Link>
        </div>
        
        <nav style={styles.nav}>
          {/* АБСОЛЮТНЫЕ ССЫЛКИ (начинаются с /) */}
          {/* Всегда ведут от корня сайта, независимо от текущего URL */}
          
          <NavLink 
            to="/" 
            style={({ isActive }) => ({
              ...styles.link,
              ...(isActive ? styles.activeLink : {})
            })}
            end
          >
            Главная
          </NavLink>
          
          <NavLink 
            to="/about" 
            style={({ isActive }) => ({
              ...styles.link,
              ...(isActive ? styles.activeLink : {})
            })}
          >
            О нас
          </NavLink>
          
          <NavLink 
            to="/products" 
            style={({ isActive }) => ({
              ...styles.link,
              ...(isActive ? styles.activeLink : {})
            })}
          >
            Товары
          </NavLink>
          
          {isAuthenticated ? (
            <>
              <NavLink 
                to="/dashboard" 
                style={({ isActive }) => ({
                  ...styles.link,
                  ...(isActive ? styles.activeLink : {})
                })}
              >
                Личный кабинет
              </NavLink>
              
              <span style={styles.userName}>Привет, {userName}!</span>
            </>
          ) : (
            <NavLink 
              to="/login" 
              style={({ isActive }) => ({
                ...styles.link,
                ...(isActive ? styles.activeLink : {})
              })}
            >
              Войти
            </NavLink>
          )}
        </nav>
      </header>
      
      {/* Основной контент - здесь будут подставляться страницы */}
      <main style={styles.main}>
        <Outlet />
      </main>
      
      {/* Подвал */}
      <footer style={styles.footer}>
        <p>© 2024 Мой Магазин. Все права защищены.</p>
        <div>
          <Link to="/about">О нас</Link> | 
          <Link to="/contact">Контакты</Link> | 
          <Link to="/privacy">Политика конфиденциальности</Link>
        </div>
      </footer>
    </div>
  );
}

const styles = {
  header: {
    backgroundColor: '#2c3e50',
    color: 'white',
    padding: '1rem',
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center'
  },
  logo: {
    fontSize: '1.5rem',
    fontWeight: 'bold'
  },
  nav: {
    display: 'flex',
    gap: '1rem',
    alignItems: 'center'
  },
  link: {
    color: 'white',
    textDecoration: 'none',
    padding: '0.5rem 1rem',
    borderRadius: '4px'
  },
  activeLink: {
    backgroundColor: '#3498db'
  },
  userName: {
    marginLeft: '1rem',
    padding: '0.5rem',
    backgroundColor: '#27ae60',
    borderRadius: '4px'
  },
  main: {
    minHeight: 'calc(100vh - 200px)',
    padding: '2rem'
  },
  footer: {
    backgroundColor: '#34495e',
    color: 'white',
    textAlign: 'center',
    padding: '1rem',
    marginTop: 'auto'
  }
};
```

### components/PrivateRoute.jsx

```jsx
import { Navigate, Outlet } from 'react-redux';
import { useSelector } from 'react-redux';

export function PrivateRoute() {
  const isAuthenticated = useSelector(state => state.auth.isAuthenticated);
  const loading = useSelector(state => state.auth.loading);
  
  if (loading) {
    return <div>Проверка авторизации...</div>;
  }
  
  // Если не авторизован - перенаправляем на логин
  // Абсолютная ссылка на /login
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}
```

---

## Часть 4: Страницы

### pages/Home.jsx

```jsx
import { Link } from 'react-router-dom';
import { useSelector, useDispatch } from 'react-redux';
import { useEffect } from 'react';
import { fetchProducts } from '../modules/products/productsSlice';

export function Home() {
  const dispatch = useDispatch();
  const { items, status } = useSelector(state => state.products);
  
  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchProducts());
    }
  }, [status, dispatch]);
  
  return (
    <div>
      <h1>Добро пожаловать в наш магазин!</h1>
      <p>Лучшие товары по лучшим ценам</p>
      
      <h2>Популярные товары</h2>
      
      {status === 'loading' && <div>Загрузка товаров...</div>}
      
      <div style={styles.productsGrid}>
        {items.slice(0, 3).map(product => (
          <div key={product.id} style={styles.productCard}>
            <h3>{product.title}</h3>
            <p>{product.body.substring(0, 100)}...</p>
            {/* ОТНОСИТЕЛЬНАЯ ССЫЛКА - будет работать от текущего URL */}
            <Link to={`/products/${product.id}`}>Подробнее →</Link>
          </div>
        ))}
      </div>
      
      <div style={styles.buttons}>
        {/* Абсолютная ссылка - всегда ведёт на /products */}
        <Link to="/products" style={styles.button}>
          Смотреть все товары
        </Link>
      </div>
    </div>
  );
}

const styles = {
  productsGrid: {
    display: 'grid',
    gridTemplateColumns: 'repeat(auto-fit, minmax(300px, 1fr))',
    gap: '1rem',
    marginTop: '1rem'
  },
  productCard: {
    border: '1px solid #ddd',
    borderRadius: '8px',
    padding: '1rem',
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
  },
  buttons: {
    marginTop: '2rem',
    textAlign: 'center'
  },
  button: {
    display: 'inline-block',
    padding: '0.75rem 1.5rem',
    backgroundColor: '#3498db',
    color: 'white',
    textDecoration: 'none',
    borderRadius: '4px'
  }
};
```

### pages/About.jsx

```jsx
import { Link } from 'react-router-dom';

export function About() {
  return (
    <div>
      <h1>О нас</h1>
      <p>Мы - команда профессионалов, создающих лучшие товары для вас.</p>
      
      <h2>Наши преимущества</h2>
      <ul>
        <li>Высокое качество</li>
        <li>Быстрая доставка</li>
        <li>Отличная поддержка</li>
      </ul>
      
      {/* Абсолютная ссылка */}
      <Link to="/" style={styles.link}>← На главную</Link>
    </div>
  );
}

const styles = {
  link: {
    display: 'inline-block',
    marginTop: '2rem',
    color: '#3498db',
    textDecoration: 'none'
  }
};
```

### pages/Login.jsx

```jsx
import { useState } from 'react';
import { useDispatch } from 'react-redux';
import { useNavigate, Link } from 'react-router-dom';
import { loginStart, loginSuccess, loginFailure } from '../features/auth/authSlice';

export function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useDispatch();
  const navigate = useNavigate();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Имитация авторизации
    dispatch(loginStart());
    
    setTimeout(() => {
      if (email === 'user@example.com' && password === '123456') {
        dispatch(loginSuccess({ name: 'Анна', email }));
        // Программная навигация после успешного входа
        navigate('/dashboard'); // абсолютная ссылка
      } else {
        dispatch(loginFailure('Неверный email или пароль'));
      }
    }, 1000);
  };
  
  return (
    <div style={styles.container}>
      <div style={styles.formContainer}>
        <h1>Вход в аккаунт</h1>
        
        <form onSubmit={handleSubmit} style={styles.form}>
          <input
            type="email"
            placeholder="Email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            style={styles.input}
            required
          />
          
          <input
            type="password"
            placeholder="Пароль"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            style={styles.input}
            required
          />
          
          <button type="submit" style={styles.button}>
            Войти
          </button>
        </form>
        
        <p style={styles.hint}>
          Тестовые данные: user@example.com / 123456
        </p>
        
        {/* Абсолютная ссылка на главную */}
        <Link to="/" style={styles.link}>← Вернуться на главную</Link>
      </div>
    </div>
  );
}

const styles = {
  container: {
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    minHeight: '60vh'
  },
  formContainer: {
    width: '100%',
    maxWidth: '400px',
    padding: '2rem',
    border: '1px solid #ddd',
    borderRadius: '8px',
    boxShadow: '0 2px 8px rgba(0,0,0,0.1)'
  },
  form: {
    display: 'flex',
    flexDirection: 'column',
    gap: '1rem'
  },
  input: {
    padding: '0.75rem',
    border: '1px solid #ddd',
    borderRadius: '4px',
    fontSize: '1rem'
  },
  button: {
    padding: '0.75rem',
    backgroundColor: '#3498db',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    fontSize: '1rem'
  },
  hint: {
    marginTop: '1rem',
    fontSize: '0.875rem',
    color: '#666',
    textAlign: 'center'
  },
  link: {
    display: 'block',
    marginTop: '1rem',
    textAlign: 'center',
    color: '#3498db',
    textDecoration: 'none'
  }
};
```

### pages/Dashboard.jsx (защищённая страница)

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { Link, Outlet } from 'react-router-dom';
import { logout } from '../features/auth/authSlice';

export function Dashboard() {
  const user = useSelector(state => state.auth.user);
  const dispatch = useDispatch();
  
  const handleLogout = () => {
    dispatch(logout());
  };
  
  return (
    <div style={styles.container}>
      <div style={styles.sidebar}>
        <h3>Меню</h3>
        <nav style={styles.nav}>
          {/* ОТНОСИТЕЛЬНЫЕ ССЫЛКИ - работают от /dashboard */}
          <Link to="" style={styles.link}>Обзор</Link>
          <Link to="profile" style={styles.link}>Профиль</Link>
          <Link to="orders" style={styles.link}>Заказы</Link>
          <Link to="settings" style={styles.link}>Настройки</Link>
          {/* Абсолютная ссылка для выхода */}
          <button onClick={handleLogout} style={styles.logoutBtn}>
            Выйти
          </button>
        </nav>
      </div>
      
      <div style={styles.content}>
        <h2>Добро пожаловать, {user?.name}!</h2>
        {/* Здесь будут дочерние маршруты дашборда */}
        <Outlet />
      </div>
    </div>
  );
}

const styles = {
  container: {
    display: 'flex',
    gap: '2rem'
  },
  sidebar: {
    width: '250px',
    padding: '1rem',
    backgroundColor: '#f8f9fa',
    borderRadius: '8px'
  },
  nav: {
    display: 'flex',
    flexDirection: 'column',
    gap: '0.5rem'
  },
  link: {
    padding: '0.5rem',
    color: '#333',
    textDecoration: 'none',
    borderRadius: '4px',
    transition: 'background-color 0.3s'
  },
  logoutBtn: {
    marginTop: '1rem',
    padding: '0.5rem',
    backgroundColor: '#e74c3c',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer'
  },
  content: {
    flex: 1,
    padding: '1rem'
  }
};
```

---

## Часть 5: Модуль товаров (с вложенными маршрутами)

### modules/products/ProductsPage.jsx

```jsx
import { Routes, Route, Link, Outlet, useLocation } from 'react-router-dom';
import { ProductsList } from './ProductsList';
import { ProductDetail } from './ProductDetail';

export function ProductsPage() {
  const location = useLocation();
  
  return (
    <div>
      <h1>Каталог товаров</h1>
      
      {/* Хлебные крошки */}
      <div style={styles.breadcrumbs}>
        <Link to="/">Главная</Link> / 
        <Link to="/products">Товары</Link>
        {location.pathname !== '/products' && (
          <> / <span>{location.pathname.split('/').pop()}</span></>
        )}
      </div>
      
      <div style={styles.container}>
        {/* Боковая панель с категориями */}
        <aside style={styles.sidebar}>
          <h3>Категории</h3>
          <ul style={styles.categoryList}>
            <li><Link to="/products">Все товары</Link></li>
            {/* Абсолютные ссылки на категории */}
            <li><Link to="/products?category=electronics">Электроника</Link></li>
            <li><Link to="/products?category=clothing">Одежда</Link></li>
            <li><Link to="/products?category=books">Книги</Link></li>
          </ul>
        </aside>
        
        {/* Основной контент с вложенными маршрутами */}
        <main style={styles.content}>
          <Routes>
            {/* ОТНОСИТЕЛЬНЫЕ маршруты (без ведущего /) */}
            <Route index element={<ProductsList />} />
            {/* :id - динамический параметр */}
            <Route path=":id" element={<ProductDetail />} />
          </Routes>
        </main>
      </div>
    </div>
  );
}

const styles = {
  breadcrumbs: {
    marginBottom: '1rem',
    padding: '0.5rem',
    backgroundColor: '#f8f9fa',
    borderRadius: '4px'
  },
  container: {
    display: 'flex',
    gap: '2rem'
  },
  sidebar: {
    width: '200px'
  },
  categoryList: {
    listStyle: 'none',
    padding: 0
  },
  content: {
    flex: 1
  }
};
```

### modules/products/ProductsList.jsx

```jsx
import { useSelector } from 'react-redux';
import { Link } from 'react-router-dom';
import { ProductCard } from '../../components/ProductCard';

export function ProductsList() {
  const { items, status } = useSelector(state => state.products);
  
  if (status === 'loading') {
    return <div>Загрузка товаров...</div>;
  }
  
  if (status === 'failed') {
    return <div>Ошибка загрузки товаров</div>;
  }
  
  return (
    <div>
      <h2>Все товары</h2>
      <div style={styles.grid}>
        {items.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}

const styles = {
  grid: {
    display: 'grid',
    gridTemplateColumns: 'repeat(auto-fill, minmax(280px, 1fr))',
    gap: '1.5rem'
  }
};
```

### modules/products/ProductDetail.jsx

```jsx
import { useParams, Link } from 'react-router-dom';
import { useSelector } from 'react-redux';

export function ProductDetail() {
  // Получаем id товара из URL (динамический параметр)
  const { id } = useParams();
  const product = useSelector(state => 
    state.products.items.find(p => p.id === parseInt(id))
  );
  
  if (!product) {
    return (
      <div>
        <h2>Товар не найден</h2>
        <Link to="/products">Вернуться к списку товаров</Link>
      </div>
    );
  }
  
  return (
    <div>
      <div style={styles.header}>
        <Link to="/products" style={styles.backLink}>← Назад к списку</Link>
        <h2>{product.title}</h2>
      </div>
      
      <div style={styles.content}>
        <div style={styles.image}>
          <div style={styles.placeholder}>🖼️ Изображение</div>
        </div>
        
        <div style={styles.info}>
          <p><strong>ID:</strong> {product.id}</p>
          <p><strong>Описание:</strong></p>
          <p>{product.body}</p>
          
          <button style={styles.button}>Добавить в корзину</button>
          
          {/* ОТНОСИТЕЛЬНЫЕ ссылки от текущего маршрута */}
          <div style={styles.links}>
            <Link to="reviews">Отзывы</Link>
            <Link to="similar">Похожие товары</Link>
            <Link to="delivery">Доставка</Link>
          </div>
        </div>
      </div>
    </div>
  );
}

const styles = {
  header: {
    marginBottom: '2rem'
  },
  backLink: {
    display: 'inline-block',
    marginBottom: '1rem',
    color: '#3498db',
    textDecoration: 'none'
  },
  content: {
    display: 'flex',
    gap: '2rem'
  },
  image: {
    flex: 1
  },
  placeholder: {
    backgroundColor: '#f0f0f0',
    height: '300px',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: '8px'
  },
  info: {
    flex: 1
  },
  button: {
    padding: '0.75rem 1.5rem',
    backgroundColor: '#27ae60',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    fontSize: '1rem',
    marginTop: '1rem'
  },
  links: {
    marginTop: '2rem',
    display: 'flex',
    gap: '1rem'
  }
};
```

### components/ProductCard.jsx

```jsx
import { Link } from 'react-router-dom';

export function ProductCard({ product }) {
  return (
    <div style={styles.card}>
      <div style={styles.image}>🖼️ Товар</div>
      <h3 style={styles.title}>{product.title.substring(0, 50)}...</h3>
      <p style={styles.body}>{product.body.substring(0, 80)}...</p>
      
      {/* ОТНОСИТЕЛЬНАЯ ссылка - зависит от текущего URL */}
      {/* Если мы на /products, то ссылка будет /products/{id} */}
      {/* Если мы на /products?category=books, то ссылка будет /products/{id} (базовый путь) */}
      <Link to={`${product.id}`} style={styles.link}>
        Подробнее →
      </Link>
    </div>
  );
}

const styles = {
  card: {
    border: '1px solid #ddd',
    borderRadius: '8px',
    padding: '1rem',
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
    transition: 'transform 0.3s, box-shadow 0.3s',
    cursor: 'pointer'
  },
  image: {
    height: '150px',
    backgroundColor: '#f0f0f0',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: '4px',
    marginBottom: '1rem'
  },
  title: {
    fontSize: '1rem',
    marginBottom: '0.5rem'
  },
  body: {
    fontSize: '0.875rem',
    color: '#666',
    marginBottom: '1rem'
  },
  link: {
    color: '#3498db',
    textDecoration: 'none'
  }
};
```

---

## Часть 6: Главный файл App.jsx (собранный)

```jsx
// App.jsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { Provider } from 'react-redux';
import { store } from './app/store';

// Компоненты
import { Layout } from './components/Layout';
import { PrivateRoute } from './components/PrivateRoute';
import { Home } from './pages/Home';
import { About } from './pages/About';
import { Login } from './pages/Login';
import { Dashboard } from './pages/Dashboard';
import { Settings } from './pages/Settings';
import { NotFound } from './pages/NotFound';
import { ProductsPage } from './modules/products/ProductsPage';

function App() {
  return (
    <Provider store={store}>
      <BrowserRouter>
        <Routes>
          {/* Публичные маршруты (абсолютные) */}
          <Route path="/login" element={<Login />} />
          
          {/* Маршруты с общим Layout (абсолютные) */}
          <Route element={<Layout />}>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            
            {/* Вложенные маршруты товаров */}
            {/* /products/* - звёздочка означает, что все подмаршруты */}
            {/* будут обрабатываться внутри ProductsPage */}
            <Route path="/products/*" element={<ProductsPage />} />
            
            {/* Защищённые маршруты (требуют авторизации) */}
            <Route element={<PrivateRoute />}>
              <Route path="/dashboard/*" element={<Dashboard />} />
              <Route path="/settings" element={<Settings />} />
            </Route>
          </Route>
          
          {/* 404 - не найдено (абсолютный) */}
          <Route path="*" element={<NotFound />} />
        </Routes>
      </BrowserRouter>
    </Provider>
  );
}

export default App;
```

---

## Часть 7: Таблица сравнения ссылок

### Абсолютные vs Относительные ссылки

| Тип | Синтаксис | Откуда начинается | Пример | Результат при URL `/products/123` |
|-----|-----------|-------------------|--------|----------------------------------|
| **Абсолютная** | `to="/about"` | Всегда от корня `/` | `<Link to="/about">` | `/about` |
| **Относительная** | `to="reviews"` | От текущего URL | `<Link to="reviews">` | `/products/123/reviews` |
| **Относительная (родитель)** | `to=".."` | На уровень выше | `<Link to="..">` | `/products` |
| **Относительная (родитель+)** | `to="../profile"` | На уровень выше + путь | `<Link to="../profile">` | `/products/profile` |

### Примеры использования

```jsx
// В компоненте, который рендерится по пути /products/123

// Абсолютные ссылки (всегда работают одинаково)
<Link to="/">Главная</Link>              // → /
<Link to="/products">Все товары</Link>   // → /products
<Link to="/about">О нас</Link>          // → /about

// Относительные ссылки (зависят от текущего пути)
<Link to="reviews">Отзывы</Link>         // → /products/123/reviews
<Link to="similar">Похожие</Link>        // → /products/123/similar
<Link to=".">Обновить</Link>             // → /products/123
<Link to="..">Назад к списку</Link>      // → /products
<Link to="../profile">Профиль</Link>     // → /products/profile

// Относительные с ведущей точкой
<Link to="./edit">Редактировать</Link>   // → /products/123/edit
<Link to="./reviews/new">Новый отзыв</Link> // → /products/123/reviews/new

// Динамические ссылки
const { id } = useParams();
<Link to={`/products/${id}/edit`}>Редактировать</Link>  // абсолютная
<Link to={`${id}/edit`}>Редактировать</Link>            // относительная
```

---

## Часть 8: Визуализация маршрутов

```
Структура URL и соответствующие компоненты:

/                                                                            
├── Layout                                                                   
│   ├── Home ✅ (рендерится через Outlet)                                    
│                                                                            
├── /about                                                                   
│   ├── Layout                                                               
│   │   └── About ✅                                                         
│                                                                            
├── /products                                                               
│   ├── Layout                                                               
│   │   └── ProductsPage                                                     
│   │       └── ProductsList ✅ (рендерится через Outlet внутри ProductsPage)
│                                                                            
├── /products/123                                                           
│   ├── Layout                                                               
│   │   └── ProductsPage                                                     
│   │       └── ProductDetail ✅                                             
│                                                                            
├── /login                                                                   
│   └── Login ✅ (без Layout)                                                
│                                                                            
├── /dashboard                                                              
│   ├── Layout                                                               
│   │   └── PrivateRoute (проверка авторизации)                              
│   │       └── Dashboard ✅                                                 
│                                                                            
├── /dashboard/profile                                                       
│   ├── Layout                                                               
│   │   └── PrivateRoute                                                     
│   │       └── Dashboard                                                     
│   │           └── Profile ✅ (рендерится через Outlet в Dashboard)         
│                                                                            
└── * → NotFound ✅
```

---

## Запуск приложения

```bash
# 1. Создайте новое приложение
npx create-react-app my-shop

# 2. Перейдите в папку
cd my-shop

# 3. Установите зависимости
npm install @reduxjs/toolkit react-redux react-router-dom

# 4. Скопируйте все файлы из примера

# 5. Запустите приложение
npm start
```

---

## Золотые правила для новичка

1. **Абсолютные ссылки** начинаются с `/` и всегда ведут от корня сайта
   - Используйте для главной навигации (`/about`, `/products`, `/login`)

2. **Относительные ссылки** не начинаются с `/` и зависят от текущего URL
   - Используйте для вложенных маршрутов (`reviews`, `similar`, `..`)

3. **`/*` в родительском маршруте** говорит React Router, что все подмаршруты должны обрабатываться внутри этого компонента

4. **`index` маршрут** рендерится, когда путь совпадает с родительским

5. **`<Outlet />`** — это место, где рендерятся дочерние маршруты

6. **`useParams()`** для получения динамических параметров из URL

**Запомните:** Относительные ссылки делают компоненты портативными и переиспользуемыми. Если вы переместите компонент в другое место, относительные ссылки будут работать корректно, а абсолютные — нет!