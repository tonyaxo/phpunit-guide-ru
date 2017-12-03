# Организация тестов

Одна из целей PHPUnit заключается в том, что тесты должны быть скомпонованы: мы хотим иметь возможность запускать любое количество или комбинацию тестов вместе, 
например, все тесты для всего проекта или тесты для всех классов компонента, который является частью проект или просто тесты для одного класса.

PHPUnit поддерживает различные способы организации тестов и объединения их в тестовые наборы. В этой главе пойдет речь о наиболле часто используемх подходах.

## Организация тестовых наборов используя файловую систему

Вероятно, самый простой способ хранить набор тестов - держать исходные файлы тестовых примеров в тестовом каталоге. 
PHPUnit может автоматически обнаруживать и запускать тесты путем рекурсивного прохождения каталога.

Давайте взглянем на тестовый набор следующей библиотеки [sebastianbergmann/money](http://github.com/sebastianbergmann/money/).
В структуре директорий данного проекта мы видим что все тестовые классы расположенные в директории `tests` отражают структуру пакета и классы тестируемой системы (System Under Test - SUT) директории `src`.
```
src                                 tests
`-- Currency.php                    `-- CurrencyTest.php
`-- IntlFormatter.php               `-- IntlFormatterTest.php
`-- Money.php                       `-- MoneyTest.php
`-- autoload.php
```

Для того чтобы запустить все тесты данной библиотеки нам необходимо указать PHPUnit на директорию `tests`.
```
phpunit --bootstrap src/autoload.php tests
PHPUnit 6.1.0 by Sebastian Bergmann.

.................................

Time: 636 ms, Memory: 3.50Mb

OK (33 tests, 52 assertions)
```

> Note: Когда вы указываете директорию для запуска тестов PHPUnit будет автоматически искать в ней файлы `*Test.php`.

Чтобы запустить только тесты из класса `CurrencyTest` находящегося в файле `tests/CurrencyTest.php` используется следующая команда:

```
phpunit --bootstrap src/autoload.php tests/CurrencyTest
PHPUnit 6.1.0 by Sebastian Bergmann.

........

Time: 280 ms, Memory: 2.75Mb

OK (8 tests, 8 assertions)
```

Для более точного управления запуском необходимых тестов используется опция `--filter`:

```
phpunit --bootstrap src/autoload.php --filter testObjectCanBeConstructedForValidConstructorArgument tests
PHPUnit 6.1.0 by Sebastian Bergmann.

..

Time: 167 ms, Memory: 3.00Mb

OK (2 test, 2 assertions)
```

> Note: Недостатком такого подхода является то, что мы не контролируем порядок выполнения тестов. 
  Это может привести к проблемам с зависимостями, см. Раздел «[Зависисмости](writing-tests-for-phpunit.md#Зависисмоти)». 
  В следующем разделе вы увидите, как явно задать порядок выполнения тестов используя xml файл конфигурации.
  
## Организация тестовых наборов используя XML конфигурацию

Для организации тестовых наборов PHPUnit моно использовать файл XML конфигурации. В примере 5.1 показана структура минимального файла `phpunit.xml`,
который добавляет все классы `*Test` найденные в файлах `*Test.php` при рекурсивном прохождении директории `test`.

**Пример 5.1: Организация тестовых наборов используя XML конфигурацию.**
```xml
<phpunit bootstrap="src/autoload.php">
  <testsuites>
    <testsuite name="money">
      <directory>tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

В случае если в текущей рабочей директории существует один из файлов `phpunit.xml` или `phpunit.xml.dist` (в данном порядке) и _не_ используется 
опция `--configuration`, то конфигурация будет автоматически взята из этого файла.

Порядок выполнения тестов может быть указан явным образом:

**Пример 5.2: Организация тестовых наборов используя XML конфигурацию.**
```xml
<phpunit bootstrap="src/autoload.php">
  <testsuites>
    <testsuite name="money">
      <file>tests/IntlFormatterTest.php</file>
      <file>tests/MoneyTest.php</file>
      <file>tests/CurrencyTest.php</file>
    </testsuite>
  </testsuites>
</phpunit>
```
