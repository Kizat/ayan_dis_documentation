<div align="center">  

# **Документация DIS курьер**

</div>

- [**Документация DIS курьер**](#документация-dis-курьер)
- [State Management](#state-management)
  - [GetX](#getx)
- [Экран входа](#экран-входа)
    - [Выбор сервера](#выбор-сервера)
    - [Выбор роли сотрудника](#выбор-роли-сотрудника)
- [Профиль](#профиль)
  - [Информация о сотруднике](#информация-о-сотруднике)
  - [Cмена сотрудника](#cмена-сотрудника)
  - [Кнопки для смены](#кнопки-для-смены)
    - [Кнопка открытия смены](#кнопка-открытия-смены)
    - [Кнопка выхода на перерыв](#кнопка-выхода-на-перерыв)
    - [Кнопка возвращения на вмену](#кнопка-возвращения-на-вмену)
    - [Кнопка закрытия смены](#кнопка-закрытия-смены)
  - [Календарь выходов на смену за текущий месяц](#календарь-выходов-на-смену-за-текущий-месяц)
  - [Начисление за отработанные часы, премия за количество заказов, начисление за отдельный период](#начисление-за-отработанные-часы-премия-за-количество-заказов-начисление-за-отдельный-период)
- [Комплектовщик](#комплектовщик)
- [Курьер](#курьер)
- [Заказы](#заказы)
  - [Активные заказы комплектовщика](#активные-заказы-комплектовщика)
  - [Выполненные заказы комплектовщика](#выполненные-заказы-комплектовщика)
  - [Активные заказы курьера](#активные-заказы-курьера)
  - [Выполненные заказы курьера](#выполненные-заказы-курьера)
  - [Карта](#карта)
  - [Изменение статуса](#изменение-статуса)
  - [Получение продуктов в заказе](#получение-продуктов-в-заказе)
  - [Таймер при сборке у комплектовщиков](#таймер-при-сборке-у-комплектовщиков)
  - [Сканирование штрихкода](#сканирование-штрихкода)
  - [Количество совпадает с количеством заказанного продукта](#количество-совпадает-с-количеством-заказанного-продукта)
  - [Если нет соответствующего продукта или количество меньше заказанного](#если-нет-соответствующего-продукта-или-количество-меньше-заказанного)
  - [Добавление аналогых продуктов](#добавление-аналогых-продуктов)
  - [Удаление аналогового продукта](#удаление-аналогового-продукта)
  - [Если нет заказанного продукта и нет для продукта аналога](#если-нет-заказанного-продукта-и-нет-для-продукта-аналога)


# State Management

- В приложении за управлением состоянии отвечает:
  ## GetX
    - Базовая конфигурация контроллеров разделена на **2** типа:
      1. ### BaseObxController;
         - `BaseObxController` – наблюдаемое управление состоянием, то есть контроллер слежения независящая от состояния виджета и использующее реактивные переменные, простыми словами, если изменить значение переменного, он автоматический поменяется везьде где используется эта переменная
      2. ### BaseGetxController.
         - `BaseGetxController` - управление состоянием, состояние которого меняется "в ручную" с командой `update()`, не управляет реактивными переменнами 


```dart
  class BaseObxController<T> extends GetxController {
    final Rxn<T> _result = Rxn<T>();
    final RxnString _error = RxnString();
    final RxBool _loading = RxBool(false);

    void setResult({required T? data}) => _result.value = data;
    void setError({String? error}) => _error.value = error;
    void setLoading(bool value) => _loading.value = value;

    T? get result => _result.value;

    String? get error => _error.value;

    bool get loading => _loading.value;
}
```

```dart
class BaseGetxController<T> extends GetxController {
  T? result;
  String? error;
  bool loading = false;

  void setResult({required T? data}) {
    result = data;
  }

  void setError({String? error}) {
    this.error = error;
  }

  void setLoading(bool value) {
    loading = value;
  }
}
```


# Экран входа

<div align="center"><img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/login.png?raw=true" height="500"/></div>

- Для входа в учетную запись требуется следующее API:

```dart
  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  String get endPoint => "/api/employee/auth/signin";

  @override
  get body =>{
        "phone": number,
        "password": password,
      };
  
```

### Выбор сервера

- Выбор сервера производится зажатием кнопки "Войти" на экране входа

<div align="center"><img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/choose_server.png?raw=true" height="500"/></div>

### Выбор роли сотрудника
  - Выбор роли усуществляется со следующего API во время входа:

```dart

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

  @override
  String get endPoint => "/api/employee/auth/token";

  @override
  Future<int?> onSuccess({required body, Function(String error)? onDataError}) async => body['data']['id'];

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

```

- из полученного тела метода извлекается `id` роли сотрудника – `body['data']['id']`, после этого определяется экран сотрудника [Комплектовщик](#комплектовщик) или [Курьер](#курьер)


# Профиль

<div align="center"><img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/profile.png?raw=true" height="500"/></div>


- В профиле сотрудника присутствуют следующие информации:
  - [Информация о сотруднике](#информация-о-сотруднике);
  - [Cмена сотрудника](#cмена-сотрудника):
    - Адрес работы;
    - Статус текущей смены, время начало и конец смены, количество перерывов, время перерыва;
  - [Кнопки для смены](#кнопки-для-смены):
    - [Кнопка открытия смены](#кнопка-открытия-смены);
    - [Кнопка выхода на перерыв](#кнопка-выхода-на-перерыв);
    - [Кнопка возвращения на вмену](#кнопка-возвращения-на-вмену)
    - [Кнопка закрытия смены](#кнопка-закрытия-смены).
  - [Календарь выходов на смену за текущий месяц количество отработанных смен, количество отработанных часов, количество собранных заказов](#календарь-выходов-на-смену-за-текущий-месяц):
  - [Начисление за отработанные часы, премия за количество заказов, начисление за отдельный период](#начисление-за-отработанные-часы-премия-за-количество-заказов-начисление-за-отдельный-период);
  - [Карта](#карта)


## Информация о сотруднике

- Информацию о сотруднике мы получаем со следующего API:
  
```dart

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

  @override
  String get endPoint => "/api/employee/profile/get";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};


```

## Cмена сотрудника

- Смену сотрудника(Адрес работы, Статус текущей смены, время начало и конец смены, количество перерывов, время перерыва) получаем со следующего API:

```dart

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

  @override
  String get endPoint => "/api/employee/shift/actual";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};


```


## Кнопки для смены




<p float="left">
  <img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/choose_server.png?raw=true" width="24%" alt="Запланированная смена"/>
  <img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/открытая_смена.png?raw=true" width="24%" alt="Открытая смена"/> 
  <img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/Перерыв.png?raw=true" width="24%"  alt="Перерыв"/>
  <img src="https://github.com/Kizat/ayan_dis_documentation/blob/main/screenshots/Досрочно_закрытая_смена.png?raw=true" width="24%" alt="Досрочно закрытая смена"/>
</p>

### Кнопка открытия смены

- За кнопкой для открытия смены отвечает следующее API:

```dart
  @override
  String get endPoint => "/api/employee/shift/open";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;
```

### Кнопка выхода на перерыв

- За кнопкой для перерыва отвечает следующее API:

```dart
  @override
  String get endPoint => "/api/employee/shift/break/start";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;
```

### Кнопка возвращения на вмену

- За кнопкой для возвращения на смену отвечает следующее API:

```dart
  @override
  String get endPoint => "/api/employee/shift/break/stop";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;
```

### Кнопка закрытия смены

- За кнопкой для закрытия смены отвечает следующее API:

```dart
  @override
  String get endPoint => "/api/employee/shift/close";

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;
```


## Календарь выходов на смену за текущий месяц

- Для получения отработанных дней за месяц, а также количество отработанных смен, количество отработанных часов, количество собранных заказов отвечает тот же API за которую отвечает [информация о сотруднике](#информация-о-сотруднике)

## Начисление за отработанные часы, премия за количество заказов, начисление за отдельный период

- Для получения начисление за отработанные часы, премия за количество заказов, начисление за отдельный период отвечает тот же API за которую отвечает [информация о сотруднике](#информация-о-сотруднике)



# Комплектовщик

- Роль комплектовщика определяется с помощью API `/api/employee/auth/token`, из возвращаемого тела метода извлекается `body['data']['id']`. см. [Выбор роли сотрудника](#выбор-роли-сотрудника)


# Курьер

- Роль курьера определяется с помощью API `/api/employee/auth/token`, из возвращаемого тела метода извлекается `body['data']['id']`. см. [Выбор роли сотрудника](#выбор-роли-сотрудника)

# Заказы

- Список заказов мы получаем из следующего API:

```dart

  @override
  String get endPoint => "/api/employee/order/orders";

  @override
  get body => {
        "date": DateFormat('yyyy-MM-dd').format(date),
      };

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

```


## Активные заказы комплектовщика


- В список активных заказов комплектовщика попадает заказы со статусами с `1` по `4`:

```dart

    List<Order>? get activeList => orders
      ?.where((element) =>
          element.status?.id != null &&
          element.status!.id! >= 1 &&
          element.status!.id! <= 4)
      .toList();

```



## Выполненные заказы комплектовщика

- В список выполненных заказов комплектовщика попадает заказы со статусами от `5`:

```dart

  List<Order>? get completedList => orders
      ?.where(
          (element) => element.status?.id != null && element.status!.id! >= 5)
      .toList();

```



## Активные заказы курьера


- В список активных заказов курьерва попадает заказы со статусами с `5` по `7`:


```dart

  List<Order>? get activeList => orders
      ?.where((element) =>
          element.status?.id != null &&
          element.status!.id! >= 5 &&
          element.status!.id! <= 7)
      .toList();

```



## Выполненные заказы курьера


- В список выполненных заказов курьера попадает заказы со статусами от `8`:


```dart

  List<Order>? get completedList => orders
      ?.where(
          (element) => element.status?.id != null && element.status!.id! >= 8)
      .toList();

```


## Карта

- В карте у курьеров показываются [активные заказы](#активные-заказы-курьера) и повторяются методы

## Изменение статуса 

- За изменением статуса отвечает следующий API:

```dart

  @override
  String get endPoint => "/api/employee/order/status/edit";

  @override
  Future<Status?> onSuccess({required body, Function(String error)? onDataError}) async => await compute(
        statusFromJson,
        body['data']['status'],
      );

  @override
  QueryMethod get queryMethod => QueryMethod.putMethod;

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  get body => {"statusId": statusId, "orderId": orderId};

```


## Получение продуктов в заказе

- За получением продуктов в заказе отвечает следующий API:

```dart

  @override
  String get endPoint =>"/api/employee/product/listProductFromOrder/$orderId";

  @override
  Future<List<Product>?> onSuccess({required body, Function(String error)? onDataError}) async =>
      await compute(productsFromJson, body['data']);

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

```


## Таймер при сборке у комплектовщиков

- Чтобы получить таймер на сборку заказа используется следующий API:

```dart
  @override
  String get endPoint => "/api/admin/picker/get/timer";

  @override
  Map<String, dynamic>? get queryParameters => {
        "orderId": orderId,
      };

  @override
  Future<DateTime> onSuccess({required body, Function(String error)? onDataError}) async =>
      DateTime.parse(body['data']["endAt"]);

  @override
  QueryMethod get queryMethod => QueryMethod.getMethod;

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

```

## Сканирование штрихкода

- Чтобы отсканировать штриход продукта вызывается следующий API:

```dart

  @override
  String get endPoint => "/api/employee/product/findBarcode";

  @override
  get body => {
        'barcode': barcode,
        "orderProductId": orderProductId,
      };

  @override
  Future<ScannedBarcode?> onSuccess({required body, Function(String error)? onDataError}) async {
    // print(this.body);
    return await compute(barcodeFromJson, body['data']);
  }

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  Map<String, dynamic> get auth => {
        "TOKEN": "Bearer $token",
      };

```

## Количество совпадает с количеством заказанного продукта


- Если вы подобрали правильно штрихкод и количество совпадает с количеством заказанного продукта:

```dart


  @override
  String get endPoint => "/api/employee/product/confirmedQuantity";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  get body => {'count': amount, 'orderProductId': orderProductId};


```

## Если нет соответствующего продукта или количество меньше заказанного

```dart

  @override
  String get endPoint => "/api/employee/product/absence";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

  @override
  get body => {
        'amountClient': amountClient,
        'orderId': orderId,
        'productPervId': productId,
      };

```

– далее:

```dart

  @override
  String get endPoint => "/api/employee/product/changeCount";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  get body => {'count': count, 'orderProductId': orderProductId};

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

```

## Добавление аналогых продуктов

```dart


  @override
  String get endPoint => "/api/employee/product/changeProduct";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  get body => {
        'count': amount,
        'departmentId': departmentId,
        'orderProductId': orderProductId,
        'productNextId': productNextId
      };

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

```

## Удаление аналогового продукта

```dart

  @override
  String get endPoint => "/api/employee/product/deleteProduct";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  get body => {
        'analogProductId': analogProductId,
        'oldProductId': productId,
        'orderId': orderId
      };

  @override
  Map<String, dynamic> get auth => {"TOKEN": "Bearer $token"};

```

## Если нет заказанного продукта и нет для продукта аналога

```dart

  @override
  String get endPoint => "/api/employee/product/cancelStatusProduct";

  @override
  Future<bool?> onSuccess({required body, Function(String error)? onDataError}) async => true;

  @override
  QueryMethod get queryMethod => QueryMethod.postMethod;

  @override
  get body => {'orderId': orderId, 'productId': productId};

```