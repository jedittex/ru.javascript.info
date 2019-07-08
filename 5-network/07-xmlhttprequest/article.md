# XMLHttpRequest

`XMLHttpRequest` - это встроенный в браузер объект, который позволяет делать HTTP запросы в JavaScript.

Несмотря на наличие слова "XML" в его имени, он омжет работать с любым типом данных, не только в формате XML. Мы можем загружать файлы на сервер, скачивать с сервера и даже больше.

В данные момент сущесвует еще один, более современный способ называемый `fetch`, который отчасти делает `XMLHttpRequest` устаревшим.

В современной веб-разработке `XMLHttpRequest` может быть использован по 3 причинам:

1. Исторические причины: необходимо поддреживать существующие скрипты использующие `XMLHttpRequest`.
2. Необходимо поддеривать старые браузеры, и мы не хотим использовать полифилы (чтобы наши спиркты оставались небольшого размера).
3. Необходимо то, что `fetch` еще не умеет делать. Например, отслеживать прогресс загрузки файла на сервер.

Звучит знакомо? Если да, то все в порядке, продолжай с `XMLHttpRequest`. Иначе обратитесь к `fetch` (скоро будет).

## Основной поток

У XMLHttpRequest есть два режима работы: синхронный и асинхронный.

Сперва рассмотрим синхронный. Так как он используется в большинстве случаев.

Есть 3 шага, чтоб сделать запрос:

1. Создаем `XMLHttpRequest`.
    ```js
    let xhr = new XMLHttpRequest(); // без аргументов
    ```

2. Инициализируем его.
    ```js
    xhr.open(method, URL, [async, user, password])
    ```
    
    Этот метод обычно вызывается после `new XMLHttpRequest`. Он определяет освноные параметры запроса.  

    - `method` -- метод HTTP. Обчно `"GET"` или `"POST"`.
    - `URL` -- URL запроса.
    - `async` -- если преднамеренно установлено в `false`, тогда запрос будет синхронный. Вернемся к этому немного позже.
    - `user`, `password` -- логин и пароль для базовой аутентификации HTTP (если необходимо).
    
    Пожалуйста обратите внимание, что вызов `open`, что противоположно его имени, не открывает соединение. Он только настраивает запрос. А сетевая активность начинается после вызова `send`.

3. Высылаем.

    ```js
    xhr.send([body])
    ```

    Это метод открывает соединение и посылает запрос к серверу. В необязательном параметре `body` можно передать тело запроса. 

    Некоторые методы запроса, такие как `GET`, не имеют тела. И некоторые из них, такие как `POST`, используют `body` для отправки данных на сервер. Примеры будет приведены позже. 

4. Слушаем события запроса.

    Эти три наиболее широко используемые:
    - `load` -- когда результат, который включает ошибки HTTP, такие как 404,  - готов.
    - `error` -- когда невозожно было выполнить запрос, например, упала сеть или URL - неверный.
    - `progress` -- переодически срабатывает в процессе загрузки, отчитывается о том, как много загружено.

    ```js
    xhr.onload = function() {
      alert(`Загружен: ${xhr.status} ${xhr.response}`);
    };

    xhr.onerror = function() { // срабатывает только если невозожно было выполнить запрос
      alert(`Ошибка сети`);
    };

    xhr.onprogress = function(event) { // срабатывает переодически
      // event.loaded - сколько байт загружено
      // event.lengthComputable = true если сервер послал заголовок Content-Length
      // event.total - общее количество байт (если lengthComputable)
      alert(`Получено ${event.loaded} из ${event.total}`);
    };
    ```

Вот полный пример. Код ниже загружает URL `/article/xmlhttprequest/example/load` с сервера и показывает статус прогреса:

```js run
// 1. Создаем новый объект XMLHttpRequest
let xhr = new XMLHttpRequest();

// 2. Настраиваем его: GET-запрос для URL /article/.../load
xhr.open('GET', '/article/xmlhttprequest/example/load');

// 3. Посылаем этот запрос по сети
xhr.send();

// 4. Это будет вызвано после получения ответа
xhr.onload = function() {
  if (xhr.status != 200) { // исследуем HTTP статус ответа
    alert(`Ошибка ${xhr.status}: ${xhr.statusText}`); // например, 404: Not Found
  } else { // показать ответ
    alert(`Готово, получаено ${xhr.response.length} байт`); // responseText - это ответ сервера
  }
};

xhr.onprogress = function(event) {
  if (event.lengthComputable) {
    alert(`Получено ${event.loaded} из ${event.total} байт`);
  } else {
    alert(`Получено ${event.loaded} байт`); // Content-Length не указан
  }

};

xhr.onerror = function() {
  alert("Запрос не удался");
};
```

Поскольку сервер ответил, мы можем получить результат в виде следующих свойств объекта запроса:

`status`
: код состояния HTTP (число): `200`, `404`, `403` и так далее, может быть `0` если ошибка не на уровне HTTP.

`statusText`
: сообщения состояния HTTP (строка): обычно `OK` для `200`, `Not Found` для `404`, `Forbidden` для `403` и т.д.

`response` (старые скрипты могу использовать `responseText`)
: Ответ от серера.

Если мы передумали, мы можем завершить запрос в любое время. Вызов `xhr.abort()` сделает это:

```js
xhr.abort(); // завершить запрос
```

В данном случае сработает событие `abort`.

Также можно указать время ожидания, используя соответствующее свойство:

```js
xhr.timeout = 10000; // время ожидания указывется в ms, 10 секунд
```

Если запрос не успел отработать за отведенное время, тогда он отменяется и срабатывает событие `timeout`.

## Тип ответа

Мы можем использовать свойство `xhr.responseType` для установки формата ответа:

- `""` (по умолчанию) -- получить строку,
- `"text"` -- получить строку,
- `"arraybuffer"` -- покучить как `ArrayBuffer` (для двоихных данных, см. главу <info:arraybuffer-and-views>),
- `"blob"` -- покучить как `Blob` (для двоихных данных, см. главу <info:blob>),
- `"document"` -- покучить как документ XML (допускается использование XPath и методов XML),
- `"json"` -- покучить как JSON (разбирается автоматически).

Например, давайте получим ответ как JSON:

```js run
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/example/json');

*!*
xhr.responseType = 'json';
*/!*

xhr.send();

// the response is {"message": "Привет, мир!"}
xhr.onload = function() {
  let responseObj = xhr.response;
  alert(responseObj.message); // Привет, мир!
};
```

```smart
В старых скрипта можно найти свойство `xhr.responseText` и даже `xhr.responseXML`.

Они существуют по историческим причинам, для получаения сторки или документа XML. Сейчас необходимо устанавливать формат в `xhr.responseType` и получать `xhr.response`, как показано выше.
```

## Состояния готовности

`XMLHttpRequest` меняет свое состояние в процессе работы. Текущее состояние доступно как `xhr.readyState`.

Все состония, как в [спецификации](https://xhr.spec.whatwg.org/#states):

```js
UNSENT = 0; // изначальное состояние
OPENED = 1; // вызван метод open
HEADERS_RECEIVED = 2; // получены заголовки ответа
LOADING = 3; // запрос в процессе загрузки (получен пакет данных)
DONE = 4; // запрос завершился
```

Объект `XMLHttpRequest` меняет их в порядке `0` -> `1` -> `2` -> `3` -> ... -> `3` -> `4`. Состояние `3` повторяется каждый раз, когда по сети приходит пакет данных.

Мы можем отследить их используя событие `readystatechange`:

```js
xhr.onreadystatechange = function() {
  if (xhr.readyState == 3) {
    // загрузка
  }
  if (xhr.readyState == 4) {
    // запрос закончился
  }
};
```

В очень старом коде можно найти слушателей `readystatechange`, по историческим причинам.

На данный момент обработчики `load/error/progress` выступают вместо этох состояний.

## Синхронные запросы

Если в методе `open` третьим параметром, называемым `async`, передать `false`, то запрос будет синхронный.

Другими словами, выполнение JavaScript приостанаваливается и возобновляется только тогда, когда ответ будет получен. Подобно тому, как это делают команды `alert` и `prompt`

Вот переписанный пример, третий парамерт `open` равен `false`

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/hello.txt', *!*false*/!*);

try {
  xhr.send();
  if (xhr.status != 200) {
    alert(`Error ${xhr.status}: ${xhr.statusText}`);
  } else {
    alert(xhr.response);
  }
} catch(err) { // вместо onerror
  alert("Запрос не удался");
};
```

Это может выглядень хорошо, но синхронные вызовы используются редко потому, что они блокинуют внутристраничный JavaScript, пока загрузка не будет завершена. В некоторых браузерах становится невозможо прокручивать страницу. Если синхронный запрос происходит долго, то браузер может предложить закрыть "зависшую" страницу.

Много продвинутых возможностей `XMLHttpRequest`, такие как запрос с другого домента или определенное время ожидания, недоступны для синхронных запросов. Также, как вы можете видеть, индикация процесса загрузки.

Исходя их всего этого, синхронные запросы используются очень редко, почти никогда. В дальнейшем не будем возвращаться к ним.

## Заголовки HTTP

`XMLHttpRequest` позволяет как посылать пользовательские заголовки, так и читать заголовки ответа. 

Есть 3 метода для заголовков HTTP:

`setRequestHeader(name, value)`
: Устанавливает заголовок запроса, используя данные `name` и `value`.

    Например:

    ```js
    xhr.setRequestHeader('Content-Type', 'application/json');
    ```

    ```warn header="Headers limitations"
    Некоторые заголовки управляются исключительно браузером. Такие как `Referer` и `Host`.
    Полный список [в спецификации](http://www.w3.org/TR/XMLHttpRequest/#the-setrequestheader-method).

    XMLHttpRequest не может менять их из-за соображений пользовательской безопасности и точности запроса.
    ```

    ````warn header="Не возможно удалить заголовок"
    Другая особенность `XMLHttpRequest` - он не может изменить заголовки, устрановленные в `setRequestHeader`.

    Как только заголовок установлен, не получится его изменить. Дополнительные вызовы добавляют информацию к заголовку, но не перезаписывают ее.

    Например:

    ```js
    xhr.setRequestHeader('X-Auth', '123');
    xhr.setRequestHeader('X-Auth', '456');

    // заголовок будет:
    // X-Auth: 123, 456
    ```
    ````

`getResponseHeader(name)`
: Получает заголовок ответа с данным `name` (кроме `Set-Cookie` и `Set-Cookie2`).

    Например:

    ```js
    xhr.getResponseHeader('Content-Type')
    ```

`getAllResponseHeaders()`
: Возвращает все заголовки ответ, кроме `Set-Cookie` и `Set-Cookie2`.

    Залоговки возвращаются как единственная строка. Например:

    ```
    Cache-Control: max-age=31536000
    Content-Length: 4260
    Content-Type: image/png
    Date: Sat, 08 Sep 2012 16:53:16 GMT
    ```

    Перевод строки между заголовкими всегда - `"\r\n"` (не зависит от ОС) так, чтобы можно было легко разделить на отдельные заголовки. Разделитель можду названием и значением - всегда двоеточие, за которым идет пробел, `": "`. Это зафиксированно в спецификацияи.
    
    Итак, если мы хотим получить объект с парами ключ/значение, надо применить немного JS.

    Как это (при условии, что если два заголовка с одинаковыми именами, тогда письмо перезаписывает предыдущее):

    ```js
    let headers = xhr
      .getAllResponseHeaders()
      .split('\r\n')
      .reduce((result, current) => {
        let [name, value] = current.split(': ');
        result[name] = value;
        return result;
      }, {});
    ```

## POST, FormData

Чтобы сделать запрос POST, можно использовать встроенный объект [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData).

Синтаксис:

```js
let formData = new FormData([form]); // создает объект, можно заполнить форму <form>
formData.append(name, value); // добавляет поле
```

Create it, optionally from a form, `append` more fields if needed, and then:
Создайте объект, как вариант из формы. Если необходимо, добавьте больше полей. И далее:

1. `xhr.open('POST', ...)` – используте метод `POST`.
2. `xhr.send(formData)` для отравки формы на серве.

Для примера:

```html run
<form name="person">
  <input name="name" value="John">
  <input name="surname" value="Smith">
</form>

<script>
  // предзаполняем объект FormData из данных формы
  let formData = new FormData(document.forms.person);

  // добавляем еще одно поле
  formData.append("middle", "Lee");

  // высылаем
  let xhr = new XMLHttpRequest();
  xhr.open("POST", "/article/xmlhttprequest/post/user");
  xhr.send(formData);
</script>
```

Форма полсана способ кодировки `multipart/form-data`.

Или, если мы продпочитаем JSON, используем `JSON.stringify` и посылаем как строку.

Только не забываем установить заголовок `Content-Type: application/json`. Много сервных фреймворков автоматически декодируют JSON вот так:

```js
let xhr = new XMLHttpRequest();

let json = JSON.stringify({
  name: "John",
  surname: "Smith"
});

xhr.open("POST", '/submit')
xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');

xhr.send(json);
```

Метод `.send(body)` - достаточно всеяден. Он может слать почи все, включая объекты Blob и BufferSource. 

## Прогресс загрузки

Событие `progress` работает только на стадии загрузки.

Если мы шлем что-то используя `POST`, `XMLHttpRequest` сперва загрузит наши данные, затем загрузки ответ.

Если загружаем что-то большое, тогда мы вероятнее всего заинтересованы в отслеживании прогресса загрузки. Но событие `progress` не помогает здесь.

Есть другой объект - `xhr.upload`, без методов, исключительно для событий загрузки.

Вот список:

- `loadstart` -- началась загрузка.
- `progress` -- triggers periodically during the upload.
- `progress` -- переодически срабатывает в процессе заргузки.
- `abort` -- загрузка прервана.
- `error` -- ошибка не на уровне HTTP.
- `load` -- загрузка завершилась успешно.
- `timeout` -- всемя загрузки истекло (если было установлено свойство `timeout`).
- `loadend` -- загрузка закончилась либо успешно, либо с ошибкой.

Примеры обработчиков:

```js
xhr.upload.onprogress = function(event) {
  alert(`Загружено ${event.loaded} байт из ${event.total}`);
};

xhr.upload.onload = function() {
  alert(`Загрузка завершилась успешно.`);
};

xhr.upload.onerror = function() {
  alert(`Ошибка в процессе загрузки: ${xhr.status}`);
};
```

Наже приведен пример из реальной жизни: загрузка файл с отображением прогресса:

```html run
<input type="file" onchange="upload(this.files[0])">

<script>
function upload(file) {
  let xhr = new XMLHttpRequest();

  // отслеживаем прогресс загрузки
*!*
  xhr.upload.onprogress = function(event) {
    console.log(`Загружено ${event.loaded} из ${event.total}`);
  };
*/!*

  // отслеживаем выполнение: успешно или нет
  xhr.onloadend = function() {
    if (xhr.status == 200) {
      console.log("успех");
    } else {
      console.log("ошибка " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
</script>
```

## Запрос между источкиками (cross-origin)

`XMLHttpRequest` может делать междоменные запросы, использу политику CORS как [fetch](info:fetch-crossorigin).

Так же как и `fetch`, по-умолчаню он не может слать куки и авторизацию HTTP на другой источник. Чтобы включить это, установите `xhr.withCredentials` в `true`.

```js
let xhr = new XMLHttpRequest();
*!*
xhr.withCredentials = true;
*/!*

xhr.open('POST', 'http://anywhere.com/request');
...
```


## Резюме

Типичный код запроса GET используя `XMLHttpRequest`:

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/my/url');

xhr.send(); // для POST, может слать строку или данные формы 

xhr.onload = function() {
  if (xhr.status != 200) { // ошибка HTTP?
    // обработать ошибку
    alert( 'Ошибка: ' + xhr.status);
    return;
  }

  // получить ответ из xhr.response
};

xhr.onprogress = function(event) {
  // отчет о прогрессе
  alert(`Загружено ${event.loaded} из ${event.total}`);
};

xhr.onerror = function() {
  // обработать ошибку не уровня HTTP (такую как падение сети)
};
```

There are actually more events, the [modern specification](http://www.w3.org/TR/XMLHttpRequest/#events) lists them (in the lifecycle order):
В общем, существует больше событий. В [современной спецификации](http://www.w3.org/TR/XMLHttpRequest/#events) перечисленные все (в порядке времени существования).

- `loadstart` -- запрос начался.
- `progress` -- прибыл пакет данных ответа, все тело ответа в этот момент находится в `responseText`.
- `abort` -- запрос был отменен вызовом `xhr.abort()`.
- `error` -- произошка ошибка соединения, такая как неверное доменное имя. Не срабатывает при ошибках HTTP, например, 404.
- `load` -- запрос выполнен успешно.
- `timeout` -- запрос был отменен в следствии тайм-аута (происходит, если это было установлено).
- `loadend` -- запрос выполнен (успешно или нет).

Самые используемые события - это завершение загрузки (`load`), ошибка загрузки (`error`), а также `progress` для отслеживания прогресса.

Мы уже вижели другое событие: `readystatechange`. Исторически, оно появлось давно, перед тем, как спецификация устоялась. В наше время нет нужды использовать его. Можно заменить его более новыми событиями, но все же оно может быть найдено в старых скрипта.

Если необходимо отслеживать именно загрузку, то нужно слушать то же событие на объекте `xhr.upload`.
