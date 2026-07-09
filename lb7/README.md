# Лабораторная работа №7

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

Лабораторная работа №7  
**«Добавление второго экрана (детали задачи). Переход по клику на элемент списка»**  
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

Научиться создавать многоэкранные приложения, осуществлять переход между экранами с передачей данных через Intent, обрабатывать клики на элементах RecyclerView. В результате модернизировать приложение ToDo-списка, добавив экран для просмотра и редактирования деталей задачи, а также реализовать удаление задачи с экрана деталей.

## Индивидуальное задание

Реализованы два индивидуальных задания:

1. **Удаление задачи с экрана деталей** – при нажатии на кнопку «Удалить» на экране деталей задача удаляется из списка, второй экран закрывается, список обновляется.  
2. **Редактирование деталей задачи** – на экране деталей отображается неизменяемое имя задачи и редактируемое поле «Детали задачи». При нажатии «Сохранить» детали обновляются в списке и сохраняются в памяти.

При этом имя задачи и её детали не связаны – редактируются только детали. Кнопки на экране деталей расположены внизу для удобства использования.

Весь функционал из лабораторной работы №6 (счётчик, поле ввода текста, добавление задач, удаление свайпом, удаление по индексу, очистка списка, чередование цветов карточек) сохранён.

## Скриншоты

![Главный экран с RecyclerView и карточками](screenshot_main_lab7.png)  
*Рисунок 1 – Главный экран приложения: счётчик, поле ввода, список задач с чередованием цветов фона*

<br>

![Экран деталей задачи](screenshot_detail.png)  
*Рисунок 2 – Экран деталей: неизменяемое имя задачи, поле для ввода деталей, кнопки внизу экрана*

<br>

![Структура проекта с новыми файлами](screenshot_structure_lab7.png)  
*Рисунок 3 – Структура проекта: добавлены DetailActivity.kt, activity_detail.xml, Task.kt*

## Листинги

### 1. Модель данных `Task.kt`

```kotlin
package com.example.todoapp

import java.io.Serializable

data class Task(
    val name: String,
    var details: String
) : Serializable
```

### 2. Файл разметки экрана деталей `activity_detail.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/textTaskName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="22sp"
        android:textStyle="bold"
        android:layout_marginBottom="24sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Детали задачи:"
        android:textSize="18sp"
        android:layout_below="@id/textTaskName"
        android:layout_marginBottom="8dp" />

    <EditText
        android:id="@+id/editTaskDetails"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_below="@id/textTaskName"
        android:layout_above="@id/buttonsLayout"
        android:layout_marginTop="32dp"
        android:layout_marginBottom="16dp"
        android:gravity="top"
        android:inputType="textMultiLine"
        android:hint="Введите детали задачи..."
        android:minHeight="150dp" />

    <LinearLayout
        android:id="@+id/buttonsLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_horizontal"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="16dp">

        <Button
            android:id="@+id/buttonSave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Сохранить"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/buttonDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Удалить"
            android:backgroundTint="@android:color/holo_red_dark"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/buttonBack"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Назад" />
    </LinearLayout>

</RelativeLayout>
```

### 3. `DetailActivity.kt`

```kotlin
package com.example.todoapp

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class DetailActivity : AppCompatActivity() {

    private lateinit var textTaskName: TextView
    private lateinit var editTaskDetails: EditText
    private lateinit var buttonSave: Button
    private lateinit var buttonDelete: Button
    private lateinit var buttonBack: Button

    private var taskPosition: Int = -1
    private var originalDetails: String = ""

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        textTaskName = findViewById(R.id.textTaskName)
        editTaskDetails = findViewById(R.id.editTaskDetails)
        buttonSave = findViewById(R.id.buttonSave)
        buttonDelete = findViewById(R.id.buttonDelete)
        buttonBack = findViewById(R.id.buttonBack)

        // Получаем переданные данные
        taskPosition = intent.getIntExtra("task_position", -1)
        val taskName = intent.getStringExtra("task_name") ?: ""
        originalDetails = intent.getStringExtra("task_details") ?: ""

        textTaskName.text = taskName
        editTaskDetails.setText(originalDetails)

        buttonSave.setOnClickListener {
            val newDetails = editTaskDetails.text.toString()
            if (newDetails != originalDetails) {
                val resultIntent = Intent().apply {
                    putExtra("action", "edit")
                    putExtra("position", taskPosition)
                    putExtra("new_details", newDetails)
                }
                setResult(Activity.RESULT_OK, resultIntent)
                finish()
            } else {
                // Ничего не изменилось - просто закрываем
                finish()
            }
        }

        buttonDelete.setOnClickListener {
            val resultIntent = Intent().apply {
                putExtra("action", "delete")
                putExtra("position", taskPosition)
            }
            setResult(Activity.RESULT_OK, resultIntent)
            finish()
        }

        buttonBack.setOnClickListener {
            finish()
        }
    }
}
```

### 4. Адаптер `TaskAdapter.kt`

```kotlin
package com.example.todoapp

import android.graphics.Color
import android.graphics.Paint
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.cardview.widget.CardView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(
    private val tasks: MutableList<Task>,
    private val onItemClick: (Int) -> Unit,
    private val onItemLongClick: (Int) -> Unit
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
        holder.textTask.text = task.name

        // Сброс зачёркивания
        holder.textTask.paintFlags = holder.textTask.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
        holder.checkTask.isChecked = false

        // Чередование фона карточки и цвета текста
        val cardView = holder.itemView as CardView
        if (position % 2 == 0) {
            cardView.setCardBackgroundColor(Color.BLACK)
            holder.textTask.setTextColor(Color.WHITE)
        } else {
            cardView.setCardBackgroundColor(Color.LTGRAY)
            holder.textTask.setTextColor(Color.BLACK)
        }

        // Обработка чекбокса (визуальное зачёркивание)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
        }

        holder.itemView.setOnClickListener { onItemClick(position) }
        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }
    }

    override fun getItemCount(): Int = tasks.size
}
```

### 5. `MainActivity.kt`

```kotlin
package com.example.todoapp

import android.app.Activity
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar

class MainActivity : AppCompatActivity() {

    private var counter = 0
    private val tasks = mutableListOf<Task>()
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    // Лаунчер для получения результата из DetailActivity
    private val detailActivityLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            val data = result.data
            val action = data?.getStringExtra("action")
            val position = data?.getIntExtra("position", -1) ?: -1

            when (action) {
                "edit" -> {
                    val newDetails = data.getStringExtra("new_details") ?: ""
                    if (position in tasks.indices) {
                        tasks[position].details = newDetails
                        adapter.notifyItemChanged(position)
                        saveTasks()
                        Toast.makeText(this, "Детали обновлены", Toast.LENGTH_SHORT).show()
                    }
                }
                "delete" -> {
                    if (position in tasks.indices) {
                        val removed = tasks.removeAt(position)
                        adapter.notifyItemRemoved(position)
                        saveTasks()
                        Toast.makeText(this, "Удалено: ${removed.name}", Toast.LENGTH_SHORT).show()
                    }
                }
            }
        }
    }

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

        adapter = TaskAdapter(
            tasks = tasks,
            onItemClick = { position ->
                val task = tasks[position]
                val intent = Intent(this, DetailActivity::class.java).apply {
                    putExtra("task_position", position)
                    putExtra("task_name", task.name)
                    putExtra("task_details", task.details)
                }
                detailActivityLauncher.launch(intent)
            },
            onItemLongClick = { position ->
                deleteTask(position)
            }
        )
        recyclerView.adapter = adapter

        // Удаление свайпом
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

        // --- Добавление задачи (имя) ---
        buttonAddTask.setOnClickListener {
            val taskName = editTextTask.text.toString()
            if (taskName.isNotBlank()) {
                val newTask = Task(taskName, "") // детали пока пустые
                tasks.add(newTask)
                adapter.notifyItemInserted(tasks.size - 1)
                editTextTask.text.clear()
                saveTasks()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите название задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Очистить всё ---
        buttonClearAll.setOnClickListener {
            tasks.clear()
            adapter.notifyDataSetChanged()
            saveTasks()
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        // --- Удаление по индексу ---
        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in tasks.indices) {
                    val removed = tasks.removeAt(index)
                    adapter.notifyItemRemoved(index)
                    saveTasks()
                    Toast.makeText(this, "Удалена задача: ${removed.name}", Toast.LENGTH_SHORT).show()
                    editTextIndex.text.clear()
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${tasks.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun deleteTaskWithUndo(position: Int) {
        val removedTask = tasks[position]
        tasks.removeAt(position)
        adapter.notifyItemRemoved(position)
        saveTasks()

        Snackbar.make(recyclerView, "Задача удалена", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                tasks.add(position, removedTask)
                adapter.notifyItemInserted(position)
                saveTasks()
            }
            .show()
    }

    private fun deleteTask(position: Int) {
        tasks.removeAt(position)
        adapter.notifyItemRemoved(position)
        saveTasks()
        Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
    }

    private fun updateCounterDisplay(textView: TextView) {
        textView.text = "Счётчик: $counter"
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    // Сохранение списка задач в SharedPreferences (имя|||детали, разделитель задач |||)
    private fun saveTasks() {
        val tasksString = tasks.joinToString("|||") { task ->
            "${task.name}|${task.details}"
        }
        sharedPreferences.edit().putString("tasks", tasksString).apply()
    }

    // Загрузка списка задач из SharedPreferences
    private fun loadSavedData() {
        counter = sharedPreferences.getInt("counter", 0)
        val tasksString = sharedPreferences.getString("tasks", "")
        if (!tasksString.isNullOrEmpty()) {
            tasks.clear()
            val items = tasksString.split("|||")
            for (item in items) {
                val parts = item.split("|")
                if (parts.size == 2) {
                    tasks.add(Task(parts[0], parts[1]))
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        if (::adapter.isInitialized) {
            adapter.notifyDataSetChanged()
        }
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

### 6. Файл разметки элемента списка `item_task.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

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
            android:textSize="18sp" />

        <CheckBox
            android:id="@+id/checkTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

### 7. Зависимости в `build.gradle (Module: app)`

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

**1. Что такое Intent? Какие виды Intent существуют?**  
Intent — объект для запуска компонентов (Activity, сервисов). Виды: явный (указывает конкретный класс) и неявный (описывает действие, систему выбирает компонент).

**2. Как передать данные из одной Activity в другую?**  
Через методы `putExtra()` объекта Intent перед запуском. В целевой Activity данные получают через `getStringExtra()` и аналогичные методы.

**3. Какие способы обработки кликов на элементах RecyclerView вы знаете?**  
Передача лямбды в адаптер, создание интерфейса-слушателя, установка слушателя в onBindViewHolder. Рекомендуемый — передача лямбды.

**4. Как создать новую Activity в Android Studio?**  
ПКМ по пакету → New → Activity → Empty Activity. Указать имя, снять галочку Launcher Activity.

**5. Для чего используется метод `finish()`?**  
Закрывает текущую Activity, возвращая пользователя к предыдущей.

## Вывод

В ходе лабораторной работы №7 приложение ToDo-списка модернизировано: добавлен второй экран для просмотра и редактирования деталей задач. Реализованы индивидуальные задания — удаление задачи с экрана деталей и редактирование деталей (имя задачи остаётся неизменным). Кнопки на экране деталей расположены внизу для удобства. Данные (имя + детали) сохраняются в SharedPreferences и восстанавливаются после перезапуска без использования плагинов. Весь функционал лабораторной работы №6 (счётчик, добавление, удаление свайпом, удаление по индексу, очистка, чередование цветов карточек) сохранён. Цель работы достигнута.
