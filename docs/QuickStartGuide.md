# Quick Start Guide - From Zero to Production Android App

This guide walks you through building a production-ready Android app from scratch using Jetpack Compose, MVVM, and Material Design 3.

## Step 1: Create New Android Project

1. Open Android Studio
2. **New Project** → **Empty Activity** (Compose)
3. Configure:
   - Name: YourAppName
   - Package: com.yourcompany.yourapp
   - Language: Kotlin
   - Minimum SDK: API 24 (Android 7.0)

## Step 2: Configure Gradle

### Project-level `build.gradle.kts`

```kotlin
plugins {
    id("com.android.application") version "8.2.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.20" apply false
    id("com.google.dagger.hilt.android") version "2.48" apply false
}
```

### App-level `build.gradle.kts`

Copy the template from `templates/build.gradle.kts.template` and adjust:
- Change `namespace` to your package name
- Update `applicationId`

## Step 3: Setup Hilt

### Create Application Class

```kotlin
package com.yourcompany.yourapp

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application()
```

### Update AndroidManifest.xml

```xml
<application
    android:name=".MyApplication"
    ...>
```

## Step 4: Create Package Structure

```
com.yourcompany.yourapp/
├── MyApplication.kt
├── di/
│   ├── AppModule.kt
│   ├── NetworkModule.kt
│   └── DatabaseModule.kt
├── domain/
│   ├── model/
│   ├── repository/
│   └── usecase/
├── data/
│   ├── repository/
│   ├── remote/
│   │   ├── api/
│   │   └── dto/
│   ├── local/
│   │   ├── database/
│   │   ├── dao/
│   │   └── entity/
│   └── mapper/
└── presentation/
    ├── theme/
    ├── navigation/
    ├── components/
    └── screens/
```

## Step 5: Setup Material 3 Theme

1. Create `presentation/theme/` package
2. Copy `templates/Theme.kt.template` to `Theme.kt`
3. Rename `YourAppTheme` to match your app
4. Customize colors if needed

## Step 6: Define Your First Feature

Let's build a "Users" feature as an example.

### Domain Layer

**1. Create Domain Model**
```kotlin
// domain/model/User.kt
data class User(
    val id: Int,
    val name: String,
    val email: String
)
```

**2. Create Repository Interface**
```kotlin
// domain/repository/UserRepository.kt
interface UserRepository {
    fun getUsers(): Flow<Result<List<User>>>
}
```

**3. Create Use Case**
```kotlin
// domain/usecase/GetUsersUseCase.kt
class GetUsersUseCase @Inject constructor(
    private val repository: UserRepository
) {
    operator fun invoke(): Flow<Result<List<User>>> {
        return repository.getUsers()
    }
}
```

### Data Layer

**4. Create API Service**
```kotlin
// data/remote/api/ApiService.kt
interface ApiService {
    @GET("users")
    suspend fun getUsers(): Response<List<UserDto>>
}
```

**5. Create DTO**
```kotlin
// data/remote/dto/UserDto.kt
data class UserDto(
    @SerializedName("id") val id: Int,
    @SerializedName("name") val name: String,
    @SerializedName("email") val email: String
)
```

**6. Create Mapper**
```kotlin
// data/mapper/UserMapper.kt
fun UserDto.toDomain() = User(
    id = id,
    name = name,
    email = email
)
```

**7. Implement Repository**
```kotlin
// data/repository/UserRepositoryImpl.kt
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService
) : UserRepository {
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        try {
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                val users = response.body()?.map { it.toDomain() } ?: emptyList()
                emit(Result.success(users))
            } else {
                emit(Result.failure(Exception("API Error")))
            }
        } catch (e: Exception) {
            emit(Result.failure(e))
        }
    }
}
```

### Dependency Injection

**8. Create Network Module**
```kotlin
// di/NetworkModule.kt
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
```

**9. Create Repository Module**
```kotlin
// di/RepositoryModule.kt
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

### Presentation Layer

**10. Create UI State**
```kotlin
// presentation/screens/users/UserUiState.kt
data class UserUiState(
    val isLoading: Boolean = false,
    val users: List<User> = emptyList(),
    val error: String? = null
)
```

**11. Create ViewModel**
```kotlin
// presentation/screens/users/UserViewModel.kt
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
    
    init {
        loadUsers()
    }
    
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

**12. Create Screen**
```kotlin
// presentation/screens/users/UserScreen.kt
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(title = { Text("Users") })
        }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            when {
                uiState.isLoading -> CircularProgressIndicator()
                uiState.error != null -> Text("Error: ${uiState.error}")
                else -> {
                    LazyColumn {
                        items(uiState.users, key = { it.id }) { user ->
                            Text(user.name)
                        }
                    }
                }
            }
        }
    }
}
```

### Setup Navigation

**13. Create NavGraph**
```kotlin
// presentation/navigation/NavGraph.kt
@Composable
fun AppNavGraph() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "users"
    ) {
        composable("users") {
            UserScreen()
        }
    }
}
```

**14. Update MainActivity**
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            YourAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    AppNavGraph()
                }
            }
        }
    }
}
```

## Step 7: Run Your App

1. **Sync Gradle** - Wait for dependencies to download
2. **Build** → **Make Project**
3. **Run** → **Run 'app'**

## Step 8: Add More Features

Repeat Step 6 for each new feature:
1. Domain model → Repository interface → Use case
2. DTO → Mapper → Repository impl
3. Hilt module bindings
4. UI State → ViewModel → Screen

## Common Next Steps

### Add Database Caching

**Create Room Database:**
```kotlin
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

**Create DAO:**
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    suspend fun getAll(): List<UserEntity>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
}
```

**Update Repository to use cache:**
```kotlin
override fun getUsers(): Flow<Result<List<User>>> = flow {
    try {
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
        // Return cache on error
        val cached = userDao.getAll().map { it.toDomain() }
        if (cached.isNotEmpty()) {
            emit(Result.success(cached))
        } else {
            emit(Result.failure(e))
        }
    }
}
```

### Add Testing

**ViewModel Test:**
```kotlin
@Test
fun `loadUsers updates state correctly`() = runTest {
    val mockUsers = listOf(User(1, "Test", "test@example.com"))
    coEvery { getUsersUseCase() } returns flowOf(Result.success(mockUsers))
    
    val viewModel = UserViewModel(getUsersUseCase)
    
    assertEquals(mockUsers, viewModel.uiState.value.users)
}
```

## Checklist

- [ ] Hilt setup complete
- [ ] Domain layer defined (models, repos, use cases)
- [ ] Data layer implemented (API, DTOs, mappers, repo impl)
- [ ] Dependency injection configured
- [ ] ViewModels created with StateFlow
- [ ] Screens built with Compose
- [ ] Navigation setup
- [ ] Material 3 theme applied
- [ ] Error handling implemented
- [ ] Loading states handled
- [ ] Tests written

## Troubleshooting

**Hilt errors?**
- Ensure `@HiltAndroidApp` on Application class
- Check AndroidManifest.xml has correct `android:name`
- Verify all modules are installed in `SingletonComponent`

**Compose not working?**
- Check `buildFeatures { compose = true }`
- Verify `composeOptions { kotlinCompilerExtensionVersion }`
- Use Compose BOM for version management

**Navigation not working?**
- Add `navigation-compose` dependency
- Use `hiltViewModel()` in screens, not parent composables

## Resources

- See `examples/UserManagementExample.md` for complete code
- See `SKILL.md` for patterns and best practices
- See `templates/` for reusable code templates
