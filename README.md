// Android Kotlin - MainActivity.kt
package com.example.queenapp

import android.os.Bundle
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.google.firebase.database.ktx.database
import com.google.firebase.ktx.Firebase
import com.example.queenapp.ui.theme.QueenappTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val database = Firebase.database
        val ledRef = database.getReference("Led/status")
        val tempRef = database.getReference("Sensor/temperature")
        val humRef = database.getReference("Sensor/humidity")

        setContent {
            QueenappTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    ControlScreen(
                        onLedChange = { isOn -> ledRef.setValue(isOn) },
                        onRefresh = {
                            tempRef.get().addOnSuccessListener { temperature ->
                                temperature.getValue(Double::class.java)?.let {
                                    Toast.makeText(this, "Nhiệt độ: $it°C", Toast.LENGTH_SHORT).show()
                                }
                            }
                            humRef.get().addOnSuccessListener { humidity ->
                                humidity.getValue(Double::class.java)?.let {
                                    Toast.makeText(this, "Độ ẩm: $it%", Toast.LENGTH_SHORT).show()
                                }
                            }
                        }
                    )
                }
            }
        }
    }
}

@Composable
fun ControlScreen(onLedChange: (Boolean) -> Unit, onRefresh: () -> Unit) {
    var isLedOn by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("Điều khiển ESP32", fontSize = 24.sp)
        Spacer(modifier = Modifier.height(16.dp))

        Row(verticalAlignment = Alignment.CenterVertically) {
            Text("Bật/Tắt đèn", fontSize = 20.sp)
            Spacer(modifier = Modifier.width(8.dp))
            Switch(
                checked = isLedOn,
                onCheckedChange = {
                    isLedOn = it
                    onLedChange(it)
                }
            )
        }

        Spacer(modifier = Modifier.height(32.dp))

        Button(
            onClick = onRefresh,
            colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF4CAF50))
        ) {
            Text("Làm mới dữ liệu", fontSize = 18.sp, color = Color.White)
        }
    }
}
