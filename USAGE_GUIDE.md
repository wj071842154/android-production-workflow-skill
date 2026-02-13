# Usage Guide - Android Production Workflow Skill

## 🎯 How AI Agents Use This Skill

Once installed, AI agents automatically reference this skill when you work on Android projects. The skill provides:

1. **Architecture Patterns** - MVVM, Clean Architecture, Repository patterns
2. **Code Templates** - Ready-to-use ViewModel, Repository, Screen templates
3. **Best Practices** - Performance optimization, state management, dependency injection
4. **Working Examples** - Complete User Management feature example

## 📝 Example Prompts That Trigger This Skill

### Creating New Features

**Prompt:**
```
Create a new Products feature with MVVM architecture and Compose UI. 
Include:
- Product domain model
- Repository with API and Room caching
- ViewModel with StateFlow
- Compose screen with Material 3
```

**AI Will:**
- Use templates from `templates/ViewModel.kt.template`
- Follow patterns from `SKILL.md`
- Apply Clean Architecture layers
- Generate Repository with caching strategy
- Create Material 3 themed UI

---

### Refactoring Existing Code

**Prompt:**
```
Refactor this Fragment-based screen to use Jetpack Compose with MVVM.
Follow the android-production-workflow skill patterns.
```

**AI Will:**
- Convert to Composable function
- Extract state to ViewModel with StateFlow
- Apply state hoisting patterns
- Use collectAsStateWithLifecycle()
- Add proper error handling

---

### Architecture Review

**Prompt:**
```
Review my ViewModel code and suggest improvements based on 
android-production-workflow best practices.
```

**AI Will:**
- Check for StateFlow usage (not LiveData)
- Verify immutable state exposure
- Check error handling
- Suggest viewModelScope usage
- Review state update patterns

---

### Setting Up New Projects

**Prompt:**
```
Set up a new Android project structure following the 
android-production-workflow skill. Include Hilt, Retrofit, and Room.
```

**AI Will:**
- Use `templates/build.gradle.kts.template`
- Create package structure (domain/data/presentation)
- Setup Hilt Application class
- Configure Network and Database modules
- Apply Material 3 theme from templates

---

## 🧩 Specific Use Cases

### 1. Building Login Feature

**Prompt:**
```
Build a login feature with:
- Email/password validation
- API authentication
- Token storage
- Navigation to home screen
Follow android-production-workflow patterns
```

**Generated Structure:**
```
domain/
  model/
    LoginCredentials.kt
    AuthToken.kt
  repository/
    AuthRepository.kt
  usecase/
    LoginUseCase.kt
    
data/
  repository/
    AuthRepositoryImpl.kt
  remote/
    api/AuthApiService.kt
    dto/LoginDto.kt
    
presentation/
  screens/login/
    LoginScreen.kt
    LoginViewModel.kt
    LoginUiState.kt
```

---

### 2. Adding Offline Support

**Prompt:**
```
Add offline caching to my Users repository using Room.
Follow the caching strategy from android-production-workflow.
```

**AI Will:**
```kotlin
// Generate Room Entity
@Entity(tableName = "users")
data class UserEntity(...)

// Generate DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    suspend fun getAll(): List<UserEntity>
    
    @Insert(onConflict = REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
}

// Update Repository with caching
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        try {
            // Try remote first
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                val users = response.body()!!.map { it.toDomain() }
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
}
```

---

### 3. Creating Reusable Components

**Prompt:**
```
Create reusable Compose components for common UI elements
following Material 3 and android-production-workflow patterns:
- Loading indicator
- Error view
- Empty state
- List item card
```

**AI Will Generate:**
```kotlin
@Composable
fun LoadingIndicator(modifier: Modifier = Modifier) {
    CircularProgressIndicator(
        modifier = modifier,
        color = MaterialTheme.colorScheme.primary
    )
}

@Composable
fun ErrorView(
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
            text = message,
            style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.error
        )
        Button(onClick = onRetry) {
            Text("Retry")
        }
    }
}
// ... more components
```

---

## 🎨 Customizing for Your Team

### Adding Team-Specific Patterns

1. **Copy templates to your project:**
   ```bash
   cp ~/.agents/skills/android-production-workflow/templates/* your-project/templates/
   ```

2. **Modify for your needs:**
   - Update package names
   - Add your error handling patterns
   - Include team-specific logging
   - Add analytics integration

3. **Create team skill:**
   - Fork this skill
   - Add your customizations
   - Install team version for all developers

---

## 📊 Verification

### Check AI Agent Has Access

Ask your AI:
```
Do you have access to the android-production-workflow skill? 
What templates are available?
```

Expected response:
```
Yes, I have access to android-production-workflow skill (v1.0.0).

Available templates:
- build.gradle.kts.template (Gradle configuration)
- Theme.kt.template (Material 3 theme)
- ViewModel.kt.template (StateFlow ViewModel)
- Repository.kt.template (Repository with caching)
- Screen.kt.template (Compose screen)

Documentation:
- SKILL.md (640 lines of patterns and best practices)
- QuickStartGuide.md (462 lines step-by-step tutorial)
- UserManagementExample.md (621 lines complete example)
```

---

## 🔄 Workflow Integration

### In Daily Development

1. **Starting a feature:**
   - Describe the feature to AI
   - Mention "use android-production-workflow patterns"
   - AI generates complete structure

2. **Code review:**
   - Ask AI to review against skill patterns
   - AI checks for best practices compliance
   - Suggests improvements

3. **Refactoring:**
   - Point AI to legacy code
   - Request modernization with skill patterns
   - AI applies StateFlow, Compose, Hilt, etc.

### Example Daily Workflow

```
Morning:
User: "Create a new Orders feature with list and detail screens"
AI: [Uses skill] Generates domain/data/presentation layers

Afternoon:
User: "Review my OrderViewModel for best practices"
AI: [Uses skill] Checks StateFlow, error handling, suggests improvements

Evening:
User: "Add unit tests for OrderViewModel"
AI: [Uses skill] Generates tests following patterns from SKILL.md
```

---

## 🚀 Advanced Usage

### Combining with Other Skills

```
Use android-production-workflow for architecture
+ android-jetpack-compose for advanced UI patterns
+ android-design-guidelines for accessibility
+ android-kotlin-coroutines for complex async flows
```

### Custom Prompt Templates

Create reusable prompts:

**"feature-scaffold":**
```
Create a {{FEATURE_NAME}} feature following android-production-workflow:
- Domain layer: {{FEATURE_NAME}} model, repository interface, use case
- Data layer: API, DTO, mapper, repository impl with caching
- Presentation: ViewModel with StateFlow, Compose screen with Material 3
- Hilt dependency injection setup
```

---

## 📚 Learning Resources in Skill

| Resource | Lines | Purpose |
|----------|-------|---------|
| SKILL.md | 640 | Core patterns, architecture, best practices |
| QuickStartGuide.md | 462 | Step-by-step from zero to running app |
| UserManagementExample.md | 621 | Complete working CRUD example |
| Templates | 5 files | Ready-to-use code scaffolding |

---

## 💡 Tips for Best Results

1. **Be specific with prompts:**
   ❌ "Make an Android app"
   ✅ "Create a task management app with offline support using android-production-workflow patterns"

2. **Reference the skill explicitly:**
   ❌ "Add a ViewModel"
   ✅ "Add a ViewModel following android-production-workflow StateFlow patterns"

3. **Request reviews:**
   - "Review this against android-production-workflow best practices"
   - AI will cite specific sections from SKILL.md

4. **Use examples:**
   - "Like the UserManagementExample but for Products"
   - AI adapts the example to your needs

---

## 🎓 Next Steps

1. **Try the Quick Start:**
   - Follow `docs/QuickStartGuide.md`
   - Build the example project

2. **Study Patterns:**
   - Read `SKILL.md` sections relevant to your task
   - Reference when asking AI for help

3. **Build Real Features:**
   - Apply patterns to your actual project
   - Ask AI to follow skill patterns

4. **Contribute:**
   - Add your own templates
   - Share team-specific patterns
   - Submit improvements

---

**Happy Coding with Production-Ready Android Patterns!** 🚀
