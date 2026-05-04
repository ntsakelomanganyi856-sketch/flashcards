package com.example.flashcards

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.flashcards.ui.theme.FlashcardsTheme

/**
 * Data model for a Life Hack flashcard.
 */
data class LifeHack(
    val statement: String,
    val isHack: Boolean,
    val explanation: String
)

/**
 * Sample list of life hacks and myths.
 */
val lifeHacks = listOf(
    LifeHack(
        "A wooden spoon over a boiling pot prevents it from boiling over.",
        true,
        "True! The spoon breaks the bubbles' surface tension and is a poor heat conductor."
    ),
    LifeHack(
        "Putting a wet phone in rice is the most effective way to dry it.",
        false,
        "Myth. Rice can actually slow down the drying process. Silica gel or air-drying is better."
    ),
    LifeHack(
        "You can use toothpaste to clear up cloudy car headlights.",
        true,
        "True! The mild abrasives in toothpaste help polish the plastic surface."
    ),
    LifeHack(
        "Batteries last longer if you store them in the freezer.",
        false,
        "Myth. Cold and moisture can actually damage batteries and lead to corrosion."
    ),
    LifeHack(
        "Using a binder clip can help organize messy cables on your desk.",
        true,
        "True! They are a cheap and effective way to keep cables in place."
    )
)

enum class AppScreen {
    Welcome, Quiz, Result, Review
}

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            FlashcardsTheme {
                LifeHackApp()
            }
        }
    }
}

@Composable
fun LifeHackApp() {
    var currentScreen by remember { mutableStateOf(AppScreen.Welcome) }
    var score by remember { mutableIntStateOf(0) }
    var currentIndex by remember { mutableIntStateOf(0) }

    Surface(
        modifier = Modifier.fillMaxSize(),
        color = MaterialTheme.colorScheme.background
    ) {
        when (currentScreen) {
            AppScreen.Welcome -> WelcomeScreen(onStart = {
                score = 0
                currentIndex = 0
                currentScreen = AppScreen.Quiz
            })
            AppScreen.Quiz -> QuizScreen(
                hack = lifeHacks[currentIndex],
                onAnswer = { isHackSelected ->
                    if (isHackSelected == lifeHacks[currentIndex].isHack) {
                        score++
                    }
                    if (currentIndex < lifeHacks.size - 1) {
                        currentIndex++
                    } else {
                        currentScreen = AppScreen.Result
                    }
                }
            )
            AppScreen.Result -> ResultScreen(
                score = score,
                total = lifeHacks.size,
                onReview = { currentScreen = AppScreen.Review },
                onRestart = { currentScreen = AppScreen.Welcome }
            )
            AppScreen.Review -> ReviewScreen(
                onBack = { currentScreen = AppScreen.Result }
            )
        }
    }
}

@Composable
fun WelcomeScreen(onStart: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Hack or Myth?",
            style = MaterialTheme.typography.headlineLarge,
            fontWeight = FontWeight.Bold,
            color = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = "Test your knowledge of common life hacks! This app helps you verify popular tips and learn the truth behind them.",
            style = MaterialTheme.typography.bodyLarge,
            textAlign = TextAlign.Center
        )
        Spacer(modifier = Modifier.height(48.dp))
        Button(
            onClick = onStart,
            modifier = Modifier.fillMaxWidth().height(56.dp)
        ) {
            Text("Start Quiz", fontSize = 18.sp)
        }
    }
}

@Composable
fun QuizScreen(hack: LifeHack, onAnswer: (Boolean) -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .height(280.dp),
            elevation = CardDefaults.cardElevation(defaultElevation = 8.dp)
        ) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(24.dp),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    text = hack.statement,
                    style = MaterialTheme.typography.headlineSmall,
                    textAlign = TextAlign.Center,
                    lineHeight = 32.sp
                )
            }
        }
        Spacer(modifier = Modifier.height(48.dp))
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Button(
                onClick = { onAnswer(true) },
                modifier = Modifier.weight(1f).height(56.dp),
                colors = ButtonDefaults.buttonColors(containerColor = MaterialTheme.colorScheme.primary)
            ) {
                Text("Hack (True)")
            }
            Button(
                onClick = { onAnswer(false) },
                modifier = Modifier.weight(1f).height(56.dp),
                colors = ButtonDefaults.buttonColors(containerColor = MaterialTheme.colorScheme.secondary)
            ) {
                Text("Myth (False)")
            }
        }
    }
}

@Composable
fun ResultScreen(score: Int, total: Int, onReview: () -> Unit, onRestart: () -> Unit) {
    val feedback = when {
        score == total -> "Perfect! You're a Life Hack Pro!"
        score >= total / 2 -> "Good job! You know your hacks."
        else -> "Keep learning! Some of those myths are tricky."
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Quiz Finished!",
            style = MaterialTheme.typography.headlineMedium,
            fontWeight = FontWeight.Bold
        )
        Spacer(modifier = Modifier.height(8.dp))
        Text(
            text = "Total Correct: $score / $total",
            style = MaterialTheme.typography.headlineSmall,
            color = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = feedback,
            style = MaterialTheme.typography.bodyLarge,
            textAlign = TextAlign.Center
        )
        Spacer(modifier = Modifier.height(48.dp))
        Button(
            onClick = onReview,
            modifier = Modifier.fillMaxWidth().height(56.dp)
        ) {
            Text("Review Statements")
        }
        Spacer(modifier = Modifier.height(12.dp))
        OutlinedButton(
            onClick = onRestart,
            modifier = Modifier.fillMaxWidth().height(56.dp)
        ) {
            Text("Try Again")
        }
    }
}

@Composable
fun ReviewScreen(onBack: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
            .statusBarsPadding()
    ) {
        Row(
            modifier = Modifier.fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text(
                text = "Review",
                style = MaterialTheme.typography.headlineMedium,
                fontWeight = FontWeight.Bold
            )
            TextButton(onClick = onBack) {
                Text("Back")
            }
        }
        Spacer(modifier = Modifier.height(16.dp))
        LazyColumn(
            modifier = Modifier.fillMaxSize(),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            items(lifeHacks) { hack ->
                Card(
                    modifier = Modifier.fillMaxWidth(),
                    colors = CardDefaults.cardColors(
                        containerColor = if (hack.isHack) 
                            MaterialTheme.colorScheme.primaryContainer 
                        else 
                            MaterialTheme.colorScheme.secondaryContainer
                    )
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text(
                            text = if (hack.isHack) "HACK" else "MYTH",
                            style = MaterialTheme.typography.labelLarge,
                            fontWeight = FontWeight.ExtraBold,
                            color = if (hack.isHack) 
                                MaterialTheme.colorScheme.onPrimaryContainer 
                            else 
                                MaterialTheme.colorScheme.onSecondaryContainer
                        )
                        Spacer(modifier = Modifier.height(4.dp))
                        Text(
                            text = hack.statement,
                            style = MaterialTheme.typography.bodyLarge,
                            fontWeight = FontWeight.Medium
                        )
                        Spacer(modifier = Modifier.height(8.dp))
                        HorizontalDivider(
                            modifier = Modifier.padding(vertical = 4.dp),
                            thickness = 1.dp,
                            color = MaterialTheme.colorScheme.outlineVariant
                        )
                        Spacer(modifier = Modifier.height(4.dp))
                        Text(
                            text = hack.explanation,
                            style = MaterialTheme.typography.bodyMedium,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
            }
        }
    }
}
