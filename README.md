# 🎬 Movie Watchlist App

A simple Android Movie Watchlist application built with **Kotlin** and **Jetpack Compose**. Users can add movies to their watchlist and remove them. The screen updates automatically whenever the watchlist changes.

## 📱 App Screenshots

### 1. Initial screen
The app starts with Avengers and Titanic in the watchlist. Avatar, Joker, and Inception are available to add.

![Initial Movie Watchlist](screenshots/watchlist-initial-state.webp)

### 2. After adding a movie
Avatar is added to the watchlist. The total movie count updates automatically.

![Watchlist after adding Avatar](screenshots/watchlist-after-adding-avatar.webp)

### 3. After removing movies
Movies can be removed from the watchlist, and they return to the Available Movies section.

![Watchlist after removing movies](screenshots/watchlist-after-removing-movies.webp)

## ✨ Features

- View available movies.
- Add a movie to the watchlist.
- Remove a movie from the watchlist.
- Display the current number of watchlisted movies.
- Automatically update the UI when the watchlist changes.
- Prevent duplicate movies from being added.
- Use **MVVM Architecture**, **Repository Pattern**, and **Single Source of Truth (SSOT)**.

## 🧰 Tech Stack

- Kotlin
- Android Studio
- Jetpack Compose
- Material 3
- AndroidX ViewModel
- Kotlin `StateFlow`

## 🏗️ Architecture

```text
WatchlistScreen (Jetpack Compose UI)
             |
             v
       MovieViewModel
             |
             v
       MovieRepository
             |
             v
   MutableStateFlow<List<Movie>>
       (Single Source of Truth)
```

- **Model:** `Movie` data class stores movie details.
- **View:** `WatchlistScreen` displays available movies and the user's watchlist.
- **ViewModel:** `MovieViewModel` exposes the watchlist and handles UI actions.
- **Repository:** `MovieRepository` owns and updates the watchlist data.
- **StateFlow:** Emits the latest watchlist whenever a movie is added or removed.

## 🚀 How to Run

1. Open Android Studio.
2. Create or open an Android project using **Kotlin** and **Jetpack Compose**.
3. Set the package name to `com.example.moviewatchlist`, or update the package declaration in the code to match your project.
4. Add the dependencies listed below to the app-level `build.gradle.kts`.
5. Replace the contents of `MainActivity.kt` with the code in the **Complete Source Code** section.
6. Sync Gradle and run the app on an emulator or Android device.

## 📦 Dependencies

Add these dependencies inside the `dependencies { }` block in `app/build.gradle.kts`. If your project already has compatible versions, keep the versions managed by your project or version catalog.

```kotlin
dependencies {
    implementation("androidx.activity:activity-compose:1.11.0")

    implementation(platform("androidx.compose:compose-bom:2025.10.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.foundation:foundation")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-core")

    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.9.4")

    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.10.2")
}
```

## 💻 Complete Source Code

**File:** `MainActivity.kt`

The complete implementation is provided below in one Kotlin file. It includes the data class, repository, ViewModel, Activity, and Compose screen.

```kotlin
package com.example.moviewatchlist

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material3.Card
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

// 1. DATA CLASS
data class Movie(
    val id: Int,
    val name: String,
    val genre: String
)

// 2. REPOSITORY
class MovieRepository {

    val availableMovies = listOf(
        Movie(1, "Avengers", "Action"),
        Movie(2, "Titanic", "Romance"),
        Movie(3, "Avatar", "Sci-Fi"),
        Movie(4, "Joker", "Drama"),
        Movie(5, "Inception", "Thriller")
    )

    private val _watchlist = MutableStateFlow(
        listOf(
            availableMovies[0],
            availableMovies[1]
        )
    )

    val watchlist: StateFlow<List<Movie>> = _watchlist

    fun addMovie(movie: Movie) {
        if (_watchlist.value.none { it.id == movie.id }) {
            _watchlist.value = _watchlist.value + movie
        }
    }

    fun removeMovie(movie: Movie) {
        _watchlist.value =
            _watchlist.value.filter { it.id != movie.id }
    }
}

// 3. VIEWMODEL
class MovieViewModel : ViewModel() {

    private val repository = MovieRepository()

    val watchlist = repository.watchlist
    val availableMovies = repository.availableMovies

    fun addMovie(movie: Movie) {
        repository.addMovie(movie)
    }

    fun removeMovie(movie: Movie) {
        repository.removeMovie(movie)
    }
}

// 4. MAIN ACTIVITY
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        enableEdgeToEdge()

        setContent {
            MaterialTheme {
                WatchlistScreen()
            }
        }
    }
}

// 5. WATCHLIST SCREEN
@Composable
fun WatchlistScreen(
    vm: MovieViewModel = viewModel()
) {
    val watchlist by vm.watchlist.collectAsState()

    val available = vm.availableMovies.filter { movie ->
        watchlist.none { it.id == movie.id }
    }

    Scaffold { innerPadding ->

        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
                .padding(horizontal = 16.dp),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {

            item {
                Text(
                    text = "My Movie Watchlist",
                    style = MaterialTheme.typography.headlineMedium
                )

                Text(
                    text = "Total Movies: ${watchlist.size}",
                    style = MaterialTheme.typography.titleMedium
                )
            }

            item {
                Text(
                    text = "Available Movies",
                    style = MaterialTheme.typography.titleLarge
                )
            }

            items(available, key = { it.id }) { movie ->

                Card(
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        horizontalArrangement = Arrangement.SpaceBetween,
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column(
                            modifier = Modifier.weight(1f)
                        ) {
                            Text(
                                text = movie.name,
                                style = MaterialTheme.typography.titleMedium
                            )

                            Text(
                                text = movie.genre,
                                style = MaterialTheme.typography.bodyMedium,
                                color = MaterialTheme.colorScheme.onSurfaceVariant
                            )
                        }

                        IconButton(
                            onClick = { vm.addMovie(movie) }
                        ) {
                            Icon(
                                imageVector = Icons.Filled.Add,
                                contentDescription = "Add to Watchlist",
                                tint = MaterialTheme.colorScheme.primary
                            )
                        }
                    }
                }
            }

            item {
                Text(
                    text = "My Watchlist",
                    style = MaterialTheme.typography.titleLarge
                )
            }

            if (watchlist.isEmpty()) {
                item {
                    Text("No movies in your watchlist")
                }
            }

            items(watchlist, key = { it.id }) { movie ->

                Card(
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        horizontalArrangement = Arrangement.SpaceBetween,
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column(
                            modifier = Modifier.weight(1f)
                        ) {
                            Text(
                                text = movie.name,
                                style = MaterialTheme.typography.titleMedium
                            )

                            Text(
                                text = movie.genre,
                                style = MaterialTheme.typography.bodyMedium,
                                color = MaterialTheme.colorScheme.onSurfaceVariant
                            )
                        }

                        IconButton(
                            onClick = { vm.removeMovie(movie) }
                        ) {
                            Icon(
                                imageVector = Icons.Filled.Delete,
                                contentDescription = "Remove Movie",
                                tint = MaterialTheme.colorScheme.error
                            )
                        }
                    }
                }
            }
        }
    }
}
```

## 🔄 How Automatic UI Updates Work

### Adding a movie

1. The user taps the **+** icon.
2. `WatchlistScreen` calls `vm.addMovie(movie)`.
3. `MovieViewModel` calls `repository.addMovie(movie)`.
4. The repository updates `_watchlist`.
5. `StateFlow` emits the new list.
6. `collectAsState()` observes the new value and Compose recomposes the screen.

### Removing a movie

1. The user taps the delete icon.
2. `WatchlistScreen` calls `vm.removeMovie(movie)`.
3. The ViewModel calls the repository.
4. The repository removes the movie from `_watchlist`.
5. `StateFlow` emits the updated list.
6. The movie disappears from the watchlist and becomes available to add again.

## 📚 Key Concepts

| Concept | Use in this project |
|---|---|
| MVVM | Separates UI, state handling, and data management |
| Repository Pattern | `MovieRepository` manages watchlist data |
| Single Source of Truth | The repository's `_watchlist` is the only watchlist state |
| StateFlow | Publishes updated watchlist data |
| `collectAsState()` | Observes StateFlow in Compose |
| LazyColumn | Displays the movie lists efficiently |
| `addMovie()` | Adds a movie without duplicates |
| `removeMovie()` | Removes a movie from the watchlist |

## 📝 Note

This academic implementation stores the watchlist in memory using `MutableStateFlow`. The data is not persisted in a database, so the initial watchlist is restored when the app process is recreated. Database persistence was not included in the given task requirements.
