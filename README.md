# PHP Style Guide() {
How to use: форкните и отредактируйте под нужды своей компании. 

## что-то
  - явное лучшее неявного
    ```php
    // ok
    $currentOffset = $pageNum * self::CHUNK_SIZE + $key;

    // good
    $currentOffset = ($pageNum * self::CHUNK_SIZE) + $key;
    ```
    
## сравнение
  - отношение времени указывается: прошлое < сейчас < будущее
  > Why? Мозг привыкает работать с воображаемым отрезком времени:  
  > вчера --------------сейчас------------ завтра
  
  > между startTime and expiredTime:  startTime --- < --- сейчас --- < expiredTime
  
  ```php
  // bad
  if ($now >= $hour24ago) {
    // ..
  }
  
  if ($startTime < $now && $expiredTime > $now) {
    // .. between startTime and expiredTime
  }
  
  if ($now > $expiredTime) {
    // .. expired time out
  }


  // good
  if ($hour24ago <= $now) {
    // ..
  }
  
  if ($startTime < $now && $now < $expiredTime) {
    // .. between startTime and expiredTime
  }
  
  // проверяем, что expired вышел
  if ($expiredTime < $now) {
    // .. expired time out
  }
  ```
  
  - выделять отрицание:
  ```php
  // bad
  if (!$user) {
    return;
  }

  if (!$user && !$pizza) {
    return;
  }


  // good
  if (! $user) {
    return;
  }

  if (! $user && ! $pizza) {
    return;
  }
  ```     
## Array
  - именовать массивы лучше явно
  
  ```php
  // bad
  $users = User::whereActive();
  foreach ($users as $user) { /** .. **/ }

  // good
  $userList = User::whereActive();
  foreach ($userList as $user) { /** .. **/ }
  ```
  
## Naming
  - Везде, где не сказано обратно используется CamelCase
  ```php
  // bad
  $day_of_month = 31;
  
  // good
  $dayOfMonth = 31;
  ```
  
  - Объкты называются так же как классы, только с маленькой буквы
  ```php
  // bad
  $user = new TaxiUser();
  
  // good
  $taxiUser = new TaxiUser();
  ```  
  
  - Сначала идет обобщающее слово, а потом уточняющее
  ```php
  // bad
  $currentTaxiUser = TaxiUser::find($id);
  $newTaxiUser = new TaxiUser();
  
  // good
  $taxiUserCurrent = TaxiUser::find($id);
  $taxiUserNew = new TaxiUser();
  ```    

  - Имена классов по возможности должны быть уникальны для проекта
  > Why? 
  
  > 1. Ваша IDE должна однозначно понять какой class вы хотите использовать, чтобы при "import class" у нее не возникало "какой именно класс из 10ка других User вы хотите импортировать?"
  
  > 2. Если использовать алиасы классов `User as TaxiUser` то может возникнуть ситация, что их называют по разному
  
  > 3. Снижает вероятность использования полного пути класса `new \App\Service\Taxi\User()`
  
  ```php
  // bad
  use App\Db\Main\User;
  use App\Service\Taxi\User as TaxiUser;
  
  $user = new User();
  $taxiUser = new TaxiUser();
  
  // bad 
  $user = new \App\Db\Main\User();
  $taxiUser = new \App\Service\Taxi\User();

  // good
  use App\Db\Main\User;
  use App\Service\Taxi\TaxiUser;
  
  $user = new User();
  $taxiUser = new TaxiUser();
  ```    
## Framework

  - Обращение к магическим полям лучше явно
  > Why? 
  
  > Чтобы ваша IDE могла помочь вам при рефакторинге поля или нахождения обращения к полю. Например: вам сказали, что надо переименовать phone в telephone. В явном варианте вам достаточно будет воспользоваться переименованием поля при помощи IDE и IDE сама изменит везде это поле в коде. В неявном варианте, вы будуте вынуждены вручную искать обращения к полю и переименовывать его 
  
  ```php
  // bad
  $user = new User();
  $user->fill([
    'phone' => $phone,
    'email' => $email,
  ]);
  $user->save();
  
  // good
  $user = new User();
  $user->phone = $phone;
  $user->email = $email;
  $user->save();
  
  // bad
  $taxiOrder = TaxiOrder::firstOrNew([
    'id_external' => $idExternal,
  ])

  // good
  if (! $taxiOrder = TaxiOrder::whereIdExternal($idExternal)->first()) {
    $taxiOrder = new TaxiOrder;
    $taxiOrder->id_external = $idExternal;
  }
  ```
  
# Best practices

## Хранение и работа с датами
Все даты хранятся в базе данных в UTC (+0 часовой пояс)
Вся работа внутри кода происходит в UTC (+0 часовой пояс)
При возврате пользователю: преобразовывается в его часовой пояс (или часовой пояс по умолчанию)
```php
Carbon::now(); // --> возвращает время в UTC (наше -3 часа)
Carbon::now(Cities::DEFAULT_TIMEZONE); // --> возвращает наше время

// --- записываем в базу ---
// парсим указанную дату и формируем ее в UTC чтобы положить в базу
Carbon::parse($request->input('date'), Auth::user()->timezone ?? Cities::DEFAULT_TIMEZONE)->setTimezone(config('app.timezone'));
// > 2018-01-25 15:01:00

$request->input('date');
// > 2018-01-25 18:01

Auth::user()->timezone;
// > Europe/Moscow

config('app.timezone');
// > UTC

// --- получаем из базы данных ---
Carbon::parse($order->date)->setTimezone(Auth::user()->timezone ?? Cities::DEFAULT_TIMEZONE)->format('Y-m-d H:i:s');
// > 2018-01-25 18:01:00

$order->date;
// > 2018-01-25 15:01:00
```
