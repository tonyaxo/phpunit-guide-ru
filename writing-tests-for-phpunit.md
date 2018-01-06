# Написание тестов для PHPUnit

Пример 2.1 показывает как выгляит тест для проверки встроенных функий для работы с массивами. В этом примере представлены основные соглашения и шаги при написание тестов с помощью PHPUnit:

1.  Тесты для класса `Class` именуются как `ClassTest`.
2.  `ClassTest` наследуется (в большинстве случаев) от `PHPUnit\Framework\TestCase`.
3.  В качестве тестов выступают публичные методы класса начинающиеся с `test*`.
    Другой способ указать на тестовый метод - использовать `@test` аннотацию в docblock комментарие.
4.  Внутри тестового метода методы **утверждений**, такие как `assertEquals()`, используются чтобы проверить соответствие факстического и ожидаемого значения.

**Пример 2.1: Тестирование операций для работы с массивами с помощью PHPUnit**
```php
<?php
use PHPUnit\Framework\TestCase;

class StackTest extends TestCase
{
    public function testPushAndPop()
    {
        $stack = [];
        $this->assertEquals(0, count($stack));

        array_push($stack, 'foo');
        $this->assertEquals('foo', $stack[count($stack)-1]);
        $this->assertEquals(1, count($stack));

        $this->assertEquals('foo', array_pop($stack));
        $this->assertEquals(0, count($stack));
    }
}
?>
```
> Всякий раз, когда возникает соблазн вывести что-то с помощью `print` или отладчика, напишите его как тест.
    _- Martin Fowler_
    


## Зависисмоти

> Модульные тесты в первую очередь написаны как хорошая практика, помогающая разработчикам выявлять и исправлять ошибки, проводить рефакторинг кода и служить документацией для тестируемого модуля. 
Для достижения этих преимуществ unit-тесты в идеале должны охватывать все возможные пути в программе. Один тест обычно охватывает один конкретный путь в одной функции или методе. 
Однако, метод проверки не нужен инкапсулированному независимому объекту. Часто существуют неявные зависимости между методами тестирования скрытые в сценарии реализации теста.
_- Adrian Kuhn et. al._
  					
PHPUnit поддерживает декларацию явных зависимостей между методами тестирования. Такие зависимости не определяют порядок в котором должны выполняться тестовые методы, но они позволяют возвращать экземпляр тестовой фикстуры от производителя(producer) и передавать его зависимым потребителям(consumers).    

*   Производитель - это тестовый метод, который возвращает свой модуль тестирования в качестве возвращаемого значения.
*   Потребитель - это метод тестирования, который зависит от одного или нескольких производителей и их возвращаемых значений.

В примере 2.2 показано как использовать декларацию @depends для определения зависимостей между методами.

**Пример 2.2: Использование @depends для определения зависимостей между методами.**
```php
<?php
use PHPUnit\Framework\TestCase;

class StackTest extends TestCase
{
    public function testEmpty()
    {
        $stack = [];
        $this->assertEmpty($stack);

        return $stack;
    }

    /**
     * @depends testEmpty
     */
    public function testPush(array $stack)
    {
        array_push($stack, 'foo');
        $this->assertEquals('foo', $stack[count($stack)-1]);
        $this->assertNotEmpty($stack);

        return $stack;
    }

    /**
     * @depends testPush
     */
    public function testPop(array $stack)
    {
        $this->assertEquals('foo', array_pop($stack));
        $this->assertEmpty($stack);
    }
}
?>
```

В приведенном выше примере первый тест `testEmpty()` создает новый массив и проверяет что он пуст. Затем возвращает фикстуру в качестве результата. Второй тест `testPush()` зависит от `testEmpty()` и передается результат этого зависимого теста в качестве аргумента. Наконец, `testPop()` зависит от `testPush()`.

> Note: По умолчанию возвращаемое значение, полученное производителем, передается потребителям «как есть». Это означает что когда производитель возвращает объект, ссылка на этот объект передается потребителям. Когда вместо ссылки следует использовать копию, тогда вместо @depends следует использовать клон @depends.

Чтобы быстро локализовать дефекты мы хотим чтобы наше внимание было сосредоточено на неудачных тестах. Поэтому PHPUnit пропускает выполнение теста когда зависимый от него не прошел. Это улучшает локализацию дефектов, используя зависимости между тестами, как показано в примере 2.3.

**Пример 2.3: Использование зависимостей между тестами.**
```php
<?php
use PHPUnit\Framework\TestCase;

class DependencyFailureTest extends TestCase
{
    public function testOne()
    {
        $this->assertTrue(false);
    }

    /**
     * @depends testOne
     */
    public function testTwo()
    {
    }
}
?>
```
```
phpunit --verbose DependencyFailureTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

FS

Time: 0 seconds, Memory: 5.00Mb

There was 1 failure:

1) DependencyFailureTest::testOne
Failed asserting that false is true.

/home/sb/DependencyFailureTest.php:6

There was 1 skipped test:

1) DependencyFailureTest::testTwo
This test depends on "DependencyFailureTest::testOne" to pass.


FAILURES!
Tests: 1, Assertions: 1, Failures: 1, Skipped: 1.
```

Тест может содержать более одной аннотации `@depends`. PHPUnit не изменяет порядок выполнения тестов, вы должны убедиться, что зависимости теста могут быть выполнены до его запуска.

Тесты включающие больше одной аннотации `@depends` берут фикстуру от первой зависимости как первый аршумент, фикстуру от втрой зависсомости как второй аргумент и т. д. 

**Пример 2.4: Тест с несколькими зависимостями**
```php
<?php
use PHPUnit\Framework\TestCase;

class MultipleDependenciesTest extends TestCase
{
    public function testProducerFirst()
    {
        $this->assertTrue(true);
        return 'first';
    }

    public function testProducerSecond()
    {
        $this->assertTrue(true);
        return 'second';
    }

    /**
     * @depends testProducerFirst
     * @depends testProducerSecond
     */
    public function testConsumer()
    {
        $this->assertEquals(
            ['first', 'second'],
            func_get_args()
        );
    }
}
?>
```
```
phpunit --verbose MultipleDependenciesTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

...

Time: 0 seconds, Memory: 3.25Mb

OK (3 tests, 3 assertions)
```

## Провайдеры данных

Тестовый метод может принимать произвольные аргументы. 
Эти аргументы могут быть получены с помощью метода провайдера данных(`additionProvider()` см. [Пример 2.5](#ex-2-5)). 
Используемый поставщик данных, указывается с помощью аннотации `@dataProvider`.

Метод поставщика данных должен быть `public` и либо возвращать массив массивов, либо объект, который реализует интерфейс `Iterator`, и возвращает массив для каждого шага итерации.

**Пример 2.5: Использование провайдера данных возвращающего массив массивов**
```php
<?php
use PHPUnit\Framework\TestCase;

class DataTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        $this->assertEquals($expected, $a + $b);
    }

    public function additionProvider()
    {
        return [
            [0, 0, 0],
            [0, 1, 1],
            [1, 0, 1],
            [1, 1, 3]
        ];
    }
}
?>
```
```
phpunit DataTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set #3 (1, 1, 3)
Failed asserting that 2 matches expected 3.

/home/sb/DataTest.php:9

FAILURES!
Tests: 4, Assertions: 4, Failures: 1.
```

При использовании больших массивов данных можно использовать ассоцитиативные массивы. Вывод будет более подробным, так как он будет содержать ключ массива, на котором останавливается тест.

**Пример 2.6: Использование провайдера с именованными данными**
```php
<?php
use PHPUnit\Framework\TestCase;

class DataTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        $this->assertEquals($expected, $a + $b);
    }

    public function additionProvider()
    {
        return [
            'adding zeros'  => [0, 0, 0],
            'zero plus one' => [0, 1, 1],
            'one plus zero' => [1, 0, 1],
            'one plus one'  => [1, 1, 3]
        ];
    }
}
?>
```
```
phpunit DataTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set "one plus one" (1, 1, 3)
Failed asserting that 2 matches expected 3.

/home/sb/DataTest.php:9

FAILURES!
Tests: 4, Assertions: 4, Failures: 1.
```

**Пример 2.7: Использование провайдера данных возврщающего `Iterator`**
```php
<?php
use PHPUnit\Framework\TestCase;

require 'CsvFileIterator.php';

class DataTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        $this->assertEquals($expected, $a + $b);
    }

    public function additionProvider()
    {
        $file = new SplFileObject("animals.csv");
        $file->setFlags(SplFileObject::READ_CSV);
        return $file;
    }
}
?>
```
```
phpunit DataTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set #3 ('1', '1', '3')
Failed asserting that 2 matches expected '3'.

/home/sb/DataTest.php:11

FAILURES!
Tests: 4, Assertions: 4, Failures: 1.
```

Когда тест получает входные данные как от `@dataProvider`, так и от одного или нескольких тестов, от которых он зависит `@depends`, то аргументы от поставщика данных будут идти до аргументов полученных от зависимых тестов. Апи этом аргументы от зависимых тестов будут одинаковыми для каждого набора данных провайдера. См. Пример 2.9

**Пример 2.9: Одновеменное использование `@depends` и `@dataProvider` в одном тесте**
```php
<?php
use PHPUnit\Framework\TestCase;

class DependencyAndDataProviderComboTest extends TestCase
{
    public function provider()
    {
        return [['provider1'], ['provider2']];
    }

    public function testProducerFirst()
    {
        $this->assertTrue(true);
        return 'first';
    }

    public function testProducerSecond()
    {
        $this->assertTrue(true);
        return 'second';
    }

    /**
     * @depends testProducerFirst
     * @depends testProducerSecond
     * @dataProvider provider
     */
    public function testConsumer()
    {
        $this->assertEquals(
            ['provider1', 'first', 'second'],
            func_get_args()
        );
    }
}
?>
```
```
phpunit --verbose DependencyAndDataProviderComboTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 3.50Mb

There was 1 failure:

1) DependencyAndDataProviderComboTest::testConsumer with data set #1 ('provider2')
Failed asserting that two arrays are equal.
--- Expected
+++ Actual
@@ @@
Array (
-    0 => 'provider1'
+    0 => 'provider2'
1 => 'first'
2 => 'second'
)

/home/sb/DependencyAndDataProviderComboTest.php:31

FAILURES!
Tests: 4, Assertions: 4, Failures: 1.
```

> Note: Когда тест зависит от теста, использующего поставщик данных, 
зависящий тест будет выполняться, когда тест, от которого он зависит, успешно выполняется, 
по крайней мере, для одного набора данных. Результат теста, который использует поставщик данных, не может быть внедрен в зависимый тест.

> Note: Все поставщики данных выполняются перед вызовом статического метода `setUpBeforClass` и первым вызовом метода `setUp`. 
Из-за этого вы не можете получить доступ к любым переменным, которые вы создаете внутри поставщика данных. Это необходимо для того, чтобы phpunit мог вычислить общее количество тестов.

## Тестирование исключений

В примере 2.10 показано как использовать метод `expectException()` для проверки генерации исключения в тестируемом коде.

**Пример 2.10: Использование метода `expectException()`**
```php
<?php
use PHPUnit\Framework\TestCase;

class ExceptionTest extends TestCase
{
    public function testException()
    {
        $this->expectException(InvalidArgumentException::class);
    }
}
?>
```
```
phpunit ExceptionTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

F

Time: 0 seconds, Memory: 4.75Mb

There was 1 failure:

1) ExceptionTest::testException
Expected exception InvalidArgumentException

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

Помимло метода `expectException()` так же существуют другие методы для проверки генерации исключений `expectExceptionCode()`, `expectExceptionMessage()` и `expectExceptionMessageRegExp()`. 

Также вы можете использовать аннотации `@expectedException`, `@expectedExceptionCode`, `@expectedExceptionMessage` и `@expectedExceptionMessageRegExp` как показано в примере 2.11ю

**Пример 2.11: Использование аннтотации `@expectedException`**
```php
<?php
use PHPUnit\Framework\TestCase;

class ExceptionTest extends TestCase
{
    /**
     * @expectedException InvalidArgumentException
     */
    public function testException()
    {
    }
}
?>
```
```
phpunit ExceptionTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

F

Time: 0 seconds, Memory: 4.75Mb

There was 1 failure:

1) ExceptionTest::testException
Expected exception InvalidArgumentException

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

## Тестирование ошибок PHP

По умолчанию PHPUnit конвертирует PHP `errors`, `warnings` и `notices`, возникающие во время выполнения, в исключения.
Используя эти исключения вы можете, например, ожидать что тест вызовет ошибку php как показано в примере 2.12.

> Note: Так же можно использовать `error_reporting` чтобы указать какие ошибки PHPUnit будет конвертировать. 
Если у вас возникли проблемы с этой функцией убедитесь что php не настроен на подавление типа ошибок которые вы тестируете.

**Пример 2.12: Обработка ошибок PHP используя аннтотацию `@expectedException`**
```php
<?php
use PHPUnit\Framework\TestCase;

class ExpectedErrorTest extends TestCase
{
    /**
     * @expectedException PHPUnit_Framework_Error
     */
    public function testFailingInclude()
    {
        include 'not_existing_file.php';
    }
}
?>
```
```
phpunit -d error_reporting=2 ExpectedErrorTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

.

Time: 0 seconds, Memory: 5.25Mb

OK (1 test, 1 assertion)
```

`PHPUnit_Framework_Error_Notice` and `PHPUnit_Framework_Error_Warning` соответственно представляют PHP notices и warnings. 

> Note: Будьте более конкретными при тестировании исключений. Тестирование слишком общих классов может привести к нежелательным побочным эффектам. Поэтому использование `@expectedException` или `setExpectedException()` для тестирования класса `Exception` не разрешено.

В тестах которые зависят php функций которые вызывают ошибки, такие как `fopen`, иногда бывает полезно использовать подавление ошибок. Это позволяет вам проверять возвращаемые значения, подавляя уведомления, которые приведут к `PHPUnit_Framework_Error_Notice`.

**Пример 2.13: Проверка возвращаемого значения в коде использующем php-ошибки**
```php
<?php
use PHPUnit\Framework\TestCase;

class ErrorSuppressionTest extends TestCase
{
    public function testFileWriting() {
        $writer = new FileWriter;
        $this->assertFalse(@$writer->write('/is-not-writeable/file', 'stuff'));
    }
}
class FileWriter
{
    public function write($file, $content) {
        $file = fopen($file, 'w');
        if($file == false) {
            return false;
        }
        // ...
    }
}

?>
```
```
phpunit ErrorSuppressionTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

.

Time: 1 seconds, Memory: 5.25Mb

OK (1 test, 1 assertion)
```

Без подавления ошибок тест завершиться с ошибкой ` fopen(/is-not-writeable/file): failed to open stream: No such file or directory`

## Тестирование вывода

Иногда нужно проверить результат выполенения метода сгенертрованный в стандартный вывод (например, через `echo` или `print`).
Данная возможность осущетсвляется классом `PHPUnit\Framework\TestCase` с помощью [Функции контроля вывода](http://php.net/manual/ru/ref.outcontrol.php).
В примере 2.14 показано как проверить ожидаевый результат вывода с помощью функции `expectOutputString()`. Вслучае если ожидаемый вывод не сгенерирован, тест считается не проеденным.

**Пример 2.14: Проверка результата вывода для функции или метода**
```php
<?php
use PHPUnit\Framework\TestCase;

class OutputTest extends TestCase
{
    public function testExpectFooActualFoo()
    {
        $this->expectOutputString('foo');
        print 'foo';
    }

    public function testExpectBarActualBaz()
    {
        $this->expectOutputString('bar');
        print 'baz';
    }
}
?>
```
```
phpunit OutputTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

.F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) OutputTest::testExpectBarActualBaz
Failed asserting that two strings are equal.
--- Expected
+++ Actual
@@ @@
-'bar'
+'baz'


FAILURES!
Tests: 2, Assertions: 2, Failures: 1.
```

В таблице 2.1 представлен список методов для тестирования вывода.

**Таблица 2.1: Методы для тестирования вывода**

| Метод                                               | Описание                                                                   |
|-----------------------------------------------------|----------------------------------------------------------------------------|
| `void expectOutputRegex(string $regularExpression)` | Сопоставляет ожидаемый вывод с регулярным выражением `$regularExpression`. |
| `void expectOutputString(string $expectedString)`   | Сопоставляет ожидаемый вывод со строкой `$expectedString`.                 |
| `bool setOutputCallback(callable $callback)`        | Устанавливает функцию для нормализации текущего вывода.                    |
| `string getActualOutput()`                          | Возвращает текущее значение вывода.                                        |

> Тесты которые генерируют вывод не будут пройдены в `strict mode`.

## Вывод ошибок

Всякий раз когда тест не проходит PHPUnit старается передоставить наиболее подходящий контекст для определения проблемы.

**Пример 2.15: Генерация ошибке для не пройденного теста сравнения массивов**
```php
<?php
use PHPUnit\Framework\TestCase;

class ArrayDiffTest extends TestCase
{
    public function testEquality() {
        $this->assertEquals(
            [1, 2,  3, 4, 5, 6],
            [1, 2, 33, 4, 5, 6]
        );
    }
}
?>
```
```
phpunit ArrayDiffTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) ArrayDiffTest::testEquality
Failed asserting that two arrays are equal.
--- Expected
+++ Actual
@@ @@
 Array (
     0 => 1
     1 => 2
-    2 => 3
+    2 => 33
     3 => 4
     4 => 5
     5 => 6
 )

/home/sb/ArrayDiffTest.php:7

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

В данном примере только один элемент массива не совпадает, остальные элементы представленны оучшего представления причины возникновения ошибки.
Если сгенерированый вывод слишком длинный, PHPUnit  разделяет его на несколько строк в контексте каждой ошибки.

**Пример 2.16: Генерация ошибке для не пройденного теста сравнения больших массивов**
```php
<?php
use PHPUnit\Framework\TestCase;

class LongArrayDiffTest extends TestCase
{
    public function testEquality() {
        $this->assertEquals(
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2,  3, 4, 5, 6],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 33, 4, 5, 6]
        );
    }
}
?>
```
```
phpunit LongArrayDiffTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) LongArrayDiffTest::testEquality
Failed asserting that two arrays are equal.
--- Expected
+++ Actual
@@ @@
     13 => 2
-    14 => 3
+    14 => 33
     15 => 4
     16 => 5
     17 => 6
 )


/home/sb/LongArrayDiffTest.php:7

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

### Особые случаи

В случае несовпадения сравниваемых значений PHPUnit создает текстовые представления входных значений и сравнивает их. Из-за такой реализации разница может показывать больше проблем чем их реально существует.
Это происходит только при использовании `assertEquals()` или других «слабых» функций сравнения для массивов или объектов.

**Пример 2.16: Особый случай при генерации сообщения о различие при использование слабого сравнения**
```php
<?php
use PHPUnit\Framework\TestCase;

class ArrayWeakComparisonTest extends TestCase
{
    public function testEquality() {
        $this->assertEquals(
            [1, 2, 3, 4, 5, 6],
            ['1', 2, 33, 4, 5, 6]
        );
    }
}
?>
```
```
phpunit ArrayWeakComparisonTest
PHPUnit 6.1.0 by Sebastian Bergmann and contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) ArrayWeakComparisonTest::testEquality
Failed asserting that two arrays are equal.
--- Expected
+++ Actual
@@ @@
 Array (
-    0 => 1
+    0 => '1'
     1 => 2
-    2 => 3
+    2 => 33
     3 => 4
     4 => 5
     5 => 6
 )


/home/sb/ArrayWeakComparisonTest.php:7

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

В этом примере показана разница в первом индексе между `1` и `'1'` хотя функция `assertEquals` считает их одинаковыми. 
