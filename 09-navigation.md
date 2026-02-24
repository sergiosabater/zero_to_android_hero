





@Composable
fun ProfileScreen(onNavigateBack: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            "Profile Screen",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Button(onClick = onNavigateBack) {
            Text("Go Back")
        }
    }
}
```

**What's happening:**
1. `rememberNavController()` — Creates the navigation controller
2. `NavHost` — Container for all your screens
3. `composable("route")` — Defines each screen
4. `navController.navigate("route")` — Navigate to a screen
5. `navController.popBackStack()` — Go back

</details>

---

<br>

## 📦 Part 2 · Passing Data Between Screens

<div align="center">

### *The Real Power of Navigation*

Most apps need to pass data: user IDs, product details, order info.

</div>

---

<br>

### 🎯 Method 1: Route Arguments

<br>

<details>
<summary><b>🔗 Passing Data in the Route</b></summary>

<br>

```kotlin
@Composable
fun NavigationWithArguments() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "user_list"
    ) {
        // Screen 1: User List
        composable("user_list") {
            UserListScreen(
                onUserClick = { userId ->
                    // Navigate with argument in route
                    navController.navigate("user_detail/$userId")
                }
            )
        }
        
        // Screen 2: User Detail
        composable(
            route = "user_detail/{userId}",  // Define parameter
            arguments = listOf(
                navArgument("userId") {
                    type = NavType.IntType  // Specify type
                }
            )
        ) { backStackEntry ->
            // Retrieve the argument
            val userId = backStackEntry.arguments?.getInt("userId") ?: 0
            
            UserDetailScreen(
                userId = userId,
                onNavigateBack = { navController.popBackStack() }
            )
        }
    }
}

@Composable
fun UserListScreen(onUserClick: (Int) -> Unit) {
    val users = listOf(
        User(1, "Alice"),
        User(2, "Bob"),
        User(3, "Charlie")
    )
    
    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(users) { user ->
            Card(
                modifier = Modifier.fillMaxWidth(),
                onClick = { onUserClick(user.id) }
            ) {
                Text(
                    text = user.name,
                    modifier = Modifier.padding(16.dp),
                    style = MaterialTheme.typography.titleMedium
                )
            }
        }
    }
}

@Composable
fun UserDetailScreen(userId: Int, onNavigateBack: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text(
            "User Details",
            style = MaterialTheme.typography.headlineMedium
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Text("User ID: $userId")
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Button(onClick = onNavigateBack) {
            Text("Back to List")
        }
    }
}

data class User(val id: Int, val name: String)
```

**Argument types supported:**
- `NavType.IntType` → Int
- `NavType.LongType` → Long
- `NavType.FloatType` → Float
- `NavType.StringType` → String
- `NavType.BoolType` → Boolean

</details>

---

<br>

### 🎯 Method 2: Optional Arguments

<br>

<details>
<summary><b>🔗 With Default Values</b></summary>

<br>

```kotlin
composable(
    route = "product_detail/{productId}?category={category}",
    arguments = listOf(
        navArgument("productId") {
            type = NavType.IntType
        },
        navArgument("category") {
            type = NavType.StringType
            defaultValue = "all"  // Optional with default
            nullable = true
        }
    )
) { backStackEntry ->
    val productId = backStackEntry.arguments?.getInt("productId") ?: 0
    val category = backStackEntry.arguments?.getString("category") ?: "all"
    
    ProductDetailScreen(productId, category)
}

// Navigate
navController.navigate("product_detail/42")  // category = "all"
navController.navigate("product_detail/42?category=electronics")  // specified
```

</details>

---

<br>

### 🎯 Method 3: Complex Objects (Serialization)

<br>

<details>
<summary><b>📦 Passing Custom Objects</b></summary>

<br>

For complex data, encode to JSON and pass as string:

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json
import java.net.URLEncoder
import java.net.URLDecoder
import java.nio.charset.StandardCharsets

@Serializable
data class Order(
    val id: Int,
    val items: List<String>,
    val total: Double
)

// Sending screen
fun navigateToSummary(navController: NavController, order: Order) {
    val orderJson = Json.encodeToString(order)
    val encodedJson = URLEncoder.encode(orderJson, StandardCharsets.UTF_8.toString())
    navController.navigate("order_summary/$encodedJson")
}

// Receiving screen
composable(
    route = "order_summary/{orderJson}",
    arguments = listOf(
        navArgument("orderJson") { type = NavType.StringType }
    )
) { backStackEntry ->
    val encodedJson = backStackEntry.arguments?.getString("orderJson") ?: ""
    val orderJson = URLDecoder.decode(encodedJson, StandardCharsets.UTF_8.toString())
    val order = Json.decodeFromString<Order>(orderJson)
    
    OrderSummaryScreen(order)
}
```

> [!WARNING]
> For very large objects, consider using a ViewModel shared between screens instead.

</details>

---

<br>

## ⬅️ Part 3 · Back Stack Management

<br>

<details>
<summary><b>🔙 Controlling Navigation History</b></summary>

<br>

```kotlin
// ══════════════════════════════════════════════
// Navigate normally (adds to back stack)
// ══════════════════════════════════════════════
navController.navigate("profile")
// Stack: Home → Profile

navController.navigate("settings")
// Stack: Home → Profile → Settings

navController.popBackStack()
// Stack: Home → Profile ← Back to this

// ══════════════════════════════════════════════
// Navigate and clear back stack
// ══════════════════════════════════════════════
navController.navigate("home") {
    popUpTo("home") { inclusive = true }
}
// Stack: Home (everything else removed)
// Useful for: Login → Home (can't go back to login)

// ══════════════════════════════════════════════
// Navigate and remove previous screen
// ══════════════════════════════════════════════
navController.navigate("confirmation") {
    popUpTo("cart") { inclusive = true }
}
// Cart → Checkout → Confirmation
//                   ← Back goes to Cart, skips Checkout
// Useful for: Payment confirmation (can't go back to payment)

// ══════════════════════════════════════════════
// Single top (don't duplicate if already there)
// ══════════════════════════════════════════════
navController.navigate("home") {
    launchSingleTop = true
}
// If already on Home, don't add another Home to stack

// ══════════════════════════════════════════════
// Check if you can go back
// ══════════════════════════════════════════════
if (navController.previousBackStackEntry != null) {
    navController.popBackStack()
} else {
    // Already at root, maybe exit app
}

// ══════════════════════════════════════════════
// Handle system back button
// ══════════════════════════════════════════════
BackHandler {
    if (navController.previousBackStackEntry != null) {
        navController.popBackStack()
    } else {
        // Exit app or show confirmation
    }
}
```

**Common patterns:**

| Scenario | Code |
|:---|:---|
| **Normal navigation** | `navigate("screen")` |
| **Go back** | `popBackStack()` |
| **Login → Home (clear history)** | `navigate("home") { popUpTo(0) }` |
| **Payment done (can't go back)** | `navigate("success") { popUpTo("cart") { inclusive = true } }` |
| **Tab bar navigation** | `navigate("tab") { launchSingleTop = true }` |

</details>

---

<br>

## 🧁 Part 4 · Project — Cupcake Order App

<div align="center">

### *Multi-Screen Ordering Flow*

Let's build a complete **Cupcake Order App** with navigation!

</div>

<br>

<table>
<tr>
<td align="center" width="25%">

🧁  
**Start**

Choose quantity

</td>
<td align="center" width="25%">

🎨  
**Flavor**

Select flavor

</td>
<td align="center" width="25%">

📅  
**Pickup**

Choose date

</td>
<td align="center" width="25%">

📝  
**Summary**

Review & order

</td>
</tr>
</table>

---

<br>

### 🎯 App Flow

<br>

```
┌──────────────┐
│  Start       │  Choose: 1, 6, or 12 cupcakes
│  🧁          │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│  Flavor      │  Choose: Vanilla, Chocolate, etc.
│  🎨          │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│  Pickup Date │  Select pickup day
│  📅          │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│  Summary     │  Review order & confirm
│  📝 ✅       │
└──────────────┘
```

---

<br>

<details>
<summary><b>🧁 Complete Cupcake Order App Code</b></summary>

<br>

**Create new project:**
- Name: `CupcakeApp`
- Package: `com.yourname.cupcake`
- Language: Kotlin · Minimum SDK: API 24

**Add Navigation dependency in `build.gradle.kts`:**
```kotlin
implementation("androidx.navigation:navigation-compose:2.7.7")
```

<br>

**Step 1: Create data model**

Create file: `OrderUiState.kt`

```kotlin
package com.yourname.cupcake

data class OrderUiState(
    val quantity: Int = 0,
    val flavor: String = "",
    val date: String = "",
    val price: String = "$0.00"
)
```

**Step 2: Create ViewModel**

Create file: `OrderViewModel.kt`

```kotlin
package com.yourname.cupcake

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import java.text.NumberFormat
import java.text.SimpleDateFormat
import java.util.*

private const val PRICE_PER_CUPCAKE = 2.00

class OrderViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(OrderUiState())
    val uiState: StateFlow<OrderUiState> = _uiState.asStateFlow()
    
    fun setQuantity(numberCupcakes: Int) {
        _uiState.update { currentState ->
            currentState.copy(
                quantity = numberCupcakes,
                price = calculatePrice(quantity = numberCupcakes)
            )
        }
    }
    
    fun setFlavor(desiredFlavor: String) {
        _uiState.update { currentState ->
            currentState.copy(flavor = desiredFlavor)
        }
    }
    
    fun setDate(pickupDate: String) {
        _uiState.update { currentState ->
            currentState.copy(date = pickupDate)
        }
    }
    
    fun resetOrder() {
        _uiState.value = OrderUiState()
    }
    
    private fun calculatePrice(quantity: Int = _uiState.value.quantity): String {
        val calculatedPrice = quantity * PRICE_PER_CUPCAKE
        return NumberFormat.getCurrencyInstance().format(calculatedPrice)
    }
    
    fun dateOptions(): List<String> {
        val dateOptions = mutableListOf<String>()
        val formatter = SimpleDateFormat("E MMM d", Locale.getDefault())
        val calendar = Calendar.getInstance()
        
        // Add today + next 3 days
        repeat(4) {
            dateOptions.add(formatter.format(calendar.time))
            calendar.add(Calendar.DATE, 1)
        }
        
        return dateOptions
    }
}
```

**Step 3: Create Navigation**

Create file: `CupcakeScreen.kt`

```kotlin
package com.yourname.cupcake

enum class CupcakeScreen {
    Start,
    Flavor,
    Pickup,
    Summary
}
```

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
