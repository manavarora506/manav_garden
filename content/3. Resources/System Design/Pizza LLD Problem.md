
Designing a **Pizza Ordering System** is a classic **low-level design (LLD)** problem often asked in Amazon SDE 1 interviews. It tests your ability to:

- Break a real-world domain into clean classes/interfaces
    
- Apply OOP principles (Encapsulation, Inheritance, Composition)
    
- Handle extensibility (e.g., customizations, offers, multiple stores)
    
- Think about system boundaries (UI, DB, backend services)
    

---

## 🍕 Problem Prompt (Typical Version)

> Design the backend of a pizza ordering system. Customers can place orders with multiple pizzas, each with different toppings and sizes. There are fixed prices for base pizzas and additional prices for toppings. The system should allow tracking orders and calculating the final price.

---

## 🧠 Step 1: Clarify Requirements

**Functional Requirements:**

- Add pizzas to an order
    
- Choose pizza size (S, M, L) and base type
    
- Add toppings (customizable per pizza)
    
- Calculate price per pizza and for the whole order
    
- Track order status (e.g., PLACED, PREPARING, DELIVERED)
    

**Optional Enhancements:**

- Offers/coupons
    
- Payment
    
- Multiple stores
    
- Delivery tracking
    

---

## 🧱 Step 2: Identify Core Entities

|Entity|Description|
|---|---|
|`Topping`|Name + price|
|`Pizza`|Size, base type, list of toppings, price|
|`Order`|List of pizzas, total price, order status|
|`Customer`|(Optional) name, contact info|

---

## 🧩 Step 3: Define Class Diagram (Simplified)

```
Order
│
├── List<Pizza>
│   └── List<Topping>
```

---

## 🐍 Step 4: Python Code (LLD Style)

### `topping.py`

```python
class Topping:
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

    def __str__(self):
        return f"{self.name} (${self.price})"
```

---

### `pizza.py`

```python
class Pizza:
    BASE_PRICES = {"small": 5.0, "medium": 7.0, "large": 9.0}

    def __init__(self, size: str):
        if size not in self.BASE_PRICES:
            raise ValueError("Invalid pizza size")

        self.size = size
        self.toppings = []

    def add_topping(self, topping):
        self.toppings.append(topping)

    def calculate_price(self):
        base_price = self.BASE_PRICES[self.size]
        toppings_price = sum(t.price for t in self.toppings)
        return base_price + toppings_price

    def __str__(self):
        toppings_str = ", ".join(str(t) for t in self.toppings)
        return f"{self.size.capitalize()} Pizza with [{toppings_str}] - ${self.calculate_price()}"
```

---

### `order.py`

```python
from enum import Enum

class OrderStatus(Enum):
    PLACED = "PLACED"
    PREPARING = "PREPARING"
    DELIVERED = "DELIVERED"

class Order:
    def __init__(self):
        self.pizzas = []
        self.status = OrderStatus.PLACED

    def add_pizza(self, pizza):
        self.pizzas.append(pizza)

    def calculate_total(self):
        return sum(p.calculate_price() for p in self.pizzas)

    def set_status(self, status):
        self.status = status

    def __str__(self):
        pizzas_str = "\n".join(str(p) for p in self.pizzas)
        return f"Order ({self.status.value}):\n{pizzas_str}\nTotal: ${self.calculate_total()}"
```

---

### ✅ Example Usage

```python
if __name__ == "__main__":
    cheese = Topping("Cheese", 1.0)
    olives = Topping("Olives", 1.5)
    pepperoni = Topping("Pepperoni", 2.0)

    pizza1 = Pizza("medium")
    pizza1.add_topping(cheese)
    pizza1.add_topping(pepperoni)

    pizza2 = Pizza("small")
    pizza2.add_topping(olives)

    order = Order()
    order.add_pizza(pizza1)
    order.add_pizza(pizza2)

    print(order)

    order.set_status(OrderStatus.DELIVERED)
    print(f"\nAfter delivery: {order.status}")
```

---

## 💡 Extensibility Ideas

|Feature|How you'd extend it|
|---|---|
|Offers / Coupons|Add `apply_coupon(coupon_code)` to `Order`|
|Multiple Store Support|Add `Store` class with menu & location|
|Inventory|Add a `Stock` manager per topping|
|Persistence|Add `OrderRepository`, store in DB|
|API Layer|Wrap with FastAPI for order placement|

---

## 📌 Interview Tips

- Start with classes: `Pizza`, `Topping`, `Order`
    
- Show composition: Pizza → List of Toppings, Order → List of Pizzas
    
- Use enums for state (order status)
    
- Keep methods clean: `calculate_price()`, `add_pizza()`
    
- Don’t forget to mention: extensibility (discounts, users, etc.)
    

---

Want me to mock out REST endpoints for this (like `POST /order`)? Or simulate applying coupons?


Great — let’s walk through how to **add coupon support** to your pizza ordering system. We’ll cover:

1. 🧱 **What a coupon is**
    
2. 🧠 **How to represent it (class design)**
    
3. 🧩 **How to integrate it into the order flow**
    
4. ✅ **Examples of usage**
    

---

## 1. 🧱 What is a Coupon?

At a basic level, a coupon has:

- A `code` (like `"WELCOME10"`)
    
- A `discount_type`: `%` off or flat $ amount
    
- A `value`: e.g., `10%` or `$5`
    
- Optional rules:
    
    - Min order value
        
    - Expiry date
        
    - One-time use
        

---

## 2. 🧠 Coupon Class

```python
from enum import Enum
from datetime import datetime

class DiscountType(Enum):
    PERCENTAGE = "percentage"
    FLAT = "flat"

class Coupon:
    def __init__(self, code: str, discount_type: DiscountType, value: float, min_order: float = 0.0, expires_at: datetime = None):
        self.code = code
        self.discount_type = discount_type
        self.value = value
        self.min_order = min_order
        self.expires_at = expires_at

    def is_valid(self, order_total: float) -> bool:
        if self.expires_at and datetime.now() > self.expires_at:
            return False
        if order_total < self.min_order:
            return False
        return True

    def apply_discount(self, order_total: float) -> float:
        if not self.is_valid(order_total):
            return order_total
        if self.discount_type == DiscountType.PERCENTAGE:
            return order_total * (1 - self.value / 100)
        elif self.discount_type == DiscountType.FLAT:
            return max(0, order_total - self.value)
        return order_total
```

---

## 3. 🧩 Update the Order Class

```python
class Order:
    def __init__(self):
        self.pizzas = []
        self.status = OrderStatus.PLACED
        self.coupon = None

    def add_pizza(self, pizza):
        self.pizzas.append(pizza)

    def apply_coupon(self, coupon: Coupon):
        total = self.calculate_total()
        if coupon.is_valid(total):
            self.coupon = coupon
        else:
            raise ValueError("Coupon is not valid")

    def calculate_total(self):
        subtotal = sum(p.calculate_price() for p in self.pizzas)
        if self.coupon:
            return round(self.coupon.apply_discount(subtotal), 2)
        return subtotal

    def __str__(self):
        pizzas_str = "\n".join(str(p) for p in self.pizzas)
        discount_str = f"Coupon Applied: {self.coupon.code}" if self.coupon else "No coupon"
        return f"Order ({self.status.value}):\n{pizzas_str}\n{discount_str}\nTotal: ${self.calculate_total()}"
```

---

## 4. ✅ Example Usage

```python
if __name__ == "__main__":
    from datetime import timedelta

    cheese = Topping("Cheese", 1.0)
    olives = Topping("Olives", 1.5)

    pizza = Pizza("large")
    pizza.add_topping(cheese)
    pizza.add_topping(olives)

    order = Order()
    order.add_pizza(pizza)

    # Create coupon: 10% off, min order $5, valid for 1 day
    welcome_coupon = Coupon(
        code="WELCOME10",
        discount_type=DiscountType.PERCENTAGE,
        value=10,
        min_order=5,
        expires_at=datetime.now() + timedelta(days=1)
    )

    order.apply_coupon(welcome_coupon)
    print(order)
```

---

## 🧠 Output (Example)

```
Order (PLACED):
Large Pizza with [Cheese ($1.0), Olives ($1.5)] - $11.5
Coupon Applied: WELCOME10
Total: $10.35
```

---

## ✅ Bonus Features You Can Add

|Feature|How|
|---|---|
|Coupon repository|Store/load from DB or JSON|
|One-time use|Track `used` flag per coupon|
|Multi-coupon logic|Add `List[Coupon]` and apply best or chain|
|Expiry email jobs|Add cron job that disables coupons after expiry|

---

Want me to add REST API endpoints for applying a coupon to an order (`POST /order/{id}/apply-coupon`)?