# Отчёт по лабораторной работе №11

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

Лабораторная работа №11  
**«Рефакторинг: добавление слоя Repository между ViewModel и Room. Обработка ошибок»**  

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

Изучить архитектурный паттерн Repository, научиться выделять слой доступа к данным, отделяя его от бизнес-логики, выполнить рефакторинг существующего приложения для использования репозитория. Добавить обработку ошибок с использованием класса Result для улучшения пользовательского опыта и отказоустойчивости приложения.

## Индивидуальное задание

Реализована обработка ошибок в приложении TodoApp с использованием паттерна Result. Все методы репозитория обёрнуты в try-catch и возвращают Result.Success или Result.Error. В ViewModel добавлен StateFlow для передачи сообщений об ошибках в UI, где они отображаются в виде Snackbar. Это позволяет приложению корректно обрабатывать сбои (ошибки базы данных, проблемы с сетью) без вылетов.

## Скриншоты

*(Вставьте сюда свои скриншоты)*

![Главный экран приложения](main_screen.png)  
*Рисунок 1 – Главный экран TodoApp со списком задач*

<br>

![Успешное добавление задачи](success_toast.png)  
*Рисунок 3 – Toast-уведомление при успешном выполнении операции*

<br>

![Удаление задачи с отменой](undo_delete.png)  
*Рисунок 4 – Функция отмены удаления задачи через Snackbar*

## Листинги кода

В ходе выполнения лабораторной работы были созданы/изменены следующие файлы:

### Обработка ошибок (индивидуальное задание)
- `utils/Result.kt`
```kt
package com.example.todoapp.utils

sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Exception, val message: String = exception.message ?: "Unknown error") : Result<Nothing>()

    fun isSuccess(): Boolean = this is Success
    fun isError(): Boolean = this is Error

    fun getOrNull(): T? = if (this is Success) data else null
    fun getErrorOrNull(): Exception? = if (this is Error) exception else null
}
```
- `data/repository/TaskRepository.kt`
```kt
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskEntity
import com.example.todoapp.utils.Result
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(title: String, details: String): Result<Unit>
    suspend fun addTask(task: TaskEntity): Result<Unit>
    suspend fun deleteTask(task: TaskEntity): Result<Unit>
    suspend fun updateTask(task: TaskEntity): Result<Unit>
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean): Result<Unit>
    suspend fun deleteAllTasks(): Result<Unit>
    suspend fun searchTasks(query: String): Flow<List<TaskEntity>>
}
```
- `data/repository/TaskRepositoryImpl.kt`
```kt
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskDao
import com.example.todoapp.database.TaskEntity
import com.example.todoapp.utils.Result
import kotlinx.coroutines.flow.Flow
import java.io.IOException

class TaskRepositoryImpl(
    private val taskDao: TaskDao
) : TaskRepository {

    override fun getAllTasks(): Flow<List<TaskEntity>> = taskDao.getAllTasks()

    override suspend fun addTask(title: String, details: String): Result<Unit> {
        return try {
            val task = TaskEntity(title = title, details = details)
            taskDao.insertTask(task)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при добавлении задачи")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при добавлении задачи: ${e.message}")
        }
    }

    override suspend fun addTask(task: TaskEntity): Result<Unit> {
        return try {
            taskDao.insertTask(task)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при добавлении задачи")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при добавлении задачи: ${e.message}")
        }
    }

    override suspend fun deleteTask(task: TaskEntity): Result<Unit> {
        return try {
            taskDao.deleteTask(task)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при удалении задачи")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при удалении задачи: ${e.message}")
        }
    }

    override suspend fun updateTask(task: TaskEntity): Result<Unit> {
        return try {
            taskDao.updateTask(task)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при обновлении задачи")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при обновлении задачи: ${e.message}")
        }
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean): Result<Unit> {
        return try {
            val updatedTask = task.copy(isCompleted = isCompleted)
            taskDao.updateTask(updatedTask)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при изменении статуса")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при изменении статуса: ${e.message}")
        }
    }

    override suspend fun deleteAllTasks(): Result<Unit> {
        return try {
            taskDao.deleteAll()
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e, "Ошибка сети при удалении всех задач")
        } catch (e: Exception) {
            Result.Error(e, "Ошибка при удалении всех задач: ${e.message}")
        }
    }

    override suspend fun searchTasks(query: String): Flow<List<TaskEntity>> = taskDao.searchTasks(query)
}
```
- `MainViewModel.kt`
```kt
package com.example.todoapp

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.todoapp.data.repository.TaskRepository
import com.example.todoapp.database.TaskEntity
import com.example.todoapp.utils.Result
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class MainViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    private val _tasks = MutableStateFlow<List<TaskEntity>>(emptyList())
    val tasks: StateFlow<List<TaskEntity>> = _tasks.asStateFlow()

    private val _isSearching = MutableStateFlow(false)
    val isSearching: StateFlow<Boolean> = _isSearching.asStateFlow()

    // StateFlow для отображения ошибок
    private val _errorMessage = MutableStateFlow<String?>(null)
    val errorMessage: StateFlow<String?> = _errorMessage.asStateFlow()

    // StateFlow для отображения успешных сообщений (НЕ ИСПОЛЬЗУЕТСЯ при ошибке)
    private val _successMessage = MutableStateFlow<String?>(null)
    val successMessage: StateFlow<String?> = _successMessage.asStateFlow()

    private var currentSearchQuery = ""

    // ФЛАГ ДЛЯ ДЕМОНСТРАЦИИ ОШИБКИ
    private val simulateError = true  // ← true = демонстрация ошибки, false = нормальная работа

    init {
        loadAllTasks()
    }

    fun clearError() {
        _errorMessage.value = null
    }

    fun clearSuccess() {
        _successMessage.value = null
    }

    private fun handleResult(result: Result<*>, successMsg: String, errorMsg: String): Boolean {
        return if (result is Result.Error) {
            _errorMessage.value = errorMsg + ": " + result.message
            false
        } else {
            // Показываем successMessage ТОЛЬКО если НЕ в режиме симуляции ошибки
            if (!simulateError) {
                _successMessage.value = successMsg
            }
            true
        }
    }

    fun loadAllTasks() {
        _isSearching.value = false
        currentSearchQuery = ""
        viewModelScope.launch {
            // ДЕМОНСТРАЦИЯ ОШИБКИ
            if (simulateError) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось загрузить задачи (симуляция сбоя)"
                return@launch
            }

            repository.getAllTasks().collect { taskList ->
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
                if (simulateError) {
                    _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось выполнить поиск (симуляция сбоя)"
                    _isSearching.value = false
                    return@launch
                }

                repository.searchTasks(query).collect { taskList ->
                    _tasks.value = taskList
                }
            }
        }
    }

    fun addTask(taskName: String, taskDetails: String = "") {
        viewModelScope.launch {
            if (simulateError && taskName.isNotBlank()) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось добавить задачу \"$taskName\" (симуляция сбоя)"
                return@launch
            }

            val result = repository.addTask(taskName, taskDetails)
            if (handleResult(result, "✅ Задача добавлена", "❌ Ошибка при добавлении задачи")) {
                if (currentSearchQuery.isBlank()) {
                    loadAllTasks()
                } else {
                    searchTasks(currentSearchQuery)
                }
            }
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            if (simulateError) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось удалить задачу \"${task.title}\" (симуляция сбоя)"
                return@launch
            }

            val result = repository.deleteTask(task)
            if (handleResult(result, "🗑️ Задача удалена", "❌ Ошибка при удалении задачи")) {
                if (currentSearchQuery.isBlank()) {
                    loadAllTasks()
                } else {
                    searchTasks(currentSearchQuery)
                }
            }
        }
    }

    fun deleteTaskAt(index: Int) {
        viewModelScope.launch {
            val task = _tasks.value.getOrNull(index)
            task?.let {
                if (simulateError) {
                    _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось удалить задачу \"${task.title}\" (симуляция сбоя)"
                    return@launch
                }

                val result = repository.deleteTask(it)
                if (handleResult(result, "🗑️ Задача удалена", "❌ Ошибка при удалении задачи")) {
                    if (currentSearchQuery.isBlank()) {
                        loadAllTasks()
                    } else {
                        searchTasks(currentSearchQuery)
                    }
                }
            }
        }
    }

    fun updateTaskDetails(task: TaskEntity, newDetails: String) {
        viewModelScope.launch {
            if (simulateError) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось обновить детали задачи \"${task.title}\" (симуляция сбоя)"
                return@launch
            }

            val updatedTask = task.copy(details = newDetails)
            val result = repository.updateTask(updatedTask)
            handleResult(result, "✏️ Детали обновлены", "❌ Ошибка при обновлении деталей")
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            if (simulateError) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось изменить статус задачи \"${task.title}\" (симуляция сбоя)"
                return@launch
            }

            val result = repository.toggleTaskCompletion(task, isCompleted)
            handleResult(result, "✅ Статус изменён", "❌ Ошибка при изменении статуса")
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            if (simulateError) {
                _errorMessage.value = "❌ ДЕМОНСТРАЦИОННАЯ ОШИБКА: Не удалось удалить все задачи (симуляция сбоя)"
                return@launch
            }

            val result = repository.deleteAllTasks()
            if (handleResult(result, "🧹 Все задачи удалены", "❌ Ошибка при удалении всех задач")) {
                if (currentSearchQuery.isBlank()) {
                    loadAllTasks()
                } else {
                    searchTasks(currentSearchQuery)
                }
            }
        }
    }

    fun getTaskAt(index: Int): TaskEntity? = _tasks.value.getOrNull(index)

    fun getCurrentSearchQuery(): String = currentSearchQuery
}
```
- `MainActivity.kt`
```kt
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
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar
import com.example.todoapp.data.repository.TaskRepositoryImpl
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val database by lazy { AppDatabase.getInstance(this) }
    private val repository by lazy { TaskRepositoryImpl(database.taskDao()) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(repository)
    }

    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var adapter: TaskAdapter
    private lateinit var recyclerView: RecyclerView

    private lateinit var editTextSearch: EditText
    private lateinit var buttonSearch: Button
    private lateinit var textSearchResult: TextView
    private lateinit var textCounter: TextView
    private lateinit var editTextTask: EditText
    private lateinit var buttonAddTask: Button
    private lateinit var buttonClearAll: Button
    private lateinit var editTextIndex: EditText
    private lateinit var buttonDeleteTask: Button
    private lateinit var buttonIncrement: Button
    private lateinit var buttonReset: Button

    private var deletedTask: TaskEntity? = null
    private var currentSnackbar: Snackbar? = null

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
                        // Успех уже обрабатывается в ViewModel через Result
                    }
                }
                "delete" -> {
                    val task = viewModel.getTaskAt(position)
                    if (task != null) {
                        viewModel.deleteTask(task)
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

        initUI()
        setupAdapter()
        setupObservers()
        setupSwipeToDelete()
        setupSearch()
        setupButtons()
    }

    private fun initUI() {
        textCounter = findViewById(R.id.textCounter)
        buttonIncrement = findViewById(R.id.buttonIncrement)
        buttonReset = findViewById(R.id.buttonReset)

        editTextTask = findViewById(R.id.editTextTask)
        buttonAddTask = findViewById(R.id.buttonAddTask)
        buttonClearAll = findViewById(R.id.buttonClearAll)

        editTextIndex = findViewById(R.id.editTextIndex)
        buttonDeleteTask = findViewById(R.id.buttonDeleteTask)

        editTextSearch = findViewById(R.id.editTextSearch)
        buttonSearch = findViewById(R.id.buttonSearch)
        textSearchResult = findViewById(R.id.textSearchResult)

        recyclerView = findViewById(R.id.recyclerViewTasks)
        recyclerView.layoutManager = LinearLayoutManager(this)
    }

    private fun setupAdapter() {
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
    }

    private fun setupObservers() {
        // Наблюдение за списком задач
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks)
                    updateSearchResultText()
                }
            }
        }

        // Наблюдение за состоянием поиска
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.isSearching.collect { isSearching ->
                    textSearchResult.visibility = if (isSearching) android.view.View.VISIBLE else android.view.View.GONE
                }
            }
        }

        // Наблюдение за ошибками
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.errorMessage.collect { errorMessage ->
                    errorMessage?.let {
                        showErrorSnackbar(it)
                        viewModel.clearError()
                    }
                }
            }
        }

        // НОВЫЙ: Наблюдение за успешными сообщениями
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.successMessage.collect { successMessage ->
                    successMessage?.let {
                        Toast.makeText(this@MainActivity, it, Toast.LENGTH_SHORT).show()
                        viewModel.clearSuccess()
                    }
                }
            }
        }
    }

    private fun showErrorSnackbar(message: String) {
        currentSnackbar?.dismiss()
        currentSnackbar = Snackbar.make(recyclerView, message, Snackbar.LENGTH_LONG)
        currentSnackbar?.setAction("ОК") {
            currentSnackbar?.dismiss()
        }
        currentSnackbar?.show()
    }

    private fun setupSwipeToDelete() {
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
    }

    private fun setupSearch() {
        buttonSearch.setOnClickListener {
            val query = editTextSearch.text.toString()
            if (query.isBlank()) {
                Toast.makeText(this, "Введите текст для поиска", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            viewModel.searchTasks(query)
            updateSearchResultText()
        }
    }

    private fun setupButtons() {
        updateCounterDisplay()
        buttonIncrement.setOnClickListener {
            counter++
            updateCounterDisplay()
            saveCounter()
        }
        buttonReset.setOnClickListener {
            counter = 0
            updateCounterDisplay()
            saveCounter()
        }

        buttonAddTask.setOnClickListener {
            val taskName = editTextTask.text.toString()
            if (taskName.isNotBlank()) {
                viewModel.addTask(taskName)  // Toast покажется из ViewModel
                editTextTask.text.clear()
                // Удалите Toast отсюда
            } else {
                Toast.makeText(this, "Введите название задачи", Toast.LENGTH_SHORT).show()
            }
        }

        buttonClearAll.setOnClickListener {
            viewModel.deleteAllTasks()  // Toast покажется из ViewModel
            // Удалите Toast отсюда
        }

        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index in 0 until viewModel.tasks.value.size) {
                    val task = viewModel.getTaskAt(index)
                    if (task != null) {
                        viewModel.deleteTask(task)  // Toast покажется из ViewModel
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
        val position = viewModel.tasks.value.indexOfFirst { it.id == task.id }
        if (position == -1) return

        deletedTask = task
        viewModel.deleteTask(task)

        Snackbar.make(recyclerView, "Задача удалена: ${task.title.take(30)}", Snackbar.LENGTH_LONG)
            .setAction("Отменить") {
                deletedTask?.let { restoredTask ->
                    viewModel.addTask(restoredTask.title, restoredTask.details)
                }
                deletedTask = null
            }
            .addCallback(object : Snackbar.Callback() {
                override fun onDismissed(transientBottomBar: Snackbar?, event: Int) {
                    if (event != DISMISS_EVENT_ACTION) {
                        deletedTask = null
                    }
                }
            })
            .show()
    }


    private var counter = 0
    private fun updateCounterDisplay() {
        textCounter.text = "Счётчик: $counter"
    }
    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }
    private fun loadCounter() {
        counter = sharedPreferences.getInt("counter", 0)
    }

    override fun onResume() {
        super.onResume()
        updateCounterDisplay()
    }

    override fun onPause() {
        super.onPause()
        saveCounter()
    }
}
```

## Сравнение архитектуры

### До рефакторинга:
```
MainActivity → MainViewModel → TaskDao (Room)
```

### После рефакторинга:
```
MainActivity → MainViewModel → TaskRepository (интерфейс) → TaskRepositoryImpl → TaskDao (Room)
                    ↓
              Result (Success/Error)
                    ↓
              errorMessage StateFlow
                    ↓
              Snackbar в Activity
```

## Ответы на контрольные вопросы

**1. Какую роль выполняет слой Repository в архитектуре приложения?**

Репозиторий инкапсулирует логику доступа к данным, предоставляя единый API для работы с данными из различных источников (база данных, сеть, кэш). Он служит посредником между источниками данных и бизнес-логикой приложения, скрывая детали реализации операций с данными.

**2. Какие преимущества даёт использование Repository по сравнению с прямым обращением к DAO из ViewModel?**

- **Разделение ответственности** – ViewModel не знает, откуда берутся данные
- **Упрощение тестирования** – можно легко подставить mock-репозиторий
- **Гибкость** – при изменении источника данных меняется только реализация репозитория
- **Единая точка доступа** – упрощает добавление кэширования, логирования и обработки ошибок

**3. Как изменится ViewModel, если мы захотим добавить ещё один источник данных (например, сетевое API)?**

ViewModel не изменится вообще! Нужно будет только модифицировать реализацию репозитория, добавив в него логику выбора источника данных (сначала из кэша, потом из сети, сохранить в БД). ViewModel продолжит вызывать те же методы.

**4. Почему методы репозитория объявлены как `suspend`?**

Методы объявлены как `suspend`, потому что они выполняют долгие операции (работа с БД, сетью) и должны вызываться из корутин, не блокируя главный поток UI.

**5. Что такое инверсия зависимостей и как она применяется в данном рефакторинге?**

Инверсия зависимостей – принцип, согласно которому модули верхнего уровня не должны зависеть от модулей нижнего уровня; оба должны зависеть от абстракций. В рефакторинге `MainViewModel` зависит от абстракции `TaskRepository`, а не от конкретной реализации, что позволяет легко заменять реализацию.

**6. Как работает обработка ошибок через Result в вашей реализации?**

- Все методы репозитория обёрнуты в `try-catch`
- При успехе возвращается `Result.Success(data)`, при ошибке – `Result.Error(exception, message)`
- ViewModel проверяет результат: если ошибка – сохраняет сообщение в `_errorMessage` StateFlow
- Activity подписывается на `errorMessage` и показывает Snackbar
- При успехе ViewModel может показать Toast

## Вывод

В ходе выполнения лабораторной работы был проведён рефакторинг приложения TodoApp с добавлением архитектурного слоя Repository и обработки ошибок.

**Освоены следующие технологии и подходы:**

- **Паттерн Repository** – инкапсуляция логики доступа к данным
- **Инверсия зависимостей** – зависимость от абстракций, а не от конкретных реализаций
- **Обработка ошибок через Result** – типобезопасный способ обработки сбоев
- **StateFlow для ошибок** – реактивная передача сообщений об ошибках в UI
- **CoroutineScope** – асинхронное выполнение операций

**Результаты рефакторинга:**

| Аспект | До рефакторинга | После рефакторинга |
|--------|-----------------|-------------------|
| Связность ViewModel и БД | Высокая | Низкая (через абстракцию) |
| Обработка ошибок | Отсутствует (вылеты) | Полная (Result + Snackbar) |
| Тестируемость | Сложная | Простая (mock-репозиторий) |
| Замена источника данных | Требует изменения ViewModel | Требует только новой реализации |
| Пользовательский опыт | Вылеты при ошибках | Понятные сообщения |
- Приложение не вылетает даже при критических ошибках

Цель работы достигнута в полном объёме. Приложение теперь соответствует рекомендациям Google по архитектуре Android-приложений и обладает улучшенной отказоустойчивостью.
