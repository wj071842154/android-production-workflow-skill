---
name: Android Production Workflow
description: Complete production-ready Android development workflow with Jetpack Compose, MVVM, Clean Architecture, Material Design 3, and Hilt DI
version: 1.0.0
author: wj071842154
tags:
  - android
  - jetpack-compose
  - mvvm
  - material-design
  - clean-architecture
  - kotlin
  - hilt
---

# Android Production Workflow - Jetpack Compose + MVVM + Material Design 3

**Version:** 1.0.0  
**Author:** wj071842154  
**Last Updated:** 2026-02-13

---

## 📋 Overview

This skill provides a complete, production-ready Android development workflow using:
- **Jetpack Compose** for declarative UI
- **MVVM + Clean Architecture** for scalable structure
- **Material Design 3** for modern UI/UX
- **Hilt** for dependency injection
- **Kotlin Coroutines + Flow** for async operations
- **Retrofit** for networking
- **Room** for local database

---

## 🏗️ Architecture Principles

### Clean Architecture Layers

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│   (UI, ViewModel, Compose Screens)      │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│          Domain Layer                   │
│   (Use Cases, Domain Models, Repo       │
│    Interfaces - Pure Kotlin)            │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│           Data Layer                    │
│   (Repository Impl, API, Database,      │
│    Data Models, Mappers)                │
└─────────────────────────────────────────┘
```

### MVVM Pattern

**ViewModel** holds UI state and business logic  
**View (Composable)** observes state and renders UI  
**Model** (Domain/Data) provides data through repositories

```
User Action → Composable → ViewModel → Use Case → Repository → API/Database
                 ↑                                                   │
                 └───────────── StateFlow ◄──────────────────────────┘
```

---

## 📁 Project Structure

```
app/
├── src/main/java/com/yourapp/
│   ├── MyApplication.kt                    # Hilt Application
│   │
│   ├── di/                                 # Dependency Injection
│   │   ├── AppModule.kt
│   │   ├── NetworkModule.kt
│   │   └── DatabaseModule.kt
│   │
│   ├── domain/                             # Domain Layer (Pure Kotlin)
│   │   ├── model/                          # Domain Models
│   │   ├── repository/                     # Repository Interfaces
│   │   └── usecase/                        # Business Logic Use Cases
│   │
│   ├── data/                               # Data Layer
│   │   ├── repository/                     # Repository Implementations
│   │   ├── remote/                         # Remote Data Source
│   │   │   ├── api/                        # Retrofit API interfaces
│   │   │   └── dto/                        # Data Transfer Objects
│   │   ├── local/                          # Local Data Source
│   │   │   ├── database/                   # Room Database
│   │   │   ├── dao/                        # Data Access Objects
│   │   │   └── entity/                     # Database Entities
│   │   └── mapper/                         # Data Mappers (DTO ↔ Domain)
│   │
│   └── presentation/                       # Presentation Layer
│       ├── theme/                          # Material 3 Theme
│       │   ├── Color.kt
│       │   ├── Theme.kt
│       │   └── Type.kt
│       ├── navigation/
│       │   └── NavGraph.kt
│       ├── components/                     # Reusable Composables
│       └── screens/                        # Feature Screens
│           └── [feature]/
│               ├── [Feature]Screen.kt
│               ├── [Feature]ViewModel.kt
│               └── [Feature]UiState.kt
```

---

## 🎯 Core Patterns

### 1. StateFlow + UI State Pattern

**Always use immutable data classes for UI state:**

```kotlin
data class HomeUiState(
    val isLoading: Boolean = false,
    val data: List<Item> = emptyList(),
    val error: String? = null
)

@HiltViewModel
class HomeViewModel @Inject constructor(
    private val useCase: GetItemsUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(HomeUiState())
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
    
    init {
        loadData()
    }
    
    fun loadData() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            
            useCase().collect { result ->
                result.fold(
                    onSuccess = { data ->
                        _uiState.update {
                            it.copy(isLoading = false, data = data, error = null)
                        }
                    },
                    onFailure = { exception ->
                        _uiState.update {
                            it.copy(isLoading = false, error = exception.message)
                        }
                    }
                )
            }
        }
    }
}
```

### 2. Repository Pattern with Flow

```kotlin
// Domain Layer - Interface
interface UserRepository {
    fun getUsers(): Flow<Result<List<User>>>
    suspend fun getUserById(id: Int): Result<User>
}

// Data Layer - Implementation
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        try {
            // Try remote first
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                val users = response.body()?.map { it.toDomain() } ?: emptyList()
                // Cache locally
                userDao.insertAll(users.map { it.toEntity() })
                emit(Result.success(users))
            } else {
                // Fallback to local cache
                val cachedUsers = userDao.getAll().map { it.toDomain() }
                emit(Result.success(cachedUsers))
            }
        } catch (e: Exception) {
            // Return cached data on error
            val cachedUsers = userDao.getAll().map { it.toDomain() }
            if (cachedUsers.isNotEmpty()) {
                emit(Result.success(cachedUsers))
            } else {
                emit(Result.failure(e))
            }
        }
    }
}
```

### 3. Use Case Pattern

**Single Responsibility - One use case per business action:**

```kotlin
class GetUsersUseCase @Inject constructor(
    private val repository: UserRepository
) {
    operator fun invoke(): Flow<Result<List<User>>> {
        return repository.getUsers()
    }
}

class LoginUseCase @Inject constructor(
    private val repository: AuthRepository
) {
    suspend operator fun invoke(email: String, password: String): Result<User> {
        // Add business logic validation here
        if (email.isBlank() || password.isBlank()) {
            return Result.failure(Exception("Email and password required"))
        }
        return repository.login(email, password)
    }
}
```

### 4. Hilt Dependency Injection

```kotlin
// Application
@HiltAndroidApp
class MyApplication : Application()

// Network Module
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

// Repository Binding
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    
    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

### 5. Compose UI with State Collection

```kotlin
@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = { TopBar() }
    ) { paddingValues ->
        when {
            uiState.isLoading -> LoadingIndicator()
            uiState.error != null -> ErrorView(uiState.error!!)
            else -> ContentView(uiState.data)
        }
    }
}
```

---

## 🎨 Material Design 3 Guidelines

### Color Scheme Structure

```kotlin
// Use Material 3 color roles
val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFFEADDFF),
    onPrimaryContainer = Color(0xFF21005D),
    secondary = Color(0xFF625B71),
    // ... complete color scheme
)
```

### Typography System

```kotlin
val Typography = Typography(
    displayLarge = TextStyle(fontSize = 57.sp, lineHeight = 64.sp),
    headlineMedium = TextStyle(fontSize = 28.sp, lineHeight = 36.sp),
    titleLarge = TextStyle(fontSize = 22.sp, lineHeight = 28.sp),
    bodyLarge = TextStyle(fontSize = 16.sp, lineHeight = 24.sp),
    labelMedium = TextStyle(fontSize = 12.sp, lineHeight = 16.sp)
)
```

### Component Usage

```kotlin
// Cards
Card(
    onClick = { /* action */ },
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
) { Content() }

// Buttons
Button(
    onClick = { /* action */ },
    colors = ButtonDefaults.buttonColors(
        containerColor = MaterialTheme.colorScheme.primary
    )
) { Text("Action") }

// Top App Bar
TopAppBar(
    title = { Text("Title") },
    colors = TopAppBarDefaults.topAppBarColors(
        containerColor = MaterialTheme.colorScheme.primaryContainer
    )
)
```

---

## ⚡ Performance Best Practices

### 1. Recomposition Optimization

```kotlin
// ✅ GOOD: Use keys for lists
LazyColumn {
    items(items = users, key = { it.id }) { user ->
        UserItem(user)
    }
}

// ✅ GOOD: Use derivedStateOf for expensive calculations
val sortedUsers by remember(users) {
    derivedStateOf { users.sortedBy { it.name } }
}

// ❌ BAD: Don't perform heavy work during composition
@Composable
fun MyScreen(users: List<User>) {
    val sorted = users.sortedBy { it.name } // Runs on every recomposition!
}
```

### 2. State Hoisting

```kotlin
// ✅ GOOD: Stateless composables
@Composable
fun SearchBar(
    query: String,
    onQueryChange: (String) -> Unit
) {
    TextField(value = query, onValueChange = onQueryChange)
}

// Usage in parent
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val searchQuery by viewModel.searchQuery.collectAsState()
    
    SearchBar(
        query = searchQuery,
        onQueryChange = { viewModel.updateSearchQuery(it) }
    )
}
```

### 3. Side Effects Management

```kotlin
// LaunchedEffect - For coroutine-based side effects
LaunchedEffect(userId) {
    viewModel.loadUser(userId)
}

// DisposableEffect - For cleanup
DisposableEffect(Unit) {
    val listener = SomeListener()
    registerListener(listener)
    onDispose {
        unregisterListener(listener)
    }
}

// SideEffect - For non-suspend side effects
SideEffect {
    analytics.logScreenView("HomeScreen")
}
```

---

## 📝 Code Quality Checklist

### ViewModel
- [ ] Uses `StateFlow` for UI state, not `LiveData`
- [ ] Exposes immutable state (`StateFlow`, not `MutableStateFlow`)
- [ ] Uses `viewModelScope` for coroutines
- [ ] Handles all error cases
- [ ] Uses `update {}` for state modifications
- [ ] No UI logic (View references, Context)

### Repository
- [ ] Returns `Flow<Result<T>>` or `Result<T>`
- [ ] Implements caching strategy (remote + local)
- [ ] Maps DTOs to domain models
- [ ] Handles network errors gracefully
- [ ] Uses Kotlin coroutines, not callbacks

### Composables
- [ ] Uses `collectAsStateWithLifecycle()` for Flow collection
- [ ] Keys are provided for `LazyColumn`/`LazyRow` items
- [ ] State hoisting applied for reusability
- [ ] No side effects in composition (use `LaunchedEffect`)
- [ ] Previews provided with `@Preview`

### Dependency Injection
- [ ] `@HiltViewModel` for ViewModels
- [ ] `@Singleton` for app-level dependencies
- [ ] Interface binding with `@Binds`
- [ ] Proper scoping (`SingletonComponent`, `ActivityComponent`)

---

## 🚀 Development Workflow

### Step 1: Define Feature Requirements
1. Identify screens needed
2. Define user actions and navigation
3. List data requirements (API, database)

### Step 2: Domain Layer First
1. Create domain models (pure Kotlin data classes)
2. Define repository interfaces
3. Write use cases with business logic

### Step 3: Data Layer
1. Create API service with Retrofit
2. Define DTOs and mappers
3. Implement repository with error handling
4. Set up Room database if needed

### Step 4: Dependency Injection
1. Create Hilt modules for network/database
2. Bind repository implementations
3. Verify dependency graph

### Step 5: Presentation Layer
1. Define UI state data class
2. Create ViewModel with StateFlow
3. Build Composable screens
4. Set up navigation

### Step 6: Testing
1. Unit test ViewModels (verify state transitions)
2. Unit test Use Cases (verify business logic)
3. Test repositories with mock data sources
4. Compose UI tests for critical flows

---

## 🧪 Testing Patterns

### ViewModel Testing

```kotlin
@Test
fun `loadUsers should update state with success`() = runTest {
    // Given
    val mockUsers = listOf(User(1, "Test"))
    coEvery { useCase() } returns flowOf(Result.success(mockUsers))
    
    val viewModel = HomeViewModel(useCase)
    
    // When
    viewModel.loadUsers()
    
    // Then
    val state = viewModel.uiState.value
    assertEquals(false, state.isLoading)
    assertEquals(mockUsers, state.users)
    assertNull(state.error)
}
```

### Compose UI Testing

```kotlin
@Test
fun homeScreen_displaysUserList() {
    composeTestRule.setContent {
        HomeScreen()
    }
    
    composeTestRule
        .onNodeWithText("User Name")
        .assertIsDisplayed()
}
```

---

## 🔗 Dependencies Template

```kotlin
// Compose BOM
val composeBom = platform("androidx.compose:compose-bom:2024.01.00")
implementation(composeBom)

// Core Compose
implementation("androidx.compose.ui:ui")
implementation("androidx.compose.material3:material3")
implementation("androidx.activity:activity-compose:1.8.2")

// ViewModel + Lifecycle
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")
implementation("androidx.lifecycle:lifecycle-runtime-compose:2.7.0")

// Navigation
implementation("androidx.navigation:navigation-compose:2.7.6")

// Hilt
implementation("com.google.dagger:hilt-android:2.48")
kapt("com.google.dagger:hilt-compiler:2.48")
implementation("androidx.hilt:hilt-navigation-compose:1.1.0")

// Coroutines
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

// Retrofit
implementation("com.squareup.retrofit2:retrofit:2.9.0")
implementation("com.squareup.retrofit2:converter-gson:2.9.0")

// Room
val roomVersion = "2.6.1"
implementation("androidx.room:room-runtime:$roomVersion")
implementation("androidx.room:room-ktx:$roomVersion")
kapt("androidx.room:room-compiler:$roomVersion")

// Testing
testImplementation("junit:junit:4.13.2")
testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
testImplementation("io.mockk:mockk:1.13.8")
androidTestImplementation("androidx.compose.ui:ui-test-junit4")
```

---

## ⚠️ Common Pitfalls to Avoid

### ❌ DON'T
```kotlin
// Don't expose MutableStateFlow
val uiState: MutableStateFlow<UiState> = MutableStateFlow(UiState())

// Don't use LiveData in new projects
val data: LiveData<List<Item>> = liveData { }

// Don't perform heavy work during composition
@Composable
fun MyScreen(items: List<Item>) {
    val sorted = items.sortedBy { it.name } // RECOMPUTES EVERY TIME!
}

// Don't use viewModelScope without error handling
viewModelScope.launch {
    val result = repository.getData() // Can crash on exception!
}

// Don't mix concerns - ViewModels shouldn't know about Views
class MyViewModel : ViewModel() {
    fun updateUI(context: Context) { /* NO! */ }
}
```

### ✅ DO
```kotlin
// Expose immutable StateFlow
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

// Use StateFlow with Kotlin Flow
val data: Flow<List<Item>> = repository.getItems()

// Use derivedStateOf for expensive calculations
val sorted by remember(items) {
    derivedStateOf { items.sortedBy { it.name } }
}

// Handle errors gracefully
viewModelScope.launch {
    try {
        val result = repository.getData()
        _uiState.update { it.copy(data = result) }
    } catch (e: Exception) {
        _uiState.update { it.copy(error = e.message) }
    }
}

// Keep ViewModels pure - no Android dependencies
class MyViewModel @Inject constructor(
    private val useCase: GetDataUseCase
) : ViewModel()
```

---

## 📚 References

- [Android Architecture Guide](https://developer.android.com/topic/architecture)
- [Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Material Design 3](https://m3.material.io/)
- [Hilt Dependency Injection](https://developer.android.com/training/dependency-injection/hilt-android)
- [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)

---

## 📄 License

MIT License - Free to use and modify for your projects.
