
# Лабораторная работа №6

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

Лабораторная работа №6  
**«Модернизация ToDo-списка: RecyclerView, CardView, удаление свайпом»**  
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

Модернизировать существующее приложение ToDo-списка (Лабораторная работа №5): заменить статическое отображение задач на динамический `RecyclerView` с использованием `CardView`, добавить чекбоксы для визуальной отметки выполнения и реализовать удаление задачи свайпом с возможностью отмены через `Snackbar`.

## Индивидуальное задание

Реализовано **удаление задачи свайпом** влево или вправо. При свайпе задача удаляется из списка, появляется `Snackbar` с кнопкой «Отменить». При нажатии на кнопку задача восстанавливается на прежнее место. Весь функционал из лабораторной работы №5 (счётчик, поле ввода текста, добавление задач, удаление по индексу, очистка списка) сохранён.

## Скриншоты

![Главный экран с RecyclerView и карточками](screenshot_main.png)  
*Рисунок 1 – Главный экран приложения: счётчик, поле ввода, список задач в виде карточек с чекбоксами*

<br>

![Удаление свайпом с Snackbar](screenshot_swipe_delete.png)  
*Рисунок 2 – После свайпа появляется Snackbar с предложением отменить удаление*

<br>

![Структура проекта с новыми файлами](screenshot_structure.png)  
*Рисунок 3 – Структура проекта: добавлены item_task.xml, TaskAdapter.kt*

## Листинги

### 1. Файл разметки элемента списка `item_task.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp"
    app:cardBackgroundColor="#FFFFFF">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp">

        <TextView
            android:id="@+id/textTask"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="18sp"
            android:textColor="#333333"/>

        <CheckBox
            android:id="@+id/checkTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

### 2. Адаптер `TaskAdapter.kt`

```kotlin
package com.example.todoapp

import android.graphics.Paint
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(
    private val tasks: MutableList<String>,
    private val onItemLongClick: (Int) -> Unit  // для долгого нажатия (опционально)
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    inner class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val task = tasks[position]
        holder.textTask.text = task

        // Сброс флага зачёркивания при переиспользовании
        holder.textTask.paintFlags = holder.textTask.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
        holder.checkTask.isChecked = false

        // Обработка чекбокса – перечёркивание текста (без сохранения состояния)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
        }

        // Долгое нажатие – удаление (альтернатива свайпу)
        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateData(newTasks: List<String>) {
        tasks.clear()
        tasks.addAll(newTasks)
        notifyDataSetChanged()
    }
}
```

### 3. Обновлённый `activity_main.xml` (фрагмент со списком)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Блок 1: Счётчик -->
    <TextView
        android:id="@+id/textCounter"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Счётчик: 0"
        android:textSize="24sp"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/buttonIncrement"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Увеличить"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonReset"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Сброс"
        android:layout_marginBottom="24dp"/>

    <!-- Блок 2: Поле ввода и отображение текста -->
    <EditText
        android:id="@+id/editTextInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Введите текст"
        android:inputType="text"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonShow"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Показать"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:id="@+id/textEntered"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Введённый текст: "
        android:textSize="18sp"
        android:layout_marginBottom="24dp"/>

    <!-- Блок 3: ToDo список (добавление новой задачи) -->
    <EditText
        android:id="@+id/editTextTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Новая задача"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonAddTask"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Добавить задачу"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonClearAll"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Очистить всё"
        android:layout_marginBottom="8dp"/>

    <!-- Индивидуальное задание: удаление по индексу -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginBottom="8dp">

        <EditText
            android:id="@+id/editTextIndex"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Индекс задачи (0..N-1)"
            android:inputType="number"
            android:layout_marginEnd="8dp"/>

        <Button
            android:id="@+id/buttonDeleteTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Удалить по индексу"/>
    </LinearLayout>

    <!-- RecyclerView для списка задач -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerViewTasks"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:background="#F5F5F5"/>

</LinearLayout>
```

### 4. `MainActivity.kt` с добавлением `RecyclerView`, адаптера и `ItemTouchHelper`

```kotlin
package com.example.todoapp

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar

class MainActivity : AppCompatActivity() {

    private var counter = 0
    private val tasks = mutableListOf<String>()
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sharedPreferences = getSharedPreferences("TodoAppPrefs", Context.MODE_PRIVATE)

        loadSavedData()

        val textCounter = findViewById<TextView>(R.id.textCounter)
        val buttonIncrement = findViewById<Button>(R.id.buttonIncrement)
        val buttonReset = findViewById<Button>(R.id.buttonReset)

        val editTextInput = findViewById<EditText>(R.id.editTextInput)
        val buttonShow = findViewById<Button>(R.id.buttonShow)
        val textEntered = findViewById<TextView>(R.id.textEntered)

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val buttonClearAll = findViewById<Button>(R.id.buttonClearAll)

        val editTextIndex = findViewById<EditText>(R.id.editTextIndex)
        val buttonDeleteTask = findViewById<Button>(R.id.buttonDeleteTask)

        recyclerView = findViewById(R.id.recyclerViewTasks)
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(tasks) { position ->
            // Обработка долгого нажатия – тоже удаление (опционально)
            deleteTask(position)
        }
        recyclerView.adapter = adapter

        // Удаление свайпом (влево или вправо)
        val swipeCallback = object : ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT) {
            override fun onMove(
                recyclerView: RecyclerView,
                viewHolder: RecyclerView.ViewHolder,
                target: RecyclerView.ViewHolder
            ): Boolean = false

            override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
                val position = viewHolder.adapterPosition
                deleteTaskWithUndo(position)
            }
        }
        ItemTouchHelper(swipeCallback).attachToRecyclerView(recyclerView)

        // --- Логика счётчика ---
        updateCounterDisplay(textCounter)

        buttonIncrement.setOnClickListener {
            counter++
            updateCounterDisplay(textCounter)
            saveCounter()
        }

        buttonReset.setOnClickListener {
            counter = 0
            updateCounterDisplay(textCounter)
            saveCounter()
        }

        // --- Поле для ввода текста (из Лаб.5) ---
        buttonShow.setOnClickListener {
            val inputText = editTextInput.text.toString()
            if (inputText.isNotBlank()) {
                textEntered.text = "Введённый текст: $inputText"
            } else {
                Toast.makeText(this, "Введите текст", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Добавление задачи ---
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                adapter.notifyItemInserted(tasks.size - 1)
                editTextTask.text.clear()
                saveTasks()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Очистить всё ---
        buttonClearAll.setOnClickListener {
            tasks.clear()
            adapter.notifyDataSetChanged()
            saveTasks()
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        // --- Удаление по индексу (индивидуальное задание) ---
        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in tasks.indices) {
                    val removed = tasks.removeAt(index)
                    adapter.notifyItemRemoved(index)
                    saveTasks()
                    Toast.makeText(this, "Удалена задача: $removed", Toast.LENGTH_SHORT).show()
                    editTextIndex.text.clear()
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${tasks.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }
    }

    // Удаление свайпом с возможностью отмены через Snackbar
    private fun deleteTaskWithUndo(position: Int) {
        val removedTask = tasks[position]
        tasks.removeAt(position)
        adapter.notifyItemRemoved(position)
        saveTasks()

        Snackbar.make(recyclerView, "Задача удалена", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                // Восстанавливаем задачу на ту же позицию
                tasks.add(position, removedTask)
                adapter.notifyItemInserted(position)
                saveTasks()
            }
            .show()
    }

    // Простое удаление (используется для долгого нажатия, если нужно)
    private fun deleteTask(position: Int) {
        tasks.removeAt(position)
        adapter.notifyItemRemoved(position)
        saveTasks()
        Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
    }

    // --- Вспомогательные методы для обновления UI и сохранения ---
    private fun updateCounterDisplay(textView: TextView) {
        textView.text = "Счётчик: $counter"
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    private fun saveTasks() {
        val tasksString = tasks.joinToString("|||")
        sharedPreferences.edit().putString("tasks", tasksString).apply()
    }

    private fun loadSavedData() {
        counter = sharedPreferences.getInt("counter", 0)
        val tasksString = sharedPreferences.getString("tasks", "")
        if (!tasksString.isNullOrEmpty()) {
            tasks.clear()
            tasks.addAll(tasksString.split("|||"))
        }
        // Обновим UI после загрузки (вызывается до создания RecyclerView, поэтому адаптер ещё не инициализирован)
        // Но адаптер будет создан позже, и он получит уже заполненный список tasks, а также вызовем notifyDataSetChanged.
    }

    override fun onResume() {
        super.onResume()
        // Если адаптер уже создан, обновляем его после загрузки
        if (::adapter.isInitialized) {
            adapter.notifyDataSetChanged()
        }
        // Обновим отображение счётчика
        findViewById<TextView>(R.id.textCounter)?.let { updateCounterDisplay(it) }
    }

    override fun onPause() {
        super.onPause()
        saveCounter()
        saveTasks()
    }

    override fun onDestroy() {
        super.onDestroy()
        saveCounter()
        saveTasks()
    }
}
```

### 5. Зависимости в `build.gradle (Module: app)`

```gradle
dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```

## Ответы на контрольные вопросы

**1. Для чего нужен `RecyclerView`? Чем он лучше `ListView`?**  

`RecyclerView` – это более гибкий и производительный компонент для отображения больших списков. Он лучше `ListView` по следующим причинам:
- **Принудительное использование `ViewHolder`** – уменьшает количество вызовов `findViewById`.
- **Разделение ответственности** – `LayoutManager` отвечает за расположение элементов, `Adapter` – за данные, `ItemAnimator` – за анимации.
- **Встроенная поддержка разных типов элементов** (например, заголовки и строки).
- **Эффективная работа с большими списками** за счёт переиспользования views.
- **Гибкая настройка анимаций** добавления/удаления.

**2. Какие компоненты необходимы для работы `RecyclerView`?**  

Для работы `RecyclerView` необходимы три ключевых компонента:
- **`RecyclerView`** – сам контейнер, размещаемый в разметке.
- **`LayoutManager`** – определяет способ расположения элементов (LinearLayoutManager, GridLayoutManager, StaggeredGridLayoutManager).
- **`Adapter`** – подготавливает данные и создаёт ViewHolder'ы.
- **`ViewHolder`** – внутренний класс адаптера, кэширующий ссылки на виджеты элемента списка (обязателен, но реализуется внутри адаптера).

**3. Что такое `ViewHolder` и для чего он используется?**  

`ViewHolder` – это объект-обёртка, который хранит ссылки на дочерние View одного элемента списка (например, `TextView` и `CheckBox`). Он используется для избежания частых вызовов `findViewById()` при прокрутке списка. Без `ViewHolder` при каждом связывании данных система искала бы компоненты по id, что замедляет работу. `RecyclerView` требует явного создания `ViewHolder` в адаптере.

**4. Чем отличается `notifyDataSetChanged()` от `notifyItemInserted()`?**  

- **`notifyDataSetChanged()`** – уведомляет, что изменился весь набор данных. `RecyclerView` перерисовывает все видимые элементы, что неэффективно. Не использует анимации по умолчанию.
- **`notifyItemInserted(int position)`** – сообщает, что в конкретной позиции вставлен новый элемент. `RecyclerView` запускает анимацию появления и перерисовывает только затронутые элементы, что оптимально.

Аналогично есть `notifyItemRemoved()`, `notifyItemChanged()` и другие.

**5. Как добавить обработку кликов на элементы `RecyclerView`?**  

В `RecyclerView` нет встроенного метода `setOnItemClickListener`. Обработку кликов нужно реализовать самостоятельно, обычно через интерфейс обратного вызова в адаптере:

```kotlin
class TaskAdapter(
    private val tasks: List<String>,
    private val onItemClick: (Int) -> Unit
) : RecyclerView.Adapter<...>() {
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.itemView.setOnClickListener { onItemClick(position) }
    }
}
```

Затем в `MainActivity` передать лямбду: `TaskAdapter(tasks) { position -> ... }`. Аналогично обрабатываются долгие нажатия, свайпы и другие жесты.

## Вывод

В ходе выполнения лабораторной работы проведена модернизация приложения ToDo-списка. Вместо устаревшего `TextView` внутри `ScrollView` внедрён современный компонент `RecyclerView` с использованием `CardView` для каждого элемента. Добавлены чекбоксы, позволяющие визуально отмечать выполненные задачи (перечёркивание текста). Реализовано удаление задачи свайпом с отменой через `Snackbar` – одно из самых популярных и удобных UX-решений. 

При этом полностью сохранён весь функционал предыдущей лабораторной работы:
- счётчик нажатий с сохранением через `SharedPreferences`;
- поле ввода произвольного текста и его отображение;
- добавление новых задач;
- удаление задачи по индексу (индивидуальное задание №5);
- очистка всего списка.

Благодаря использованию `RecyclerView` и `notifyItemInserted()` / `notifyItemRemoved()` приложение работает быстрее и анимации удаления/добавления выглядят плавно. `Snackbar` с кнопкой «Отменить» улучшает пользовательский опыт: случайно удалённую задачу можно мгновенно восстановить.

Таким образом, цель работы достигнута: получены практические навыки работы с `RecyclerView`, `CardView`, `ItemTouchHelper` и `Snackbar` в Android-приложении на Kotlin. Приложение готово к дальнейшему расширению (например, сохранение состояния чекбоксов, редактирование задачи и т.д.).
