# Отчёт по лабораторной работе №9
## Сохранение настроек темы. Тёмная/светлая тема в Compose

<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  
**Спивакова Екатерина Максимовна**

<br>
<br>
<br>
<br>

Лабораторная работа №9  
**«Сохранение настроек темы. Тёмная/светлая тема в Compose»**  
01.03.02 Прикладная математика и информатика  
3 Курс

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<div align="right">
Научный руководитель<br>
Соболев Евгений Игоревич
</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>

---

## Цель работы

Изучить механизмы смены и сохранения темы приложения в Jetpack Compose, научиться использовать DataStore для хранения пользовательских настроек, реализовать переключение между тёмной и светлой темами с сохранением выбора пользователя.

## Индивидуальное задание

Реализовано приложение с возможностью переключения между светлой и тёмной темами с сохранением выбора пользователя. Добавлена поддержка динамических цветов для Android 12+. Интерфейс включает кнопку переключения темы, карточку с примером компонентов и демонстрацию применения цветовой схемы.

## Скриншоты

<img width="316" height="694" alt="Снимок экрана (7)(1)" src="https://github.com/user-attachments/assets/ff25b9ef-36f7-4990-b786-5b34c4c8c4a1" />  
*Рисунок 1 – Главный экран приложения в светлой теме*

<br>
<img width="325" height="702" alt="Снимок экрана (8)(1)" src="https://github.com/user-attachments/assets/8c27be29-e594-492f-91d1-594a4ba732de" />

*Рисунок 2 – Главный экран приложения в тёмной теме*

<br>

<img width="280" height="560" alt="Снимок экрана (9)(1)" src="https://github.com/user-attachments/assets/dbbf17cd-63c9-4c03-a805-85eddafb19bb" />
 
*Рисунок 3 – Структура проекта*

## Листинги

### 1. `Color.kt`
```kt
package com.example.themeswitcherapp.ui.theme

import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.ui.graphics.Color

// Светлая тема
val LightColors = lightColorScheme(
    primary = Color(0xFF006C4C),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFF89F8C7),
    onPrimaryContainer = Color(0xFF002114),
    secondary = Color(0xFF4D635A),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFCFE9DD),
    onSecondaryContainer = Color(0xFF0A1F19),
    tertiary = Color(0xFF3A637A),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFC1E8FF),
    onTertiaryContainer = Color(0xFF001E2C),
    background = Color(0xFFF4FBF5),
    onBackground = Color(0xFF161D1A),
    surface = Color(0xFFF4FBF5),
    onSurface = Color(0xFF161D1A)
)

// Тёмная тема
val DarkColors = darkColorScheme(
    primary = Color(0xFF6CDBB0),
    onPrimary = Color(0xFF003825),
    primaryContainer = Color(0xFF005239),
    onPrimaryContainer = Color(0xFF89F8C7),
    secondary = Color(0xFFB3CCC1),
    onSecondary = Color(0xFF1F352D),
    secondaryContainer = Color(0xFF354B43),
    onSecondaryContainer = Color(0xFFCFE9DD),
    tertiary = Color(0xFF9DC9E5),
    onTertiary = Color(0xFF003549),
    tertiaryContainer = Color(0xFF1F4B63),
    onTertiaryContainer = Color(0xFFC1E8FF),
    background = Color(0xFF161D1A),
    onBackground = Color(0xFFE1E3DF),
    surface = Color(0xFF161D1A),
    onSurface = Color(0xFFE1E3DF)
)
```

### 2. `SettingsManager.kt`
```kt
package com.example.themeswitcherapp.data

import android.content.Context
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

val Context.dataStore by preferencesDataStore(name = "settings")

class SettingsManager(private val context: Context) {

    companion object {
        val DARK_MODE_KEY = booleanPreferencesKey("dark_mode")
    }

    val isDarkMode: Flow<Boolean> = context.dataStore.data
        .map { preferences ->
            preferences[DARK_MODE_KEY] ?: false
        }

    suspend fun saveDarkMode(enabled: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[DARK_MODE_KEY] = enabled
        }
    }
}
```

### 3. `ThemeViewModel.kt`
```kt
package com.example.themeswitcherapp.ui.theme

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.themeswitcherapp.data.SettingsManager
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class ThemeViewModel(
    private val settingsManager: SettingsManager
) : ViewModel() {

    val isDarkTheme: StateFlow<Boolean> = settingsManager.isDarkMode
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )

    fun toggleTheme() {
        viewModelScope.launch {
            val currentValue = isDarkTheme.value
            settingsManager.saveDarkMode(!currentValue)
        }
    }
}
```

### 4. `ThemeViewModelFactory.kt`
```kt
package com.example.themeswitcherapp.ui.theme

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.themeswitcherapp.data.SettingsManager

class ThemeViewModelFactory(
    private val settingsManager: SettingsManager
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(ThemeViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return ThemeViewModel(settingsManager) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

### 5. `Theme.kt`
```kt
package com.example.themeswitcherapp.ui.theme

import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.runtime.SideEffect
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.core.view.WindowCompat
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ThemeSwitcherTheme(
    viewModel: ThemeViewModel = viewModel(),
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()

    // Динамические цвета доступны на Android 12+
    val colorScheme = when {
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            if (isDarkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        isDarkTheme -> DarkColors
        else -> LightColors
    }

    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as Activity).window
            window.statusBarColor = colorScheme.primary.toArgb()
            WindowCompat.getInsetsController(window, view).isAppearanceLightStatusBars = !isDarkTheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography(),
        content = content
    )
}
```

### 6. `MainActivity.kt`
```kt
package com.example.themeswitcherapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.themeswitcherapp.data.SettingsManager
import com.example.themeswitcherapp.ui.theme.ThemeSwitcherTheme
import com.example.themeswitcherapp.ui.theme.ThemeViewModel
import com.example.themeswitcherapp.ui.theme.ThemeViewModelFactory

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val settingsManager = SettingsManager(this)

        setContent {
            ThemeSwitcherTheme(
                viewModel = viewModel(factory = ThemeViewModelFactory(settingsManager))
            ) {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    ThemeScreen()
                }
            }
        }
    }
}

@Composable
fun ThemeScreen(viewModel: ThemeViewModel = viewModel()) {
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Текущая тема: ${if (isDarkTheme) "Тёмная" else "Светлая"}",
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(
            onClick = { viewModel.toggleTheme() }
        ) {
            Text("Переключить тему")
        }

        Spacer(modifier = Modifier.height(32.dp))

        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "Пример карточки",
                    style = MaterialTheme.typography.titleLarge
                )
                Text(
                    text = "Это демонстрация того, как тема влияет на цвета компонентов. " +
                            "Primary цвет: ${MaterialTheme.colorScheme.primary}",
                    style = MaterialTheme.typography.bodyMedium
                )
            }
        }

        Spacer(modifier = Modifier.height(24.dp))

        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Button(
                onClick = { },
                colors = ButtonDefaults.buttonColors(
                    containerColor = MaterialTheme.colorScheme.secondary
                ),
                modifier = Modifier.weight(1f)
            ) {
                Text("Кнопка 1")
            }

            OutlinedButton(
                onClick = { },
                modifier = Modifier.weight(1f)
            ) {
                Text("Кнопка 2")
            }
        }
    }
}
```

## Ответы на контрольные вопросы

**1. Как в Compose определить, какая тема активна в данный момент (тёмная/светлая)?**

В Compose определить активную тему можно несколькими способами:
- Использовать `isSystemInDarkTheme()` – эта функция возвращает true, если на устройстве включена тёмная тема на системном уровне.
- Через `MaterialTheme.colorScheme` – можно анализировать цветовую схему, например, сравнить цвет фона с ожидаемым.
- В данном приложении используется `viewModel.isDarkTheme.collectAsState()`, который предоставляет сохранённое пользовательское предпочтение.

**2. Что такое `MaterialTheme.colorScheme` и какие основные цвета он содержит?**

`MaterialTheme.colorScheme` – это объект, содержащий цветовую палитру темы приложения. Основные цвета включают:
- `primary` – основной цвет приложения, используется для акцентных элементов (кнопки, панели)
- `onPrimary` – цвет элементов, отображаемых поверх primary (например, текст на кнопке)
- `secondary` – второстепенный цвет для акцентов второго плана
- `onSecondary` – цвет для элементов поверх secondary
- `background` – основной цвет фона экранов
- `onBackground` – цвет текста и иконок на фоне
- `surface` – цвет поверхностей (карточек, диалогов, боковых панелей)
- `onSurface` – цвет элементов на поверхности
- `error` – цвет для сообщений об ошибках
- `tertiary` – третичный акцентный цвет

**3. Как сохранить выбор темы пользователя между сессиями работы приложения?**

Для сохранения выбора темы используются механизмы хранения данных:
- **DataStore** – современное решение на основе Kotlin Coroutines и Flow, рекомендованное Google. Сохраняет пары ключ-значение асинхронно.
- **SharedPreferences** – классический способ, синхронный, но также подходит для простых настроек.

В данной работе используется DataStore, где сохраняется булево значение `dark_mode`. При запуске приложения значение считывается из DataStore через Flow, а при переключении темы сохраняется новое значение.

**4. В чём разница между `isSystemInDarkTheme()` и сохранённым пользовательским выбором?**

- `isSystemInDarkTheme()` – определяет системную настройку темы на устройстве пользователя. Это внешний фактор, который приложение может учитывать или игнорировать.
- **Сохранённый пользовательский выбор** – это настройка, которую пользователь явно выбрал внутри приложения. Она имеет приоритет над системной, если приложение следует за выбором пользователя, а не за системой.

В данном приложении используется исключительно сохранённый пользовательский выбор, что даёт пользователю полный контроль независимо от системных настроек.

**5. Что такое динамические цвета (dynamic color) и на каких версиях Android они доступны?**

Динамические цвета – это функция Android 12 (API 32) и выше, которая автоматически генерирует цветовую схему на основе обоев, установленных на устройстве пользователя. Система извлекает доминирующие цвета из изображения обоев и создаёт гармоничную палитру из primary, secondary, tertiary и других оттенков.

Compose предоставляет функции `dynamicDarkColorScheme(context)` и `dynamicLightColorScheme(context)` для получения этих цветов. Если приложение поддерживает динамические цвета, оно автоматически адаптируется к стилю оформления устройства, создавая более целостный пользовательский опыт.

## Вывод

В ходе выполнения лабораторной работы были достигнуты следующие результаты:

1. **Изучены механизмы темизации в Jetpack Compose** – освоена работа с `MaterialTheme`, `ColorScheme`, создание пользовательских цветовых схем для светлой и тёмной темы. Созданы две полноценные цветовые палитры в соответствии с Material Design 3.

2. **Реализовано переключение между темами** – создан интерфейс с кнопкой переключения, который мгновенно изменяет внешний вид всех компонентов приложения без перезапуска Activity. Текст на экране отображает текущую активную тему.

3. **Освоено сохранение пользовательских настроек** – с помощью DataStore реализовано сохранение выбранной темы между сессиями. После переключения темы и полного закрытия приложения выбранная настройка сохраняется и восстанавливается при следующем запуске.

4. **Реализована поддержка динамических цветов** – для устройств на Android 12+ автоматически применяются цвета на основе обоев системы, что делает приложение более интегрированным с системой.

5. **Создан демонстрационный интерфейс** – разработан экран, показывающий применение цветов темы на различных компонентах: тексте, кнопках, карточках. Это наглядно демонстрирует влияние темы на все элементы приложения.

