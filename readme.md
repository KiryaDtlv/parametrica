# Metaconfig - ru
<br>
Python модуль для удобной работы с конфигурацией приложений.
Создан с целью минимизировать время на описание конфигурации приложения, и предоставить фронтенду метаданные о модели конфигурации для автоматического создания соответствующего UI.
<br>
## Пример использования
<br>
```
# Импортируем необходимые классы из модуля
from metaconfig import IntField, Fieldset, ConfigRoot, ListField, StrField, BoolField, JsonFileConfigIO


# Создадим набор полей, содержащий поля хост и порт
class HostPort(Fieldset):
    # Определим поле host как StrField с названием "Хост", подсказкой "IP адрес" и значеним по-умолчанию 0.0.0.0
    host = StrField(label='Хост', default='0.0.0.0', hint="IP адрес")
    port = IntField(label='Порт', default=11000)


# Создадим другой набор полей
class Server(Fieldset):
    location = StrField(label="Местонахождение сервера", default='', hint="Страна")
    # Используем определенный ранее набор полей с дополнительным пояснением
    connection_data = HostPort(label='Данные для подключения')


# Создадим корневой узел конфигурации 
class Config(ConfigRoot):
    __io_class__ = JsonFileConfigIO('proxy.settings')

    # Объявим список серверов
    proxy_pool = ListField(
            Server(
                label="Прокси-сервер", 
                default={
                    'location': 'USA',
                    'connection_data': {
                        'host': 'proxy.google.com',
                        'port': '3128'
                    }
                }
            ),
            label='Пул прокси-серверов',
            default=[]
        )

    # и адрес собственного сервера
    home = Server(
                label='Домашний сервер', 
                default={
                    'location': 'Russia', 
                    'connection_data': {
                        'host': 'home.server.com', 
                        'port': 4123
                    }
                }
            )

    # а так же параметр, разрешающий отключить использование прокси
    use_proxy = BoolField(label='Использовать прокси', default=True)


# Проинициализируем получившуюся конфигурацию
config = Config()

# Получим из нее конкретное значение (автокомплиты нам помогут)
print(config.home.location)
# Получим все данные из конфигурации
print(config.as_dataset())
# Получим метаданные о конфигурации
print(config.as_metadata())
# Точечно обновим один из параметров
config.update({'use_proxy': False})


# print(config.as_dataset())
# print(json.dumps(config.as_dataset(), indent=2,ensure_ascii=False))

```

## Сущности

### Поле

Базовая сущность. Ее свойства - это тип переносимых данных, человекочитаемое название, описание, текущее значение и значение по-умолчанию.

### Список полей

Сущность, позволяющая хранить в себе любое количество полей одинакового типа. Также является полем.

### Набор полей

Сущность для компановки нескольких полей в самостоятельный объект.
Ее свойства - это экземпляры полей. Также является полем, что позволяет ей содержать в качестве свойств экземпляры самой себя.
<br>
## Типы

### metaconfig.types.MetaFieldClass

Базовый класс поля. Реализует в себе абстрактный функционал по хранению метаданных и значений.
Является общим предком для всех классов иерархии.
Предоставляет унифицированный интерфейс для создания экземпляра поля, чтения, сериализации, валидации и записи его значения.

**НЕ ПРЕДНАЗНАЧЕНО ДЛЯ САМОСТОЯТЕЛЬНОГО ИСПОЛЬЗОВАНИЯ**
<br>
#### metaconfig.types.MetaFieldClass.validate(value)

Метод для валидации значения перед записью.
На вход принимает значение, которое подлежит записи.
В случае успеха возвращает None, в случае ошибки валидации поднимает ValueError

#### metaconfig.types.MetaFieldClass.normalize(value)

Метод для нормализации значения перед записью.
На вход принимает значение, которое подлежит записи.
Возвращает нормализованное значение.
Перед выполнением вызывает validate.

#### metaconfig.types.MetaFieldClass.as\_metadata() -> dict

Метод для иерархичной генерации метаданных о поле(ях).
Возвращает dict, содержащий все свойства, присущие полю.

#### metaconfig.types.MetaFieldClass.as\_dataset()

Метод для иехархичного получения данных из поля(ей).
Тип возвращаемого значения определяется реализацией.
<br>
### metaconfig.types.IntField

Поле
Наследуется от MetaFieldClass
Реализует поле с целочисленным значением.
Поддерживает автоматическую нормализацию из строки, если это возможно.

### metaconfig.types.StrField

Поле
Наследуется от MetaFieldClass
Реализует поле со строковым значением.
Приведет любое установленное значение к строке.

### metaconfig.types.FloatField

Поле
Наследуется от MetaFieldClass
Реализует поле со значением с плавающей точкой.
Поддерживает нормализацию из строки (если возможно) и целочисленного значения

### metaconfig.types.BoolField

Поле
Наследуется от MetaFieldClass
Реализует поле с булевым значением.
Поддерживает семантическую реализацию из целого числа и строки по правилам:
`value = int(value) != 0 || value = str(value).lower() == 'true'`

### metaconfig.types.ListField

Список полей
Наследуется от MetaFieldClass
Реализует хранилище для нескольких полей одинакогово типа
Обязательным аргуементом для инициализации является объект наследника MetaFieldClass для определения типа хранимых полей.

### metaconfig.types.Fieldset

Набор полей
Наследуется от MetaFieldClass
Реализует структурную единицу описания конфигурации - модель.
Не подлежит самостоятельному использованию.
Должна являться родительским классом для пользовательских классов, реализующих конкретную структуру конфигурации.

### metaconfig.types.ConfigRoot

Набор полей
Синглтон
Наследуется от Fieldset
Реализует корневую модель конфигурации.
Не подлежит самостоятельному использованию.
Реализует абстрактный функционал по чтению/записи описанной в классе-наследнике структуры конфигурации.
Экземпляр класса-наследника будет являться точкой входа для взаимодействия клиентского кода со структурой конфигурации.

#### metaconfig.types.ConfigRoot.\_\_io\_class\_\_

Статическое поле, подлежащее записи в классе-наследнике ConfigRoot
В качестве значения разрешается экземпляр metaconfig.io.ConfigIoInterface
Если не определен в классе-наследнике, то будет использован metaconfig.io.JsonFileConfiIO по-умолчанию.

### metaconfig.io.ConfigIOInterface

Интерфейс для чтения/записи параметров конфигурации.

#### metaconfig.io.ConfigIOInterface.parse(data: str) -> dict

Абстрактный метод, который должен реализовать парсинг сырых данных в словарь, соответствующий описанной структуре конфигурации.
На вход принимает data:str - сырые данные
Должен вернуть dict - данные, соответствующие структуре конфигурации.

#### metaconfig.io.ConfigIOInterface.serialize(dataset: dict) -> str

Абстрактный метод, который должен реализовать сериализацию текущих данных конфигурации в сырые данные, подлежащие записи на постоянный носитель.
На вход принимает dataset: dict - текущие данные конфигурации
Возвращает str - сырые данные.

#### metaconfig.io.ConfigIOInterface.read() -> dict

Абстрактный метод, который должен реализовать алгоритм чтения сырых данных.
Должен вызывать parse()
Должен вернуть dict, соответствующий структуре конфигурации.

#### metaconfig.io.ConfigIOInterface.write(dataset: dict)

Абстрактный метод, который дложен реализовать алгоритм записи структуры конфигурации в постоянное хранилище.
Должен вызывать serialize()
На вход принимает dataset: dict, содержащий все текущие значения конфигурации.

### metaconfig.io.JsonFileConfigIO

Наследник ConfigIOInterface
Реализует работу с файлов на жестком диске в формате JSON