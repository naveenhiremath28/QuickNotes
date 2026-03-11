## **Low Level Design**

https://codewitharyan.com/system-design/low-level-design

Intro - https://codewitharyan.com/tech-blogs/what-is-low-level-system-design
In **LLD (Low-Level Design)**, diagrams help you explain _how your system works internally_ — classes, interactions, and flows.
```
# 1️⃣ Class Diagram

### 👉 What it shows:

Structure of the system —  
Classes, attributes, methods, and relationships between classes.

Think of it as:

> “Blueprint of objects in your system”

### It contains:

- Class name
    
- Variables (attributes)
    
- Methods (functions)
    
- Relationships (Inheritance, Association, Composition, etc.)
```

```
### Example Classes:

User  
- id  
- name  
- email  
+ login()  
+ logout()  
  
Order  
- id  
- amount  
+ createOrder()  
+ cancelOrder()  
  
Payment  
- id  
- status  
+ processPayment()

### Relationships:

- User → places → Order
    
- Order → uses → Payment
    

---

### ✅ When to use:

- Designing system structure
    
- Explaining OOP concepts
    
- Interview LLD rounds
    
- Before writing code
```

```
# 2️⃣ Sequence Diagram

### 👉 What it shows:

How objects interact over time.

Think of it as:

> “Who calls whom and in what order?”

It focuses on:

- Method calls
    
- Execution order
    
- Request → Response flow
```

```
### Flow:

1. User → OrderService: createOrder()
    
2. OrderService → PaymentService: processPayment()
    
3. PaymentService → BankAPI
    
4. Response comes back
    
5. Order confirmed
    

Time flows from **top to bottom**

---

### ✅ When to use:

- Explaining API flow
    
- Explaining microservice interaction
    
- Understanding method call chains
    
- Debugging design issues
```

```
# 3️⃣ Activity Diagram

### 👉 What it shows:

Flow of logic or business process.

Think of it as:

> “Flowchart of the system logic”

It focuses on:

- Decisions (if/else)
    
- Loops
    
- Parallel flows
    
- Start and end

### Flow:

Start  
→ Enter Card Details  
→ Validate Card  
→ Is valid?  
  Yes → Deduct Money → Success  
  No → Show Error  
→ End

---

### ✅ When to use:

- Explaining business logic
    
- Writing complex conditional flows
    
- Designing workflows
    
- Understanding algorithm flow
```

# 🔥 Simple Comparison

|Diagram|Focus|Answers What?|
|---|---|---|
|Class Diagram|Structure|What are the objects?|
|Sequence Diagram|Interaction|How do objects talk?|
|Activity Diagram|Flow Logic|What is the process?|


## Low-Level Design (LLD) Interview Approach 
https://codewitharyan.com/tech-blogs/how-to-approach-lld-problems


https://codewitharyan.com/tech-blogs/solid-principles



```python
  
The goal: **Client code should not know which DB it is using**. It only asks the **factory** for a connection.

This pattern is very useful when configuration decides the database (MySQL, PostgreSQL, etc.).

---

# 1. Production-style Folder Structure

db_factory_example/  
│  
├── app.py  
├── config.py  
│  
├── db  
│   ├── __init__.py  
│   ├── factory.py  
│   ├── base.py  
│   └── connections  
│       ├── __init__.py  
│       ├── mysql_connection.py  
│       └── postgres_connection.py  
│  
└── requirements.txt

Explanation

|Folder|Purpose|
|---|---|
|db/base.py|Abstract base class for DB connection|
|db/connections|Implementations (MySQL/Postgres)|
|db/factory.py|Factory that returns correct DB|
|config.py|Configuration|
|app.py|Client code|

---

# 2. Base Interface (Abstract Class)

`db/base.py`

from abc import ABC, abstractmethod  
  
class DatabaseConnection(ABC):  
  
    @abstractmethod  
    def connect(self):  
        pass  
  
    @abstractmethod  
    def execute(self, query: str):  
        pass

This ensures **all databases follow same interface**.

---

# 3. MySQL Implementation

`db/connections/mysql_connection.py`

from db.base import DatabaseConnection  
  
  
class MySQLConnection(DatabaseConnection):  
  
    def connect(self):  
        print("Connecting to MySQL database")  
  
    def execute(self, query: str):  
        print(f"MySQL executing query: {query}")

---

# 4. PostgreSQL Implementation

`db/connections/postgres_connection.py`

from db.base import DatabaseConnection  
  
  
class PostgresConnection(DatabaseConnection):  
  
    def connect(self):  
        print("Connecting to PostgreSQL database")  
  
    def execute(self, query: str):  
        print(f"PostgreSQL executing query: {query}")

---

# 5. Factory Class

`db/factory.py`

from db.connections.mysql_connection import MySQLConnection  
from db.connections.postgres_connection import PostgresConnection  
  
  
class DBFactory:  
  
    @staticmethod  
    def create_connection(db_type: str):  
  
        if db_type == "mysql":  
            return MySQLConnection()  
  
        if db_type == "postgres":  
            return PostgresConnection()  
  
        raise ValueError("Unsupported database type")

The **factory decides which object to create**.

Client does **not know implementation details**.

---

# 6. Configuration

`config.py`

DB_TYPE = "postgres"

In real production systems this may come from:

- `.env`
    
- YAML
    
- Kubernetes ConfigMap
    
- environment variables
    

---

# 7. Client Code

`app.py`

from db.factory import DBFactory  
from config import DB_TYPE  
  
  
def main():  
  
    db = DBFactory.create_connection(DB_TYPE)  
  
    db.connect()  
  
    db.execute("SELECT * FROM users")  
  
  
if __name__ == "__main__":  
    main()

---

# 8. Output

If `DB_TYPE = "postgres"`

Connecting to PostgreSQL database  
PostgreSQL executing query: SELECT * FROM users

If `DB_TYPE = "mysql"`

Connecting to MySQL database  
MySQL executing query: SELECT * FROM users

---

# 9. Why This Is Useful in Production

Advantages:

1️⃣ **Loose coupling**

Client does not depend on database implementation.

2️⃣ **Easy switching**

Change config only:

DB_TYPE=mysql

3️⃣ **Scalable**

Add new database easily:

db/connections/  
    oracle_connection.py

Then update factory.

4️⃣ **Common in frameworks**

Used internally by:

- Django ORM
    
- SQLAlchemy
    
- Spring Boot datasource factories
    
- Cloud SDK clients
    

---

# 10. Real Production Upgrade

In real systems the factory may also:

- load DB driver
    
- connection pooling
    
- secrets manager
    
- retries
    
- logging
    
- telemetry
    

Example:

DBFactory.create_connection(config)
```



Abstract Factory method
```python
# Project Folder Structure

db_factory_example/  
│  
├── main.py  
│  
├── db  
│   ├── __init__.py  
│   ├── database.py  
│   ├── mysql_db.py  
│   └── postgres_db.py  
│  
└── factories  
    ├── __init__.py  
    ├── db_factory.py  
    ├── mysql_factory.py  
    └── postgres_factory.py

---

# db/database.py

(Common Interface)

from abc import ABC, abstractmethod  
  
  
class Database(ABC):  
  
    @abstractmethod  
    def connect(self):  
        pass  
  
    @abstractmethod  
    def disconnect(self):  
        pass

---

# db/mysql_db.py

(MySQL Implementation)

from db.database import Database  
  
  
class MySQL(Database):  
  
    def connect(self):  
        print("Connecting to MySQL database")  
  
    def disconnect(self):  
        print("Disconnecting MySQL database")

---

# db/postgres_db.py

(PostgreSQL Implementation)

from db.database import Database  
  
  
class Postgres(Database):  
  
    def connect(self):  
        print("Connecting to PostgreSQL database")  
  
    def disconnect(self):  
        print("Disconnecting PostgreSQL database")

---

# factories/db_factory.py

(Abstract Factory Interface)

from abc import ABC, abstractmethod  
  
  
class DatabaseFactory(ABC):  
  
    @abstractmethod  
    def create_database(self):  
        pass

---

# factories/mysql_factory.py

from factories.db_factory import DatabaseFactory  
from db.mysql_db import MySQL  
  
  
class MySQLFactory(DatabaseFactory):  
  
    def create_database(self):  
        return MySQL()

---

# factories/postgres_factory.py

from factories.db_factory import DatabaseFactory  
from db.postgres_db import Postgres  
  
  
class PostgresFactory(DatabaseFactory):  
  
    def create_database(self):  
        return Postgres()

---

# main.py (Client Code)

from factories.mysql_factory import MySQLFactory  
from factories.postgres_factory import PostgresFactory  
  
  
def main():  
  
    mysql_factory = MySQLFactory()  
    mysql_db = mysql_factory.create_database()  
  
    mysql_db.connect()  
    mysql_db.disconnect()  
  
    print()  
  
    postgres_factory = PostgresFactory()  
    postgres_db = postgres_factory.create_database()  
  
    postgres_db.connect()  
    postgres_db.disconnect()  
  
  
if __name__ == "__main__":  
    main()

---

# Output

Connecting to MySQL database  
Disconnecting MySQL database  
  
Connecting to PostgreSQL database  
Disconnecting PostgreSQL database

---

# Key Idea

Client **does not directly create database objects**

❌ Not this

db = MySQL()

✅ Instead

factory = MySQLFactory()  
db = factory.create_database()

So **object creation logic is encapsulated inside factories**.
```


Builder Pattern:

```python
The idea:  
Instead of creating a complex object with many parameters in one constructor, we **build it step-by-step**.

Example problem:

DatabaseConfig(  
    host,  
    port,  
    username,  
    password,  
    database,  
    ssl,  
    timeout  
)

This constructor becomes messy.  
Builder pattern solves it.

---

# Project Folder Structure

builder_db_example/  
│  
├── main.py  
│  
├── config  
│   ├── __init__.py  
│   ├── database_config.py  
│   └── database_config_builder.py

---

# config/database_config.py

(Product Class)

class DatabaseConfig:  
  
    def __init__(self):  
        self.host = None  
        self.port = None  
        self.username = None  
        self.password = None  
        self.database = None  
        self.ssl = False  
        self.timeout = None  
  
    def show(self):  
        print("Database Configuration")  
        print("Host:", self.host)  
        print("Port:", self.port)  
        print("Username:", self.username)  
        print("Database:", self.database)  
        print("SSL Enabled:", self.ssl)  
        print("Timeout:", self.timeout)

---

# config/database_config_builder.py

(Builder Class)

from config.database_config import DatabaseConfig  
  
  
class DatabaseConfigBuilder:  
  
    def __init__(self):  
        self.config = DatabaseConfig()  
  
    def set_host(self, host):  
        self.config.host = host  
        return self  
  
    def set_port(self, port):  
        self.config.port = port  
        return self  
  
    def set_username(self, username):  
        self.config.username = username  
        return self  
  
    def set_password(self, password):  
        self.config.password = password  
        return self  
  
    def set_database(self, database):  
        self.config.database = database  
        return self  
  
    def enable_ssl(self):  
        self.config.ssl = True  
        return self  
  
    def set_timeout(self, timeout):  
        self.config.timeout = timeout  
        return self  
  
    def build(self):  
        return self.config

---

# main.py (Client Code)

from config.database_config_builder import DatabaseConfigBuilder  
  
  
def main():  
  
    db_config = (  
        DatabaseConfigBuilder()  
        .set_host("localhost")  
        .set_port(5432)  
        .set_username("admin")  
        .set_password("secret")  
        .set_database("users_db")  
        .enable_ssl()  
        .set_timeout(30)  
        .build()  
    )  
  
    db_config.show()  
  
  
if __name__ == "__main__":  
    main()

---

# Output

Database Configuration  
Host: localhost  
Port: 5432  
Username: admin  
Database: users_db  
SSL Enabled: True  
Timeout: 30

---

# Without Builder (Problem)

Without builder we would write:

config = DatabaseConfig(  
    "localhost",  
    5432,  
    "admin",  
    "secret",  
    "users_db",  
    True,  
    30  
)

Problems:

- Hard to remember order
    
- Hard to read
    
- Many optional parameters
    

---

# With Builder (Clean)

config = (  
    DatabaseConfigBuilder()  
    .set_host("localhost")  
    .set_port(5432)  
    .enable_ssl()  
    .build()  
)

Much clearer.

---

# Real Production Examples

Builder pattern is used in:

- **HTTP request builders**
    
- **Docker client libraries**
    
- **SQL query builders**
    
- **Java StringBuilder**
    
- **Cloud SDK clients**
    

Example:

query = (  
    QueryBuilder()  
    .select("*")  
    .from_table("users")  
    .where("age > 18")  
    .limit(10)  
    .build()  
)

---
```


```
# 1️⃣ Strategy Design Pattern (Very Simple)

👉 **Purpose:** Choose **one behavior/algorithm** from many.

Think: **“How should this action be done?”**

Example: **Payment method**

You want to **change the payment method without changing the main code**.

### Structure

PaymentStrategy (interface)  
      ↑  
   CardPayment  
   UpiPayment  
   PaypalPayment  
  
PaymentContext (uses strategy)

---

### Python Example (Simple)

strategy/  
├── payment_strategy.py  
├── card_payment.py  
├── upi_payment.py  
└── main.py

### payment_strategy.py

class PaymentStrategy:  
    def pay(self, amount):  
        pass

### card_payment.py

from payment_strategy import PaymentStrategy  
  
class CardPayment(PaymentStrategy):  
    def pay(self, amount):  
        print("Paid with Card:", amount)

### upi_payment.py

from payment_strategy import PaymentStrategy  
  
class UpiPayment(PaymentStrategy):  
    def pay(self, amount):  
        print("Paid with UPI:", amount)

### main.py

from card_payment import CardPayment  
from upi_payment import UpiPayment  
  
class PaymentContext:  
  
    def __init__(self, strategy):  
        self.strategy = strategy  
  
    def pay(self, amount):  
        self.strategy.pay(amount)  
  
  
payment = PaymentContext(CardPayment())  
payment.pay(100)  
  
payment = PaymentContext(UpiPayment())  
payment.pay(200)

✔ Here we **change the behavior** (payment method).

---

# 2️⃣ Abstract Factory Pattern (Very Simple)

👉 **Purpose:** Create **families of related objects**.

Think: **“Which group of objects should be created?”**

Example: **UI for different OS**

For **Mac** you need:

- MacButton
    
- MacCheckbox
    

For **Windows** you need:

- WindowsButton
    
- WindowsCheckbox
    

Factory will create the correct set.

---

### Structure

GUIFactory (interface)  
      ↑  
 MacFactory  
 WindowsFactory  
  
Creates:  
Button  
Checkbox

---

### Python Example

abstract-factory/  
├── button.py  
├── checkbox.py  
├── mac_factory.py  
├── windows_factory.py  
└── main.py

### button.py

class Button:  
    def render(self):  
        pass

### mac_factory.py

class MacButton:  
    def render(self):  
        print("Mac Button")  
  
class MacFactory:  
    def create_button(self):  
        return MacButton()

### windows_factory.py

class WindowsButton:  
    def render(self):  
        print("Windows Button")  
  
class WindowsFactory:  
    def create_button(self):  
        return WindowsButton()

### main.py

factory = MacFactory()  
button = factory.create_button()  
button.render()

✔ Here we **create objects depending on factory type**.

---

# 3️⃣ The Key Difference (SUPER IMPORTANT)

|Feature|Strategy Pattern|Abstract Factory|
|---|---|---|
|Purpose|Choose **behavior/algorithm**|Create **families of objects**|
|Focus|**How something works**|**What objects to create**|
|Example|Payment method|UI components|
|Changes|Behavior changes|Object family changes|

---

# 4️⃣ Easy Way to Remember

### Strategy

👉 **Same object, different behavior**

Example:

Payment  
 ├── Card  
 ├── UPI  
 └── PayPal

Changing **HOW payment works**

---

### Abstract Factory

👉 **Create different sets of objects**

Example:

MacFactory → MacButton + MacCheckbox  
WindowsFactory → WindowsButton + WindowsCheckbox

Changing **WHAT objects are created**

---

# 5️⃣ One-Line Trick for Interviews

**Strategy Pattern**

> Used to switch between different algorithms at runtime.

**Abstract Factory Pattern**

> Used to create families of related objects without specifying their concrete classes.

---
```


**Simple Example: Strategy Design Pattern**
```java
interface Payment {
    void pay();
}

class PhonePay implements Payment {
    public void pay() {
        System.out.println("Paying using PhonePay");
    }
}

class GooglePay implements Payment {
    public void pay() {
        System.out.println("Paying using GooglePay");
    }
}

class Strategy {

    private Payment paymentStrategy;

    Strategy(Payment payment) {
        this.paymentStrategy = payment;
    }

    void pay() {
        paymentStrategy.pay();
    }

    void setPaymentStrategy(Payment payment) {
        this.paymentStrategy = payment;
    }
}

public class Main {

    public static void main(String[] args) {

        Payment p1 = new PhonePay();

        Strategy s1 = new Strategy(p1);
        s1.pay();

        s1.setPaymentStrategy(new GooglePay());
        s1.pay();
    }
}
```


**Simple Example: Observer Design Pattern**
```java
// Subscriber interface
public interface Subscriber {
    void update(String video);
}


// YouTube subscriber
public class YouTubeSubscriber implements Subscriber {

    private String name;

    public YouTubeSubscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(String video) {
        System.out.println(name + " is watching the video: " + video);
    }
}

// Email subscriber
public class EmailSubscriber implements Subscriber {

    private String email;

    public EmailSubscriber(String email) {
        this.email = email;
    }

    @Override
    public void update(String video) {
        System.out.println("Sending email to " + email + ": New video uploaded: " + video);
    }
}

// Push notification subscriber
public class PushNotificationSubscriber implements Subscriber {

    private String device;

    public PushNotificationSubscriber(String device) {
        this.device = device;
    }

    @Override
    public void update(String video) {
        System.out.println("Sending push notification to " + device + ": New video uploaded: " + video);
    }
}

// Channel interface
public interface YouTubeChannel {

    void addSubscriber(Subscriber subscriber);

    void removeSubscriber(Subscriber subscriber);

    void notifySubscribers();
}


import java.util.ArrayList;
import java.util.List;

// Channel implementation
public class YouTubeChannelImpl implements YouTubeChannel {

    private List<Subscriber> subscribers = new ArrayList<>();
    private String video;

    @Override
    public void addSubscriber(Subscriber subscriber) {
        subscribers.add(subscriber);
    }

    @Override
    public void removeSubscriber(Subscriber subscriber) {
        subscribers.remove(subscriber);
    }

    @Override
    public void notifySubscribers() {
        for (Subscriber subscriber : subscribers) {
            subscriber.update(video);
        }
    }

    public void uploadNewVideo(String video) {
        this.video = video;
        notifySubscribers();
    }
}

// Main class
public class Main {

    public static void main(String[] args) {

        YouTubeChannelImpl channel = new YouTubeChannelImpl();

        Subscriber alice = new YouTubeSubscriber("Alice");
        Subscriber bob = new YouTubeSubscriber("Bob");
        Subscriber emailUser = new EmailSubscriber("alice@gmail.com");
        Subscriber mobileUser = new PushNotificationSubscriber("Alice's Phone");

        channel.addSubscriber(alice);
        channel.addSubscriber(bob);
        channel.addSubscriber(emailUser);
        channel.addSubscriber(mobileUser);

        channel.uploadNewVideo("Observer Pattern Explained");
    }
}


### Output

Alice is watching the video: Observer Pattern Explained
Bob is watching the video: Observer Pattern Explained
Sending email to alice@gmail.com: New video uploaded: Observer Pattern Explained
Sending push notification to Alice's Phone: New video uploaded: Observer Pattern Explained

```

Example 2:

```txt
Problem:

Design a simple Weather Station system using the Observer Design Pattern.

A weather station measures temperature. Whenever the temperature changes, all registered displays should automatically receive the updated temperature and print it.

Requirements:

1. There should be a way for displays to subscribe to the weather station.
2. Displays should also be able to unsubscribe.
3. When the temperature changes, all subscribed displays must be notified.
4. Each display should print the updated temperature when it receives the update.

Example Output:

Mobile Display: Temperature updated to 30.5
TV Display: Temperature updated to 30.5

If one display unsubscribes and the temperature changes again:

Mobile Display: Temperature updated to 32.0
```

```java
// Observer
interface DisplaySubscriber {
    void update(Float temperature);
}

// Concrete Observer 1
class MobileDisplaySubscriber implements DisplaySubscriber {

    @Override
    public void update(Float temperature) {
        System.out.println("Mobile Display: Temperature updated to " + temperature);
    }
}

// Concrete Observer 2
class DesktopDisplaySubscriber implements DisplaySubscriber {

    @Override
    public void update(Float temperature) {
        System.out.println("Desktop Display: Temperature updated to " + temperature);
    }
}

// Subject
interface WeatherStation {
    void temperatureChange(Float temperature);
    void addSubscriber(DisplaySubscriber subscriber);
    void removeSubscriber(DisplaySubscriber subscriber);
    void notifySubscribers(Float temperature);
}

// Subject Implementation
import java.util.ArrayList;
import java.util.List;

class WeatherStationImpl implements WeatherStation {

    private List<DisplaySubscriber> subscribers = new ArrayList<>();

    @Override
    public void temperatureChange(Float temperature) {
        notifySubscribers(temperature);
    }

    @Override
    public void addSubscriber(DisplaySubscriber subscriber) {
        subscribers.add(subscriber);
    }

    @Override
    public void removeSubscriber(DisplaySubscriber subscriber) {
        subscribers.remove(subscriber);
    }

    @Override
    public void notifySubscribers(Float temperature) {
        for (DisplaySubscriber subscriber : subscribers) {
            subscriber.update(temperature);
        }
    }
}

// Main
public class Main {

    public static void main(String[] args) {

        WeatherStation station = new WeatherStationImpl();

        DisplaySubscriber mobile = new MobileDisplaySubscriber();
        DisplaySubscriber desktop = new DesktopDisplaySubscriber();

        station.addSubscriber(mobile);
        station.addSubscriber(desktop);

        station.temperatureChange(30.5f);

        station.removeSubscriber(desktop);

        station.temperatureChange(31.5f);
    }
}

/*
Output:

Mobile Display: Temperature updated to 30.5
Desktop Display: Temperature updated to 30.5
Mobile Display: Temperature updated to 31.5
*/
``` 