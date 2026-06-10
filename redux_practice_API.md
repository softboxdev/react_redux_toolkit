# Учебный пример: Приложение прогноза погоды с использованием Redux Toolkit

## Обзор приложения

Мы создадим приложение, которое показывает прогноз погоды для разных городов, используя **бесплатное публичное API погоды**.

### Выбранное API: Open-Meteo (бесплатный, без ключа!)

**Почему Open-Meteo:**
- ✅ Не требует API ключа
- ✅ Бесплатный
- ✅ Отличная документация
- ✅ Поддерживает CORS
- ✅ Работает без ограничений для учебных целей

**API endpoint:** `https://api.open-meteo.com/v1/forecast`

---

## Полный код приложения

### Шаг 1: Установка зависимостей

```bash
# Создаём React приложение
npx create-react-app weather-app --template redux-typescript

# Или для обычного JavaScript:
npx create-react-app weather-app

# Переходим в папку
cd weather-app

# Дополнительные зависимости
npm install axios
```

---

### Шаг 2: Создаём слайс погоды (weatherSlice.js)

```javascript
// src/features/weather/weatherSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// ============================================
// 1. ОПРЕДЕЛЯЕМ ТИПЫ (для JavaScript можно пропустить)
// ============================================
/**
 * @typedef {Object} WeatherData
 * @property {number} temperature - Температура в градусах Цельсия
 * @property {number} windspeed - Скорость ветра (км/ч)
 * @property {number} humidity - Влажность (%)
 * @property {string} time - Дата и время
 */

/**
 * @typedef {Object} WeatherState
 * @property {Object} data - Данные о погоде для текущего города
 * @property {string} status - Статус загрузки: 'idle' | 'loading' | 'succeeded' | 'failed'
 * @property {string|null} error - Сообщение об ошибке
 * @property {string} city - Текущий выбранный город
 */

// ============================================
// 2. СОЗДАЁМ АСИНХРОННЫЙ THUNK ДЛЯ ЗАГРУЗКИ ПОГОДЫ
// ============================================
/**
 * fetchWeather - асинхронный thunk для загрузки данных о погоде
 * @param {string} cityName - Название города (например, 'London', 'Moscow')
 * @returns {Promise<Object>} - Данные о погоде
 */
export const fetchWeather = createAsyncThunk(
  'weather/fetchWeather',  // уникальное имя для Redux
  async (cityName, { rejectWithValue }) => {
    try {
      // ============================================
      // ШАГ 1: Получаем координаты города (геокодинг)
      // ============================================
      // Open-Meteo требует координаты (широта, долгота)
      // Используем бесплатный API геокодинга
      const geoResponse = await axios.get(
        `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cityName)}&count=1&language=ru&format=json`
      );
      
      // Проверяем, найден ли город
      if (!geoResponse.data.results || geoResponse.data.results.length === 0) {
        return rejectWithValue(`Город "${cityName}" не найден. Попробуйте другой город.`);
      }
      
      // Получаем координаты первого найденного города
      const { latitude, longitude, name, country } = geoResponse.data.results[0];
      
      // ============================================
      // ШАГ 2: Загружаем погоду по координатам
      // ============================================
      const weatherResponse = await axios.get(
        `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true&hourly=temperature_2m&timezone=auto`
      );
      
      // Извлекаем данные о текущей погоде
      const currentWeather = weatherResponse.data.current_weather;
      
      // ============================================
      // ШАГ 3: Формируем и возвращаем результат
      // ============================================
      return {
        city: name,
        country: country,
        temperature: currentWeather.temperature,
        windspeed: currentWeather.windspeed,
        time: currentWeather.time,
        weathercode: currentWeather.weathercode, // код погоды (0=ясно, 1=облачно и т.д.)
        coordinates: { latitude, longitude }
      };
      
    } catch (error) {
      // Обрабатываем ошибки сети или другие проблемы
      return rejectWithValue(
        error.response?.data?.message || error.message || 'Не удалось загрузить погоду'
      );
    }
  }
);

// ============================================
// 3. НАЧАЛЬНОЕ СОСТОЯНИЕ
// ============================================
const initialState = {
  data: null,           // данные о погоде (будет заполнены после загрузки)
  status: 'idle',       // 'idle' | 'loading' | 'succeeded' | 'failed'
  error: null,          // сообщение об ошибке
  city: 'London'        // город по умолчанию
};

// ============================================
// 4. СОЗДАЁМ СЛАЙС
// ============================================
const weatherSlice = createSlice({
  name: 'weather',
  initialState,
  reducers: {
    // Синхронный редюсер для смены города
    setCity: (state, action) => {
      state.city = action.payload;
      // При смене города сбрасываем предыдущие данные
      state.status = 'idle';
      state.data = null;
      state.error = null;
    },
    // Сброс состояния
    resetWeather: (state) => {
      return initialState;
    }
  },
  // ============================================
  // 5. EXTRAREDUCERS - ОБРАБОТКА АСИНХРОННЫХ ЭКШЕНОВ
  // ============================================
  extraReducers: (builder) => {
    builder
      // ---------- НАЧАЛО ЗАГРУЗКИ ----------
      .addCase(fetchWeather.pending, (state) => {
        state.status = 'loading';     // меняем статус на 'loading'
        state.error = null;           // сбрасываем прошлую ошибку
        state.data = null;            // сбрасываем старые данные
        console.log('🌤️ Загружаем погоду...');
      })
      
      // ---------- УСПЕШНАЯ ЗАГРУЗКА ----------
      .addCase(fetchWeather.fulfilled, (state, action) => {
        state.status = 'succeeded';    // загрузка успешна
        state.data = action.payload;    // сохраняем данные
        state.error = null;             // ошибок нет
        console.log(`✅ Погода загружена для города ${action.payload.city}`);
      })
      
      // ---------- ОШИБКА ЗАГРУЗКИ ----------
      .addCase(fetchWeather.rejected, (state, action) => {
        state.status = 'failed';        // статус ошибки
        state.error = action.payload;    // сохраняем сообщение об ошибке
        state.data = null;              // данных нет
        console.error('❌ Ошибка загрузки погоды:', action.payload);
      });
  }
});

// ============================================
// 6. ЭКСПОРТИРУЕМ ЭКШЕНЫ И РЕДЮСЕР
// ============================================
export const { setCity, resetWeather } = weatherSlice.actions;
export default weatherSlice.reducer;
```

---

### Шаг 3: Создаём store (store.js)

```javascript
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit';
import weatherReducer from '../features/weather/weatherSlice';

export const store = configureStore({
  reducer: {
    weather: weatherReducer,  // ключ 'weather' для доступа к состоянию
  },
});

// Для TypeScript (опционально)
export const RootState = store.getState;
export const AppDispatch = store.dispatch;
```

---

### Шаг 4: Создаём компоненты

#### Компонент поиска города (CitySearch.jsx)

```jsx
// src/components/CitySearch.jsx
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchWeather, setCity } from '../features/weather/weatherSlice';

export const CitySearch = () => {
  // Локальное состояние для поля ввода
  const [inputValue, setInputValue] = useState('');
  
  // Получаем dispatch для отправки экшенов
  const dispatch = useDispatch();
  
  // Получаем текущий город и статус из Redux
  const currentCity = useSelector((state) => state.weather.city);
  const status = useSelector((state) => state.weather.status);
  
  // Обработчик отправки формы
  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (!inputValue.trim()) return;
    
    // Сохраняем город в Redux
    dispatch(setCity(inputValue));
    
    // Загружаем погоду для нового города
    dispatch(fetchWeather(inputValue));
    
    // Очищаем поле ввода
    setInputValue('');
  };
  
  // Быстрый выбор популярных городов
  const popularCities = ['London', 'Paris', 'Tokyo', 'New York', 'Moscow', 'Sydney'];
  
  return (
    <div style={styles.container}>
      <form onSubmit={handleSubmit} style={styles.form}>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Введите название города..."
          style={styles.input}
          disabled={status === 'loading'}
        />
        <button 
          type="submit" 
          style={styles.button}
          disabled={status === 'loading'}
        >
          {status === 'loading' ? 'Загрузка...' : '🔍 Найти'}
        </button>
      </form>
      
      <div style={styles.popularCities}>
        <span style={styles.popularLabel}>Популярные:</span>
        {popularCities.map(city => (
          <button
            key={city}
            onClick={() => {
              dispatch(setCity(city));
              dispatch(fetchWeather(city));
            }}
            style={styles.cityChip}
            disabled={status === 'loading' || currentCity === city}
          >
            {city}
          </button>
        ))}
      </div>
    </div>
  );
};

// Стили для компонента
const styles = {
  container: {
    marginBottom: '2rem',
    padding: '1rem',
    backgroundColor: '#f5f5f5',
    borderRadius: '8px',
  },
  form: {
    display: 'flex',
    gap: '0.5rem',
    marginBottom: '1rem',
  },
  input: {
    flex: 1,
    padding: '0.75rem',
    fontSize: '1rem',
    border: '1px solid #ddd',
    borderRadius: '4px',
    outline: 'none',
  },
  button: {
    padding: '0.75rem 1.5rem',
    fontSize: '1rem',
    backgroundColor: '#007bff',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
  },
  popularCities: {
    display: 'flex',
    alignItems: 'center',
    gap: '0.5rem',
    flexWrap: 'wrap',
  },
  popularLabel: {
    fontSize: '0.875rem',
    color: '#666',
    marginRight: '0.5rem',
  },
  cityChip: {
    padding: '0.25rem 0.75rem',
    fontSize: '0.875rem',
    backgroundColor: '#e9ecef',
    border: '1px solid #dee2e6',
    borderRadius: '16px',
    cursor: 'pointer',
    transition: 'all 0.2s',
  },
};
```

#### Компонент отображения погоды (WeatherDisplay.jsx)

```jsx
// src/components/WeatherDisplay.jsx
import React from 'react';
import { useSelector } from 'react-redux';

// Функция для преобразования кода погоды в текст и иконку
const getWeatherDescription = (code) => {
  // Коды погоды от Open-Meteo
  // https://open-meteo.com/en/docs
  const weatherCodes = {
    0: { text: 'Ясно', icon: '☀️', description: 'Солнечно, без осадков' },
    1: { text: 'Преимущественно ясно', icon: '🌤️', description: 'В основном солнечно' },
    2: { text: 'Переменная облачность', icon: '⛅', description: 'Частично облачно' },
    3: { text: 'Пасмурно', icon: '☁️', description: 'Сплошная облачность' },
    45: { text: 'Туман', icon: '🌫️', description: 'Туман, ограниченная видимость' },
    51: { text: 'Морось', icon: '🌧️', description: 'Лёгкая морось' },
    61: { text: 'Дождь', icon: '🌧️', description: 'Небольшой дождь' },
    71: { text: 'Снег', icon: '❄️', description: 'Снегопад' }
  };
  
  return weatherCodes[code] || { text: 'Неизвестно', icon: '❓', description: 'Данные недоступны' };
};

// Компонент для отображения карточки погоды
const WeatherCard = ({ title, value, unit, icon }) => (
  <div style={styles.card}>
    <div style={styles.cardIcon}>{icon}</div>
    <div style={styles.cardTitle}>{title}</div>
    <div style={styles.cardValue}>
      {value}{unit && <span style={styles.unit}>{unit}</span>}
    </div>
  </div>
);

export const WeatherDisplay = () => {
  // Получаем данные из Redux store
  const { data, status, error, city } = useSelector((state) => state.weather);
  
  // Начальное состояние (ещё не загружали)
  if (status === 'idle') {
    return (
      <div style={styles.messageContainer}>
        <div style={styles.welcomeIcon}>🌍</div>
        <h2>Добро пожаловать в Weather App!</h2>
        <p>Введите название города или выберите из популярных, чтобы увидеть прогноз погоды.</p>
        <p style={styles.example}>Например: London, Moscow, Tokyo</p>
      </div>
    );
  }
  
  // Состояние загрузки
  if (status === 'loading') {
    return (
      <div style={styles.messageContainer}>
        <div style={styles.loader}></div>
        <p>Загружаем погоду для города <strong>{city}</strong>...</p>
        <p style={styles.hint}>Пожалуйста, подождите</p>
      </div>
    );
  }
  
  // Состояние ошибки
  if (status === 'failed') {
    return (
      <div style={styles.errorContainer}>
        <div style={styles.errorIcon}>⚠️</div>
        <h3>Ошибка загрузки</h3>
        <p style={styles.errorMessage}>{error}</p>
        <p style={styles.hint}>Проверьте название города или попробуйте другой.</p>
      </div>
    );
  }
  
  // Успешная загрузка - отображаем данные
  if (status === 'succeeded' && data) {
    const weather = getWeatherDescription(data.weathercode);
    // Форматируем дату и время
    const date = new Date(data.time);
    const formattedDate = date.toLocaleDateString('ru-RU', {
      weekday: 'long',
      day: 'numeric',
      month: 'long',
      year: 'numeric'
    });
    const formattedTime = date.toLocaleTimeString('ru-RU', {
      hour: '2-digit',
      minute: '2-digit'
    });
    
    return (
      <div style={styles.weatherContainer}>
        {/* Заголовок с городом */}
        <div style={styles.header}>
          <h2>
            {weather.icon} {data.city}
            {data.country && <span style={styles.country}> ({data.country})</span>}
          </h2>
          <p style={styles.datetime}>{formattedDate}, {formattedTime}</p>
        </div>
        
        {/* Основная информация о погоде */}
        <div style={styles.mainWeather}>
          <div style={styles.temperatureContainer}>
            <span style={styles.temperature}>{Math.round(data.temperature)}</span>
            <span style={styles.temperatureUnit}>°C</span>
          </div>
          <div style={styles.weatherDescription}>
            <div style={styles.weatherText}>{weather.text}</div>
            <div style={styles.weatherDesc}>{weather.description}</div>
          </div>
        </div>
        
        {/* Дополнительные показатели */}
        <div style={styles.detailsGrid}>
          <WeatherCard
            title="Ветер"
            value={data.windspeed}
            unit="км/ч"
            icon="💨"
          />
          <WeatherCard
            title="Температура"
            value={Math.round(data.temperature)}
            unit="°C"
            icon="🌡️"
          />
          <WeatherCard
            title="Координаты"
            value={`${data.coordinates.latitude.toFixed(1)}°, ${data.coordinates.longitude.toFixed(1)}°`}
            icon="📍"
          />
        </div>
        
        {/* Информация о времени обновления */}
        <div style={styles.footer}>
          <p style={styles.updateInfo}>
            🔄 Данные актуальны на {formattedTime}
          </p>
        </div>
      </div>
    );
  }
  
  return null;
};

// Стили компонента
const styles = {
  messageContainer: {
    textAlign: 'center',
    padding: '3rem',
    backgroundColor: '#f8f9fa',
    borderRadius: '8px',
    marginTop: '1rem',
  },
  welcomeIcon: {
    fontSize: '4rem',
    marginBottom: '1rem',
  },
  example: {
    color: '#666',
    fontSize: '0.875rem',
    fontFamily: 'monospace',
    marginTop: '1rem',
  },
  loader: {
    width: '50px',
    height: '50px',
    border: '4px solid #f3f3f3',
    borderTop: '4px solid #007bff',
    borderRadius: '50%',
    animation: 'spin 1s linear infinite',
    margin: '0 auto 1rem',
  },
  errorContainer: {
    textAlign: 'center',
    padding: '2rem',
    backgroundColor: '#fee',
    borderRadius: '8px',
    marginTop: '1rem',
    border: '1px solid #fcc',
  },
  errorIcon: {
    fontSize: '3rem',
    marginBottom: '0.5rem',
  },
  errorMessage: {
    color: '#c33',
    fontWeight: 'bold',
    margin: '0.5rem 0',
  },
  hint: {
    color: '#666',
    fontSize: '0.875rem',
    marginTop: '1rem',
  },
  weatherContainer: {
    backgroundColor: 'white',
    borderRadius: '12px',
    padding: '1.5rem',
    boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
    marginTop: '1rem',
  },
  header: {
    textAlign: 'center',
    marginBottom: '1.5rem',
  },
  country: {
    fontSize: '0.9rem',
    color: '#666',
    fontWeight: 'normal',
  },
  datetime: {
    color: '#666',
    fontSize: '0.875rem',
    marginTop: '0.25rem',
  },
  mainWeather: {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    gap: '2rem',
    marginBottom: '2rem',
    padding: '1rem',
    backgroundColor: '#f0f7ff',
    borderRadius: '8px',
  },
  temperatureContainer: {
    display: 'flex',
    alignItems: 'baseline',
  },
  temperature: {
    fontSize: '4rem',
    fontWeight: 'bold',
    color: '#007bff',
  },
  temperatureUnit: {
    fontSize: '1.5rem',
    color: '#666',
    marginLeft: '4px',
  },
  weatherDescription: {
    textAlign: 'left',
  },
  weatherText: {
    fontSize: '1.25rem',
    fontWeight: 'bold',
    marginBottom: '0.25rem',
  },
  weatherDesc: {
    color: '#666',
    fontSize: '0.875rem',
  },
  detailsGrid: {
    display: 'grid',
    gridTemplateColumns: 'repeat(auto-fit, minmax(150px, 1fr))',
    gap: '1rem',
    marginBottom: '1rem',
  },
  card: {
    backgroundColor: '#f8f9fa',
    padding: '1rem',
    borderRadius: '8px',
    textAlign: 'center',
  },
  cardIcon: {
    fontSize: '1.5rem',
    marginBottom: '0.5rem',
  },
  cardTitle: {
    fontSize: '0.875rem',
    color: '#666',
    marginBottom: '0.25rem',
  },
  cardValue: {
    fontSize: '1.25rem',
    fontWeight: 'bold',
  },
  unit: {
    fontSize: '0.875rem',
    fontWeight: 'normal',
    marginLeft: '2px',
  },
  footer: {
    textAlign: 'center',
    paddingTop: '1rem',
    borderTop: '1px solid #eee',
  },
  updateInfo: {
    fontSize: '0.75rem',
    color: '#999',
    margin: 0,
  },
};

// Добавляем анимацию для лоадера (в глобальный CSS или через styled-components)
// Для простоты можно добавить в index.css:
/*
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
*/
```

#### Главный компонент приложения (App.js)

```jsx
// src/App.js
import React from 'react';
import { Provider } from 'react-redux';
import { store } from './app/store';
import { CitySearch } from './components/CitySearch';
import { WeatherDisplay } from './components/WeatherDisplay';

function App() {
  return (
    <Provider store={store}>
      <div style={styles.app}>
        <div style={styles.container}>
          <header style={styles.header}>
            <h1>
              <span style={styles.headerIcon}>🌤️</span>
              Weather Forecast
            </h1>
            <p>Узнайте погоду в любом городе мира</p>
          </header>
          
          <CitySearch />
          <WeatherDisplay />
          
          <footer style={styles.footer}>
            <p>
              Данные предоставлены <strong>Open-Meteo.com</strong>
            </p>
            <p style={styles.apiInfo}>
              API: Open-Meteo (бесплатный, без ключа)
            </p>
          </footer>
        </div>
      </div>
    </Provider>
  );
}

const styles = {
  app: {
    minHeight: '100vh',
    backgroundColor: '#e0f2fe',
    padding: '2rem',
  },
  container: {
    maxWidth: '800px',
    margin: '0 auto',
  },
  header: {
    textAlign: 'center',
    marginBottom: '2rem',
  },
  headerIcon: {
    fontSize: '2rem',
    marginRight: '0.5rem',
  },
  footer: {
    marginTop: '2rem',
    paddingTop: '1rem',
    textAlign: 'center',
    borderTop: '1px solid #cbd5e1',
    color: '#475569',
    fontSize: '0.875rem',
  },
  apiInfo: {
    fontSize: '0.75rem',
    color: '#64748b',
    marginTop: '0.25rem',
  },
};

export default App;
```

---

## Диаграмма работы приложения

```
ПОЛЬЗОВАТЕЛЬ вводит "Moscow" и нажимает "Найти"
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│  CitySearch.jsx                                                 │
│  dispatch(setCity('Moscow'))                                   │
│  dispatch(fetchWeather('Moscow'))                              │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│  weatherSlice.js - fetchWeather.pending                        │
│  state.status = 'loading'                                      │
│  Компонент WeatherDisplay показывает спиннер                    │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│  Асинхронный запрос к API:                                      │
│                                                                 │
│  1. GET https://geocoding-api.open-meteo.com/v1/search?        │
│     name=Moscow&count=1                                        │
│     → Получаем координаты: latitude=55.75, longitude=37.62     │
│                                                                 │
│  2. GET https://api.open-meteo.com/v1/forecast?                │
│     latitude=55.75&longitude=37.62&current_weather=true       │
│     → Получаем погоду: temperature=5.2, windspeed=4.1          │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│  weatherSlice.js - fetchWeather.fulfilled                      │
│  state.status = 'succeeded'                                    │
│  state.data = {                                                │
│    city: 'Moscow',                                             │
│    temperature: 5.2,                                           │
│    windspeed: 4.1,                                             │
│    ...                                                         │
│  }                                                             │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│  WeatherDisplay.jsx - перерендеривается                        │
│  Показывает:                                                   │
│  - Температура: 5.2°C                                          │
│  - Ветер: 4.1 км/ч                                             │
│  - Время обновления                                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Запуск приложения

```bash
# 1. Установка зависимостей
npm install

# 2. Запуск dev-сервера
npm start

# 3. Открыть в браузере http://localhost:3000
```

## Примеры городов для тестирования

| Город | Ожидаемый результат |
|-------|---------------------|
| London | Погода в Лондоне |
| Moscow | Погода в Москве |
| Tokyo | Погода в Токио |
| Paris | Погода в Париже |
| InvalidCity | Сообщение об ошибке |

---

## Дополнительные улучшения (опционально)

1. **Сохранение последнего города в localStorage**
2. **Добавление иконок погоды**
3. **Прогноз на несколько дней**
4. **Переключение между °C и °F**
5. **История поиска**
