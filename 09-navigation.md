

    
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
