
# Лабораторная работа №4

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

Лабораторная работа №4  
**«Создание экрана профиля с использованием ConstraintLayout и обработка событий»**  
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

Изучить возможности ConstraintLayout для создания гибких интерфейсов Android-приложений, освоить механизмы обработки событий нажатия на элементы UI и научиться выносить ресурсы (размеры, цвета, строки) в отдельные файлы.

## Индивидуальное задание: Экран профиля

Требовалось создать экран профиля пользователя со следующими элементами:
- Аватар пользователя (ImageView)
- Имя пользователя (TextView с возможностью редактирования через EditText)
- Статус (TextView)
- Кнопка «Редактировать» (переключает режим редактирования имени)
- Кнопка «Выйти» (завершает работу приложения)

Дополнительные требования: центрирование элементов по вертикали, использование теней (elevation), сохранение данных между сессиями (SharedPreferences).

## Скриншоты

![Результат выполнения приложения](AppScreenshot.png)  
*Рисунок 1 – Работа приложения с отображением профиля*

<br>

![Режим редактирования](EditMode.png)  
*Рисунок 2 – Режим редактирования имени пользователя*

<br>

![Структура проекта](ProjectStructure.png)  
*Рисунок 3 – Структура проекта с ресурсами*

## Листинги

### 1. Итоговый файл activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    android:padding="16dp"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.3" />

    <ImageView
        android:id="@+id/imageAvatar"
        android:layout_width="@dimen/avatar_size"
        android:layout_height="@dimen/avatar_size"
        android:contentDescription="@string/profile_name"
        android:src="@drawable/ic_profile"
        android:elevation="8dp"
        android:outlineProvider="bounds"
        android:scaleType="centerCrop"
        android:shadowColor="@color/purple_500"
        android:shadowRadius="10"
        app:layout_constraintBottom_toTopOf="@+id/containerName"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="@id/guideline_center" />

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/containerName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:elevation="4dp"
        android:padding="4dp"
        android:shadowColor="@color/teal_200"
        android:shadowRadius="6"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar"
        app:layout_constraintBottom_toTopOf="@+id/textStatus">

        <TextView
            android:id="@+id/textName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/profile_name"
            android:textColor="@color/fraise"
            android:textSize="@dimen/text_size_name"
            android:textStyle="bold"
            android:elevation="4dp"
            android:shadowColor="@color/purple_200"
            android:shadowRadius="4"
            android:shadowDx="2"
            android:shadowDy="2"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent" />

        <EditText
            android:id="@+id/editName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/profile_name"
            android:textColor="@color/fraise"
            android:textSize="@dimen/text_size_name"
            android:textStyle="bold"
            android:background="@android:drawable/editbox_background"
            android:singleLine="true"
            android:visibility="gone"
            android:elevation="4dp"
            android:shadowColor="@color/teal_200"
            android:shadowRadius="4"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>

    <TextView
        android:id="@+id/textStatus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/profile_status"
        android:textColor="@color/purple_500"
        android:textSize="@dimen/text_size_status"
        android:textStyle="italic"
        android:elevation="4dp"
        android:shadowColor="@color/teal_200"
        android:shadowRadius="6"
        android:shadowDx="3"
        android:shadowDy="3"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/containerName"
        app:layout_constraintBottom_toTopOf="@+id/buttonEdit" />

    <Button
        android:id="@+id/buttonEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:backgroundTint="@color/purple_200"
        android:text="@string/button_edit"
        android:textColor="@color/black"
        android:elevation="6dp"
        android:shadowColor="@color/purple_500"
        android:shadowRadius="8"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textStatus"
        app:layout_constraintBottom_toTopOf="@+id/buttonExit" />

    <Button
        android:id="@+id/buttonExit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="32dp"
        android:backgroundTint="@color/fraise"
        android:text="@string/button_exit"
        android:textColor="@color/white"
        android:elevation="6dp"
        android:shadowColor="@color/teal_200"
        android:shadowRadius="10"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/buttonEdit"
        app:layout_constraintBottom_toBottomOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### 2. Файл ресурсов dimens.xml

```xml
<resources>
    <dimen name="avatar_size">240dp</dimen>
    <dimen name="margin_high">32dp</dimen>
    <dimen name="margin_normal">16dp</dimen>
    <dimen name="margin_small">8dp</dimen>
    <dimen name="text_size_name">24sp</dimen>
    <dimen name="text_size_status">16sp</dimen>
    <dimen name="button_corner_radius">8dp</dimen>
</resources>
```

### 3. Файл ресурсов colors.xml

```xml
<resources>
    <color name="purple_200">#FFBB86FC</color>
    <color name="purple_500">#FF6200EE</color>
    <color name="teal_200">#FF03DAC5</color>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
    <color name="gray_light">#F5F5F5</color>
    <color name="fraise">#FF99D3</color>
</resources>
```

### 4. Файл ресурсов strings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">ProfileApp</string>
    <string name="profile_name">Пахомов Виктор</string>
    <string name="profile_status">Android-разработчик</string>
    <string name="button_edit">Редактировать</string>
    <string name="button_save">Сохранить</string>
    <string name="button_exit">Выйти</string>
    <string name="toast_message">Редактирование профиля</string>
    <string name="toast_saved">Имя сохранено</string>
    <string name="toast_exit">Выход из профиля</string>
</resources>
```

### 5. MainActivity.kt

```kotlin
package com.example.profileapp

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import android.view.View
import android.content.Context
import android.content.SharedPreferences

class MainActivity : AppCompatActivity() {

    private lateinit var textName: TextView
    private lateinit var editName: EditText
    private lateinit var buttonEdit: Button
    private lateinit var buttonExit: Button
    private var isEditing = false
    private lateinit var sharedPref: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textName = findViewById(R.id.textName)
        editName = findViewById(R.id.editName)
        buttonEdit = findViewById(R.id.buttonEdit)
        buttonExit = findViewById(R.id.buttonExit)

        sharedPref = getSharedPreferences("ProfilePrefs", Context.MODE_PRIVATE)

        loadSavedData()

        buttonEdit.setOnClickListener {
            if (!isEditing) {
                textName.visibility = View.GONE
                editName.visibility = View.VISIBLE
                editName.setText(textName.text)
                buttonEdit.text = getString(R.string.button_save)
                isEditing = true
                Toast.makeText(this, getString(R.string.toast_message), Toast.LENGTH_SHORT).show()
            } else {
                val newName = editName.text.toString().trim()
                if (newName.isNotEmpty()) {
                    textName.text = newName
                    saveData(newName)
                }
                textName.visibility = View.VISIBLE
                editName.visibility = View.GONE
                buttonEdit.text = getString(R.string.button_edit)
                isEditing = false
                Toast.makeText(this, getString(R.string.toast_saved), Toast.LENGTH_SHORT).show()
            }
        }

        buttonExit.setOnClickListener {
            Toast.makeText(this, getString(R.string.toast_exit), Toast.LENGTH_SHORT).show()
            finish()
        }
    }

    private fun saveData(name: String) {
        with(sharedPref.edit()) {
            putString("user_name", name)
            apply()
        }
    }

    private fun loadSavedData() {
        val savedName = sharedPref.getString("user_name", getString(R.string.profile_name))
        textName.text = savedName
    }
}
```

## Ответы на контрольные вопросы

**1. Для чего используется ConstraintLayout? Какие у него преимущества перед LinearLayout?**

ConstraintLayout — это гибкий менеджер компоновки, который позволяет создавать сложные и плоские иерархии представлений. Основные преимущества перед LinearLayout:

- **Гибкое позиционирование** — элементы можно привязывать друг к другу, к родителю, к направляющим (guidelines) в любых комбинациях
- **Плоская иерархия** — позволяет создавать сложные интерфейсы без вложенных layout'ов, что повышает производительность
- **Процентное позиционирование** — можно размещать элементы на определённом проценте от родителя (например, на 30% высоты экрана)
- **Цепочки (chains)** — возможность создавать группы элементов с равномерным распределением пространства
- **Адаптивность** — легче создавать интерфейсы, которые хорошо выглядят на разных размерах экранов

**2. Что такое app:layout_constraint... атрибуты?**

Это атрибуты, определяющие привязки (constraints) элемента внутри ConstraintLayout. Они задают, к какой стороне другого элемента или родителя привязана текущая сторона:

- `app:layout_constraintLeft_toLeftOf` — левая сторона элемента привязана к левой стороне другого элемента
- `app:layout_constraintRight_toRightOf` — правая сторона привязана к правой стороне
- `app:layout_constraintTop_toBottomOf` — верхняя сторона привязана к нижней стороне другого элемента
- `app:layout_constraintBottom_toTopOf` — нижняя сторона привязана к верхней стороне
- `app:layout_constraintGuide_percent` — для Guideline задаёт процентное расположение

**3. Как вынести размеры и цвета в ресурсы? Зачем это нужно?**

Размеры выносятся в файл `res/values/dimens.xml`:
```xml
<dimen name="text_size_name">24sp</dimen>
```
Цвета выносятся в файл `res/values/colors.xml`:
```xml
<color name="fraise">#FF99D3</color>
```
Используются в layout так:
```xml
android:textSize="@dimen/text_size_name"
android:textColor="@color/fraise"
```

**Зачем это нужно:**
- **Единообразие дизайна** — все размеры и цвета в одном месте
- **Лёгкость поддержки** — изменение одного значения в ресурсах обновит его во всём приложении
- **Адаптация под разные устройства** — можно создавать альтернативные ресурсы для разных экранов
- **Локализация** — строки можно переводить на другие языки

**4. Каким образом можно обработать клик на кнопке в Kotlin-коде?**

Самый распространённый способ — установка слушателя через `setOnClickListener`:

```kotlin
val button = findViewById<Button>(R.id.buttonEdit)
button.setOnClickListener {
    // Действия при нажатии
    Toast.makeText(this, "Кнопка нажата", Toast.LENGTH_SHORT).show()
}
```

Альтернативные способы:
- Реализация интерфейса `View.OnClickListener` в Activity
- Использование лямбда-выражений (как в примере выше)
- Атрибут `android:onClick` в XML (менее гибкий способ)

**5. Как добавить обработчик нажатия на ImageView?**

ImageView обрабатывает нажатия так же, как и кнопка, но нужно явно указать, что он кликабельный:

```kotlin
val imageView = findViewById<ImageView>(R.id.imageAvatar)
imageView.isClickable = true  // или в XML android:clickable="true"
imageView.setOnClickListener {
    Toast.makeText(this, "Аватар нажат", Toast.LENGTH_SHORT).show()
}
```

В XML можно добавить:
```xml
android:clickable="true"
android:focusable="true"
```

## Вывод

В ходе лабораторной работы был создан экран профиля для Android-приложения с использованием ConstraintLayout и обработкой событий.

Были изучены и применены на практике:

- **ConstraintLayout** — освоено позиционирование элементов через привязки, использование направляющей (Guideline) для центрирования группы элементов по вертикали, создание вложенного контейнера для стабильности интерфейса при изменении видимости элементов

- **Обработка событий** — реализованы обработчики нажатия для двух кнопок, переключение режима редактирования с изменением видимости TextView и EditText, обратная связь через Toast

- **Ресурсы** — размеры вынесены в `dimens.xml`, цвета в `colors.xml`, строки в `strings.xml` для единообразия и лёгкости поддержки

- **Визуальные эффекты** — добавлены тени через атрибуты `elevation` и `shadowColor` с использованием цветовой палитры

- **Сохранение данных** — реализовано хранение отредактированного имени между сессиями через SharedPreferences

В приложении реализован функционал: отображение профиля (аватар, имя, статус), редактирование имени с сохранением, выход из приложения. Приложение корректно работает на эмуляторе и сохраняет данные между сессиями.

Таким образом, цель работы достигнута: получены практические навыки создания гибких интерфейсов с помощью ConstraintLayout и обработки пользовательских событий.
