# Example Android Project - Task Manager App

This is a complete example project demonstrating all patterns from the `android-production-workflow` skill.

## 🎯 Project Overview

**Task Manager App** - A production-ready task management application showcasing:
- Complete MVVM + Clean Architecture implementation
- Jetpack Compose UI with Material Design 3
- Offline-first data caching
- Hilt dependency injection
- StateFlow state management
- Comprehensive error handling

## 📁 Project Structure

```
TaskManagerApp/
├── app/
│   ├── src/main/java/com/example/taskmanager/
│   │   ├── TaskManagerApplication.kt          # Hilt application class
│   │   │
│   │   ├── di/                                 # Dependency Injection
│   │   │   ├── AppModule.kt                   # App-level dependencies
│   │   │   ├── DatabaseModule.kt              # Room database
│   │   │   └── NetworkModule.kt               # Retrofit, OkHttp
│   │   │
│   │   ├── domain/                             # Domain Layer (Pure Kotlin)
│   │   │   ├── model/
│   │   │   │   └── Task.kt                    # Domain model
│   │   │   ├── repository/
│   │   │   │   └── TaskRepository.kt          # Repository interface
│   │   │   └── usecase/
│   │   │       ├── GetTasksUseCase.kt
│   │   │       ├── CreateTaskUseCase.kt
│   │   │       ├── UpdateTaskUseCase.kt
│   │   │       └── DeleteTaskUseCase.kt
│   │   │
│   │   ├── data/                               # Data Layer
│   │   │   ├── local/
│   │   │   │   ├── TaskDatabase.kt            # Room database
│   │   │   │   ├── TaskDao.kt                 # Database operations
│   │   │   │   └── TaskEntity.kt              # Database entity
│   │   │   ├── remote/
│   │   │   │   ├── TaskApi.kt                 # Retrofit API interface
│   │   │   │   └── TaskDto.kt                 # Network data model
│   │   │   ├── mapper/
│   │   │   │   └── TaskMapper.kt              # DTO ↔ Domain mapping
│   │   │   └── repository/
│   │   │       └── TaskRepositoryImpl.kt      # Repository implementation
│   │   │
│   │   └── presentation/                       # Presentation Layer
│   │       ├── theme/
│   │       │   ├── Theme.kt                   # Material Design 3 theme
│   │       │   ├── Color.kt
│   │       │   └── Type.kt
│   │       ├── navigation/
│   │       │   └── AppNavigation.kt           # Navigation graph
│   │       ├── tasks/
│   │       │   ├── TaskListScreen.kt          # Task list UI
│   │       │   ├── TaskListViewModel.kt       # List state management
│   │       │   ├── TaskDetailScreen.kt        # Task detail UI
│   │       │   ├── TaskDetailViewModel.kt     # Detail state management
│   │       │   └── components/
│   │       │       ├── TaskItem.kt            # Reusable task card
│   │       │       └── EmptyTasksView.kt      # Empty state
│   │       └── MainActivity.kt                # Main activity
│   │
│   └── build.gradle.kts                        # App-level Gradle
│
├── build.gradle.kts                            # Project-level Gradle
└── settings.gradle.kts
```

## 🏗️ Architecture Layers

### 1️⃣ Domain Layer (Business Logic)
**Location**: `domain/`

**Purpose**: Pure Kotlin business logic, no Android dependencies

**Components**:
- **Models**: `Task` data class with business validation
- **Repository Interface**: `TaskRepository` contract
- **Use Cases**: Single-responsibility business operations
  - `GetTasksUseCase` - Retrieve all tasks
  - `CreateTaskUseCase` - Create new task
  - `UpdateTaskUseCase` - Update existing task
  - `DeleteTaskUseCase` - Delete task

**Key Pattern**:
```kotlin
// Domain Model - Pure Kotlin
data class Task(
    val id: String,
    val title: String,
    val description: String,
    val isCompleted: Boolean,
    val priority: Priority,
    val createdAt: Long,
    val dueDate: Long?
) {
    enum class Priority { LOW, MEDIUM, HIGH, URGENT }
}

// Repository Interface - Dependency Inversion
interface TaskRepository {
    fun getTasks(): Flow<Result<List<Task>>>
    suspend fun getTaskById(id: String): Result<Task>
    suspend fun createTask(task: Task): Result<Task>
    suspend fun updateTask(task: Task): Result<Task>
    suspend fun deleteTask(id: String): Result<Unit>
}
```

### 2️⃣ Data Layer (Data Sources)
**Location**: `data/`

**Purpose**: Implement repository, manage data sources (API + Database)

**Components**:
- **Local**: Room database for offline caching
- **Remote**: Retrofit API client
- **Mapper**: Convert DTOs ↔ Domain models
- **Repository Implementation**: Offline-first strategy

**Key Pattern**:
```kotlin
// Offline-First Repository
class TaskRepositoryImpl @Inject constructor(
    private val taskApi: TaskApi,
    private val taskDao: TaskDao,
    private val mapper: TaskMapper
) : TaskRepository {
    
    override fun getTasks(): Flow<Result<List<Task>>> = flow {
        try {
            // 1. Emit cached data immediately
            val cachedTasks = taskDao.getAllTasks()
                .first()
                .map { mapper.toDomain(it) }
            emit(Result.success(cachedTasks))
            
            // 2. Fetch fresh data from network
            val remoteTasks = taskApi.getTasks()
            
            // 3. Update cache
            taskDao.insertAll(remoteTasks.map { mapper.toEntity(it) })
            
            // 4. Emit updated data
            val updatedTasks = remoteTasks.map { mapper.toDomain(it) }
            emit(Result.success(updatedTasks))
            
        } catch (e: Exception) {
            // 5. On error, emit cached data if available
            val cachedTasks = taskDao.getAllTasks()
                .first()
                .map { mapper.toDomain(it) }
            
            if (cachedTasks.isNotEmpty()) {
                emit(Result.success(cachedTasks))
            } else {
                emit(Result.failure(e))
            }
        }
    }
}
```

### 3️⃣ Presentation Layer (UI)
**Location**: `presentation/`

**Purpose**: Display UI, handle user interactions, manage UI state

**Components**:
- **ViewModel**: StateFlow-based state management
- **Composables**: Declarative UI components
- **Theme**: Material Design 3 configuration
- **Navigation**: Screen routing

**Key Pattern**:
```kotlin
// ViewModel with StateFlow
@HiltViewModel
class TaskListViewModel @Inject constructor(
    private val getTasksUseCase: GetTasksUseCase,
    private val deleteTaskUseCase: DeleteTaskUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<TaskListUiState>(TaskListUiState.Loading)
    val uiState: StateFlow<TaskListUiState> = _uiState.asStateFlow()
    
    init {
        loadTasks()
    }
    
    private fun loadTasks() {
        viewModelScope.launch {
            getTasksUseCase()
                .collect { result ->
                    _uiState.value = when {
                        result.isSuccess -> {
                            val tasks = result.getOrNull() ?: emptyList()
                            if (tasks.isEmpty()) {
                                TaskListUiState.Empty
                            } else {
                                TaskListUiState.Success(tasks)
                            }
                        }
                        result.isFailure -> {
                            TaskListUiState.Error(
                                result.exceptionOrNull()?.message ?: "Unknown error"
                            )
                        }
                        else -> TaskListUiState.Loading
                    }
                }
        }
    }
    
    fun deleteTask(taskId: String) {
        viewModelScope.launch {
            deleteTaskUseCase(taskId)
        }
    }
}

// UI State sealed interface
sealed interface TaskListUiState {
    object Loading : TaskListUiState
    object Empty : TaskListUiState
    data class Success(val tasks: List<Task>) : TaskListUiState
    data class Error(val message: String) : TaskListUiState
}

// Composable Screen
@Composable
fun TaskListScreen(
    viewModel: TaskListViewModel = hiltViewModel(),
    onTaskClick: (String) -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("My Tasks") }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = { /* Navigate to create */ }) {
                Icon(Icons.Default.Add, contentDescription = "Add Task")
            }
        }
    ) { padding ->
        when (val state = uiState) {
            is TaskListUiState.Loading -> LoadingView()
            is TaskListUiState.Empty -> EmptyTasksView()
            is TaskListUiState.Success -> TaskList(
                tasks = state.tasks,
                onTaskClick = onTaskClick,
                onTaskDelete = { viewModel.deleteTask(it) }
            )
            is TaskListUiState.Error -> ErrorView(
                message = state.message,
                onRetry = { viewModel.loadTasks() }
            )
        }
    }
}
```

## 🔧 Key Features Demonstrated

### ✅ Dependency Injection with Hilt
```kotlin
@HiltAndroidApp
class TaskManagerApplication : Application()

@AndroidEntryPoint
class MainActivity : ComponentActivity()

@HiltViewModel
class TaskListViewModel @Inject constructor(
    private val getTasksUseCase: GetTasksUseCase
) : ViewModel()
```

### ✅ Offline-First Caching
- Room database for local storage
- Network requests with automatic caching
- Graceful fallback on network errors

### ✅ Material Design 3 Theming
- Dynamic color support
- Dark/Light theme switching
- Custom typography and shapes

### ✅ State Management
- `StateFlow` for UI state
- `SharedFlow` for one-time events
- `collectAsStateWithLifecycle()` for composition

### ✅ Error Handling
- Try-catch in repository layer
- `Result<T>` type for operations
- User-friendly error messages

### ✅ Navigation
- Jetpack Compose Navigation
- Type-safe routes
- Deep linking support

## 🚀 How to Build This Project

### Using the Skill with AI:

**Step 1: Generate Project Structure**
```
Using android-production-workflow skill, create the Task Manager app:
1. Set up the complete project structure with all gradle files
2. Configure Hilt dependency injection
3. Set up Material Design 3 theme
```

**Step 2: Generate Domain Layer**
```
Using android-production-workflow skill, create the domain layer:
- Task domain model with Priority enum
- TaskRepository interface
- GetTasksUseCase, CreateTaskUseCase, UpdateTaskUseCase, DeleteTaskUseCase
```

**Step 3: Generate Data Layer**
```
Using android-production-workflow skill, create the data layer:
- Room database with TaskEntity and TaskDao
- Retrofit API with TaskApi and TaskDto
- TaskMapper for conversions
- TaskRepositoryImpl with offline-first caching
```

**Step 4: Generate Presentation Layer**
```
Using android-production-workflow skill, create the presentation layer:
- TaskListViewModel with StateFlow
- TaskListScreen with Compose UI
- TaskDetailViewModel and TaskDetailScreen
- TaskItem reusable component
```

## 📊 Project Stats

- **Total Classes**: ~25 files
- **Lines of Code**: ~2,000 (excluding generated)
- **Architecture**: Clean Architecture (3 layers)
- **UI Framework**: 100% Jetpack Compose
- **Min SDK**: 24 (Android 7.0)
- **Target SDK**: 34 (Android 14)

## 🎓 Learning Outcomes

After studying this example, you'll understand:

1. **Clean Architecture**: How to properly separate concerns across layers
2. **MVVM Pattern**: ViewModel manages state, View observes and renders
3. **Dependency Injection**: Hilt provides dependencies automatically
4. **State Management**: StateFlow for reactive, lifecycle-aware state
5. **Offline-First**: Cache data locally, sync when online
6. **Compose UI**: Build declarative, reusable UI components
7. **Material Design 3**: Implement modern Android design standards
8. **Error Handling**: Graceful degradation and user feedback

## 🔗 Related Documentation

- [SKILL.md](../SKILL.md) - Complete architecture patterns
- [QuickStartGuide.md](../docs/QuickStartGuide.md) - Step-by-step tutorial
- [UserManagementExample.md](../examples/UserManagementExample.md) - Similar CRUD example
- [Templates](../templates/) - Code templates for copy-paste

## 💡 Tips for Building

1. **Start with Domain Layer** - Define your business logic first
2. **Implement Data Layer** - Set up data sources and repository
3. **Build Presentation Layer** - Create UI and ViewModels last
4. **Test as You Go** - Write unit tests for each layer
5. **Follow the Patterns** - Consistency is key for maintainability

---

**Ready to build your own production-quality Android app!** 🚀

Use this example as a reference for implementing similar features in your projects.
