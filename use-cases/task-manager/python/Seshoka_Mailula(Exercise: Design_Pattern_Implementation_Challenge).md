# Design Pattern Implementation Challenge: Factory Pattern (Python)

---

## 1. Pattern Selected

**Pattern:** Factory Pattern
**Language:** Python
**Code Base:** Database Connection Manager

---

## 2. Pattern Opportunity Analysis

The original `DatabaseConnection` class suffers from a classic design problem: it handles the creation and configuration logic for **four completely different database types** (`mysql`, `postgresql`, `mongodb`, `redis`) inside a single `__init__` and `connect()` method. This means:

- Every time a new database type is added, the existing class must be modified (violating the **Open/Closed Principle**).
- The `connect()` method grows longer with every new `elif` branch.
- It is impossible to test one database type in isolation without the entire class being loaded.

The **Factory Pattern** solves this by delegating the creation of each specific database connection to its own dedicated class, and using a central factory to decide which one to instantiate.

---

## 3. Refactored Implementation

### Base Class and Specialised Subclasses

```python
from abc import ABC, abstractmethod

# Abstract base class — defines the contract
class DatabaseConnection(ABC):
    def __init__(self, host, port, username, password, database, use_ssl=False):
        self.host = host
        self.port = port
        self.username = username
        self.password = password
        self.database = database
        self.use_ssl = use_ssl
        self.connection = None

    @abstractmethod
    def connect(self):
        pass


# Concrete class for MySQL
class MySQLConnection(DatabaseConnection):
    def __init__(self, *args, charset='utf8', connection_timeout=30, **kwargs):
        super().__init__(*args, **kwargs)
        self.charset = charset
        self.connection_timeout = connection_timeout

    def connect(self):
        connection_string = (
            f"mysql://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
            f"?charset={self.charset}&connectionTimeout={self.connection_timeout}"
        )
        if self.use_ssl:
            connection_string += "&useSSL=true"
        print(f"MySQL Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for PostgreSQL
class PostgreSQLConnection(DatabaseConnection):
    def connect(self):
        connection_string = (
            f"postgresql://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
        )
        if self.use_ssl:
            connection_string += "?sslmode=require"
        print(f"PostgreSQL Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for MongoDB
class MongoDBConnection(DatabaseConnection):
    def __init__(self, *args, pool_size=5, retry_attempts=3, **kwargs):
        super().__init__(*args, **kwargs)
        self.pool_size = pool_size
        self.retry_attempts = retry_attempts

    def connect(self):
        connection_string = (
            f"mongodb://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
            f"?retryAttempts={self.retry_attempts}&poolSize={self.pool_size}"
        )
        if self.use_ssl:
            connection_string += "&ssl=true"
        print(f"MongoDB Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for Redis
class RedisConnection(DatabaseConnection):
    def connect(self):
        print(f"Redis Connection: {self.host}:{self.port}/{self.database}")
        print("Connection successful!")
```

### The Factory

```python
class DatabaseConnectionFactory:
    _registry = {
        'mysql':      MySQLConnection,
        'postgresql': PostgreSQLConnection,
        'mongodb':    MongoDBConnection,
        'redis':      RedisConnection,
    }

    @staticmethod
    def create(db_type: str, **kwargs) -> DatabaseConnection:
        connection_class = DatabaseConnectionFactory._registry.get(db_type.lower())
        if not connection_class:
            raise ValueError(f"Unsupported database type: '{db_type}'")
        return connection_class(**kwargs)
```

### Usage

```python
# MySQL
mysql_db = DatabaseConnectionFactory.create(
    db_type='mysql',
    host='localhost',
    port=3306,
    username='db_user',
    password='password123',
    database='app_db',
    use_ssl=True
)
mysql_db.connect()

# MongoDB
mongo_db = DatabaseConnectionFactory.create(
    db_type='mongodb',
    host='mongodb.example.com',
    port=27017,
    username='mongo_user',
    password='mongo123',
    database='analytics',
    pool_size=10,
    retry_attempts=5
)
mongo_db.connect()
```

---

## 4. Tests

```python
import unittest

class TestDatabaseConnectionFactory(unittest.TestCase):

    def test_creates_mysql_connection(self):
        db = DatabaseConnectionFactory.create(
            db_type='mysql', host='localhost', port=3306,
            username='user', password='pass', database='test_db'
        )
        self.assertIsInstance(db, MySQLConnection)

    def test_creates_mongodb_connection(self):
        db = DatabaseConnectionFactory.create(
            db_type='mongodb', host='localhost', port=27017,
            username='user', password='pass', database='test_db'
        )
        self.assertIsInstance(db, MongoDBConnection)

    def test_raises_error_for_unsupported_type(self):
        with self.assertRaises(ValueError):
            DatabaseConnectionFactory.create(
                db_type='oracle', host='localhost', port=1521,
                username='user', password='pass', database='test_db'
            )

    def test_mysql_connect_runs_without_error(self):
        db = DatabaseConnectionFactory.create(
            db_type='mysql', host='localhost', port=3306,
            username='user', password='pass', database='test_db'
        )
        try:
            db.connect()
        except Exception as e:
            self.fail(f"connect() raised an exception: {e}")


if __name__ == '__main__':
    unittest.main()
```

---

## 5. Reflection Questions

**How did implementing the pattern improve the code's maintainability?**

The original class was a single large block that mixed creation logic for four different database types into one place. After refactoring, each database type lives in its own class and is fully responsible for its own connection logic. If MySQL's connection string requirements change, only `MySQLConnection` needs to be updated — nothing else is touched. This makes the codebase significantly easier to maintain and reason about.

**What future changes will be easier because of this pattern?**

Adding a new database type — say, `CassandraConnection` — now requires only two steps: creating a new subclass that extends `DatabaseConnection`, and registering it in the factory's `_registry` dictionary. The rest of the codebase does not need to change at all. Without the Factory Pattern, every new database type would require opening the original class and adding another `elif` branch, increasing the risk of introducing bugs into existing functionality.

**Were there any unexpected challenges in implementing the pattern?**

The trickiest part was handling the constructor arguments cleanly. Each database type needs slightly different parameters — MySQL needs `charset` and `connection_timeout`, MongoDB needs `pool_size` and `retry_attempts` — while all of them share the common base parameters. Using Python's `*args` and `**kwargs` to pass shared parameters up to the base class via `super().__init__()` while keeping type-specific parameters at the subclass level required careful thought but resulted in a much cleaner interface.# Design Pattern Implementation Challenge: Factory Pattern (Python)

---

## 1. Pattern Selected

**Pattern:** Factory Pattern
**Language:** Python
**Code Base:** Database Connection Manager

---

## 2. Pattern Opportunity Analysis

The original `DatabaseConnection` class suffers from a classic design problem: it handles the creation and configuration logic for **four completely different database types** (`mysql`, `postgresql`, `mongodb`, `redis`) inside a single `__init__` and `connect()` method. This means:

- Every time a new database type is added, the existing class must be modified (violating the **Open/Closed Principle**).
- The `connect()` method grows longer with every new `elif` branch.
- It is impossible to test one database type in isolation without the entire class being loaded.

The **Factory Pattern** solves this by delegating the creation of each specific database connection to its own dedicated class, and using a central factory to decide which one to instantiate.

---

## 3. Refactored Implementation

### Base Class and Specialised Subclasses

```python
from abc import ABC, abstractmethod

# Abstract base class — defines the contract
class DatabaseConnection(ABC):
    def __init__(self, host, port, username, password, database, use_ssl=False):
        self.host = host
        self.port = port
        self.username = username
        self.password = password
        self.database = database
        self.use_ssl = use_ssl
        self.connection = None

    @abstractmethod
    def connect(self):
        pass


# Concrete class for MySQL
class MySQLConnection(DatabaseConnection):
    def __init__(self, *args, charset='utf8', connection_timeout=30, **kwargs):
        super().__init__(*args, **kwargs)
        self.charset = charset
        self.connection_timeout = connection_timeout

    def connect(self):
        connection_string = (
            f"mysql://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
            f"?charset={self.charset}&connectionTimeout={self.connection_timeout}"
        )
        if self.use_ssl:
            connection_string += "&useSSL=true"
        print(f"MySQL Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for PostgreSQL
class PostgreSQLConnection(DatabaseConnection):
    def connect(self):
        connection_string = (
            f"postgresql://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
        )
        if self.use_ssl:
            connection_string += "?sslmode=require"
        print(f"PostgreSQL Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for MongoDB
class MongoDBConnection(DatabaseConnection):
    def __init__(self, *args, pool_size=5, retry_attempts=3, **kwargs):
        super().__init__(*args, **kwargs)
        self.pool_size = pool_size
        self.retry_attempts = retry_attempts

    def connect(self):
        connection_string = (
            f"mongodb://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
            f"?retryAttempts={self.retry_attempts}&poolSize={self.pool_size}"
        )
        if self.use_ssl:
            connection_string += "&ssl=true"
        print(f"MongoDB Connection: {connection_string}")
        print("Connection successful!")


# Concrete class for Redis
class RedisConnection(DatabaseConnection):
    def connect(self):
        print(f"Redis Connection: {self.host}:{self.port}/{self.database}")
        print("Connection successful!")
```

### The Factory

```python
class DatabaseConnectionFactory:
    _registry = {
        'mysql':      MySQLConnection,
        'postgresql': PostgreSQLConnection,
        'mongodb':    MongoDBConnection,
        'redis':      RedisConnection,
    }

    @staticmethod
    def create(db_type: str, **kwargs) -> DatabaseConnection:
        connection_class = DatabaseConnectionFactory._registry.get(db_type.lower())
        if not connection_class:
            raise ValueError(f"Unsupported database type: '{db_type}'")
        return connection_class(**kwargs)
```

### Usage

```python
# MySQL
mysql_db = DatabaseConnectionFactory.create(
    db_type='mysql',
    host='localhost',
    port=3306,
    username='db_user',
    password='password123',
    database='app_db',
    use_ssl=True
)
mysql_db.connect()

# MongoDB
mongo_db = DatabaseConnectionFactory.create(
    db_type='mongodb',
    host='mongodb.example.com',
    port=27017,
    username='mongo_user',
    password='mongo123',
    database='analytics',
    pool_size=10,
    retry_attempts=5
)
mongo_db.connect()
```

---

## 4. Tests

```python
import unittest

class TestDatabaseConnectionFactory(unittest.TestCase):

    def test_creates_mysql_connection(self):
        db = DatabaseConnectionFactory.create(
            db_type='mysql', host='localhost', port=3306,
            username='user', password='pass', database='test_db'
        )
        self.assertIsInstance(db, MySQLConnection)

    def test_creates_mongodb_connection(self):
        db = DatabaseConnectionFactory.create(
            db_type='mongodb', host='localhost', port=27017,
            username='user', password='pass', database='test_db'
        )
        self.assertIsInstance(db, MongoDBConnection)

    def test_raises_error_for_unsupported_type(self):
        with self.assertRaises(ValueError):
            DatabaseConnectionFactory.create(
                db_type='oracle', host='localhost', port=1521,
                username='user', password='pass', database='test_db'
            )

    def test_mysql_connect_runs_without_error(self):
        db = DatabaseConnectionFactory.create(
            db_type='mysql', host='localhost', port=3306,
            username='user', password='pass', database='test_db'
        )
        try:
            db.connect()
        except Exception as e:
            self.fail(f"connect() raised an exception: {e}")


if __name__ == '__main__':
    unittest.main()
```

---

## 5. Reflection Questions

**How did implementing the pattern improve the code's maintainability?**

The original class was a single large block that mixed creation logic for four different database types into one place. After refactoring, each database type lives in its own class and is fully responsible for its own connection logic. If MySQL's connection string requirements change, only `MySQLConnection` needs to be updated — nothing else is touched. This makes the codebase significantly easier to maintain and reason about.

**What future changes will be easier because of this pattern?**

Adding a new database type — say, `CassandraConnection` — now requires only two steps: creating a new subclass that extends `DatabaseConnection`, and registering it in the factory's `_registry` dictionary. The rest of the codebase does not need to change at all. Without the Factory Pattern, every new database type would require opening the original class and adding another `elif` branch, increasing the risk of introducing bugs into existing functionality.

**Were there any unexpected challenges in implementing the pattern?**

The trickiest part was handling the constructor arguments cleanly. Each database type needs slightly different parameters — MySQL needs `charset` and `connection_timeout`, MongoDB needs `pool_size` and `retry_attempts` — while all of them share the common base parameters. Using Python's `*args` and `**kwargs` to pass shared parameters up to the base class via `super().__init__()` while keeping type-specific parameters at the subclass level required careful thought but resulted in a much cleaner interface.
