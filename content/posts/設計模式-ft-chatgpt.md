---
title: 設計模式(ft chatgpt)
date: 2023-04-20 06:42:23
tags:
categories:
---

# 前言
2022年底chatgpt的出現讓AI出現了重大的進展，連帶著我們的學習方式也跟著改變了
下面的筆記是和BING聊天後整理出來的

# 創建型


## 工廠模式
Q:工廠模式有哪幾種?

### 簡單工廠模式
Q:用最簡單的PHP代碼來實現簡單工廠模式

```php
interface Car {
    public function drive();
}

class Tesla implements Car {
    public function drive() {
        echo "Driving a Tesla car!\n";
    }
}

class BMW implements Car {
    public function drive() {
        echo "Driving a BMW car!\n";
    }
}

class CarFactory {
    public static function createCar($type) {
        if ($type == 'Tesla') {
            return new Tesla();
        } else if ($type == 'BMW') {
            return new BMW();
        } else {
            throw new Exception("Invalid car type.");
        }
    }
}

$tesla = CarFactory::createCar('Tesla');
$tesla->drive();

$bmw = CarFactory::createCar('BMW');
$bmw->drive();
```
從上面方法我們看出需要interface定義相同類型的產品
需要class的對應來判斷要news哪個class，而這個判斷就是createCar方法

Q:簡單工廠模式一定要用靜態方法嗎
>A:不一定。简单工厂模式有时也被称为静态工厂模式，因为它通常使用静态方法来创建对象。但是，这并不是必须的。您也可以使用非静态方法来实现简单工厂模式。 
>
>使用静态方法的好处是您可以直接调用该方法而无需创建工厂类的实例。但是，这也意味着您无法通过继承或实现接口来扩展或更改工厂类的行为。
>
>如果您希望能够扩展或更改工厂类的行为，可以使用非静态方法并创建工厂类的实例来调用该方法。

Q:簡單工廠模式一定要搭配interface嗎
>不一定。简单工厂模式并不要求一定要使用接口。它的目的是通过一个工厂类来创建不同类型的对象，而这些对象通常具有相同的基类或实现相同的接口。


### 工廠方法模式
Q:用最簡單的PHP代碼來實現工廠方法模式

```php
interface Car {
    public function drive();
}

class Tesla implements Car {
    public function drive() {
        echo "Driving a Tesla car!\n";
    }
}

class BMW implements Car {
    public function drive() {
        echo "Driving a BMW car!\n";
    }
}

interface CarFactory {
    public function createCar();
}

class TeslaFactory implements CarFactory {
    public function createCar() {
        return new Tesla();
    }
}

class BMWFactory implements CarFactory {
    public function createCar() {
        return new BMW();
    }
}

$teslaFactory = new TeslaFactory();
$tesla = $teslaFactory->createCar();
$tesla->drive();

$bmwFactory = new BMWFactory();
$bmw = $bmwFactory->createCar();
$bmw->drive();
```
從上面方法我們看出需要interface定義相同類型的產品，以及相同的的工廠

依舊需要class的對應來判斷要news哪個class，只不過這次變成new工廠，相當於將簡單工廠再包一層，createCar不再是靜態方法


### 抽象工廠模式
Q:用最簡單的PHP代碼來實現抽象工廠模式

```php
interface Car {
    public function drive();
}

class Tesla implements Car {
    public function drive() {
        echo "Driving a Tesla car!\n";
    }
}

class BMW implements Car {
    public function drive() {
        echo "Driving a BMW car!\n";
    }
}

interface Bike {
    public function ride();
}

class BMXBike implements Bike {
    public function ride() {
        echo "Riding a BMX bike!\n";
    }
}

class RoadBike implements Bike {
    public function ride() {
        echo "Riding a road bike!\n";
    }
}

interface VehicleFactory {
    public function createCar();
    public function createBike();
}

class TeslaFactory implements VehicleFactory {
    public function createCar() {
        return new Tesla();
    }

    public function createBike() {
        return new BMXBike();
    }
}

class BMWFactory implements VehicleFactory {
    public function createCar() {
        return new BMW();
    }

    public function createBike() {
        return new RoadBike();
    }
}

$teslaFactory = new TeslaFactory();
$tesla = $teslaFactory->createCar();
$tesla->drive();
$bmx = $teslaFactory->createBike();
$bmx->ride();

$bmwFactory = new BMWFactory();
$bmw = $bmwFactory->createCar();
$bmw->drive();
$road = $bmwFactory->createBike();
$road->ride();
```
從上面方法我們看出需要interface定義相同類型的產品，以及相同的的工廠，只不過這次的工廠產出兩種不同的產品