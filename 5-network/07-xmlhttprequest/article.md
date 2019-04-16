# XMLHttpRequest

`XMLHttpRequest` - это встроенный в броузер объект, который позволяет делать HTTP запросы в JavaScript.

Несмотря на наличие слова "XML" в его имени, он омжет работать с любым типом данных, не только в формате XML. Мы можем загружать файлы на сервер, скачивать с сервера и даже больше.

В данные момент сущесвует еще один, более современный способ называемый `fetch`, который отчасти делает `XMLHttpRequest` устаревшим.

В современной веб-разработке `XMLHttpRequest` может быть использован по 3 причинам:

1. Исторические причины: необходимо поддреживать существующие скрипты использующие `XMLHttpRequest`.
2. Необходимо поддеривать старые броузеры, и мы не хотим использовать полифилы (чтобы наши спиркты оставались небольшого размера).
3. Необходимо то, что `fetch` еще не умеет делать. Например, отслеживать прогресс загрузки файла на сервер.

Звучит знакомо? Если да, то все в порядке, продолжай с `XMLHttpRequest`. Иначе обратить к `fetch` (скоро будет).

## Основной поток

У XMLHttpRequest есть два режима работы: синхронный и асинхронный.

Сперва рассмотрим синхронный. Так как он используется в большинстве случаев.

Есть 3 шага, чтоб сделать запрос:

1. Создаем `XMLHttpRequest`.
    ```js
    let xhr = new XMLHttpRequest(); // no arguments
    ```

2. Инициализируем его.
    ```js
    xhr.open(method, URL, [async, user, password])
    ```
    
    Этот метод обычно вызывается после `new XMLHttpRequest`. Он определяет освноные параметры запроса.  

    - `method` -- метод HTTP. Обчно `"GET"` или `"POST"`.
    - `URL` -- URL запроса.
    - `async` -- если преднамеренно установлено в `false`, тогда запрос будет синхронный. Вернемся к этому не много позже.
    - `user`, `password` -- логин и пароль для базовой аутентификации HTTP (если необходимо).
    
    Пожалуйста обратите внимание, что вызов `open`, что противоположно его имени, не открывает соединение. Он только настраивает запрос. А сетевая активность начинается после вызова `send`.

3. Высылаем.

    ```js
    xhr.send([body])
    ```

    Это метод открывает соединение и посылает запрос к серверу. В необязательном параметре `body` можно передать тело запроса. 

    Некоторые методы запроса, такие как `GET`, не именно не имеют тела. И некоторыее из них, такие как `POST`, используют `body` для отправки данных на сервер. Примеры будет приведены позже. 

4. Слушаем события запроса.

    Эти три наиболее широко используемые:
    - `load` -- когда результат, который включает ошибки HTTP, такие как 404,  - готов.
    - `error` -- когда невозожно было выполнить запрос, например, упла сеть или URL - неверный.
    - `progress` -- triggers periodically during the download, reports how much downloaded.
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

Here's a full example. The code below loads the URL at `/article/xmlhttprequest/example/load` from the server and prints the progress:

Вот полный пример. Код ниже загружает URL `/article/xmlhttprequest/example/load` с сервера и показывает статус прогреса:

```js run
// 1. Создаем новый объект XMLHttpRequest
let xhr = new XMLHttpRequest();

// 2. Настраиваем его: GET-запрос для URL /article/.../load
xhr.open('GET', '/article/xmlhttprequest/example/load');

// 3. Посылаем этот запро по сети
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

If we changed our mind, we can terminate the request at any time. The call to `xhr.abort()` does that:

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

## Response Type

We can use `xhr.responseType` property to set the response format:

- `""` (default) -- get as string,
- `"text"` -- get as string,
- `"arraybuffer"` -- get as `ArrayBuffer` (for binary data, see chapter <info:arraybuffer-and-views>),
- `"blob"` -- get as `Blob` (for binary data, see chapter <info:blob>),
- `"document"` -- get as XML document (can use XPath and other XML methods),
- `"json"` -- get as JSON (parsed automatically).

For example, let's get the response as JSON:

```js run
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/example/json');

*!*
xhr.responseType = 'json';
*/!*

xhr.send();

// the response is {"message": "Hello, world!"}
xhr.onload = function() {
  let responseObj = xhr.response;
  alert(responseObj.message); // Hello, world!
};
```

```smart
In the old scripts you may also find `xhr.responseText` and even `xhr.responseXML` properties.

They exist for historical reasons, to get either a string or XML document. Nowadays, we should set the format in `xhr.responseType` and get `xhr.response` as demonstrated above.
```

## Ready states

`XMLHttpRequest` changes between states as it progresses. The current state is accessible as  `xhr.readyState`.

All states, as in [the specification](https://xhr.spec.whatwg.org/#states):

```js
UNSENT = 0; // initial state
OPENED = 1; // open called
HEADERS_RECEIVED = 2; // response headers received
LOADING = 3; // response is loading (a data packed is received)
DONE = 4; // request complete
```

An `XMLHttpRequest` object travels them in the order `0` -> `1` -> `2` -> `3` -> ... -> `3` -> `4`. State `3` repeats every time a data packet is received over the network.

We can track them using `readystatechange` event:

```js
xhr.onreadystatechange = function() {
  if (xhr.readyState == 3) {
    // loading
  }
  if (xhr.readyState == 4) {
    // request finished
  }
};
```

You can find `readystatechange` listeners in really old code, for historical reasons.

Nowadays, `load/error/progress` handlers deprecate it.

## Synchronous requests

If in the `open` method the third parameter `async` is set to `false`, the request is made synchronously.

In other words, Javascript execution pauses at `send()` and resumes when the response is received. Somewhat like `alert` or `prompt` commands.

Here's the rewritten example, the 3rd parameter of `open` is `false`:

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
} catch(err) { // instead of onerror
  alert("Request failed");
};
```

It might look good, but synchronous calls are used rarely, because they block in-page Javascript till the loading is complete. In some browsers it becomes impossible to scroll. If a synchronous call takes too much time, the browser may suggest to close the "hanging" webpage.

Many advanced capabilities of `XMLHttpRequest`, like requesting from another domain or specifying a timeout, are unavailable for synchronous requests. Also, as you can see, no progress indication.

Because of all that, synchronous requests are used very sparingly, almost never. We won't talk about them any more.

## HTTP-headers

`XMLHttpRequest` allows both to send custom headers and read headers from the response.

There are 3 methods for HTTP-headers:

`setRequestHeader(name, value)`
: Sets the request header with the given `name` and `value`.

    For instance:

    ```js
    xhr.setRequestHeader('Content-Type', 'application/json');
    ```

    ```warn header="Headers limitations"
    Several headers are managed exclusively by the browser, e.g. `Referer` and `Host`.
    The full list is [in the specification](http://www.w3.org/TR/XMLHttpRequest/#the-setrequestheader-method).

    XMLHttpRequest is not allowed to change them, for the sake of user safety and correctness of the request.
    ```

    ````warn header="Can't remove a header"
    Another peciliarity of `XMLHttpRequest` is that one can't undo `setRequestHeader`.

    Once the header is set, it's set. Additional calls add information to the header, don't overwrite it.

    For instance:

    ```js
    xhr.setRequestHeader('X-Auth', '123');
    xhr.setRequestHeader('X-Auth', '456');

    // the header will be:
    // X-Auth: 123, 456
    ```
    ````

`getResponseHeader(name)`
: Gets the response header with the given `name` (except `Set-Cookie` and `Set-Cookie2`).

    For instance:

    ```js
    xhr.getResponseHeader('Content-Type')
    ```

`getAllResponseHeaders()`
: Returns all response headers, except `Set-Cookie` and `Set-Cookie2`.

    Headers are returned as a single line, e.g.:

    ```
    Cache-Control: max-age=31536000
    Content-Length: 4260
    Content-Type: image/png
    Date: Sat, 08 Sep 2012 16:53:16 GMT
    ```

    The line break between headers is always `"\r\n"` (doesn't depend on OS), so we can easily split it into individual headers. The separator between the name and the value is always a colon followed by a space `": "`. That's fixed in the specification.

    So, if we want to get an object with name/value pairs, we need to throw in a bit JS.

    Like this (assuming that if two headers have the same name, then the latter one overwrites the former one):

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

To make a POST request, we can use the built-in [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object.

The syntax:

```js
let formData = new FormData([form]); // creates an object, optionally fill from <form>
formData.append(name, value); // appends a field
```

Create it, optionally from a form, `append` more fields if needed, and then:

1. `xhr.open('POST', ...)` – use `POST` method.
2. `xhr.send(formData)` to submit the form to the server.

For instance:

```html run
<form name="person">
  <input name="name" value="John">
  <input name="surname" value="Smith">
</form>

<script>
  // pre-fill FormData from the form
  let formData = new FormData(document.forms.person);

  // add one more field
  formData.append("middle", "Lee");

  // send it out
  let xhr = new XMLHttpRequest();
  xhr.open("POST", "/article/xmlhttprequest/post/user");
  xhr.send(formData);

  xhr.
</script>
```

The form is sent with `multipart/form-data` encoding.

Or, if we like JSON more, then `JSON.stringify` and send as a string.

Just don't forget to set the header `Content-Type: application/json`, many server-side frameworks automatically decode JSON with it:

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

The `.send(body)` method is pretty omnivore. It can send almost everything, including Blob and BufferSource objects.

## Upload progress

The `progress` event only works on the downloading stage.

That is: if we `POST` something, `XMLHttpRequest` first uploads our data, then downloads the response.

If we're uploading something big, then we're surely more interested in tracking the upload progress. But `progress` event doesn't help here.

There's another object `xhr.upload`, without methods, exclusively for upload events.

Here's the list:

- `loadstart` -- upload started.
- `progress` -- triggers periodically during the upload.
- `abort` -- upload aborted.
- `error` -- non-HTTP error.
- `load` -- upload finished successfully.
- `timeout` -- upload timed out (if `timeout` property is set).
- `loadend` -- upload finished with either success or error.

Example of handlers:

```js
xhr.upload.onprogress = function(event) {
  alert(`Uploaded ${event.loaded} of ${event.total} bytes`);
};

xhr.upload.onload = function() {
  alert(`Upload finished successfully.`);
};

xhr.upload.onerror = function() {
  alert(`Error during the upload: ${xhr.status}`);
};
```

Here's a real-life example: file upload with progress indication:

```html run
<input type="file" onchange="upload(this.files[0])">

<script>
function upload(file) {
  let xhr = new XMLHttpRequest();

  // track upload progress
*!*
  xhr.upload.onprogress = function(event) {
    console.log(`Uploaded ${event.loaded} of ${event.total}`);
  };
*/!*

  // track completion: both successful or not
  xhr.onloadend = function() {
    if (xhr.status == 200) {
      console.log("success");
    } else {
      console.log("error " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
</script>
```

## Cross-origin requests

`XMLHttpRequest` can make cross-domain requests, using the same CORS policy as [fetch](info:fetch-crossorigin).

Just like `fetch`, it doesn't send cookies and HTTP-authorization to another origin by default. To enable them, set `xhr.withCredentials` to `true`:

```js
let xhr = new XMLHttpRequest();
*!*
xhr.withCredentials = true;
*/!*

xhr.open('POST', 'http://anywhere.com/request');
...
```


## Summary

Typical code of the GET-request with `XMLHttpRequest`:

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/my/url');

xhr.send(); // for POST, can send a string or FormData

xhr.onload = function() {
  if (xhr.status != 200) { // HTTP error?
    // handle error
    alert( 'Error: ' + xhr.status);
    return;
  }

  // get the response from xhr.response
};

xhr.onprogress = function(event) {
  // report progress
  alert(`Loaded ${event.loaded} of ${event.total}`);
};

xhr.onerror = function() {
  // handle non-HTTP error (e.g. network down)
};
```

There are actually more events, the [modern specification](http://www.w3.org/TR/XMLHttpRequest/#events) lists them (in the lifecycle order):

- `loadstart` -- the request has started.
- `progress` -- a data packet of the response has arrived, the whole response body at the moment is in `responseText`.
- `abort` -- the request was canceled by the call `xhr.abort()`.
- `error` -- connection error has occured, e.g. wrong domain name. Doesn't happen for HTTP-errors like 404.
- `load` -- the request has finished successfully.
- `timeout` -- the request was canceled due to timeout (only happens if it was set).
- `loadend` -- the request has finished (succeffully or not).

The most used events are load completion (`load`), load failure (`error`), and also `progress` to track the progress.

We've already seen another event: `readystatechange`. Historically, it appeared long ago, before the specification settled. Nowadays, there's no need to use it, we can replace it with newer events, but it can often be found in older scripts.

If we need to track uploading specifically, then we should listen to same events on `xhr.upload` object.
