










    
    f


**Step 4: MainActivity.kt**

```kotlin
package com.yourname.cupcake

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.selection.selectable
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.ArrowBack
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                CupcakeApp()
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CupcakeApp(
    viewModel: OrderViewModel = viewModel(),
    navController: NavHostController = rememberNavController()
) {
    val backStackEntry by navController.currentBackStackEntryAsState()
    val currentScreen = CupcakeScreen.valueOf(
        backStackEntry?.destination?.route ?: CupcakeScreen.Start.name
    )
    
    Scaffold(
        topBar = {
            CupcakeAppBar(
                currentScreen = currentScreen,
                canNavigateBack = navController.previousBackStackEntry != null,
                navigateUp = { navController.navigateUp() }
            )
        }
    ) { innerPadding ->
        val uiState by viewModel.uiState.collectAsState()
        
        NavHost(
            navController = navController,
            startDestination = CupcakeScreen.Start.name,
            modifier = Modifier.padding(innerPadding)
        ) {
            composable(route = CupcakeScreen.Start.name) {
                StartOrderScreen(
                    quantityOptions = listOf(
                        Pair(R.string.one_cupcake, 1),
                        Pair(R.string.six_cupcakes, 6),
                        Pair(R.string.twelve_cupcakes, 12)
                    ),
                    onNextButtonClicked = { quantity ->
                        viewModel.setQuantity(quantity)
                        navController.navigate(CupcakeScreen.Flavor.name)
                    }
                )
            }
            
            composable(route = CupcakeScreen.Flavor.name) {
                SelectOptionScreen(
                    subtotal = uiState.price,
                    options = listOf("Vanilla", "Chocolate", "Red Velvet", "Salted Caramel", "Coffee"),
                    onSelectionChanged = { viewModel.setFlavor(it) },
                    onNextButtonClicked = { navController.navigate(CupcakeScreen.Pickup.name) },
                    onCancelButtonClicked = {
                        cancelOrderAndNavigateToStart(viewModel, navController)
                    }
                )
            }
            
            composable(route = CupcakeScreen.Pickup.name) {
                SelectOptionScreen(
                    subtotal = uiState.price,
                    options = viewModel.dateOptions(),
                    onSelectionChanged = { viewModel.setDate(it) },
                    onNextButtonClicked = { navController.navigate(CupcakeScreen.Summary.name) },
                    onCancelButtonClicked = {
                        cancelOrderAndNavigateToStart(viewModel, navController)
                    }
                )
            }
            
            composable(route = CupcakeScreen.Summary.name) {
                OrderSummaryScreen(
                    orderUiState = uiState,
                    onCancelButtonClicked = {
                        cancelOrderAndNavigateToStart(viewModel, navController)
                    },
                    onSendButtonClicked = { subject, summary ->
                        // In real app: share intent or API call
                        cancelOrderAndNavigateToStart(viewModel, navController)
                    }
                )
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CupcakeAppBar(
    currentScreen: CupcakeScreen,
    canNavigateBack: Boolean,
    navigateUp: () -> Unit,
    modifier: Modifier = Modifier
) {
    TopAppBar(
        title = { Text("Cupcake") },
        colors = TopAppBarDefaults.mediumTopAppBarColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer
        ),
        modifier = modifier,
        navigationIcon = {
            if (canNavigateBack) {
                IconButton(onClick = navigateUp) {
                    Icon(
                        imageVector = Icons.Filled.ArrowBack,
                        contentDescription = "Back"
                    )
                }
            }
        }
    )
}

private fun cancelOrderAndNavigateToStart(
    viewModel: OrderViewModel,
    navController: NavHostController
) {
    viewModel.resetOrder()
    navController.popBackStack(CupcakeScreen.Start.name, inclusive = false)
}

// ══════════════════════════════════════════════════════════
// SCREEN 1: START ORDER
// ══════════════════════════════════════════════════════════
@Composable
fun StartOrderScreen(
    quantityOptions: List<Pair<Int, Int>>,
    onNextButtonClicked: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Spacer(modifier = Modifier.height(16.dp))
        
        Text(
            text = "Order Cupcakes",
            style = MaterialTheme.typography.headlineMedium
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        quantityOptions.forEach { (stringResourceId, quantity) ->
            Button(
                onClick = { onNextButtonClicked(quantity) },
                modifier = Modifier.fillMaxWidth()
            ) {
                Text(getQuantityString(quantity))
            }
        }
    }
}

fun getQuantityString(quantity: Int): String = when (quantity) {
    1 -> "One Cupcake"
    6 -> "Six Cupcakes"
    12 -> "Twelve Cupcakes"
    else -> "$quantity Cupcakes"
}

// ══════════════════════════════════════════════════════════
// SCREEN 2 & 3: SELECT OPTION (used for Flavor and Pickup)
// ══════════════════════════════════════════════════════════
@Composable
fun SelectOptionScreen(
    subtotal: String,
    options: List<String>,
    onSelectionChanged: (String) -> Unit,
    onCancelButtonClicked: () -> Unit,
    onNextButtonClicked: () -> Unit,
    modifier: Modifier = Modifier
) {
    var selectedValue by remember { mutableStateOf("") }
    
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        options.forEach { item ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .selectable(
                        selected = selectedValue == item,
                        onClick = {
                            selectedValue = item
                            onSelectionChanged(item)
                        }
                    )
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                RadioButton(
                    selected = selectedValue == item,
                    onClick = {
                        selectedValue = item
                        onSelectionChanged(item)
                    }
                )
                Text(item, modifier = Modifier.padding(start = 8.dp))
            }
        }
        
        Divider(
            thickness = 1.dp,
            modifier = Modifier.padding(vertical = 16.dp)
        )
        
        Text(
            text = "Subtotal: $subtotal",
            style = MaterialTheme.typography.headlineSmall,
            fontWeight = FontWeight.Bold
        )
        
        Spacer(modifier = Modifier.weight(1f))
        
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            OutlinedButton(
                onClick = onCancelButtonClicked,
                modifier = Modifier.weight(1f)
            ) {
                Text("Cancel")
            }
            
            Button(
                onClick = onNextButtonClicked,
                enabled = selectedValue.isNotEmpty(),
                modifier = Modifier.weight(1f)
            ) {
                Text("Next")
            }
        }
    }
}

// ══════════════════════════════════════════════════════════
// SCREEN 4: ORDER SUMMARY
// ══════════════════════════════════════════════════════════
@Composable
fun OrderSummaryScreen(
    orderUiState: OrderUiState,
    onCancelButtonClicked: () -> Unit,
    onSendButtonClicked: (String, String) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text(
            text = "Order Summary",
            style = MaterialTheme.typography.headlineMedium
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Card(modifier = Modifier.fillMaxWidth()) {
            Column(modifier = Modifier.padding(16.dp)) {
                Text("Quantity: ${orderUiState.quantity} cupcakes")
                Text("Flavor: ${orderUiState.flavor}")
                Text("Pickup date: ${orderUiState.date}")
                
                Divider(modifier = Modifier.padding(vertical = 8.dp))
                
                Text(
                    text = "Total: ${orderUiState.price}",
                    style = MaterialTheme.typography.titleLarge,
                    fontWeight = FontWeight.Bold
                )
            }
        }
        
        Spacer(modifier = Modifier.weight(1f))
        
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            OutlinedButton(
                onClick = onCancelButtonClicked,
                modifier = Modifier.weight(1f)
            ) {
                Text("Cancel")
            }
            
            Button(
                onClick = {
                    onSendButtonClicked(
                        "New Cupcake Order",
                        "Order details:\nQuantity: ${orderUiState.quantity}\nFlavor: ${orderUiState.flavor}\nDate: ${orderUiState.date}\nTotal: ${orderUiState.price}"
                    )
                },
                modifier = Modifier.weight(1f)
            ) {
                Text("Send Order")
            }
        }
    }
}
```

**Step 5: Add string resources**

In `res/values/strings.xml`:
```xml
<resources>
    <string name="app_name">Cupcake</string>
    <string name="one_cupcake">One Cupcake</string>
    <string name="six_cupcakes">Six Cupcakes</string>
    <string name="twelve_cupcakes">Twelve Cupcakes</string>
</resources>
```

</details>

---

<br>

### 🎨 Features Breakdown

<br>

<details>
<summary><b>✨ What Makes This App Great</b></summary>

<br>

**1. ViewModel for Shared State:**
```kotlin
class OrderViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(OrderUiState())
    val uiState: StateFlow<OrderUiState> = _uiState.asStateFlow()
    // State survives navigation and configuration changes
}
```

**2. Navigation Flow:**
```kotlin
Start → (select quantity) → Flavor → Pickup → Summary
                ↓           ↓         ↓          ↓
             Cancel      Cancel   Cancel   Send/Cancel
                ↓           ↓         ↓          ↓
              Start       Start    Start     Start
```

**3. TopAppBar with Back:**
```kotlin
navigationIcon = {
    if (canNavigateBack) {
        IconButton(onClick = navigateUp) {
            Icon(Icons.Filled.ArrowBack, "Back")
        }
    }
}
```

**4. Reusable SelectOptionScreen:**
```kotlin
// Same screen used for both Flavor and Pickup
// Just pass different options!
```

**5. Price Calculation:**
```kotlin
private const val PRICE_PER_CUPCAKE = 2.00
private fun calculatePrice(quantity: Int): String {
    return NumberFormat.getCurrencyInstance().format(quantity * PRICE_PER_CUPCAKE)
}
```

</details>

---

<br>

## 🎯 Mission · Chapter 09

<div align="center">

### 💻 Master Navigation!

</div>

<br>

### Core Tasks:

- [ ] 🗺️ **Create NavHost** — 2-screen navigation
- [ ] 📦 **Pass data** — Navigate with arguments
- [ ] ⬅️ **Handle back** — popBackStack properly
- [ ] 🧁 **Build Cupcake App** — Complete 4-screen flow
- [ ] 🎨 **Add TopAppBar** — With back button
- [ ] ✅ **Test flow** — Complete an order end-to-end

<br>

<details>
<summary><b>⭐ Bonus Challenges</b></summary>

<br>

- [ ] 🎨 Add **order customization** (toppings, size)
- [ ] 💾 **Save draft** orders (persist to DataStore)
- [ ] 📧 Implement **real share** intent for order
- [ ] 🛒 Add **order history** screen
- [ ] 🎯 Add **favorites** (save common orders)
- [ ] 🌈 Add **themed cupcakes** (birthday, wedding, etc.)
- [ ] 📊 Add **analytics** (track which flavors are popular)

</details>

---

<br>

<div align="center">

## 🏆 Achievement Unlocked

### **The Navigator** 🧭

<br>

**You now understand:**

- Navigation Component in Jetpack Compose
- NavController and NavHost
- Passing data between screens
- Back stack management
- Multi-screen app architecture
- ViewModel for shared state
- TopAppBar with navigation
- Complete user flows

<br>

*You built a Cupcake Order App*  
*with 4 screens, data passing,*  
*proper navigation, and order state management.*  
**That's a real-world app architecture.**

<br>

![Achievement](https://img.shields.io/badge/🏆-Achievement_Unlocked-gold?style=for-the-badge)

</div>

---

<br>

<div align="center">

### 🎓 Remember This

> *"Navigation isn't just moving between screens.*  
> *It's about creating a coherent user journey.*  
> *Every tap, every back button, every confirmation*  
> *should feel natural and intuitive.*  
> *Master navigation, and your apps*  
> *will feel professional and polished."*

</div>

---

<br>

<div align="center">

### 🔜 What's Next?

In **Chapter 10**, we explore **Data Persistence** —  
saving data locally with Room Database.  
You'll build a **Task Manager App** that remembers everything!

</div>

<br>

<div align="center">

[![Continue](https://img.shields.io/badge/➡️_Continue_to_Chapter_10-Data_Persistence-success?style=for-the-badge&logo=android)](./10-data-persistence.md)

</div>

<br>
