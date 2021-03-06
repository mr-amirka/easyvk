# Изменения

Все доступные изменения, начиная с <b>2.1.1</b> версии
## \[2.4.13\] - 2019-05-20
-   Исправлена ошибка работы `http(s)` прокси при включенной авторизации по логину и паролю

## \[2.4.12\] - 2019-04-28
-   Исправлена работа метода `HTTPClient.readStories()`
-   Исправлена работа переавторизации
-   Исправлена работа `highload` режима. Теперь будет видно полное описание ошибки
-   Исправлена работа debugger'а в секции `vk.call` при включенном `highload`. Теперь все запросы точно будут доходить до него 

## \[2.4.11\] - 2019-04-24
### Исправления
-   Исправлена кодировка GET запросов на `utf8`
-   Исправлена авторизация по сохраненной сессии приложений

## \[2.4.1\] - 2019-04-17
### Исправления
-   Исправлена работа с капчей при авторизации по сессии. Ранее капча могла не обрабатывалась через `captchaHandler` в кейсах, когда данные авторизации не менялись и не происходило `reauth: true`

## \[2.4.0\] - 2019-04-08
### Исправления
-   Исправлена работа режима `highload` при ошибке капчи. Теперь капча будет обрабатываться самым последним отправленным запросом из очереди
```javascript
easyvk({...{
  mode: 'highload'
}}).then(vk => {
  for (let i = 0; i < 100; i++) {
    vk.call('messages.send', {
      peer_id: 356607530,
      message: 'Дароу!'
    }).catch(e => {
      console.log(e, i);
    });
  }
});
````
-   Исправлена работа Bots LongPoll при неизвестной ошибке ВК (неожиданный ответ), теперь Easy VK самостоятельно сделает переподключение при остановке в таком случае
-   Исправлена авторизация из сессии
-   Исправлена работа метода `client.readFeedStories()` для прочтения историй со стены

### Добавления и изменения
-   Добавлена работа с `fs.ReadStream` для метода `uploader.uploadFile`. Теперь кроме имени файла можно передавать объект `ReadStream`
```javascript
const fs = require('fs')
const path = require('path')

easyvk({... {
  utils: {
    uploader: true
  }
}}).then(vk => {
  vk.uploader.getUploadURL('docs.getUploadServer').then(({url}) => {
    
    let stream = fs.createReadStream(path.join(__dirname, 't.txt'))
    
    vk.uploader.uploadFile(url, stream, 'file').then(({vkr}) => {
      console.log(vkr);
    }).catch(console.log)

  })
})

```
-   Для `static` методов добавлен новый метод `static.createExecute()`

Метод создает строковое представление запроса для VK Script
```javascript
easyvk.static.createExecute('messages.send', {
  peer_id: 1,
  v: '5.90',
  lang: 'en'
}) // 'API.messages.send({"peer_id": 1})'
```
-   Добавлен новый объект (переписанный старый) `Debugger`, теперь дебагинг всех запросов и их ответов возможен с помощью него <b>В следующих обновлениях все методы, связанные с работой прошлого Debugger'а, в том числе и DebuggerRun, будут удалены (сейчас об этом идет предупреждение)</b>
```javascript
const easyvk = require('easyvk');

const { Debugger } = easyvk;

let debug = new Debugger();

debug.on('response', ({ body }) => {
  console.log(body);
});

debug.on('request', (b) => {
  console.log(b.toString());
});

debug.on('request', ({ method, url, query }) => {
  console.log(`[${method}] on ${url}: \n`, query);
});

easyvk({
  debug,
  username: '...',
  ...{}
}).then(async vk => {

  let { vkr: friends } = await vk.call('friends.get');

  console.log('Got friends!', friends);

});

```
-  В связи с новым способом дебагинга, был добавлен новый параметр `debug`, куда необходимо передавать объект дебагера
-  Добавлена автоматическая "переавторизация", если изменены какие-то данные в настройках easyvk. Это убирает большинство типичных проблем, с которыми сталкивались разработчики ранее
-  В связи с исправлениями переавторизации, были добавлены новые поля сессии
<b>Для аккаунта</b>
```javascript
{
  username: 'имя_пользователя'
}
```
<b>Для приложений</b>
```javascript
{
  client_id: parseInt('233_ID_приложения', 10)
}
```
<b>Для групп ничего не изменялось, проверка идет по access_token'у</b>

## \[2.3.0\] - 2019-03-17
## Исправления
-   Исправлен `captchaHandler` для авторизации. Теперь капча поступает сразу в handler. Раньше можно было ловить капчу при авторизации только с помощью `.catch()` метода
-   Исправлена кодировка методов `Audio.get()` и `Audio.search()`
-   Исправлена функция декодирования Mp3 URL, были внесены последние обновления с сайта ВКонтакте
-   Исправлена ошибка поиска аудиозаписей
-   Исправлена ошибка закрытых аудиозаписей. Ранее Easy VK не возвращал аудиозапись, к которой закрыт доступ правообладателем или самой платформой. Теперь стало возможно получить аудиозапись, а вместе с этим, у таких аудиозаписей появились два новых поля - `is_restriction` и `extra` (несет в себе JSON информацию о причине блокировке аудио)

## Добавления
-  Были добавлены два новых поля для аудиозаписей: `is_restriction` и `extra`, которые дают знать, заблокирован ли трек и если да, то по какой причине
-  Был добавлен режим `highload`, который позволяет отправлять все запросы к API через метод `execute`. Его настройка максимально точна и проста.
```javascript
const easyvk = require('easyvk')

easyvk({
  access_token: '{ТОКЕН_ГРУППЫ}',
  mode: 'highload' || { // Можно настроить режим точнее
    // Имя режима работы. Пока что только одно
    name: 'highload',

    // Время, через которое запрос гарантированно выполнится, если не поступит новых
    timeout: 15
  },
  api_v: '5.80', // Все запросы будут выполнятся под этой версией
  lang: 'en' // Все запросы будут возвращать в едином языке - English
}).then(async (vk) => {

  // Подключение к LongPoll происходит тоже через execute метод
  let {connection: group} = await vk.longpoll.connect()

  group.on("message", async (msg) => {
    
    let _msg = {
      out: msg[2] & 2,
      peer_id: msg[3],
      text: msg[5],
      payload: msg[6].payload
    }
    
    if (!_msg.out) {
      
      let {vkr} = await vk.call('messages.send', {
        message: "Hello!",
        peer_id: _msg.peer_id
      });

      console.log(vkr);
    }

  })

})
```
-   Была добавлена возможность настроить утилиты и компоненты, которые вы теперь можете отключить за ненадобностью, или, наоборот, подключить, если Easy VK их автоматически отключил

```javascript

const easyvk = require('easyvk')

easyvk({
  access_token: '{ТОКЕН_ГРУППЫ}',
  utils: {
    http: true,
    widgets: true,
    bots: false, // Отключаем секию vk.bots
    uploader: true, // Включаем Uploader
    longpoll: true, // Включаем User LongPoll
    callbackAPI: true, // Callback API изначально выключен, его мы подключаем сами
    streamingAPI: true // То же самое
  }
}).then(async (vk) => {

  // Подключение к LongPoll происходит тоже через execute метод
  let {connection: group} = await vk.longpoll.connect()

  group.on("message", async (msg) => {
    
    let _msg = {
      out: msg[2] & 2,
      peer_id: msg[3],
      text: msg[5],
      payload: msg[6].payload
    }
    
    if (!_msg.out) {
      
      let {vkr} = await vk.call('messages.send', {
        message: "Hello!",
        peer_id: _msg.peer_id
      });

      console.log(vkr);
    }

  })

})

```

## \[2.2.17\] - 2019-03-10
## Исправления
-   Исправлен метод HTTP клиента audio.get() для большого количества аудиозаписей (больше 200) с помощью добавления параметра `count` - количество максимально обрабатываемых аудиозаписей. Ранее обрабатывались все получаемые аудио (без ограничения, т.к ВК возвращает почти все треки)

```javascript
let { client } = await vk.http.loginByForm()

client.audio.get({
  count: 150,
  offset: 30,
  owner_id: -45703770
}).then(({vkr}) => {
  
  console.log(vkr.length)

})
```
## \[2.2.16\] - 2019-02-22
## Исправления
-   Исправлена отправка post запроса
-   Исправлена работа с прокси на более новых версиях Node.JS

## \[2.2.14\] - 2019-02-09
### Добавления
-   Добавлен параметр <code>userAgent</code> для запросов <code>vk.call()</code>
```javascript
const easyvk = require('easyvk')

easyvk({
  userAgent: 'KateMobileAndroid/52.2.2 lite-448 (Android 6.0; SDK 23; arm64-v8a; alps Razar; ru)',
  access_token: 'USER_TOKEN',
  reauth: true
}).then(async (vk) => {
  let {vkr: audios} = vk.call("audio.get")

  console.log(audios.items.length)
})
```

### Исправления
-   Исправлена отправка post запроса через метод vk.post()

## \[2.2.13\] - 2019-02-04
### Изменения
-   Возвращена поддержка Node JS >= 8.0.0

## \[2.2.12\] - 2019-02-03
### Добавления
-   Добавлен файл CHANGELOG.md

## \[2.2.1\] - 2019-02-03
### Добавления
-   Добавлен метод поиска множества аудиозаписей сразу 
```javascript
let { client } = await vk.http.loginByForm()
let count = 1000 // Если аудиозаписей меньше, вернет полученное количество

client.audio.searchAll(query, count).then(console.log)
```

### Исправления
-   Исправлена ошибка авторизации при <code>save_session: false</code>

### Изменения
-   В Streaming API теперь событие pullEvent будет доступно, даже если нету слушателей на это событие. Таким образом, можно будет слушать все события с помощью одного.