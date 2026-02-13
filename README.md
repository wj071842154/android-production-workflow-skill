# Android Production Workflow Skill

**Complete production-ready Android development workflow** using modern best practices.

## 🚀 What This Skill Provides

A comprehensive guide and templates for building Android apps with:

- ✅ **Jetpack Compose** - Modern declarative UI
- ✅ **MVVM + Clean Architecture** - Scalable, testable structure  
- ✅ **Material Design 3** - Beautiful, consistent UI/UX
- ✅ **Hilt** - Dependency injection made simple
- ✅ **Kotlin Coroutines + Flow** - Reactive async programming
- ✅ **Retrofit** - Type-safe HTTP client
- ✅ **Room** - Powerful local database with caching

## 📁 What's Included

```
android-production-workflow-skill/
├── SKILL.md                          # Complete patterns & best practices
├── skill.json                        # Skill metadata
│
├── docs/
│   └── QuickStartGuide.md            # Step-by-step tutorial
│
├── examples/
│   └── UserManagementExample.md      # Full working example
│
└── templates/
    ├── build.gradle.kts.template     # Gradle configuration
    ├── Theme.kt.template             # Material 3 theme setup
    ├── ViewModel.kt.template         # ViewModel pattern
    ├── Repository.kt.template        # Repository with caching
    └── Screen.kt.template            # Compose UI screen
```

## 🎯 Key Features

### Architecture Patterns

- **Clean Architecture** with 3 clear layers (Domain/Data/Presentation)
- **MVVM** with StateFlow for reactive UI
- **Repository Pattern** with offline-first caching
- **Use Cases** for reusable business logic
- **Dependency Injection** with Hilt

### UI Development

- **Material Design 3** color schemes and typography
- **Jetpack Compose** best practices
- **State Management** with StateFlow and collectAsStateWithLifecycle
- **Navigation** with Navigation Compose
- **Reusable Components** library

### Code Quality

- **Type-safe** API calls with Retrofit
- **Null-safe** Kotlin throughout
- **Error Handling** with Result types
- **Loading States** for better UX
- **Testable** code structure

## 🚀 Quick Start

### 1. Install This Skill

```bash
# Option 1: Install from local directory
npx skills add /path/to/android-production-workflow-skill -g

# Option 2: Install from GitHub (once published)
npx skills add yourusername/android-skills@android-production-workflow -g
```

### 2. Start a New Project

Follow the **Quick Start Guide** in `docs/QuickStartGuide.md`:

1. Create new Android project in Android Studio
2. Copy Gradle configuration from `templates/build.gradle.kts.template`
3. Setup Hilt with `@HiltAndroidApp`
4. Create package structure (domain/data/presentation)
5. Copy Material 3 theme from `templates/Theme.kt.template`
6. Build your first feature following the patterns

### 3. Use Templates for New Features

When adding a new feature:

```kotlin
// 1. Copy ViewModel template
templates/ViewModel.kt.template
// Replace: {{FEATURE}}, {{MODEL}}, {{USE_CASE}}

// 2. Copy Repository template  
templates/Repository.kt.template
// Replace: {{REPOSITORY}}, {{MODEL}}, {{DAO}}

// 3. Copy Screen template
templates/Screen.kt.template
// Replace: {{FEATURE}}, {{MODEL}}
```

## 📚 Documentation

### Core Documentation

- **[SKILL.md](./SKILL.md)** - Complete architecture guide, patterns, and best practices
- **[QuickStartGuide.md](./docs/QuickStartGuide.md)** - Step-by-step tutorial from zero to running app
- **[UserManagementExample.md](./examples/UserManagementExample.md)** - Full working code example

### Key Sections

| Document | What You'll Learn |
|----------|-------------------|
| SKILL.md | Architecture layers, MVVM pattern, StateFlow usage, Hilt setup, performance optimization |
| QuickStartGuide.md | Project setup, package structure, first feature implementation, navigation setup |
| UserManagementExample.md | Complete User CRUD example with API, database, and UI |

## 🎨 Architecture Overview

```
┌─────────────────────────────────────────┐
│      Presentation Layer (UI)            │
│   • Composables (Jetpack Compose)      │
│   • ViewModels (StateFlow)              │
│   • Navigation                          │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│       Domain Layer (Business)           │
│   • Use Cases (Business Logic)          │
│   • Domain Models (Pure Kotlin)         │
│   • Repository Interfaces               │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│        Data Layer (Sources)             │
│   • Repository Implementations          │
│   • API Service (Retrofit)              │
│   • Database (Room)                     │
│   • DTOs & Mappers                      │
└─────────────────────────────────────────┘
```

## 🔥 Code Examples

### ViewModel with StateFlow

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            
            getUsersUseCase().collect { result ->
                result.fold(
                    onSuccess = { users ->
                        _uiState.update {
                            it.copy(isLoading = false, users = users)
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

### Repository with Caching

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        try {
            // Try remote API
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                val users = response.body()?.map { it.toDomain() } ?: emptyList()
                // Cache locally
                userDao.insertAll(users.map { it.toEntity() })
                emit(Result.success(users))
            } else {
                // Fallback to cache
                val cached = userDao.getAll().map { it.toDomain() }
                emit(Result.success(cached))
            }
        } catch (e: Exception) {
            // Return cache on network error
            val cached = userDao.getAll().map { it.toDomain() }
            if (cached.isNotEmpty()) {
                emit(Result.success(cached))
            } else {
                emit(Result.failure(e))
            }
        }
    }
}
```

### Compose Screen with Material 3

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Users") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                )
            )
        }
    ) { paddingValues ->
        when {
            uiState.isLoading -> LoadingIndicator()
            uiState.error != null -> ErrorView(uiState.error!!)
            else -> UserList(uiState.users)
        }
    }
}
```

## ✅ Best Practices Checklist

### Architecture
- [ ] Clean separation of Domain/Data/Presentation layers
- [ ] Domain layer has NO Android dependencies
- [ ] Repository interfaces in Domain, implementations in Data
- [ ] Use Cases for business logic

### State Management
- [ ] ViewModels use `StateFlow`, not `LiveData`
- [ ] Expose immutable `StateFlow`, not `MutableStateFlow`
- [ ] Use `collectAsStateWithLifecycle()` in Composables
- [ ] Use `update {}` for state modifications

### Dependency Injection
- [ ] `@HiltAndroidApp` on Application class
- [ ] `@HiltViewModel` for ViewModels
- [ ] `@Singleton` for app-level dependencies
- [ ] Interface binding with `@Binds`

### Compose UI
- [ ] Keys provided for `LazyColumn` items
- [ ] State hoisting for reusable components
- [ ] No side effects in composition
- [ ] Use `LaunchedEffect` for coroutines

## 🧪 Testing

This skill promotes testable code:

```kotlin
@Test
fun `loadUsers should update state with success`() = runTest {
    // Given
    val mockUsers = listOf(User(1, "Test", "test@example.com"))
    coEvery { useCase() } returns flowOf(Result.success(mockUsers))
    
    // When
    val viewModel = UserViewModel(useCase)
    
    // Then
    assertEquals(mockUsers, viewModel.uiState.value.users)
    assertFalse(viewModel.uiState.value.isLoading)
}
```

## 🎓 Learning Path

1. **Beginners**: Start with `docs/QuickStartGuide.md`
2. **Intermediate**: Study patterns in `SKILL.md`
3. **Advanced**: Explore `examples/UserManagementExample.md`
4. **Reference**: Use templates for new features

## 🔧 Requirements

- Android Studio Arctic Fox or later
- Kotlin 1.9.20+
- Gradle 8.2+
- Min SDK 24 (Android 7.0)
- Target SDK 34 (Android 14)

## 📦 Dependencies Managed

This skill provides configurations for:

- Compose BOM 2024.01.00
- Hilt 2.48
- Retrofit 2.9.0
- Room 2.6.1
- Kotlin Coroutines 1.7.3
- Navigation Compose 2.7.6

## 🤝 Contributing

To improve this skill:

1. Add new templates for common patterns
2. Expand examples with more use cases
3. Add testing patterns and examples
4. Document advanced techniques

## 📄 License

MIT License - Free to use and modify for your projects.

## 🙋 Support

- Read `SKILL.md` for detailed patterns
- Check `examples/` for working code
- Follow `docs/QuickStartGuide.md` step-by-step

---

**Happy Coding! Build amazing Android apps with confidence.** 🚀
