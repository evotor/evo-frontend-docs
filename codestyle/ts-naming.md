# Рекомендации по неймингу

**Для чего это всё**

Семантичность наименования сущностей сильно упрощает восприятие кода. Корректные названия дают понимание о содержимом и его типе или о действии, которое будет совершено, и его результате непосредственно в месте применения, избавляя от необходимости в просмотре декларации. 

### Булевы переменные

СЛЕДУЕТ использовать префикс `is` или `has`.
НЕЖЕЛАТЕЛЬНО использовать другие префиксы для булевых переменных и методов.
[Хорошая статья на эту тему](https://dev.to/michi/tips-on-naming-boolean-variables-cleaner-code-35ig).

**Исключение**

Данное правило не применяется для входящих свойств компонентов, отражающих их состояние:
- loader
- loading
- disabled
- expanded
- collapsed

### Массивы

Название списка сущностей НУЖНО формировать, используя маску:

`<существительное во множественном числе> + List`

В местах применения переменной это даст +10 к читаемости.

```
const characteristics: Array<Characteristic> = []; // плохо

const characteristicsList: Array<Characteristic> = []; // хорошо
```

### Количество, счётчики, индексы и номера

#### Для счётчиков и количества в общем случае используем маску:

`<существительное во множественном числе> + Count`

```
retriesCount = 0; // хорошо
basketItemsCount = 0; // хорошо
totalItemsCount = 0; // хорошо
```

#### Для индексов используем маску: 

`<существительное в единственном числе> + Index`

```
itemIndex = 0; // хорошо
pageNumber = 0; // некорректно, так как имеет значение 0

ordersNumber = 100; // плохо
orderNumber = 0; // допускается, но стоит учитывать, что номер может содержать не только цифры
```

#### Для номеров сущностей (страниц, заказов и тд) используем маску:

`<существительное в единственном числе> + Number`

**Замечение:** Постфикс `Number` СЛЕДУЕТ использовать только в случае, если это действительно **номер** чего-либо.

### Минимальное и максимальное значения

#### Для обозначения минимума и максимума используем префиксы `min` и  `max`, соответственно. 

```
const minDate; // хорошо
const maxRetriesCount; // хорошо
```

### Мапы и хэши

Типы данных, которые формируют маппинг или хэш, а также переменные, которые содержат такие значения СЛЕДУЕТ называть соответствующим образом:

#### Группировка списков сущностей по признаку:

`<сущность во множественном числе>By<признак в единственном числе>Map`

Примеры:

```
type ProductCharacteristics = Record<ProductCharacteristicType, ProductCharacteristic[]>; // плохо

type ProductCharacteristicsByTypeMap = Record<ProductCharacteristicType, ProductCharacteristic[]>; // хорошо

basketItems$: Observable<Record<ProductType, BasketItem[]>>; // плохо

basketItemsByTypeMap$: Observable<Record<ProductType, BasketItem[]>>; // ок

basketItemsByProductTypeMap$: Observable<Record<ProductType, BasketItem[]>>; // хорошо
```
  
#### Маппинг (хэш) сущностей 1 к 1 (например, по идентификатору)

```
<сущность во множественном числе>Map

или

<сущность во множественном числе>Hash
```

Примеры:

```
type CatalogItems = Record<CatalogItem['id'], CatalogItem>; // плохо

type CatalogItemsHash = Record<CatalogItem['id'], CatalogItem>; // хорошо
type CatalogItemsMap = Record<CatalogItem['id'], CatalogItem>; // тоже ок
```