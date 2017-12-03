## Недописанные и пропускаемые тесты

### Не законченные тесты

Когда вы создаете новый тестовый клас, вы скорее всего начнете с пустых тестовых методов, например:

```php
public function testSomething()
{
}
```
чтобы обозначить тесты которые вы должны написать. Проблема с такими тетовыми методами в том, что по умолчанию PHPUnit 
интерпретирует такие методы как `успешные`. Данное поведение делает отчеты о прохождение тестов неинформативными - вы не 
можете определить был ли тест успешно пройден или просто еще не реализован. Вызов `$this->fail()` в нереализованном 
тестовом методе тоже не помогает, так как в этом случае тест считается `не пройденным`. Что так же не верно как 
интерпритация не реализованного метода пройденным.

Если мы помечаем успешный тест как зеленый, а провалившийся тест как красный, то нам нужен дополнительный желтый цвето чтобы помечать
недописанные или не реализоавнные тесты.
`PHPUnit_Framework_IncompleteTest` используется для обозначения исключения, которое поднимается методом 
тестирования если тест является неполным или в настоящее время не реализован. Стандартной реализацией данного интерфейса
является `PHPUnit_Framework_IncompleteTestError`.

В примере 7.1 демонстриует класс `SampleTest`, в котором реализован один тестовый метод `testSomething()`. В данном методе вызывается
`markTestIncomplete()` (который автоматически генерирует исключение `PHPUnit_Framework_IncompleteTestError`) с помощью чего мы помечаем 
данный тест как не завершенный.

**Пример 7.1: Помечаем тест как не завершенный**
```php
<?php
use PHPUnit\Framework\TestCase;

class SampleTest extends TestCase
{
    public function testSomething()
    {
        // Не обязательно: тестируем како-то метод.
        $this->assertTrue(true, 'This should already work.');

        // Останавливаем выполнение и помечаем тест как не законченный.
        $this->markTestIncomplete(
          'This test has not been implemented yet.'
        );
    }
}
?>
```

Незавершенны метод обозначается с помошью `I` в выводе консольной утилиты PHPUnit, как показано в примере ниже:
```
phpunit --verbose SampleTest
PHPUnit 6.5.0 by Sebastian Bergmann and contributors.

I

Time: 0 seconds, Memory: 3.95Mb

There was 1 incomplete test:

1) SampleTest::testSomething
This test has not been implemented yet.

/home/sb/SampleTest.php:12
OK, but incomplete or skipped tests!
Tests: 1, Assertions: 1, Incomplete: 1.
```

В таблице 7.1 перечислены методы для обозначения теста как не законченного.

**Таблица 7.1: API для незавершенных методов**

| Метод                                     | Описание                                                                 |
|-------------------------------------------|--------------------------------------------------------------------------|
| `void markTestIncomplete()`               | Помечает текущий тест как незаконченный                                  |
| `void markTestIncomplete(string $message)`| Помечает текущий тест как незаконченный используя `$message` для описания|

### Пропуск тестов

Иногда нет смысла запускать некоторые тесты в поределенной среде. Для примера, расмотрим слой абстакции базы данных который
имеет несколько драйверов для поддержки различных СУБД. Естественно, тесты для MySQL должны быть запущены только если сервер 
MySQL доступен.

В примере 7.2 показан класс `DatabaseTest`, в котором реализован один тестовый метод `testConnection()`. В данном классе 
мы проверяем доступно ли расширение MySQL при помощи шаблонного метода `setUP()` и используем вызов `markTestSkipped()` 
для пропуска теста если это не так.

**Пример 7.2: Пропуск тестов**
```php
<?php
use PHPUnit\Framework\TestCase;

class DatabaseTest extends TestCase
{
    protected function setUp()
    {
        if (!extension_loaded('mysqli')) {
            $this->markTestSkipped(
              'Расширение MySQLi не доступно.'
            );
        }
    }

    public function testConnection()
    {
        // ...
    }
}
?>
```

Пропущенные тесты обозначается с помошью `S` в выводе консольной утилиты PHPUnit, как показано в примере ниже:
```
phpunit --verbose DatabaseTest
PHPUnit 6.5.0 by Sebastian Bergmann and contributors.

S

Time: 0 seconds, Memory: 3.95Mb

There was 1 skipped test:

1) DatabaseTest::testConnection
The MySQLi extension is not available.

/home/sb/DatabaseTest.php:9
OK, but incomplete or skipped tests!
Tests: 1, Assertions: 0, Skipped: 1.
```

В таблице 7.2 перечислены методы для обозначения теста как пропускаемого.

**Таблица 7.2: API для пропускаемых методов**

| Метод                                   | Описание                                                                 |
|-----------------------------------------|--------------------------------------------------------------------------|
| `void markTestSkipped()`                | Помечает текущий тест как пропускаемый                                   |
| `void markTestSkipped(string $message)` | Помечает текущий тест как пропускаемый используя `$message` для описания |

### Пропуск тестов используя @requires

В дополнение к вышеизложенным методам вы так же можете использовать аннотацию `@requires` для быстрого обозначения общих 
условий выполнения теста.

**Таблица 7.3: Варианты использования @requires**

| Тип         | Возможное значение                                                                                 | Пример                       | Другой пример                                      |
|-------------|----------------------------------------------------------------------------------------------------|------------------------------|----------------------------------------------------|
| `PHP`       | Любой идентификатор версии PHP                                                                     | @requires PHP 5.3.3          | @requires PHP 7.1-dev                              |
| `PHPUnit`   | Любой идентификатор версии PHPUnit                                                                 | @requires PHPUnit 3.6.3      | @requires PHPUnit 4.6                              |
| `OS`        | Регулярное выражение для [PHP_OS](http://php.net/manual/en/reserved.constants.php#constant.php-os) | @requires OS Linux           | @requires OS WIN32|WINNT                           |
| `function`  | Валидное значение для передачи в функцию [function_exists](http://php.net/function_exists)         | @requires function imap_open | @requires function ReflectionMethod::setAccessible |
| `extension` | Название расширения с необязательным идентификатором версии                                        | @requires extension mysqli   | @requires extension redis 2.2.0                    |

**Пример 7.3: Пропуск тестов с использованием @requires**
```php
<?php
use PHPUnit\Framework\TestCase;

/**
 * @requires extension mysqli
 */
class DatabaseTest extends TestCase
{
    /**
     * @requires PHP 5.3
     */
    public function testConnection()
    {
        // Test requires the mysqli extension and PHP >= 5.3
    }

    // ... All other tests require the mysqli extension
}
?>
```

Если вы используете синтаксис, который не компилируется с определенной версией PHP, посмотрите на 
конфигурацию xml для зависимостей [в разделе «Test Suites»]((writing-tests-for-phpunit.md#Зависисмоти))