# ExtraReducers в Redux Toolkit — Полное руководство 

## Что такое extraReducers?

**ExtraReducers** — это специальное поле в `createSlice`, которое позволяет обрабатывать **экшены, созданные НЕ внутри этого слайса**.

### Простая аналогия

Представьте, что ваше приложение — это большой офис:

```
Обычные reducers (внутренние):
═══════════════════════════════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────┐
│ Отдел продаж (слайс products) получает команды ТОЛЬКО из своего│
│ отдела: "добавить товар", "удалить товар"                       │
│ → Это обычные reducers                                          │
└─────────────────────────────────────────────────────────────────┘

extraReducers (внешние):
═══════════════════════════════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────┐
│ Отдел продаж может реагировать на команды из других отделов:   │
│ "Пользователь авторизовался" → очистить корзину                 │
│ "Приложение загрузилось" → загрузить товары                     │
│ "Сменился язык" → обновить названия товаров                     │
│ → Это extraReducers                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Зачем нужны extraReducers?

### Проблема, которую решают extraReducers

```javascript
// У нас есть два независимых слайса:

// authSlice.js - управляет авторизацией
const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null, isAuth: false },
  reducers: {
    login: (state, action) => {
      state.user = action.payload;
      state.isAuth = true;
    },
    logout: (state) => {
      state.user = null;
      state.isAuth = false;
    }
  }
});

// cartSlice.js - управляет корзиной
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addToCart: (state, action) => {
      state.items.push(action.payload);
    }
  }
  // ❌ ПРОБЛЕМА: как очистить корзину при logout?
});
```

**Вопрос:** Как заставить `cartSlice` реагировать на экшен `logout` из `authSlice`?

**Ответ:** Использовать `extraReducers`!

---

## Синтаксис extraReducers

### Два способа написания

```javascript
import { createSlice } from '@reduxjs/toolkit';

// СПОСОБ 1: Объектный синтаксис (старый, не рекомендуется)
const slice1 = createSlice({
  name: 'example',
  initialState: {},
  reducers: {},
  extraReducers: {
    'some/action': (state, action) => { ... }
  }
});

// СПОСОБ 2: Builder синтаксис (НОВЫЙ, РЕКОМЕНДУЕТСЯ!)
const slice2 = createSlice({
  name: 'example',
  initialState: {},
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(action1, reducer1)
      .addCase(action2, reducer2)
      .addCase(action3, reducer3)
      .addMatcher(condition, reducer)  // для нескольких экшенов
      .addDefaultCase(reducer);        // для всех остальных
  }
});
```

---

## Диаграмма: Поток экшенов через extraReducers

```
ДИАГРАММА ПОТОКА ЭКШЕНОВ
═══════════════════════════════════════════════════════════════════

                    dispatch(action)
                          │
                          ▼
              ┌───────────────────────┐
              │  Redux Store          │
              │  (пропускает action   │
              │   через все редюсеры) │
              └───────────┬───────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
   ┌────────┐       ┌────────┐       ┌────────┐
   │ Слайс 1│       │ Слайс 2│       │ Слайс 3│
   │        │       │        │       │        │
   │reducers│       │reducers│       │reducers│
   │   +    │       │   +    │       │   +    │
   │extra   │       │extra   │       │extra   │
   │Reducers│       │Reducers│       │Reducers│
   └────────┘       └────────┘       └────────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   Новое состояние     │
              │   (объединённое из    │
              │    всех слайсов)      │
              └───────────────────────┘
```

### Конкретный пример потока

```
Экшен: { type: 'auth/logout' }
           │
           ▼
    ┌──────────────┐
    │  authSlice   │
    │              │
    │ reducers: {  │
    │   logout     │ ← обрабатывает logout
    │ }            │
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │  cartSlice   │
    │              │
    │ extraReducers│
    │ .addCase(    │ ← ТОЖЕ обрабатывает logout!
    │   logout,    │
    │   (state) => │
    │     state    │
    │     .items=[]│
    │ )            │
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │  userSlice   │
    │              │
    │ extraReducers│
    │ .addCase(    │ ← И ЭТОТ обрабатывает logout!
    │   logout,    │
    │   (state) => │
    │     state    │
    │     .lastLogout│
    │     = Date.now│
    │ )            │
    └──────────────┘
```

---

## Полный пример: Реагируем на logout из другого слайса

### authSlice.js

```javascript
import { createSlice } from '@reduxjs/toolkit';

// Создаём слайс авторизации
const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: { name: 'Анна', email: 'anna@example.com' },
    isAuthenticated: true,
    token: 'secret-token-123'
  },
  reducers: {
    // Экшен для входа
    login: (state, action) => {
      state.user = action.payload;
      state.isAuthenticated = true;
      state.token = 'new-token';
    },
    // Экшен для выхода (БУДЕТ ИСПОЛЬЗОВАН В ДРУГИХ СЛАЙСАХ)
    logout: (state) => {
      state.user = null;
      state.isAuthenticated = false;
      state.token = null;
    }
  }
});

// Экспортируем экшены
export const { login, logout } = authSlice.actions;
export default authSlice.reducer;
```

### cartSlice.js (использует extraReducers для реакции на logout)

```javascript
import { createSlice } from '@reduxjs/toolkit';
import { logout } from './authSlice'; // ← импортируем экшен из другого слайса!

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [
      { id: 1, name: 'iPhone', price: 999, quantity: 2 },
      { id: 2, name: 'MacBook', price: 1999, quantity: 1 }
    ],
    totalPrice: 3997
  },
  reducers: {
    // Обычные редюсеры для работы с корзиной
    addToCart: (state, action) => {
      state.items.push(action.payload);
      state.totalPrice += action.payload.price;
    },
    removeFromCart: (state, action) => {
      const item = state.items.find(i => i.id === action.payload);
      if (item) {
        state.totalPrice -= item.price * item.quantity;
        state.items = state.items.filter(i => i.id !== action.payload);
      }
    }
  },
  // ⭐ EXTRA REDUCERS - реагируем на экшены из ДРУГИХ слайсов
  extraReducers: (builder) => {
    builder.addCase(logout, (state) => {
      // Когда пользователь выходит из системы (экшен logout из authSlice)
      // очищаем корзину!
      state.items = [];
      state.totalPrice = 0;
      console.log('Корзина очищена при выходе из аккаунта');
    });
  }
});

export const { addToCart, removeFromCart } = cartSlice.actions;
export default cartSlice.reducer;
```

### notificationsSlice.js (тоже реагирует на logout)

```javascript
import { createSlice } from '@reduxjs/toolkit';
import { logout } from './authSlice';

const notificationsSlice = createSlice({
  name: 'notifications',
  initialState: {
    list: [
      { id: 1, text: 'Добро пожаловать!', read: false },
      { id: 2, text: 'У вас новое сообщение', read: false }
    ],
    unreadCount: 2
  },
  reducers: {
    addNotification: (state, action) => {
      state.list.push(action.payload);
      state.unreadCount++;
    },
    markAsRead: (state, action) => {
      const notification = state.list.find(n => n.id === action.payload);
      if (notification && !notification.read) {
        notification.read = true;
        state.unreadCount--;
      }
    }
  },
  extraReducers: (builder) => {
    // Тоже реагируем на logout!
    builder.addCase(logout, (state) => {
      state.list = [];
      state.unreadCount = 0;
      console.log('Уведомления очищены');
    });
  }
});

export const { addNotification, markAsRead } = notificationsSlice.actions;
export default notificationsSlice.reducer;
```

### Диаграмма: Реакция на logout

```
Пользователь нажимает кнопку "Выйти"
                │
                ▼
    dispatch(logout())
                │
                ▼
    ┌───────────────────────────────────────────────────────┐
    │                    STORE                              │
    │                                                       │
    │  ┌─────────────────┐                                │
    │  │   authSlice     │                                │
    │  │   reducers: {   │                                │
    │  │     logout      │ ← user = null, isAuth = false  │
    │  │   }             │                                │
    │  └─────────────────┘                                │
    │                                                       │
    │  ┌─────────────────┐                                │
    │  │   cartSlice     │                                │
    │  │   extraReducers │                                │
    │  │   addCase(logout)│ ← items = [], totalPrice = 0  │
    │  └─────────────────┘                                │
    │                                                       │
    │  ┌─────────────────┐                                │
    │  │notificationsSlice│                                │
    │  │   extraReducers │                                │
    │  │   addCase(logout)│ ← list = [], unreadCount = 0  │
    │  └─────────────────┘                                │
    └───────────────────────────────────────────────────────┘
                │
                ▼
    Все компоненты, использующие useSelector,
    автоматически обновляются!
```

---

## Работа с createAsyncThunk (САМЫЙ ЧАСТЫЙ СЛУЧАЙ!)

### Полный пример с асинхронной загрузкой данных

```javascript
// productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// 1. СОЗДАЁМ АСИНХРОННЫЙ THUNK
// createAsyncThunk автоматически создаёт три экшена:
// - fetchProducts.pending
// - fetchProducts.fulfilled
// - fetchProducts.rejected
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',  // уникальное имя
  async (_, { rejectWithValue }) => {
    try {
      const response = await axios.get('https://fakestoreapi.com/products');
      return response.data; // эти данные попадут в action.payload
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// 2. СОЗДАЁМ СЛАЙС С extraReducers ДЛЯ ОБРАБОТКИ АСИНХРОННЫХ ЭКШЕНОВ
const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],           // массив товаров
    status: 'idle',      // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null,         // сообщение об ошибке
    lastUpdate: null     // время последнего обновления
  },
  reducers: {
    // Синхронные редюсеры (для ручного управления)
    clearProducts: (state) => {
      state.items = [];
      state.status = 'idle';
    },
    updateTimestamp: (state) => {
      state.lastUpdate = Date.now();
    }
  },
  // ⭐ EXTRA REDUCERS для обработки асинхронных экшенов
  extraReducers: (builder) => {
    builder
      // 1. Обработка состояния "ЗАГРУЗКА"
      .addCase(fetchProducts.pending, (state) => {
        state.status = 'loading';    // меняем статус на 'loading'
        state.error = null;          // сбрасываем ошибку
        console.log('📡 Начинаем загрузку товаров...');
      })
      
      // 2. Обработка состояния "УСПЕШНО"
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded';   // загрузка успешна
        state.items = action.payload;  // сохраняем полученные данные
        state.lastUpdate = Date.now(); // запоминаем время обновления
        console.log(`✅ Загружено ${action.payload.length} товаров`);
      })
      
      // 3. Обработка состояния "ОШИБКА"
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed';       // статус ошибки
        state.error = action.payload || action.error.message;
        console.error('❌ Ошибка загрузки:', state.error);
      })
      
      // 4. ДОПОЛНИТЕЛЬНО: можно добавить обработку других экшенов
      .addCase('user/login', (state) => {
        // Реагируем на вход пользователя (если есть такой экшен)
        console.log('Пользователь вошёл, обновляем товары...');
        // можно автоматически перезагрузить товары
      });
  }
});

export const { clearProducts, updateTimestamp } = productsSlice.actions;
export default productsSlice.reducer;
```

### Диаграмма: Жизненный цикл асинхронного экшена с extraReducers

```
ВРЕМЕННАЯ ШКАЛА
═══════════════════════════════════════════════════════════════════

t=0ms: dispatch(fetchProducts())
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  createAsyncThunk автоматически диспатчит:                      │
│  { type: 'products/fetchProducts/pending' }                    │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  extraReducers.addCase(fetchProducts.pending)                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ state.status = 'loading'                                 │ │
│  │ state.error = null                                       │ │
│  │ Компонент показывает спиннер "Загрузка..."               │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=100ms: Выполняется асинхронный запрос
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  await axios.get('/api/products')                              │
│  🔄 Ожидание ответа от сервера...                              │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
t=500ms: Сервер вернул данные
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  createAsyncThunk автоматически диспатчит:                      │
│  { type: 'products/fetchProducts/fulfilled',                   │
│    payload: [{id:1,...}, {id:2,...}] }                        │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  extraReducers.addCase(fetchProducts.fulfilled)                │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ state.status = 'succeeded'                               │ │
│  │ state.items = action.payload  (массив с данными)         │ │
│  │ Компонент показывает список товаров                      │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

Если ОШИБКА:
═══════════════════════════════════════════════════════════════════

t=500ms: Сервер вернул ошибку (500, 404, и т.д.)
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  createAsyncThunk автоматически диспатчит:                      │
│  { type: 'products/fetchProducts/rejected',                    │
│    error: { message: 'Network Error' } }                       │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  extraReducers.addCase(fetchProducts.rejected)                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ state.status = 'failed'                                  │ │
│  │ state.error = 'Network Error'                            │ │
│  │ Компонент показывает сообщение об ошибке                 │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Продвинутые возможности extraReducers

### 1. addMatcher (для групп экшенов)

```javascript
import { isAnyOf } from '@reduxjs/toolkit';

const slice = createSlice({
  name: 'ui',
  initialState: { loadingCount: 0, errors: [] },
  reducers: {},
  extraReducers: (builder) => {
    builder
      // Сработает для ЛЮБОГО экшена с 'pending'
      .addMatcher(
        (action) => action.type.endsWith('/pending'),
        (state) => {
          state.loadingCount++;
        }
      )
      // Сработает для ЛЮБОГО экшена с 'fulfilled' или 'rejected'
      .addMatcher(
        isAnyOf(
          (action) => action.type.endsWith('/fulfilled'),
          (action) => action.type.endsWith('/rejected')
        ),
        (state) => {
          state.loadingCount--;
        }
      )
      // Сработает для всех rejected экшенов
      .addMatcher(
        (action) => action.type.endsWith('/rejected'),
        (state, action) => {
          state.errors.push({
            type: action.type,
            error: action.error.message,
            time: Date.now()
          });
        }
      );
  }
});
```

### 2. addDefaultCase (для всех остальных экшенов)

```javascript
const analyticsSlice = createSlice({
  name: 'analytics',
  initialState: { actionLog: [] },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.fulfilled, (state) => {
        state.actionLog.push('Продукты загружены');
      })
      .addCase(login, (state) => {
        state.actionLog.push('Пользователь вошёл');
      })
      // Логируем ВСЕ остальные экшены
      .addDefaultCase((state, action) => {
        state.actionLog.push(`Выполнен экшен: ${action.type}`);
        // Ограничиваем лог 100 записями
        if (state.actionLog.length > 100) {
          state.actionLog.shift();
        }
      });
  }
});
```

### 3. Реакция на экшены из нескольких слайсов

```javascript
// userSlice.js
export const updateUserProfile = createAsyncThunk(
  'user/updateProfile',
  async (userData) => {
    const response = await fetch('/api/user/profile', {
      method: 'POST',
      body: JSON.stringify(userData)
    });
    return response.json();
  }
);

// auditSlice.js (логирует ВСЕ изменения пользователя)
const auditSlice = createSlice({
  name: 'audit',
  initialState: { logs: [] },
  reducers: {},
  extraReducers: (builder) => {
    builder
      // Реагируем на успешное обновление профиля
      .addCase(updateUserProfile.fulfilled, (state, action) => {
        state.logs.push({
          action: 'PROFILE_UPDATED',
          data: action.payload,
          timestamp: Date.now()
        });
      })
      // Реагируем на ошибку обновления
      .addCase(updateUserProfile.rejected, (state, action) => {
        state.logs.push({
          action: 'PROFILE_UPDATE_FAILED',
          error: action.error.message,
          timestamp: Date.now()
        });
      });
  }
});
```

---

## Диаграмма: addMatcher и несколько условий

```
Поток экшенов через addMatcher
═══════════════════════════════════════════════════════════════════

                    dispatch(action)
                          │
                          ▼
              ┌───────────────────────┐
              │  extraReducers        │
              │  (проверяет условия)  │
              └───────────┬───────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
   addCase           addMatcher        addDefaultCase
   (точное         (условие true)     (все остальные)
   совпадение)
        │                 │                 │
        ▼                 ▼                 ▼
   выполняет          выполняет          выполняет
   reducer            reducer            reducer
```

---

## Практический пример: Приложение с корзиной и заказами

```javascript
// ========== orderSlice.js ==========
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Асинхронный thunk для создания заказа
export const createOrder = createAsyncThunk(
  'order/create',
  async (orderData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/orders', {
        method: 'POST',
        body: JSON.stringify(orderData)
      });
      if (!response.ok) throw new Error('Ошибка создания заказа');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const orderSlice = createSlice({
  name: 'order',
  initialState: {
    currentOrder: null,
    history: [],
    status: 'idle',
    error: null
  },
  reducers: {
    clearCurrentOrder: (state) => {
      state.currentOrder = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(createOrder.pending, (state) => {
        state.status = 'loading';
        state.error = null;
      })
      .addCase(createOrder.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.currentOrder = action.payload;
        state.history.push(action.payload);
      })
      .addCase(createOrder.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload;
      });
  }
});

// ========== cartSlice.js ==========
// Импортируем экшен из orderSlice
import { createOrder } from './orderSlice';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    totalPrice: 0
  },
  reducers: {
    addToCart: (state, action) => {
      state.items.push(action.payload);
      state.totalPrice += action.payload.price;
    },
    clearCart: (state) => {
      state.items = [];
      state.totalPrice = 0;
    }
  },
  extraReducers: (builder) => {
    // ⭐ Реагируем на УСПЕШНОЕ создание заказа!
    builder.addCase(createOrder.fulfilled, (state) => {
      // Заказ создан — очищаем корзину
      state.items = [];
      state.totalPrice = 0;
      console.log('Корзина очищена после успешного заказа');
    });
  }
});

// ========== analyticsSlice.js ==========
const analyticsSlice = createSlice({
  name: 'analytics',
  initialState: {
    events: []
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      // Логируем ВСЕ асинхронные действия
      .addMatcher(
        (action) => action.type.endsWith('/fulfilled'),
        (state, action) => {
          state.events.push({
            type: action.type,
            timestamp: Date.now(),
            success: true
          });
        }
      )
      .addMatcher(
        (action) => action.type.endsWith('/rejected'),
        (state, action) => {
          state.events.push({
            type: action.type,
            timestamp: Date.now(),
            success: false,
            error: action.error?.message
          });
        }
      );
  }
});
```

### Диаграмма: Взаимодействие слайсов через extraReducers

```
Пользователь оформляет заказ
            │
            ▼
    dispatch(createOrder(cartData))
            │
            ▼
    ┌───────────────────────────────────────────────────────────┐
    │                    orderSlice                             │
    │  extraReducers.addCase(createOrder.pending)              │
    │  → status = 'loading'                                    │
    └───────────────────────────────────────────────────────────┘
            │
            ▼
    Запрос к серверу (ожидание...)
            │
            ▼
    ┌───────────────────────────────────────────────────────────┐
    │                    orderSlice                             │
    │  extraReducers.addCase(createOrder.fulfilled)            │
    │  → currentOrder = orderData                              │
    │  → history.push(orderData)                               │
    │  → status = 'succeeded'                                  │
    └───────────────────────────────────────────────────────────┘
            │
            ▼
    ┌───────────────────────────────────────────────────────────┐
    │                    cartSlice                              │
    │  extraReducers.addCase(createOrder.fulfilled)            │
    │  → items = []  (ОЧИЩАЕМ КОРЗИНУ!)                        │
    │  → totalPrice = 0                                        │
    └───────────────────────────────────────────────────────────┘
            │
            ▼
    ┌───────────────────────────────────────────────────────────┐
    │                 analyticsSlice                            │
    │  addMatcher(endsWith('/fulfilled'))                      │
    │  → events.push({ type: 'order/create/fulfilled' })      │
    └───────────────────────────────────────────────────────────┘
```

---

## Частые ошибки и их решение

### Ошибка 1: Забыли импортировать экшен

```javascript
// ❌ ОШИБКА
import { createSlice } from '@reduxjs/toolkit';
// забыли импортировать logout

const slice = createSlice({
  extraReducers: (builder) => {
    builder.addCase(logout, (state) => { // ReferenceError: logout is not defined
      // ...
    });
  }
});

// ✅ ПРАВИЛЬНО
import { logout } from './authSlice'; // добавляем импорт
```

### Ошибка 2: Мутация состояния без Immer

```javascript
// ❌ ОШИБКА (хотя в extraReducers Immer тоже работает!)
extraReducers: (builder) => {
  builder.addCase(action, (state, action) => {
    state = { ...state, data: action.payload }; // так не работает!
  });
}

// ✅ ПРАВИЛЬНО (мутируем напрямую)
extraReducers: (builder) => {
  builder.addCase(action, (state, action) => {
    state.data = action.payload; // так работает через Immer
  });
}
```

### Ошибка 3: Неправильный порядок addCase

```javascript
// ❌ ОШИБКА (порядок не важен, но нужно понимать приоритет)
extraReducers: (builder) => {
  builder
    .addMatcher(isAnyOf(action1, action2), reducer)
    .addCase(action1, reducer); // может не сработать, если addMatcher перехватил
}

// ✅ ПРАВИЛЬНО (addCase имеет приоритет над addMatcher)
extraReducers: (builder) => {
  builder
    .addCase(action1, specificReducer)  // сначала точные совпадения
    .addMatcher(isAnyOf(action1, action2), generalReducer); // потом matcher
}
```

---

## Шпаргалка по extraReducers

```
┌─────────────────────────────────────────────────────────────────┐
│                      EXTRAREDUCERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ЧТО ЭТО: Поле в createSlice для обработки чужих экшенов       │
│                                                                 │
│  ЗАЧЕМ:                                                         │
│  • Обрабатывать экшены из ДРУГИХ слайсов                        │
│  • Обрабатывать асинхронные экшены (createAsyncThunk)          │
│  • Реагировать на глобальные события                           │
│                                                                 │
│  СИНТАКСИС:                                                     │
│  extraReducers: (builder) => {                                 │
│    builder                                                      │
│      .addCase(exactAction, reducer)      // точное совпадение   │
│      .addMatcher(condition, reducer)     // по условию         │
│      .addDefaultCase(reducer)            // все остальные      │
│  }                                                              │
│                                                                 │
│  ЧАСТЫЙ СЛУЧАЙ:                                                │
│  export const fetchData = createAsyncThunk(...)                │
│                                                                 │
│  extraReducers: (builder) => {                                 │
│    builder                                                      │
│      .addCase(fetchData.pending, (state) => {                  │
│        state.status = 'loading'                                │
│      })                                                         │
│      .addCase(fetchData.fulfilled, (state, action) => {        │
│        state.data = action.payload                             │
│        state.status = 'succeeded'                              │
│      })                                                         │
│      .addCase(fetchData.rejected, (state, action) => {         │
│        state.error = action.error.message                      │
│        state.status = 'failed'                                 │
│      })                                                         │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Золотое правило:** Используйте `extraReducers` когда нужно, чтобы слайс реагировал на:
1. Асинхронные операции (`createAsyncThunk`)
2. Экшены из других слайсов (`logout`, `login`, `resetApp`)
3. Глобальные события (`theme/change`, `language/set`)
