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


