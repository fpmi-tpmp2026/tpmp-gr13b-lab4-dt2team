# Схема базы данных

## SQL-скрипт для SQLite

```sql
CREATE TABLE IF NOT EXISTS cars (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    plate_number TEXT NOT NULL UNIQUE,
    brand TEXT NOT NULL,
    initial_mileage INTEGER NOT NULL CHECK (initial_mileage >= 0),
    load_capacity REAL NOT NULL CHECK (load_capacity > 0)
);

CREATE TABLE IF NOT EXISTS drivers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    personnel_number TEXT NOT NULL UNIQUE,
    last_name TEXT NOT NULL,
    category TEXT NOT NULL,
    experience INTEGER NOT NULL CHECK (experience >= 0),
    birth_year INTEGER NOT NULL CHECK (birth_year > 1940),
    address TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    order_date TEXT NOT NULL,
    driver_id INTEGER NOT NULL,
    car_id INTEGER NOT NULL,
    mileage INTEGER NOT NULL CHECK (mileage >= 0),
    cargo_weight REAL NOT NULL CHECK (cargo_weight > 0),
    cost REAL NOT NULL CHECK (cost > 0),
    FOREIGN KEY (driver_id) REFERENCES drivers(id) ON DELETE CASCADE,
    FOREIGN KEY (car_id) REFERENCES cars(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('driver', 'manager')),
    driver_id INTEGER UNIQUE,
    FOREIGN KEY (driver_id) REFERENCES drivers(id) ON DELETE SET NULL
);

CREATE TABLE IF NOT EXISTS driver_earnings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    driver_id INTEGER NOT NULL,
    period_start TEXT NOT NULL,
    period_end TEXT NOT NULL,
    total_trips INTEGER NOT NULL CHECK (total_trips >= 0),
    total_weight REAL NOT NULL CHECK (total_weight >= 0),
    total_earned REAL NOT NULL CHECK (total_earned >= 0),
    FOREIGN KEY (driver_id) REFERENCES drivers(id) ON DELETE CASCADE
);

INSERT INTO cars (plate_number, brand, initial_mileage, load_capacity) VALUES
('A123BC', 'КАМАЗ', 10000, 15.0),
('B456DE', 'MAN', 5000, 20.0),
('C789FG', 'Volvo', 8000, 18.5),
('X001XX', 'ГАЗель', 15000, 3.5),
('Y002YY', 'Mercedes', 2000, 22.0);

INSERT INTO drivers (personnel_number, last_name, category, experience, birth_year, address) VALUES
('001', 'Иванов', 'CE', 10, 1985, 'г. Москва, ул. Ленина, 1'),
('002', 'Петров', 'C', 5, 1990, 'г. Москва, ул. Гагарина, 2'),
('003', 'Сидоров', 'CE', 15, 1980, 'г. Москва, ул. Пушкина, 3'),
('004', 'Козлов', 'B', 2, 1995, 'г. Москва, ул. Тверская, 10'),
('005', 'Михайлов', 'C', 8, 1988, 'г. Москва, ул. Арбат, 5');

INSERT INTO orders (order_date, driver_id, car_id, mileage, cargo_weight, cost) VALUES
('2025-03-01', 1, 1, 200, 10.0, 50000),
('2025-03-05', 1, 1, 150, 12.0, 60000),
('2025-03-10', 2, 2, 300, 18.0, 90000),
('2025-03-15', 3, 3, 100, 8.0, 40000),
('2025-03-20', 1, 4, 80, 3.0, 15000),
('2025-03-25', 4, 4, 120, 3.2, 16000),
('2025-04-01', 2, 2, 250, 16.0, 80000),
('2025-04-05', 3, 5, 400, 20.0, 120000),
('2025-04-10', 5, 5, 350, 18.0, 100000),
('2025-04-12', 1, 1, 180, 11.0, 55000);

INSERT INTO users (username, password_hash, role, driver_id) VALUES
('ivanov', 'pass123', 'driver', 1),
('petrov', 'pass123', 'driver', 2),
('sidorov', 'pass123', 'driver', 3),
('kozlov', 'pass123', 'driver', 4),
('mikhailov', 'pass123', 'driver', 5),
('manager', 'pass123', 'manager', NULL);

CREATE TRIGGER check_cargo_weight
BEFORE INSERT ON orders
BEGIN
    SELECT CASE
        WHEN NEW.cargo_weight > (SELECT load_capacity FROM cars WHERE id = NEW.car_id)
        THEN RAISE(ABORT, 'Ошибка: масса груза превышает грузоподъемность автомобиля')
    END;
END;

CREATE TRIGGER check_cargo_weight_update
BEFORE UPDATE ON orders
BEGIN
    SELECT CASE
        WHEN NEW.cargo_weight > (SELECT load_capacity FROM cars WHERE id = NEW.car_id)
        THEN RAISE(ABORT, 'Ошибка: масса груза превышает грузоподъемность автомобиля')
    END;
END;
```

## Таблицы

| Таблица | Описание |
|---------|----------|
| cars | Автомобили |
| drivers | Водители |
| orders | Заказы |
| users | Пользователи (аутентификация) |
| driver_earnings | Результаты расчёта заработной платы |

## Тестовые данные

- 5 автомобилей
- 5 водителей
- 10 заказов
- 6 пользователей (5 водителей + 1 менеджер)

## Триггеры

- `check_cargo_weight` - проверяет, не превышает ли масса груза грузоподъемность автомобиля при вставке заказа
- `check_cargo_weight_update` - то же самое при обновлении заказа
```

