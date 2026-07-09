# Отчёт по лабораторной работе №10 (с интеграцией Room и поиском)

<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  

<br>
<br>
<br>
<br>

Лабораторная работа №10  
**«Интеграция Room в проект. Сохранение списка задач в БД»**  

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

</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>

---

## Цель работы

Изучить основы работы с Room Database — официальной библиотекой для работы с SQLite в Android. Научиться создавать Entity, DAO, Database, интегрировать Room с ViewModel и корутинами, обеспечить сохранение списка задач между сессиями приложения. Реализовать функцию поиска задач по названию с использованием SQL-запроса LIKE.

## Индивидуальное задание

Реализован полнофункциональный TodoApp с использованием Room Database для постоянного хранения данных. Добавлена функция поиска задач по названию с использованием `@Query` и оператора `LIKE`. Приложение сохраняет все задачи, их статус выполнения и детали между перезапусками. Реализована реактивная загрузка данных через StateFlow.

## Скриншоты

*(Вставьте сюда свои скриншоты)*

![Главный экран со списком задач](main_screen.png)  
*Рисунок 1 – Главный экран приложения со списком задач*

<br>

![Поиск задач](search_screen.png)  
*Рисунок 2 – Результат поиска задач по ключевому слову*

<br>

![Детальный экран задачи](detail_screen.png)  
*Рисунок 3 – Детальный экран с редактированием задачи*

<br>

## Листинги кода

В ходе выполнения лабораторной работы были созданы/изменены следующие файлы:

### Entity (модель данных для БД)
- `database/TaskEntity.kt`
```kt
package com.example.todoapp.database

import androidx.room.Entity
import androidx.room.PrimaryKey
import java.io.Serializable

@Entity(tableName = "tasks")
data class TaskEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val title: String,
    val details: String = "",
    val isCompleted: Boolean = false,
    val createdTime: Long = System.currentTimeMillis()
) : Serializable
```

### DAO (Data Access Object)
- `database/TaskDao.kt`
```kt
package com.example.todoapp.database

import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
interface TaskDao {

    @Query("SELECT * FROM tasks ORDER BY createdTime DESC")
    fun getAllTasks(): Flow<List<TaskEntity>>

    // Новый метод для поиска задач по названию
    @Query("SELECT * FROM tasks WHERE title LIKE '%' || :query || '%' ORDER BY createdTime DESC")
    fun searchTasks(query: String): Flow<List<TaskEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertTask(task: TaskEntity)

    @Update
    suspend fun updateTask(task: TaskEntity)

    @Delete
    suspend fun deleteTask(task: TaskEntity)

    @Query("DELETE FROM tasks")
    suspend fun deleteAll()

    @Query("SELECT * FROM tasks WHERE id = :id")
    suspend fun getTaskById(id: Long): TaskEntity?
}
```

### Database
- `database/AppDatabase.kt`
```kt
package com.example.todoapp.database

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [TaskEntity::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun taskDao(): TaskDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "todo_database"
                )
                    .fallbackToDestructiveMigration() // Для разработки
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### ViewModel
- `MainViewModel.kt`
```kt
package com.example.todoapp

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.launch

class MainViewModel(
    private val database: AppDatabase
) : ViewModel() {

    private val taskDao = database.taskDao()

    private val _tasks = MutableStateFlow<List<TaskEntity>>(emptyList())
    val tasks: StateFlow<List<TaskEntity>> = _tasks.asStateFlow()

    private val _isSearching = MutableStateFlow(false)
    val isSearching: StateFlow<Boolean> = _isSearching.asStateFlow()

    private var currentSearchQuery = ""

    init {
        loadAllTasks()
    }

    fun loadAllTasks() {
        _isSearching.value = false
        currentSearchQuery = ""
        viewModelScope.launch {
            taskDao.getAllTasks().collect { taskList ->
                _tasks.value = taskList
            }
        }
    }

    fun searchTasks(query: String) {
        currentSearchQuery = query
        if (query.isBlank()) {
            loadAllTasks()
        } else {
            _isSearching.value = true
            viewModelScope.launch {
                taskDao.searchTasks(query).collect { taskList ->
                    _tasks.value = taskList
                }
            }
        }
    }

    fun addTask(taskName: String, taskDetails: String = "") {
        viewModelScope.launch {
            val task = TaskEntity(title = taskName, details = taskDetails)
            taskDao.insertTask(task)
            if (currentSearchQuery.isBlank()) {
                loadAllTasks()
            } else {
                searchTasks(currentSearchQuery)
            }
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            taskDao.deleteTask(task)
            if (currentSearchQuery.isBlank()) {
                loadAllTasks()
            } else {
                searchTasks(currentSearchQuery)
            }
        }
    }

    fun deleteTaskAt(index: Int) {
        viewModelScope.launch {
            val task = _tasks.value.getOrNull(index)
            task?.let {
                taskDao.deleteTask(it)
                if (currentSearchQuery.isBlank()) {
                    loadAllTasks()
                } else {
                    searchTasks(currentSearchQuery)
                }
            }
        }
    }

    fun updateTaskDetails(task: TaskEntity, newDetails: String) {
        viewModelScope.launch {
            val updatedTask = task.copy(details = newDetails)
            taskDao.updateTask(updatedTask)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            val updatedTask = task.copy(isCompleted = isCompleted)
            taskDao.updateTask(updatedTask)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            taskDao.deleteAll()
            if (currentSearchQuery.isBlank()) {
                loadAllTasks()
            } else {
                searchTasks(currentSearchQuery)
            }
        }
    }

    fun getTaskAt(index: Int): TaskEntity? = _tasks.value.getOrNull(index)

    fun getCurrentSearchQuery(): String = currentSearchQuery
}
```

### ViewModelFactory
- `MainViewModelFactory.kt`
```kt
package com.example.todoapp

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.todoapp.database.AppDatabase

class MainViewModelFactory(
    private val database: AppDatabase
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MainViewModel(database) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

### Адаптер
- `TaskAdapter.kt`
```kt
package com.example.todoapp

import android.graphics.Color
import android.graphics.Paint
import android.view.LayoutInflater
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.cardview.widget.CardView
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.database.TaskEntity

class TaskAdapter(
    private var tasks: List<TaskEntity>,
    private val onItemClick: (TaskEntity) -> Unit,
    private val onItemLongClick: (TaskEntity) -> Unit,
    private val onCheckChange: (TaskEntity, Boolean) -> Unit
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
        holder.textTask.text = task.title
        holder.checkTask.isChecked = task.isCompleted

        // Зачёркивание текста в зависимости от статуса
        if (task.isCompleted) {
            holder.textTask.paintFlags = holder.textTask.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
        } else {
            holder.textTask.paintFlags = holder.textTask.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
        }

        // Чередование фона
        val cardView = holder.itemView as CardView
        if (position % 2 == 0) {
            cardView.setCardBackgroundColor(Color.BLACK)
            holder.textTask.setTextColor(Color.WHITE)
        } else {
            cardView.setCardBackgroundColor(Color.LTGRAY)
            holder.textTask.setTextColor(Color.BLACK)
        }

        holder.itemView.setOnClickListener { onItemClick(task) }
        holder.itemView.setOnLongClickListener {
            onItemLongClick(task)
            true
        }

        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            onCheckChange(task, isChecked)
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateData(newTasks: List<TaskEntity>) {
        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

### Активности
- `MainActivity.kt`
```kt
package com.example.todoapp

import android.app.Activity
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.os.Bundle
import android.text.Editable
import android.text.TextWatcher
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val database by lazy { AppDatabase.getInstance(this) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(database)
    }

    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    // UI элементы для поиска
    private lateinit var editTextSearch: EditText
    private lateinit var buttonSearch: Button
    private lateinit var buttonClearSearch: Button
    private lateinit var textSearchResult: TextView

    private var deletedTask: TaskEntity? = null
    private var deletedTaskPosition: Int = -1

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
                    val task = viewModel.getTaskAt(position)
                    if (task != null) {
                        viewModel.updateTaskDetails(task, newDetails)
                        Toast.makeText(this, "Детали обновлены", Toast.LENGTH_SHORT).show()
                    }
                }
                "delete" -> {
                    val task = viewModel.getTaskAt(position)
                    if (task != null) {
                        viewModel.deleteTask(task)
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
        loadCounter()

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

        // Инициализация элементов поиска
        editTextSearch = findViewById(R.id.editTextSearch)
        buttonSearch = findViewById(R.id.buttonSearch)
        buttonClearSearch = findViewById(R.id.buttonClearSearch)
        textSearchResult = findViewById(R.id.textSearchResult)

        recyclerView = findViewById(R.id.recyclerViewTasks)
        recyclerView.layoutManager = LinearLayoutManager(this)

        adapter = TaskAdapter(
            tasks = emptyList(),
            onItemClick = { task ->
                val position = viewModel.tasks.value.indexOf(task)
                val intent = Intent(this, DetailActivity::class.java).apply {
                    putExtra("task_position", position)
                    putExtra("task_name", task.title)
                    putExtra("task_details", task.details)
                }
                detailActivityLauncher.launch(intent)
            },
            onItemLongClick = { task ->
                deleteTaskWithUndo(task)
            },
            onCheckChange = { task, isChecked ->
                viewModel.toggleTaskCompletion(task, isChecked)
            }
        )
        recyclerView.adapter = adapter

        // Подписка на изменения StateFlow
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks)
                    updateSearchResultText()
                }
            }
        }

        // Подписка на состояние поиска
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.isSearching.collect { isSearching ->
                    if (!isSearching) {
                        textSearchResult.visibility = android.view.View.GONE
                    } else {
                        textSearchResult.visibility = android.view.View.VISIBLE
                    }
                }
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
                val task = viewModel.getTaskAt(position)
                task?.let { deleteTaskWithUndo(it) }
            }
        }
        ItemTouchHelper(swipeCallback).attachToRecyclerView(recyclerView)

        // --- Обработка поиска ---

        // Вариант 1: Поиск по кнопке
        buttonSearch.setOnClickListener {
            val query = editTextSearch.text.toString()
            performSearch(query)
        }

        // Очистка поиска
        buttonClearSearch.setOnClickListener {
            editTextSearch.text.clear()
            viewModel.loadAllTasks()
            textSearchResult.visibility = android.view.View.GONE
            Toast.makeText(this, "Поиск очищен", Toast.LENGTH_SHORT).show()
        }

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
                viewModel.addTask(taskName)
                editTextTask.text.clear()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите название задачи", Toast.LENGTH_SHORT).show()
            }
        }

        // --- Очистить всё ---
        buttonClearAll.setOnClickListener {
            viewModel.deleteAllTasks()
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        // --- Удаление по индексу ---
        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in 0 until viewModel.tasks.value.size) {
                    val task = viewModel.getTaskAt(index)
                    if (task != null) {
                        viewModel.deleteTask(task)
                        Toast.makeText(this, "Удалена задача: ${task.title}", Toast.LENGTH_SHORT).show()
                        editTextIndex.text.clear()
                    }
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${viewModel.tasks.value.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun performSearch(query: String) {
        if (query.isBlank()) {
            Toast.makeText(this, "Введите текст для поиска", Toast.LENGTH_SHORT).show()
            return
        }

        viewModel.searchTasks(query)
        updateSearchResultText()

        val resultCount = viewModel.tasks.value.size
        //if (resultCount > 0) {
        //    Toast.makeText(this, "Найдено задач: $resultCount", Toast.LENGTH_SHORT).show()
        //} else {
        //    Toast.makeText(this, "Задачи не найдены", Toast.LENGTH_SHORT).show()
        //}
    }

    private fun updateSearchResultText() {
        if (viewModel.isSearching.value) {
            val query = viewModel.getCurrentSearchQuery()
            val count = viewModel.tasks.value.size
            textSearchResult.text = "Результаты поиска \"$query\": найдено $count задач"
            textSearchResult.visibility = android.view.View.VISIBLE
        } else {
            textSearchResult.visibility = android.view.View.GONE
        }
    }

    private fun deleteTaskWithUndo(task: TaskEntity) {
        val position = viewModel.tasks.value.indexOf(task)
        if (position == -1) return

        deletedTask = task
        deletedTaskPosition = position

        viewModel.deleteTask(task)

        Snackbar.make(recyclerView, "Задача удалена: ${task.title.take(30)}", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                deletedTask?.let {
                    viewModel.addTask(it.title, it.details)
                    // Восстанавливаем состояние выполнения и детали
                    if (it.isCompleted || it.details.isNotEmpty()) {
                        // Обновляем задачу после восстановления
                        val tasks = viewModel.tasks.value
                        val restoredTask = tasks.find { task -> task.title == it.title && task.details == it.details }
                        restoredTask?.let { restored ->
                            if (it.isCompleted != restored.isCompleted) {
                                viewModel.toggleTaskCompletion(restored, it.isCompleted)
                            }
                            if (it.details != restored.details) {
                                viewModel.updateTaskDetails(restored, it.details)
                            }
                        }
                    }
                }
                deletedTask = null
                deletedTaskPosition = -1
            }
            .show()
    }

    // --- Счётчик ---
    private var counter = 0

    private fun updateCounterDisplay(textView: TextView) {
        textView.text = "Счётчик: $counter"
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    private fun loadCounter() {
        counter = sharedPreferences.getInt("counter", 0)
    }

    override fun onResume() {
        super.onResume()
        findViewById<TextView>(R.id.textCounter)?.let { updateCounterDisplay(it) }
    }

    override fun onPause() {
        super.onPause()
        saveCounter()
    }
}
```
- `DetailActivity.kt`
```kt
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

### Layout файлы
- `res/layout/activity_main.xml`
```kt
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

    <!-- НОВЫЙ БЛОК: Поиск задач -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Поиск задач:"
        android:textSize="16sp"
        android:textStyle="bold"
        android:layout_marginBottom="8dp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginBottom="16dp">

        <EditText
            android:id="@+id/editTextSearch"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Введите название задачи..."
            android:inputType="text"
            android:layout_marginEnd="8dp"/>

        <Button
            android:id="@+id/buttonSearch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Найти"/>

        <Button
            android:id="@+id/buttonClearSearch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Очистить"
            android:layout_marginStart="8dp"/>
    </LinearLayout>

    <!-- Результат поиска -->
    <TextView
        android:id="@+id/textSearchResult"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="14sp"
        android:textColor="@android:color/holo_blue_dark"
        android:layout_marginBottom="8dp"
        android:visibility="gone"/>

    <!-- Блок 3: ToDo список (добавление новой задачи) -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Добавление задачи:"
        android:textSize="16sp"
        android:textStyle="bold"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"/>

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
- `res/layout/activity_detail.xml`
```kt
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
- `res/layout/item_task.xml`
```kt
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


## Ответы на контрольные вопросы

**1. Для чего нужна библиотека Room? Какие проблемы она решает по сравнению с прямым использованием SQLite?**

Room — это библиотека-обёртка над SQLite, предоставляющая удобный и безопасный способ работы с базами данных в Android. Она решает следующие проблемы:

- **Абстракция над SQLite** — минимум шаблонного кода, не нужно писать сложные `SQLiteOpenHelper` и ручные запросы
- **Валидация запросов на этапе компиляции** — если SQL-запрос некорректен, ошибка появится при компиляции, а не в рантайме
- **Интеграция с корутинами и Flow** — асинхронная работа "из коробки", автоматическое обновление UI при изменении данных
- **Снижение количества ошибок** — типизированные запросы, автоматическое преобразование между SQL и Kotlin-объектами

**2. Назовите три основных компонента Room и объясните их назначение.**

1. **Entity** — класс данных, представляющий таблицу в базе данных. Каждое поле класса — это столбец таблицы. Аннотация `@Entity` указывает Room, что этот класс нужно сохранять.

2. **DAO (Data Access Object)** — интерфейс, определяющий методы для работы с данными (вставка, удаление, запросы). Room автоматически генерирует реализацию. Аннотации `@Query`, `@Insert`, `@Update`, `@Delete` описывают операции.

3. **Database** — абстрактный класс, наследующий от `RoomDatabase`, служит точкой доступа к БД. Связывает Entity и DAO, управляет версионированием и миграциями.

**3. Почему методы DAO, изменяющие данные, объявляются как `suspend`?**

Методы DAO объявляются как `suspend`, потому что:
- Операции с базой данных (вставка, обновление, удаление) могут занимать время
- Выполнение таких операций в главном потоке заблокирует UI
- `suspend` функции позволяют выполнять их в фоновом потоке без блокировки
- Room автоматически переключает выполнение на `Dispatchers.IO` для suspend-функций
- При вызове из корутины (например, `viewModelScope.launch`) код остаётся простым и читаемым

**4. Что такое `Flow` и почему его удобно использовать с Room?**

`Flow` — это реактивный поток данных в Kotlin корутинах, который эмитирует последовательность значений. Его удобно использовать с Room потому, что:

- Room может возвращать `Flow<List<T>>` из запросов SELECT
- При каждом изменении в таблице (вставка, обновление, удаление) Room автоматически отправляет новый список в Flow
- UI автоматически обновляется при изменении данных без дополнительного кода
- `Flow` работает с корутинами и поддерживает отмену при уничтожении ViewModel
- Можно легко преобразовывать, фильтровать и комбинировать потоки данных

**5. Как Room обеспечивает проверку SQL-запросов на этапе компиляции?**

Room проверяет SQL-запросы на этапе компиляции с помощью аннотаций и процессора аннотаций (KSP/kapt):

- Каждый `@Query` аннотированный метод анализируется во время компиляции
- Room проверяет синтаксис SQL и наличие таблиц/колонок в Entity
- Если в запросе есть ошибка (неправильное имя таблицы, опечатка в колонке), компиляция не пройдёт
- Тип возвращаемого значения сверяется с результатом запроса
- При изменении Entity (добавление/удаление полей) все связанные запросы перепроверяются

**6. Зачем нужен паттерн Singleton для экземпляра базы данных?**

Singleton для экземпляра базы данных нужен потому, что:

- Создание экземпляра RoomDatabase — тяжёлая операция (открытие файла БД, создание таблиц)
- Room использует пул соединений, и несколько экземпляров могут привести к ошибкам и утечкам памяти
- Паттерн Singleton гарантирует, что во всём приложении будет только один экземпляр БД
- `@Volatile` обеспечивает видимость изменений между потоками
- `synchronized` предотвращает создание нескольких экземпляров в многопоточной среде

## Вывод

В ходе выполнения лабораторной работы было модернизировано приложение TodoApp с переходом от SharedPreferences к Room Database.

**Освоены следующие технологии и подходы:**

- **Room Database** — создание Entity, DAO, Database для постоянного хранения данных
- **KSP (Kotlin Symbol Processing)** — для генерации кода Room вместо kapt
- **Корутины** — для асинхронной работы с базой данных в фоновом потоке (`viewModelScope.launch`)
- **StateFlow** — для реактивного обновления UI при изменениях в БД
- **Поиск с LIKE** — реализована функция поиска задач по названию через SQL-запрос
- **MVVM** — чёткое разделение ответственности между Activity, ViewModel и Repository/DAO

**Результаты:**

- Задачи сохраняются после перезапуска приложения
- Статус выполнения (чекбокс) сохраняется в БД через поле `isCompleted`
- Поиск задач работает в реальном времени и показывает количество найденных результатов
- При добавлении/удалении задач во время поиска результаты автоматически обновляются
- Все операции с БД выполняются асинхронно, не блокируя UI-поток

**Сравнение с предыдущей версией:**

| Аспект | SharedPreferences | Room Database |
|--------|-------------------|---------------|
| Хранение | Текстовый файл | SQLite БД |
| Скорость | Медленнее при 100+ задач | Быстрее, индексы |
| Поиск | Ручная фильтрация в коде | SQL LIKE оптимизирован |
| Состояние чекбокса | Не сохранялось | Сохраняется |
| Масштабируемость | Низкая | Высокая |

Цель работы достигнута в полном объёме.
