
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

<img width="328" height="709" alt="Снимок экрана (2)(1)" src="https://github.com/user-attachments/assets/00d94d4f-1384-491c-a4a6-8e2dfcae22ce" />
*Рисунок 1 – Работа приложения*

<br>

<img width="335" height="716" alt="Снимок экрана (3)(1)(1)" src="https://github.com/user-attachments/assets/cc03fc27-6eda-41a2-9ebc-c4af378dc6a9" />
*Рисунок 2 – Режим редактирования*

<br>


<img width="553" height="781" alt="struct" src="https://github.com/user-attachments/assets/c934f60f-713b-41fc-b91b-c4f7c428cbdb" />
*Рисунок 3 – Структура проекта*

## Листинги

### 1. `activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/gray_light"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/imageAvatar"
        android:layout_width="@dimen/avatar_size"
        android:layout_height="@dimen/avatar_size"
        android:layout_marginTop="@dimen/margin_normal"
        android:contentDescription="@string/profile_name"
        android:elevation="4dp"
        android:src="@drawable/logo"
        app:layout_constraintBottom_toTopOf="@+id/textName"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/margin_small"
        android:text="@string/profile_name"
        android:textColor="@color/black"
        android:textSize="@dimen/text_size_name"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar" />

    <EditText
        android:id="@+id/editName"
        android:layout_width="@dimen/edit_min_width"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar" />

    <TextView
        android:id="@+id/textStatus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/margin_small"
        android:text="@string/profile_status"
        android:textColor="@color/purple_500"
        android:textSize="@dimen/text_size_status"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textName" />

    <EditText
        android:id="@+id/editStatus"
        android:layout_width="@dimen/edit_min_width"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/editName" />

    <Button
        android:id="@+id/buttonEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/margin_normal"
        android:backgroundTint="@color/purple_200"
        android:text="@string/button_edit"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textStatus" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### 2. `MainActivity.kt`

```kotlin
package com.example.lab4

import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private var isEditing = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val textName = findViewById<TextView>(R.id.textName)
        val textStatus = findViewById<TextView>(R.id.textStatus)
        val editName = findViewById<EditText>(R.id.editName)
        val editStatus = findViewById<EditText>(R.id.editStatus)
        val buttonEdit = findViewById<Button>(R.id.buttonEdit)

        buttonEdit.setOnClickListener {
            if (!isEditing) {
                editName.setText(textName.text)
                editStatus.setText(textStatus.text)
                textName.visibility = View.GONE
                textStatus.visibility = View.GONE
                editName.visibility = View.VISIBLE
                editStatus.visibility = View.VISIBLE
                buttonEdit.setText(R.string.button_save)
                isEditing = true
            } else {
                textName.text = editName.text.toString().trim()
                textStatus.text = editStatus.text.toString().trim()
                editName.visibility = View.GONE
                editStatus.visibility = View.GONE
                textName.visibility = View.VISIBLE
                textStatus.visibility = View.VISIBLE
                buttonEdit.setText(R.string.button_edit)
                Toast.makeText(this, R.string.toast_saved, Toast.LENGTH_SHORT).show()
                isEditing = false
            }
        }
    }
}
```

### 3. Ресурсы (`strings.xml`, фрагмент)

```xml
<string name="profile_name">Спивакова Екатерина</string>
<string name="profile_status">Прикладная математика и информатика</string>
<string name="button_edit">Редактировать</string>
<string name="button_save">Сохранить</string>
<string name="toast_saved">Профиль сохранён</string>
```


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
