# Отчёт по лабораторной работе №13

## Создание простого API клиента. Запрос списка постов с jsonplaceholder.typicode.com

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

Лабораторная работа №13  
**«Создание простого API клиента. Запрос списка постов с jsonplaceholder.typicode.com»**  
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

Научиться выполнять сетевые запросы в Android-приложении с использованием библиотеки Retrofit и корутин, обрабатывать ответы сервера, парсить JSON-данные и отображать их в RecyclerView. Реализовать переход на детальный экран при клике на элемент списка, обновление данных через SwipeRefreshLayout и отдельный экран для отображения отсутствия интернет-соединения.

## Индивидуальное задание

Реализовано приложение для получения и отображения списка постов с тестового REST API JSONPlaceholder. Добавлена возможность обновления данных при помощи SwipeRefreshLayout (потянуть вниз для обновления). Реализован детальный экран поста: при клике на элемент списка открывается новый экран с полной информацией о посте (id, title, body) с использованием Intent для передачи данных. Добавлен отдельный экран "Нет интернета", который отображается при возникновении IOException (отсутствие сетевого соединения), с кнопкой повторной попытки подключения.

## Скриншоты

![Главный экран со списком постов](main_screen.png)  
*Рисунок 1 – Главный экран приложения со списком постов*

<br>

![Детальный экран поста](detail_screen.png)  
*Рисунок 2 – Детальный экран с полной информацией о выбранном посте*

<br>

![Состояние загрузки](loading_state.png)  
*Рисунок 3 – Отображение ProgressBar при загрузке данных*

<br>

![Состояние ошибки](error_state.png)  
*Рисунок 4 – Отображение ошибки при проблемах с сервером*

<br>

![Экран "Нет интернета"](no_internet_screen.png)  
*Рисунок 5 – Экран, отображаемый при отсутствии интернет-соединения*

## Листинги кода

В ходе выполнения лабораторной работы были созданы следующие файлы:

### Модели данных
- `models/Post.kt` – data class, представляющий структуру поста (id, title, body)

```kotlin
package com.example.postsapp.models

data class Post(
    val id: Int,
    val title: String,
    val body: String
)
```

### Сетевые запросы (API)
- `api/ApiService.kt` – интерфейс с описанием GET-запроса для получения списка постов

```kotlin
package com.example.postsapp.api

import com.example.postsapp.models.Post
import retrofit2.http.GET

interface ApiService {
    @GET("posts")
    suspend fun getPosts(): List<Post>
}
```

- `api/RetrofitClient.kt` – объект для настройки и создания экземпляра Retrofit

```kotlin
package com.example.postsapp.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://jsonplaceholder.typicode.com/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```

### Репозиторий
- `repositories/PostsRepository.kt` – класс, отвечающий за получение данных из API

```kotlin
package com.example.postsapp.repositories

import com.example.postsapp.api.RetrofitClient
import com.example.postsapp.models.Post
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class PostsRepository {
    private val apiService = RetrofitClient.apiService

    suspend fun getPosts(): List<Post> = withContext(Dispatchers.IO) {
        apiService.getPosts()
    }
}
```

### ViewModel
- `viewmodels/PostsViewModel.kt` – ViewModel с состояниями UI (Loading, Success, NoInternet, Error) и логикой загрузки данных

```kotlin
package com.example.postsapp.viewmodels

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.postsapp.models.Post
import com.example.postsapp.repositories.PostsRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import java.io.IOException

sealed class PostsUiState {
    object Loading : PostsUiState()
    data class Success(val posts: List<Post>) : PostsUiState()
    object NoInternet : PostsUiState()
    data class Error(val message: String) : PostsUiState()
}

class PostsViewModel : ViewModel() {
    private val repository = PostsRepository()

    private val _uiState = MutableStateFlow<PostsUiState>(PostsUiState.Loading)
    val uiState: StateFlow<PostsUiState> = _uiState.asStateFlow()

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            _uiState.value = PostsUiState.Loading
            try {
                val posts = repository.getPosts()
                _uiState.value = PostsUiState.Success(posts)
            } catch (e: Exception) {
                if (e is IOException) {
                    _uiState.value = PostsUiState.NoInternet
                } else {
                    _uiState.value = PostsUiState.Error(e.message ?: "Unknown error")
                }
            }
        }
    }
}
```

### Адаптер
- `adapters/PostsAdapter.kt` – адаптер для RecyclerView с обработкой кликов по элементам

```kotlin
package com.example.postsapp.adapters

import android.content.Intent
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.postsapp.PostDetailActivity
import com.example.postsapp.R
import com.example.postsapp.models.Post

class PostsAdapter : RecyclerView.Adapter<PostsAdapter.PostViewHolder>() {

    private var posts = emptyList<Post>()

    fun submitList(newPosts: List<Post>) {
        posts = newPosts
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_post, parent, false)
        return PostViewHolder(view)
    }

    override fun onBindViewHolder(holder: PostViewHolder, position: Int) {
        holder.bind(posts[position])
    }

    override fun getItemCount() = posts.size

    inner class PostViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        private val textPostId: TextView = itemView.findViewById(R.id.textPostId)
        private val textPostTitle: TextView = itemView.findViewById(R.id.textPostTitle)
        private val textPostBody: TextView = itemView.findViewById(R.id.textPostBody)

        fun bind(post: Post) {
            textPostId.text = "ID: ${post.id}"
            textPostTitle.text = post.title
            textPostBody.text = post.body

            itemView.setOnClickListener {
                val context = itemView.context
                val intent = Intent(context, PostDetailActivity::class.java).apply {
                    putExtra("POST_ID", post.id)
                    putExtra("POST_TITLE", post.title)
                    putExtra("POST_BODY", post.body)
                }
                context.startActivity(intent)
            }
        }
    }
}
```

### UI компоненты
- `MainActivity.kt` – главный экран с RecyclerView, SwipeRefreshLayout, экраном "Нет интернета" и наблюдателем за состоянием

```kotlin
package com.example.postsapp

import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.ProgressBar
import android.widget.TextView
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import androidx.swiperefreshlayout.widget.SwipeRefreshLayout
import com.example.postsapp.adapters.PostsAdapter
import com.example.postsapp.viewmodels.PostsUiState
import com.example.postsapp.viewmodels.PostsViewModel
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: PostsViewModel by viewModels()
    private lateinit var adapter: PostsAdapter
    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textError: TextView
    private lateinit var swipeRefreshLayout: SwipeRefreshLayout
    private lateinit var noInternetView: View
    private lateinit var retryButton: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerViewPosts)
        progressBar = findViewById(R.id.progressBar)
        textError = findViewById(R.id.textError)
        swipeRefreshLayout = findViewById(R.id.swipeRefreshLayout)
        noInternetView = findViewById(R.id.noInternetView)
        retryButton = noInternetView.findViewById(R.id.buttonRetry)

        setupRecyclerView()
        observeUiState()
        setupSwipeRefresh()

        retryButton.setOnClickListener {
            viewModel.loadPosts()
        }
    }

    private fun setupRecyclerView() {
        adapter = PostsAdapter()
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter
    }

    private fun setupSwipeRefresh() {
        swipeRefreshLayout.setOnRefreshListener {
            viewModel.loadPosts()
        }
    }

    private fun observeUiState() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is PostsUiState.Loading -> showLoading()
                        is PostsUiState.Success -> showPosts(state.posts)
                        is PostsUiState.Error -> showError(state.message)
                        is PostsUiState.NoInternet -> showNoInternet()
                    }
                    swipeRefreshLayout.isRefreshing = false
                }
            }
        }
    }

    private fun showLoading() {
        if (adapter.itemCount == 0) {
            recyclerView.visibility = View.GONE
            progressBar.visibility = View.VISIBLE
        }
        textError.visibility = View.GONE
        noInternetView.visibility = View.GONE
    }

    private fun showPosts(posts: List<com.example.postsapp.models.Post>) {
        recyclerView.visibility = View.VISIBLE
        progressBar.visibility = View.GONE
        textError.visibility = View.GONE
        noInternetView.visibility = View.GONE
        adapter.submitList(posts)
    }

    private fun showError(message: String) {
        if (adapter.itemCount == 0) {
            recyclerView.visibility = View.GONE
            progressBar.visibility = View.GONE
            textError.visibility = View.VISIBLE
            textError.text = "Ошибка: $message"
            noInternetView.visibility = View.GONE
        }
    }

    private fun showNoInternet() {
        recyclerView.visibility = View.GONE
        progressBar.visibility = View.GONE
        textError.visibility = View.GONE
        noInternetView.visibility = View.VISIBLE
    }
}
```

- `PostDetailActivity.kt` – детальный экран для отображения полной информации о посте

```kotlin
package com.example.postsapp

import android.os.Bundle
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class PostDetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_post_detail)

        supportActionBar?.setDisplayHomeAsUpEnabled(true)
        title = "Детали поста"

        val textDetailId = findViewById<TextView>(R.id.textDetailId)
        val textDetailTitle = findViewById<TextView>(R.id.textDetailTitle)
        val textDetailBody = findViewById<TextView>(R.id.textDetailBody)

        val postId = intent.getIntExtra("POST_ID", 0)
        val postTitle = intent.getStringExtra("POST_TITLE") ?: ""
        val postBody = intent.getStringExtra("POST_BODY") ?: ""

        textDetailId.text = postId.toString()
        textDetailTitle.text = postTitle
        textDetailBody.text = postBody
    }

    override fun onSupportNavigateUp(): Boolean {
        finish()
        return true
    }
}
```

### Layout файлы
- `res/layout/activity_main.xml` – разметка главного экрана

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
        android:id="@+id/swipeRefreshLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recyclerViewPosts"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:visibility="gone" />

            <ProgressBar
                android:id="@+id/progressBar"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:visibility="gone" />

            <TextView
                android:id="@+id/textError"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="Ошибка загрузки"
                android:visibility="gone" />

            <include
                android:id="@+id/noInternetView"
                layout="@layout/no_internet_screen"
                android:visibility="gone" />

        </FrameLayout>

    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>

</LinearLayout>
```

- `res/layout/activity_post_detail.xml` – разметка детального экрана

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:cardCornerRadius="12dp"
        app:cardElevation="8dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="20dp">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="ID ПОСТА"
                android:textSize="12sp"
                android:textColor="#666666" />

            <TextView
                android:id="@+id/textDetailId"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="1"
                android:textSize="24sp"
                android:textStyle="bold"
                android:layout_marginBottom="24dp" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="#DDDDDD"
                android:layout_marginBottom="24dp" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="ЗАГОЛОВОК"
                android:textSize="12sp"
                android:textColor="#666666" />

            <TextView
                android:id="@+id/textDetailTitle"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Title"
                android:textSize="22sp"
                android:textStyle="bold"
                android:layout_marginBottom="24dp" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="#DDDDDD"
                android:layout_marginBottom="24dp" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="ТЕКСТ ПОСТА"
                android:textSize="12sp"
                android:textColor="#666666" />

            <TextView
                android:id="@+id/textDetailBody"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Body"
                android:textSize="16sp"
                android:layout_marginTop="8dp" />

        </LinearLayout>

    </androidx.cardview.widget.CardView>

</ScrollView>
```

- `res/layout/item_post.xml` – разметка отдельного элемента списка

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
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:id="@+id/textPostId"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ID: "
            android:textSize="14sp"
            android:textStyle="bold" />

        <TextView
            android:id="@+id/textPostTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:text="Title"
            android:textSize="18sp"
            android:textStyle="bold" />

        <TextView
            android:id="@+id/textPostBody"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="Body"
            android:textSize="14sp" />

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

- `res/layout/no_internet_screen.xml` – экран, отображаемый при отсутствии интернет-соединения

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="32dp">

    <ImageView
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:src="@android:drawable/stat_notify_error"
        android:alpha="0.7" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Нет подключения к интернету"
        android:textSize="20sp"
        android:textStyle="bold"
        android:layout_marginTop="24dp"
        android:textAlignment="center" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Проверьте соединение и попробуйте снова"
        android:textSize="14sp"
        android:layout_marginTop="8dp"
        android:textAlignment="center"
        android:textColor="#666666" />

    <Button
        android:id="@+id/buttonRetry"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Повторить"
        android:layout_marginTop="32dp" />

</LinearLayout>
```

### Конфигурационные файлы
- `app/build.gradle.kts` – зависимости проекта (Retrofit, корутины, SwipeRefreshLayout, RecyclerView, CardView)

```kotlin
plugins {
    alias(libs.plugins.android.application)
}

android {
    namespace = "com.example.postsapp"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.example.postsapp"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    
    buildFeatures {
        viewBinding = true
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)
    
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    implementation("androidx.cardview:cardview:1.0.0")
    implementation("androidx.swiperefreshlayout:swiperefreshlayout:1.1.0")
    
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```

- `AndroidManifest.xml` – разрешение на доступ к интернету, регистрация активностей

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.PostsApp">
        <activity
            android:name=".PostDetailActivity"
            android:exported="false" />
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## Ответы на контрольные вопросы

**1. Для чего используется библиотека Retrofit? Какие аннотации вы знаете?**

Retrofit – это типобезопасный HTTP-клиент для Android и Java, разработанный компанией Square. Он позволяет декларативно описывать API с помощью аннотаций и автоматически преобразовывать JSON-ответы в Kotlin-объекты.

Основные аннотации Retrofit:
- `@GET` – для GET-запросов
- `@POST` – для POST-запросов
- `@PUT` / `@PATCH` – для обновления данных
- `@DELETE` – для удаления данных
- `@Path` – для передачи параметров в URL
- `@Query` – для передачи query-параметров
- `@Body` – для передачи тела запроса

**2. Почему сетевые запросы нельзя выполнять в главном потоке?**

Сетевые запросы нельзя выполнять в главном потоке (UI-потоке), потому что:
- Главный поток отвечает за отрисовку интерфейса и обработку действий пользователя
- Сетевые запросы могут занимать неопределённое время (от долей секунды до нескольких минут)
- Если главный поток блокируется более чем на 5 секунд, система выбрасывает исключение `Application Not Responding` (ANR)
- Это приводит к зависанию приложения и негативному пользовательскому опыту

Поэтому сетевые запросы всегда выполняются в фоновых потоках с использованием корутин, RxJava или AsyncTask.

**3. Что такое `suspend` функция и как она работает с корутинами?**

`suspend` – это ключевое слово в Kotlin, которое обозначает функцию, которая может приостанавливать выполнение без блокировки потока. Особенности:
- Такая функция может быть вызвана только из другой suspend-функции или из корутины
- При приостановке функция не блокирует поток, а освобождает его для других задач
- После завершения асинхронной операции функция возобновляется с того же места

В контексте Retrofit: объявление метода как `suspend` позволяет Retrofit автоматически переключиться на фоновый поток и вернуть результат в корутину.

**4. Для чего нужен `Dispatchers.IO`?**

`Dispatchers.IO` – это планировщик (диспетчер) корутин, оптимизированный для операций ввода-вывода. Он предназначен для:
- Сетевых запросов (Retrofit, OkHttp)
- Работы с базой данных (Room, SQLite)
- Чтения/записи файлов
- Любых операций, которые могут блокировать поток

Особенности `Dispatchers.IO`:
- Имеет пул потоков, оптимизированный для большого количества блокирующих операций
- Может создавать дополнительные потоки при необходимости
- В отличие от `Dispatchers.Main`, не связан с UI-потоком

**5. Как обрабатывать ошибки при сетевых запросах?**

Ошибки при сетевых запросах обрабатываются с помощью конструкции `try-catch`. В данном приложении реализована следующая логика:

```kotlin
try {
    val posts = repository.getPosts()
    _uiState.value = PostsUiState.Success(posts)
} catch (e: Exception) {
    if (e is IOException) {
        _uiState.value = PostsUiState.NoInternet
    } else {
        _uiState.value = PostsUiState.Error(e.message ?: "Unknown error")
    }
}
```

Типы возможных ошибок:
- `IOException` – отсутствие интернета, проблемы с сетью (отображается специальный экран)
- `HttpException` – ошибки HTTP (404, 500 и т.д.)
- `JsonParseException` – ошибка парсинга JSON

В приложении ошибка отображается пользователю в виде текстового сообщения вместо списка постов. При отсутствии интернета показывается отдельный экран с кнопкой повторной попытки.

**6. Что такое JSONPlaceholder и для чего он используется?**

JSONPlaceholder – это бесплатный тестовый REST API, предоставляющий фейковые данные для прототипирования и тестирования. Особенности:
- Не требует регистрации и API-ключей
- Поддерживает все HTTP-методы (GET, POST, PUT, PATCH, DELETE)
- Возвращает данные в формате JSON

Основные ресурсы JSONPlaceholder:
- `/posts` – 100 постов (id, title, body)
- `/comments` – 500 комментариев
- `/albums` – 100 альбомов
- `/photos` – 5000 фотографий
- `/todos` – 200 задач
- `/users` – 10 пользователей

В данной работе использован ресурс `/posts` для получения списка постов.

## Вывод

В ходе выполнения лабораторной работы было создано Android-приложение для получения и отображения списка постов с тестового API JSONPlaceholder.

Освоены следующие технологии и подходы:
- **Retrofit** – для выполнения сетевых запросов и парсинга JSON
- **Корутины** – для асинхронной загрузки данных в фоновом потоке
- **MVVM** – архитектурный паттерн с ViewModel и StateFlow
- **RecyclerView** – для отображения списка с кастомным адаптером
- **SwipeRefreshLayout** – для обновления данных жестом "потянуть вниз"
- **Intent** – для передачи данных между Activity
- **StateFlow** – для управления состояниями UI (Loading, Success, Error, NoInternet)

Реализованы следующие функции:
- Детальный экран поста с полной информацией
- Обновление списка при помощи SwipeRefreshLayout
- Отдельный экран "Нет интернета" с кнопкой повторной попытки
- Обработка ошибок с различением IOException и других исключений

Приложение корректно работает при повороте экрана, обрабатывает отсутствие интернета и показывает соответствующие состояния загрузки. Определение отсутствия интернета происходит только на основе IOException при попытке выполнения запроса, без использования системных сервисов определения состояния сети.
