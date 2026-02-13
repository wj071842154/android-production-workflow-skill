# Complete User Management Example

This example demonstrates a full MVVM + Compose + Clean Architecture implementation for a User Management feature.

## Architecture Flow

```
UserScreen (UI)
    ↓ collectAsStateWithLifecycle
UserViewModel (Presentation)
    ↓ invoke()
GetUsersUseCase (Domain)
    ↓ getUsers()
UserRepository Interface (Domain)
    ↓ implements
UserRepositoryImpl (Data)
    ↓ calls
ApiService + UserDao (Data Sources)
```

## File Structure

```
domain/
├── model/
│   └── User.kt
├── repository/
│   └── UserRepository.kt
└── usecase/
    └── GetUsersUseCase.kt

data/
├── repository/
│   └── UserRepositoryImpl.kt
├── remote/
│   ├── api/
│   │   └── ApiService.kt
│   └── dto/
│       └── UserDto.kt
├── local/
│   ├── dao/
│   │   └── UserDao.kt
│   └── entity/
│       └── UserEntity.kt
└── mapper/
    └── UserMapper.kt

presentation/
└── screens/
    └── users/
        ├── UserScreen.kt
        ├── UserViewModel.kt
        └── UserUiState.kt
```

## Code Files

### Domain Layer

**User.kt** - Domain Model
```kotlin
package com.example.app.domain.model

data class User(
    val id: Int,
    val name: String,
    val email: String,
    val avatarUrl: String,
    val isActive: Boolean
)
```

**UserRepository.kt** - Repository Interface
```kotlin
package com.example.app.domain.repository

import com.example.app.domain.model.User
import kotlinx.coroutines.flow.Flow

interface UserRepository {
    fun getUsers(): Flow<Result<List<User>>>
    suspend fun getUserById(id: Int): Result<User>
    suspend fun updateUser(user: User): Result<Unit>
    suspend fun deleteUser(id: Int): Result<Unit>
}
```

**GetUsersUseCase.kt** - Business Logic
```kotlin
package com.example.app.domain.usecase

import com.example.app.domain.model.User
import com.example.app.domain.repository.UserRepository
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class GetUsersUseCase @Inject constructor(
    private val repository: UserRepository
) {
    operator fun invoke(): Flow<Result<List<User>>> {
        return repository.getUsers()
    }
}
```

### Data Layer

**UserDto.kt** - API Response Model
```kotlin
package com.example.app.data.remote.dto

import com.google.gson.annotations.SerializedName

data class UserDto(
    @SerializedName("id")
    val id: Int,
    @SerializedName("name")
    val name: String,
    @SerializedName("email")
    val email: String,
    @SerializedName("avatar")
    val avatarUrl: String,
    @SerializedName("active")
    val isActive: Boolean
)
```

**UserEntity.kt** - Database Entity
```kotlin
package com.example.app.data.local.entity

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey
    val id: Int,
    val name: String,
    val email: String,
    val avatarUrl: String,
    val isActive: Boolean
)
```

**UserMapper.kt** - Data Mappers
```kotlin
package com.example.app.data.mapper

import com.example.app.data.local.entity.UserEntity
import com.example.app.data.remote.dto.UserDto
import com.example.app.domain.model.User

// DTO to Domain
fun UserDto.toDomain(): User {
    return User(
        id = id,
        name = name,
        email = email,
        avatarUrl = avatarUrl,
        isActive = isActive
    )
}

// Entity to Domain
fun UserEntity.toDomain(): User {
    return User(
        id = id,
        name = name,
        email = email,
        avatarUrl = avatarUrl,
        isActive = isActive
    )
}

// Domain to Entity
fun User.toEntity(): UserEntity {
    return UserEntity(
        id = id,
        name = name,
        email = email,
        avatarUrl = avatarUrl,
        isActive = isActive
    )
}
```

**ApiService.kt** - Retrofit API
```kotlin
package com.example.app.data.remote.api

import com.example.app.data.remote.dto.UserDto
import retrofit2.Response
import retrofit2.http.*

interface ApiService {
    
    @GET("users")
    suspend fun getUsers(): Response<List<UserDto>>
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): Response<UserDto>
    
    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: Int,
        @Body user: UserDto
    ): Response<UserDto>
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int): Response<Unit>
}
```

**UserDao.kt** - Room DAO
```kotlin
package com.example.app.data.local.dao

import androidx.room.*
import com.example.app.data.local.entity.UserEntity

@Dao
interface UserDao {
    
    @Query("SELECT * FROM users")
    suspend fun getAll(): List<UserEntity>
    
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getById(id: Int): UserEntity?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
    
    @Update
    suspend fun update(user: UserEntity)
    
    @Delete
    suspend fun delete(user: UserEntity)
    
    @Query("DELETE FROM users WHERE id = :id")
    suspend fun deleteById(id: Int)
}
```

**UserRepositoryImpl.kt** - Repository Implementation
```kotlin
package com.example.app.data.repository

import com.example.app.data.local.dao.UserDao
import com.example.app.data.mapper.toDomain
import com.example.app.data.mapper.toEntity
import com.example.app.data.remote.api.ApiService
import com.example.app.domain.model.User
import com.example.app.domain.repository.UserRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import javax.inject.Inject

class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        try {
            val response = apiService.getUsers()
            
            if (response.isSuccessful && response.body() != null) {
                val users = response.body()!!.map { it.toDomain() }
                userDao.insertAll(users.map { it.toEntity() })
                emit(Result.success(users))
            } else {
                val cachedUsers = userDao.getAll().map { it.toDomain() }
                if (cachedUsers.isNotEmpty()) {
                    emit(Result.success(cachedUsers))
                } else {
                    emit(Result.failure(Exception("Failed to load users")))
                }
            }
        } catch (e: Exception) {
            val cachedUsers = userDao.getAll().map { it.toDomain() }
            if (cachedUsers.isNotEmpty()) {
                emit(Result.success(cachedUsers))
            } else {
                emit(Result.failure(e))
            }
        }
    }
    
    override suspend fun getUserById(id: Int): Result<User> {
        return try {
            val response = apiService.getUserById(id)
            if (response.isSuccessful && response.body() != null) {
                val user = response.body()!!.toDomain()
                userDao.insert(user.toEntity())
                Result.success(user)
            } else {
                val cached = userDao.getById(id)?.toDomain()
                if (cached != null) {
                    Result.success(cached)
                } else {
                    Result.failure(Exception("User not found"))
                }
            }
        } catch (e: Exception) {
            val cached = userDao.getById(id)?.toDomain()
            if (cached != null) {
                Result.success(cached)
            } else {
                Result.failure(e)
            }
        }
    }
    
    override suspend fun updateUser(user: User): Result<Unit> {
        return try {
            userDao.update(user.toEntity())
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun deleteUser(id: Int): Result<Unit> {
        return try {
            userDao.deleteById(id)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Presentation Layer

**UserUiState.kt** - UI State
```kotlin
package com.example.app.presentation.screens.users

import com.example.app.domain.model.User

data class UserUiState(
    val isLoading: Boolean = false,
    val users: List<User> = emptyList(),
    val error: String? = null,
    val selectedUser: User? = null
)
```

**UserViewModel.kt** - ViewModel
```kotlin
package com.example.app.presentation.screens.users

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.app.domain.model.User
import com.example.app.domain.usecase.GetUsersUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import javax.inject.Inject

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
            _uiState.update { it.copy(isLoading = true, error = null) }
            
            getUsersUseCase().collect { result ->
                result.fold(
                    onSuccess = { users ->
                        _uiState.update {
                            it.copy(
                                isLoading = false,
                                users = users,
                                error = null
                            )
                        }
                    },
                    onFailure = { exception ->
                        _uiState.update {
                            it.copy(
                                isLoading = false,
                                error = exception.message ?: "Unknown error"
                            )
                        }
                    }
                )
            }
        }
    }
    
    fun onRefresh() {
        loadUsers()
    }
    
    fun onUserSelected(user: User) {
        _uiState.update { it.copy(selectedUser = user) }
    }
    
    fun clearSelection() {
        _uiState.update { it.copy(selectedUser = null) }
    }
}
```

**UserScreen.kt** - Composable UI
```kotlin
package com.example.app.presentation.screens.users

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Person
import androidx.compose.material.icons.filled.Refresh
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.example.app.domain.model.User

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel(),
    onUserClick: (User) -> Unit = {}
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Users") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                ),
                actions = {
                    IconButton(onClick = { viewModel.onRefresh() }) {
                        Icon(Icons.Default.Refresh, contentDescription = "Refresh")
                    }
                }
            )
        }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            when {
                uiState.isLoading -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                
                uiState.error != null -> {
                    ErrorContent(
                        message = uiState.error!!,
                        onRetry = { viewModel.onRefresh() },
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                
                uiState.users.isEmpty() -> {
                    EmptyContent(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                
                else -> {
                    UserList(
                        users = uiState.users,
                        onUserClick = onUserClick
                    )
                }
            }
        }
    }
}

@Composable
private fun UserList(
    users: List<User>,
    onUserClick: (User) -> Unit
) {
    LazyColumn(
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        items(items = users, key = { it.id }) { user ->
            UserCard(user = user, onClick = { onUserClick(user) })
        }
    }
}

@Composable
private fun UserCard(
    user: User,
    onClick: () -> Unit
) {
    Card(
        onClick = onClick,
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Icon(
                imageVector = Icons.Default.Person,
                contentDescription = null,
                modifier = Modifier.size(48.dp),
                tint = MaterialTheme.colorScheme.primary
            )
            
            Column {
                Text(
                    text = user.name,
                    style = MaterialTheme.typography.titleMedium
                )
                Text(
                    text = user.email,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}

@Composable
private fun ErrorContent(
    message: String,
    onRetry: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier.padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "Error: $message",
            style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.error
        )
        Button(onClick = onRetry) {
            Text("Retry")
        }
    }
}

@Composable
private fun EmptyContent(modifier: Modifier = Modifier) {
    Text(
        text = "No users found",
        modifier = modifier,
        style = MaterialTheme.typography.bodyLarge
    )
}
```

## Testing

**UserViewModelTest.kt**
```kotlin
@Test
fun `loadUsers should update state with success`() = runTest {
    // Given
    val mockUsers = listOf(User(1, "Test", "test@example.com", "", true))
    coEvery { getUsersUseCase() } returns flowOf(Result.success(mockUsers))
    
    // When
    val viewModel = UserViewModel(getUsersUseCase)
    
    // Then
    assertEquals(false, viewModel.uiState.value.isLoading)
    assertEquals(mockUsers, viewModel.uiState.value.users)
    assertNull(viewModel.uiState.value.error)
}
```

## Usage

This complete example shows:
- ✅ Clean Architecture separation
- ✅ MVVM pattern with StateFlow
- ✅ Repository pattern with caching
- ✅ Hilt dependency injection
- ✅ Compose UI with Material 3
- ✅ Error handling
- ✅ Loading states
- ✅ Testable code
