
# SOLID

## Содержание

**SOLID** - это мнемонический акроним, введенный Michael Feathers для первых пяти принципов объектно-ориентированного программирования и дизайна, описанных Robert Martin.

 * [S: Принцип единственной ответственности (Single Responsibility Principle, SRP)](#Принцип-единственной-ответственности-single-responsibility-principle-srp)
 * [O: Принцип открытости/закрытости (Open/Closed Principle, OCP)](#Принцип-открытостизакрытости-openclosed-principle-ocp)
 * [L: Принцип подстановки Барбары Лисков (Liskov Substitution Principle, LSP)](#Принцип-подстановки-Барбары-Лисков-liskov-substitution-principle-lsp)
 * [I: Принцип разделения интерфейса (Interface Segregation Principle, ISP)](#Принцип-разделения-интерфейса-interface-segregation-principle-isp)
 * [D: Принцип инверсии зависимостей (Dependency Inversion Principle, DIP)](#Принцип-инверсии-зависимостей-dependency-inversion-principle-dip)

### Принцип единственной ответственности (Single Responsibility Principle, SRP)

Как говорится в книге Clean Code: "Для изменения класса никогда не должно быть более одной причины". Порой заманчиво набить класс всевозможной функциональностью, как мы это делаем с сумками и рюкзаками, которые разрешается взять в качестве ручной клади в самолёт. Проблема в том, что ваш класс не будет концептуально связанным (conceptually cohesive), и поэтому возникнет много причин изменить его. Важно минимизировать количество случаев, когда вам нужно изменять класс. А важно потому, что когда в классе избыток функциональности и вам нужно поменять её часть, то может быть трудно понять, как это отразится на зависимых модулях в кодовой базе.

**Плохо:**

```php
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```

**Хорошо:**

```php
class UserAuth
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings
{
    private $user;
    private $auth;

    public function __construct(User $user)
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```

**[⬆ вернуться к началу](#Содержание)**

### Принцип открытости/закрытости (Open/Closed Principle, OCP)

Как сказал Bertrand Meyer: "Программные сущности (классы, модули, функции и т. д.) должны быть открыты для расширения, но закрыты для модифицирования". Что это означает? Позвольте пользователям добавлять новую функциональность без изменения кода.

**Плохо:**

```php
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```

**Хорошо:**

```php
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆ вернуться к началу](#Содержание)**

### Принцип подстановки Барбары Лисков (Liskov Substitution Principle, LSP)

За этим пугающим термином скрывается очень простая идея. Формальное определение: "Если S — это подтип Т, то объекты типа Т могут быть заменены объектами типа S (например, вместо объектов типа Т можно подставить объекты типа S) без изменения каких-либо свойств программы (корректность, задачи и т. д.)". Ещё более пугающее определение.

Можно объяснить проще: если у вас есть родительский и дочерний классы, тогда они могут быть взаимозаменяемы без получения некорректных результатов. Рассмотрим классический пример с квадратом и прямоугольником. С точки зрения математики квадрат — это прямоугольник, но если смоделировать эту взаимосвязь is-a посредством наследования, то у вас будут проблемы.

**Плохо:**

```php
class Rectangle
{
    protected $width = 0;
    protected $height = 0;

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(4);
    $rectangle->setHeight(5);

    // BAD: Will return 25 for Square. Should be 20.
    echo sprintf('%s has area %d.', get_class($rectangle), $rectangle->getArea()).PHP_EOL;
}

$rectangles = [new Rectangle(), new Square()];

foreach ($rectangles as $rectangle) {
    printArea($rectangle);
}
```

**Хорошо:**

Лучший способ - разделить четырехугольники и выделить более общий подтип для обеих фигур.

Несмотря на кажущееся сходство квадрата и прямоугольника, они разные.
Квадрат имеет много общего с ромбом, а прямоугольник с параллелограммом, но они не являются подтипом.
Квадрат, прямоугольник, ромб и параллелограмм - это отдельные фигуры со своими собственными свойствами, хотя и схожими.

```php
interface Shape
{
    public function getArea(): int;
}

class Rectangle implements Shape
{
    private $width = 0;
    private $height = 0;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square implements Shape
{
    private $length = 0;

    public function __construct(int $length)
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return $this->length ** 2;
    }
}

function printArea(Shape $shape): void
{
    echo sprintf('%s has area %d.', get_class($shape), $shape->getArea()).PHP_EOL;
}

$shapes = [new Rectangle(4, 5), new Square(5)];

foreach ($shapes as $shape) {
    printArea($shape);
}
```

**[⬆ вернуться к началу](#Содержание)**

### Принцип разделения интерфейса (Interface Segregation Principle, ISP)

Согласно ISP, "Клиенты не должны зависеть от интерфейсов, которые не используют".

Хороший пример демонстрации принципа: классы, для которых требуются большие объекты настроек (settings objects). Рекомендуется не требовать от клиентов настраивать много параметров, потому что по большей части они им не нужны. Если сделать их опциональными, то это поможет избежать раздутости интерфейса.

**Плохо:**

```php
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class RobotEmployee implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**Хорошо:**

Не каждый работник является сотрудником, но каждый сотрудник является работником.

```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class RobotEmployee implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```

**[⬆ вернуться к началу](#Содержание)**

### Принцип инверсии зависимостей (Dependency Inversion Principle, DIP)

Этот принцип гласит:

1. Высокоуровневые модули не должны зависеть от низкоуровневых. Оба вида должны зависеть от абстракций.
2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

Сначала это может быть трудным для понимания, но если вы работали с PHP-фреймворками (вроде Symfony), то уже встречались с реализацией этого принципа в виде инъекции зависимости (Dependency Injection, DI). Однако эти принципы не идентичны, DI ограждает высокоуровневые модули от деталей своих низкоуровневых модулей и их настройки. Это может быть сделано посредством DI. Огромное преимущество в том, что снижается сцепление (coupling) между модулями. Сцепление — очень плохой шаблон разработки, затрудняющий рефакторинг кода.

**Плохо:**

```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**Хорошо:**

```php
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**[⬆ вернуться к началу](#Содержание)**
