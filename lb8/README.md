# ОТЧЁТ ПО ЛАБОРАТОРНОЙ РАБОТЕ №8

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

Лабораторная работа №8  
**«Перенос логики списка задач из Activity в ViewModel. Использование StateFlow для хранения состояния»**  
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

Изучить архитектурный компонент ViewModel, научиться выносить логику и состояние UI из Activity, использовать StateFlow для реактивного обновления данных, обеспечить сохранение состояния при изменении конфигурации (повороте экрана). В качестве индивидуального задания реализовать удаление задачи свайпом с возможностью отмены через Snackbar.

---

## Индивидуальное задание

Реализовано **удаление задачи свайпом** влево или вправо с использованием `ItemTouchHelper`. При свайпе задача удаляется из списка, появляется `Snackbar` с кнопкой «Отменить». При нажатии на кнопку задача восстанавливается на прежнее место. Весь функционал из лабораторной работы №7 (список задач, переход на экран деталей, добавление задач) сохранён и перенесён в ViewModel.

---

## Скриншоты
<br>

<img width="321" height="700" alt="Снимок экрана (6)(1)" src="https://github.com/user-attachments/assets/9bd1b7c0-7e98-4aa6-b680-7b51c30b2fe0" />
*Рисунок 3 – После свайпа появляется Snackbar с предложением отменить удаление*

<br>
---

## Листинги

### 1. `MainViewModel.kt`

```kotlin
package com.example.todoapp

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class MainViewModel : ViewModel() {

    private val _tasks = MutableStateFlow<List<Task>>(emptyList())
    val tasks: StateFlow<List<Task>> = _tasks.asStateFlow()

    fun setTasks(newTasks: List<Task>) {
        _tasks.value = newTasks.toList()
    }

    fun addTask(task: Task) {
        val currentList = _tasks.value.toMutableList()
        currentList.add(task)
        _tasks.value = currentList
    }

    fun deleteTask(index: Int) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList.removeAt(index)
            _tasks.value = currentList
        }
    }

    fun updateTaskDetails(index: Int, newDetails: String) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList[index] = currentList[index].copy(details = newDetails)
            _tasks.value = currentList
        }
    }

    fun restoreTask(index: Int, task: Task) {
        val currentList = _tasks.value.toMutableList()
        if (index <= currentList.size) {
            currentList.add(index, task)
            _tasks.value = currentList
        }
    }
}
```

---

### 2. `TaskAdapter.kt`

```kotlin
package com.example.todoapp

import android.graphics.Color
import android.graphics.Paint
import android.view.LayoutInflater
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.cardview.widget.CardView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(
    private var tasks: List<Task>,
    private val onItemClick: (Int) -> Unit,
    private val onItemLongClick: (Int) -> Unit
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    class TaskViewHolder(val itemView: android.view.View) : RecyclerView.ViewHolder(itemView) {
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

    fun updateData(newTasks: List<Task>) {
        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

---

### 3. `MainActivity.kt`

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
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    private var deletedTask: Task? = null
    private var deletedTaskPosition: Int = -1

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
                    if (position in 0 until viewModel.tasks.value.size) {
                        viewModel.updateTaskDetails(position, newDetails)
                        Toast.makeText(this, "Детали обновлены", Toast.LENGTH_SHORT).show()
                    }
                }
                "delete" -> {
                    if (position in 0 until viewModel.tasks.value.size) {
                        viewModel.deleteTask(position)
                        Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
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
            tasks = emptyList(),
            onItemClick = { position ->
                val task = viewModel.tasks.value[position]
                val intent = Intent(this, DetailActivity::class.java).apply {
                    putExtra("task_position", position)
                    putExtra("task_name", task.name)
                    putExtra("task_details", task.details)
                }
                detailActivityLauncher.launch(intent)
            },
            onItemLongClick = { position ->
                deleteTaskWithUndo(position)
            }
        )
        recyclerView.adapter = adapter

        // Подписка на изменения StateFlow
        lifecycleScope.launch {
            viewModel.tasks.collect { tasks ->
                adapter.updateData(tasks)
                saveTasks()
            }
        }

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

        // --- Поле для ввода текста ---
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
            val taskName = editTextTask.text.toString()
            if (taskName.isNotBlank()) {
                val newTask = Task(taskName, "")
                viewModel.addTask(newTask)
                editTextTask.text.clear()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите название задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Очистить всё ---
        buttonClearAll.setOnClickListener {
            viewModel.setTasks(emptyList())
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        // --- Удаление по индексу ---
        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in 0 until viewModel.tasks.value.size) {
                    val removed = viewModel.tasks.value[index]
                    viewModel.deleteTask(index)
                    Toast.makeText(this, "Удалена задача: ${removed.name}", Toast.LENGTH_SHORT).show()
                    editTextIndex.text.clear()
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${viewModel.tasks.value.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // Загрузка тестовых данных (если нужно)
        if (viewModel.tasks.value.isEmpty()) {
            // Можно оставить пустым или загрузить тестовые
            // viewModel.addTask(Task("Купить продукты", "Молоко, хлеб"))
        }
    }

    private fun deleteTaskWithUndo(position: Int) {
        val tasks = viewModel.tasks.value
        if (position !in tasks.indices) return

        deletedTask = tasks[position]
        deletedTaskPosition = position

        viewModel.deleteTask(position)

        Snackbar.make(recyclerView, "Задача удалена: ${deletedTask?.name?.take(30)}", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                deletedTask?.let {
                    viewModel.restoreTask(deletedTaskPosition, it)
                }
                deletedTask = null
                deletedTaskPosition = -1
            }
            .show()
    }

    // --- Счётчик и сохранение ---
    private var counter = 0

    private fun updateCounterDisplay(textView: TextView) {
        textView.text = "Счётчик: $counter"
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    private fun saveTasks() {
        val tasksString = viewModel.tasks.value.joinToString("|||") { task ->
            "${task.name}|${task.details}"
        }
        sharedPreferences.edit().putString("tasks", tasksString).apply()
    }

    private fun loadSavedData() {
        counter = sharedPreferences.getInt("counter", 0)
        val tasksString = sharedPreferences.getString("tasks", "")
        val loadedTasks = mutableListOf<Task>()
        if (!tasksString.isNullOrEmpty()) {
            val items = tasksString.split("|||")
            for (item in items) {
                val parts = item.split("|")
                if (parts.size == 2) {
                    loadedTasks.add(Task(parts[0], parts[1]))
                }
            }
        }
        viewModel.setTasks(loadedTasks)
    }

    override fun onResume() {
        super.onResume()
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

---

### 4. `DetailActivity.kt`

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
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    private var deletedTask: Task? = null
    private var deletedTaskPosition: Int = -1

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
                    if (position in 0 until viewModel.tasks.value.size) {
                        viewModel.updateTaskDetails(position, newDetails)
                        Toast.makeText(this, "Детали обновлены", Toast.LENGTH_SHORT).show()
                    }
                }
                "delete" -> {
                    if (position in 0 until viewModel.tasks.value.size) {
                        viewModel.deleteTask(position)
                        Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
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
            tasks = emptyList(),
            onItemClick = { position ->
                val task = viewModel.tasks.value[position]
                val intent = Intent(this, DetailActivity::class.java).apply {
                    putExtra("task_position", position)
                    putExtra("task_name", task.name)
                    putExtra("task_details", task.details)
                }
                detailActivityLauncher.launch(intent)
            },
            onItemLongClick = { position ->
                deleteTaskWithUndo(position)
            }
        )
        recyclerView.adapter = adapter

        // Подписка на изменения StateFlow
        lifecycleScope.launch {
            viewModel.tasks.collect { tasks ->
                adapter.updateData(tasks)
                saveTasks()
            }
        }

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

        // --- Поле для ввода текста ---
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
            val taskName = editTextTask.text.toString()
            if (taskName.isNotBlank()) {
                val newTask = Task(taskName, "")
                viewModel.addTask(newTask)
                editTextTask.text.clear()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите название задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Очистить всё ---
        buttonClearAll.setOnClickListener {
            viewModel.setTasks(emptyList())
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        // --- Удаление по индексу ---
        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in 0 until viewModel.tasks.value.size) {
                    val removed = viewModel.tasks.value[index]
                    viewModel.deleteTask(index)
                    Toast.makeText(this, "Удалена задача: ${removed.name}", Toast.LENGTH_SHORT).show()
                    editTextIndex.text.clear()
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${viewModel.tasks.value.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // Загрузка тестовых данных (если нужно)
        if (viewModel.tasks.value.isEmpty()) {
            // Можно оставить пустым или загрузить тестовые
            // viewModel.addTask(Task("Купить продукты", "Молоко, хлеб"))
        }
    }

    private fun deleteTaskWithUndo(position: Int) {
        val tasks = viewModel.tasks.value
        if (position !in tasks.indices) return

        deletedTask = tasks[position]
        deletedTaskPosition = position

        viewModel.deleteTask(position)

        Snackbar.make(recyclerView, "Задача удалена: ${deletedTask?.name?.take(30)}", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                deletedTask?.let {
                    viewModel.restoreTask(deletedTaskPosition, it)
                }
                deletedTask = null
                deletedTaskPosition = -1
            }
            .show()
    }

    // --- Счётчик и сохранение ---
    private var counter = 0

    private fun updateCounterDisplay(textView: TextView) {
        textView.text = "Счётчик: $counter"
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    private fun saveTasks() {
        val tasksString = viewModel.tasks.value.joinToString("|||") { task ->
            "${task.name}|${task.details}"
        }
        sharedPreferences.edit().putString("tasks", tasksString).apply()
    }

    private fun loadSavedData() {
        counter = sharedPreferences.getInt("counter", 0)
        val tasksString = sharedPreferences.getString("tasks", "")
        val loadedTasks = mutableListOf<Task>()
        if (!tasksString.isNullOrEmpty()) {
            val items = tasksString.split("|||")
            for (item in items) {
                val parts = item.split("|")
                if (parts.size == 2) {
                    loadedTasks.add(Task(parts[0], parts[1]))
                }
            }
        }
        viewModel.setTasks(loadedTasks)
    }

    override fun onResume() {
        super.onResume()
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

---

### 5. `activity_detail.xml`

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

---

## Ответы на контрольные вопросы

**1. Для чего нужен ViewModel? Как он помогает при повороте экрана?**

`ViewModel` – это компонент архитектуры Android, предназначенный для хранения и управления данными, связанными с UI. Основное преимущество – `ViewModel` переживает изменения конфигурации (повороты экрана, смену языка и т.д.). При повороте экрана Activity пересоздаётся, но `ViewModel` остаётся в памяти и данные не теряются. Это позволяет избежать сложного кода для сохранения и восстановления состояния (onSaveInstanceState).

**2. Чем StateFlow отличается от LiveData? В каких случаях предпочтительнее использовать StateFlow?**

| Характеристика | LiveData | StateFlow |
|----------------|----------|-----------|
| Платформа | Android-специфичный | Кроссплатформенный (Kotlin) |
| Подписка | Только на MainThread | Любой корутинный контекст |
| Обработка ошибок | Отсутствует | Есть (catch, retry) |
| Операторы преобразования | Ограниченные | Полный набор Flow-операторов |
| Jetpack Compose | Требуется адаптация | Нативная поддержка |

`StateFlow` предпочтительнее при использовании корутин, в многоплатформенных проектах, при необходимости сложных манипуляций с потоками данных.

**3. Что такое `lifecycleScope` и `repeatOnLifecycle`? Зачем они нужны при подписке на StateFlow?**

- **`lifecycleScope`** – корутина, привязанная к жизненному циклу LifecycleOwner (Activity/Fragment). Она автоматически отменяется при уничтожении владельца.
- **`repeatOnLifecycle`** – перезапускает корутину при достижении указанного состояния жизненного цикла (например, STARTED) и отменяет при уходе из него. Это предотвращает утечки памяти и ненужную фоновую работу.

В данном проекте используется упрощённый вариант – `lifecycleScope.launch { viewModel.tasks.collect {...} }`. Более правильный подход – `repeatOnLifecycle`, но в рамках лабораторной работы допустим и упрощённый.

**4. Как обновить данные в StateFlow?**

Данные в `StateFlow` обновляются через его `MutableStateFlow` версию. В ViewModel создаётся приватный `MutableStateFlow` и публичный неизменяемый `StateFlow`. Обновление происходит через изменение значения `_tasks.value`:

```kotlin
private val _tasks = MutableStateFlow<List<String>>(emptyList())
val tasks: StateFlow<List<String>> = _tasks.asStateFlow()

fun addTask(task: String) {
    val currentList = _tasks.value.toMutableList()
    currentList.add(task)
    _tasks.value = currentList  // автоматически уведомляет подписчиков
}
```

**5. Какие преимущества даёт вынос логики в ViewModel с точки зрения тестирования?**

- **Изоляция от Android-зависимостей** – ViewModel не содержит ссылок на Context, Activity, View. Это позволяет тестировать её без эмулятора или Robolectric.
- **Предсказуемость** – вся бизнес-логика собрана в одном месте, легко проверить реакции на события.
- **Тестирование корутин** – StateFlow легко тестировать с помощью `runTest` и `collect`.
- **Разделение ответственности** – Activity отвечает только за отображение UI, ViewModel – за данные и логику.

---

## Вывод

В ходе выполнения лабораторной работы №8 проведена архитектурная модернизация приложения ToDo-списка. Основные результаты:

1. **Создан `MainViewModel`**, который перенял на себя хранение списка задач и всю логику работы с данными (добавление, удаление, восстановление).

2. **Внедрён `StateFlow`** для реактивного обновления UI. Activity подписывается на изменения списка и автоматически обновляет адаптер при любых изменениях в данных.

3. **Решена проблема потери данных при повороте экрана** – ViewModel переживает изменение конфигурации, поэтому список задач сохраняется.

4. **Реализовано индивидуальное задание** – удаление задачи свайпом с помощью `ItemTouchHelper`. При свайпе появляется `Snackbar` с кнопкой «Отменить», позволяющей восстановить случайно удалённую задачу.

5. **Код стал более чистым и тестируемым** – Activity теперь отвечает только за отображение и события UI, вся бизнес-логика вынесена в ViewModel.

