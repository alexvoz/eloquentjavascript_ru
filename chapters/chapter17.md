#HTTP

<i>Мечта, ради которой создавалась Сеть – это общее информационное пространство, в котором мы общаемся, делясь информацией. Его универсальность является его неотъемлемой частью: ссылка в гипертексте может вести куда угодно, будь то персональная, локальная или глобальная информация, черновик или выверенный текст.

Тим Бернес-Ли, Всемирная паутина: Очень короткая личная история</i>

##Протокол
Если в адресной строке браузера набрать eloquentjavascript.net/17_http.html, браузер сначала распознает адрес сервера, связанный с именем eloquentjavascript.net и попробует открыть TCP соединение по 80 порту – порт для HTTP по умолчанию. Если сервер существует и принимает соединение, браузер отправляет что-то вроде:

```
GET /17_http.html HTTP/1.1
Host: eloquentjavascript.net
User-Agent: Название браузера
```

Сервер отвечает по тому же соединению:

```
HTTP/1.1 200 OK
Content-Length: 65585
Content-Type: text/html
Last-Modified: Wed, 09 Apr 2014 10:48:09 GMT

<!doctype html>
… остаток документа
```

Браузер берёт ту часть, что идёт за ответом после пустой строки и показывает её в виде HTML-документа.

Информация, отправленная клиентом, называется запросом. Он начинается со строки:

```
GET /17_http.html HTTP/1.1
```

Первое слово – метод запроса. GET означает, что нам нужно получить определённый ресурс. Другие распространённые методы – DELETE для удаления, PUT для замещения и POST для отправки информации. Заметьте, что сервер не обязан выполнять каждый полученный запрос. Если вы выберете случайный сайт и скажете ему DELETE главную страницу – он, скорее всего, откажется.

Часть после названия метода – путь к ресурсу, к которому отправлен запрос. В простейшем случае, ресурс – просто файл на сервере, но протокол не ограничивается этой возможностью. Ресурс может быть чем угодно, что можно передать в качестве файла. Многие серверы создают ответы на лету. К примеру, если вы откроете twitter.com/marijnjh, сервер посмотрит в базе данных пользователя marijnjh, и если найдёт – создаст страницу профиля этого пользователя.

После пути к ресурсу первая строка запроса упоминает HTTP/1.1, чтобы сообщить о версии HTTP – протокола, которую она использует.

Ответ сервера также начинается с версии протокола, за которой идёт статус ответа – сначала код из трёх цифр, затем строчка.

```
HTTP/1.1 200 OK
```

Коды статуса, начинающиеся с 2, обозначают успешные запросы. Коды, начинающиеся с 4, означают, что что-то пошло не так. 404 – самый знаменитый статус HTTP, обозначающий, что запрошенный ресурс не найден. Коды, начинающиеся с 5, обозначают, что на сервере произошла ошибка, но не по вине запроса.

За первой строкой запроса или ответа может идти любое число строк заголовка. Это строки в виде “имя: значение”, которые обозначают дополнительную информацию о запросе или ответе. Эти заголовки были включены в пример:

```
Content-Length: 65585
Content-Type: text/html
Last-Modified: Wed, 09 Apr 2014 10:48:09 GMT
```

Тут определяется размер и тип документа, полученного в ответ. В данном случае это HTML-документ размером 65’585 байт. Также тут указано, когда документ был изменён последний раз.

По большей части клиент или сервер определяют, какие заголовки необходимо включать в запрос или ответ, хотя некоторые заголовки обязательны. К примеру, Host, обозначающий имя хоста, должен быть включён в запрос, потому что один сервер может обслуживать много имён хостов на одном ip-адресе, и без этого заголовка сервер не узнает, с каким хостом клиент пытается общаться.

После заголовков, как запрос, так и ответ могут указать пустую строку, за которой следует тело, содержащее передаваемые данные. Запросы GET и DELETE не пересылают дополнительных данных, а PUT и POST пересылают. Некоторые ответы, например, сообщения об ошибке, не требуют наличия тела.

##Браузер и HTTP
Как мы видели в примере, браузер отправляет запрос, когда мы вводим URL в адресную строку. Когда в полученном HTML документе содержатся упоминания других файлов, такие, как картинки или файлы JavaScript, они тоже запрашиваются с сервера.

Веб-сайт средней руки легко может содержать от 10 до 200 ресурсов. Чтобы иметь возможность запросить их побыстрее, браузеры делают несколько запросов одновременно, а не ждут окончания запросов одного за другим. Такие документы всегда запрашиваются через запросы GET.

На страницах HTML могут быть формы, которые позволяют пользователям вписывать информацию и отправлять её на сервер. Вот пример формы:

```
<form method="GET" action="example/message.html">
  <p>Имя: <input type="text" name="name"></p>
  <p>Сообщение:<br><textarea name="message"></textarea></p>
  <p><button type="submit">Отправить </button></p>
</form>
```

Код описывает форму с двумя полями: маленькое запрашивает имя, а большое – сообщение. При нажатии кнопки «Отправить» информация из этих полей будет закодирована в строку запроса (query string). Когда атрибут method элемента  равен GET, или когда он вообще не указан, строка запроса помещается в URL из поля action, и браузер делает GET-запрос с этим URL.

```
GET /example/message.html?name=Jean&message=Yes%3F HTTP/1.1
```

Начало строки запроса обозначено знаком вопроса. После этого идут пары имён и значений, соответствующие атрибуту name полей формы и содержимому этих полей. Амперсанд (&amp;) используется для их разделения.

Сообщение, отправляемое в примере, содержит строку “Yes?”, хотя знак вопроса и заменён каким-то странным кодом. Некоторые символы в строке запроса нужно экранировать (escape). Знак вопроса в том числе, и он представляется кодом %3F. Есть какое-то неписаное правило, по которому у каждого формата должен быть способ экранировать символы. Это правило под названием кодирование URL использует процент, за которым идут две шестнадцатеричные цифры, которые представляют код символа. 3F в десятичной системе будет 63, и это код знака вопроса. У JavaScript есть функции encodeURIComponent и decodeURIComponent для кодирования и раскодирования.

```
console.log(encodeURIComponent("Hello & goodbye"));
// → Hello%20%26%20goodbye
console.log(decodeURIComponent("Hello%20%26%20goodbye"));
// → Hello & goodbye
```

Если мы поменяем атрибут method в форме в предыдущем примере на POST, запрос HTTP с отправкой формы пройдёт при помощи метода POST, который отправит строку запроса в теле запроса, вместо добавления её к URL.

```
POST /example/message.html HTTP/1.1
Content-length: 24
Content-type: application/x-www-form-urlencoded

name=Jean&message=Yes%3F
```

По соглашению метод GET используется для запросов, не имеющих побочных эффектов, таких, как поиск. Запросы, которые что-то меняют на сервере – создают новый аккаунт или размещают сообщение, должны отправляться методом POST. Клиентские программы типа браузера знают, что просто так делать запросы формата POST не нужно, и иногда незаметно для пользователя делают запросы GET – к примеру, чтобы загрузить заранее контент, который может вскоре понадобиться пользователю.

В следующей главе мы вернёмся к формам и поговорим про то, как мы можем делать их при помощи JavaScript.

##XMLHttpRequest
Интерфейс, через который JavaScript в браузере может делать HTTP-запросы, называется XMLHttpRequest (заметьте, как прыгает размер букв). Он был разработан в Microsoft для браузера Internet Explorer в конце 1990-х. В это время формат XML был очень популярным в мире бизнес-программ – а в этом мире Microsoft всегда чувствовал себя, как дома. Он был настолько популярным, что аббревиатура XML была пришпилена перед названием интерфейса для работы с HTTP, хотя последний с XML вообще не связан.

И всё же имя не полностью бессмысленное. Интерфейс позволяет разбирать вам ответы, как если бы это были документы XML. Смешивать две разные вещи (запрос и разбор ответа) в одну – это, конечно, отвратительный дизайн, но что поделаешь.

Когда интерфейс XMLHttpRequest был добавлен в Internet Explorer, стало можно делать вещи, которые раньше было делать очень сложно. К примеру, сайты стали показывать списки из подсказок, пока пользователь вводит что-либо в текстовое поле. Скрипт отправляет текст на сервер через HTTP одновременно с набором текста пользователем. Сервер, у которого есть база данных для возможных вариантов ввода, ищет среди записей подходящие и возвращает их назад для показа. Это выглядело очень круто – люди до этого привыкли ждать перезагрузки всей страницы после каждого взаимодействия с сайтом.

Другой важный браузер того времени Mozilla (позже Firefox), не хотел отставать. Чтобы разрешить делать сходные вещи, Mozilla скопировал интерфейс вместе с названием. Следующее поколение браузеров последовало этому примеру, и сегодня XMLHttpRequest является стандартом de facto.

##Отправка запроса
Чтобы отправить простой запрос, мы создаём объект запроса с конструктором XMLHttpRequest и вызываем методы open и send.

```
var req = new XMLHttpRequest();
req.open("GET", "example/data.txt", false);
req.send(null);
console.log(req.responseText);
// → This is the content of data.txt
```

Метод open настраивает запрос. В нашем случае мы решили сделать GET запрос на файл example/data.txt. URL, не начинающиеся с названия протокола (например, http:) называются относительными, то есть они интерпретируются относительно текущего документа. Когда они начинаются со слеша (/), они заменяют текущий путь – часть после названия сервера. В ином случае часть текущего пути вплоть до последнего слеша помещается перед относительным URL.

После открытия запроса мы можем отправить его методом send. Аргументом служит тело запроса. Для запросов GET используется null. Если третий аргумент для open был false, то send вернётся только после того, как был получен ответ на наш запрос. Для получения тела ответа мы можем прочесть свойство responseText объекта request.

Можно получить из объекта response и другую информацию. Код статуса доступен в свойстве status, а текст статуса – в statusText. Заголовки можно прочесть из getResponseHeader.

```
var req = new XMLHttpRequest();
req.open("GET", "example/data.txt", false);
req.send(null);
console.log(req.status, req.statusText);
// → 200 OK
console.log(req.getResponseHeader("content-type"));
// → text/plain
```

Названия заголовков не чувствительны к регистру. Они обычно пишутся с заглавной буквы в начале каждого слова, например “Content-Type”, но “content-type” или “cOnTeNt-TyPe” будут описывать один и тот же заголовок.

Браузер сам добавит некоторые заголовки, такие, как “Host” и другие, которые нужны серверу, чтобы вычислить размер тела. Но вы можете добавлять свои собственные заголовки методом setRequestHeader. Это нужно для особых случаев и требует содействия сервера, к которому вы обращаетесь – он волен игнорировать заголовки, которые он не умеет обрабатывать.

##Асинхронные запросы
В примере запрос был окончен, когда заканчивается вызов send. Это удобно потому, что свойства вроде responseText становятся доступными сразу. Но это значит, что программа наша будет ожидать, пока браузер и сервер общаются меж собой. При плохой связи, слабом сервере или большом файле это может занять длительное время. Это плохо ещё и потому, что никакие обработчики событий не сработают, пока программа находится в режиме ожидания – документ перестанет реагировать на действия пользователя.

Если третьим аргументом open мы передадим true, запрос будет асинхронным. Это значит, что при вызове send запрос ставится в очередь на отправку. Программа продолжает работать, а браузер позаботиться об отправке и получении данных в фоне.

Но пока запрос обрабатывается, мы не получим ответ. Нам нужен механизм оповещения о том, что данные поступили и готовы. Для этого нам нужно будет слушать событие “load”.

```
var req = new XMLHttpRequest();
req.open("GET", "example/data.txt", true);
req.addEventListener("load", function() {
  console.log("Done:", req.status);
});
req.send(null);
```

Так же, как вызов requestAnimationFrame в главе 15, этот код вынуждает нас использовать асинхронный стиль программирования, оборачивая в функцию тот код, который должен быть выполнен после запроса, и устраивая вызов этой функции в нужное время. Мы вернёмся к этому позже.

##Получение данных XML

Когда ресурс, возвращённый объектом XMLHttpRequest, является документом XML, свойство responseXML будет содержать разобранное представление о документе. Оно работает схожим с DOM образом, за исключением того, что у него нет присущей HTML функциональности навроде свойства style. Объект, содержащийся в responseXML, соответствует объекту document. Его свойство documentElement ссылается на внешний тег документа XML. В следующем документе (example/fruit.xml) таким тегом будет :

```
<fruits>
  <fruit name="banana" color="yellow"/>
  <fruit name="lemon" color="yellow"/>
  <fruit name="cherry" color="red"/>
</fruits>
```

Мы можем получить такой файл следующим образом:

```
var req = new XMLHttpRequest();
req.open("GET", "example/fruit.xml", false);
req.send(null);
console.log(req.responseXML.querySelectorAll("fruit").length);
// → 3
```

Документы XML можно использовать для обмена с сервером структурированной информацией. Их форма – вложенные теги – хорошо подходит для хранения большинства данных, ну или по крайней мере лучше, чем текстовые файлы. Интерфейс DOM неуклюж в плане извлечения информации, и XML документы получаются довольно многословными. Обычно лучше общаться при помощи данных в формате JSON, которые проще читать и писать – как программам, так и людям.

```
var req = new XMLHttpRequest();
req.open("GET", "example/fruit.json", false);
req.send(null);
console.log(JSON.parse(req.responseText));
// → {banana: "yellow", lemon: "yellow", cherry: "red"}
```

##Песочница для HTTP
HTTP-запросы из веб-страницы вызывают вопросы касаемо безопасности. Человек, контролирующий скрипт, может иметь интересы отличные от интересов пользователя, на чьём компьютере он запущен. Конкретно, если я зашёл на сайт themafia.org, я не хочу, чтобы их скрипты могли делать запросы к mybank.com, используя информацию моего браузера в качестве идентификатора, и давая команду отправить все мои деньги на какой-нибудь счёт мафии.

Вебсайты могут защитить себя от подобных атак, но для этого требуются определённые усилия, и многие сайты с этим не справляются. Из-за этого браузеры защищают их, запрещая скриптам делать запросы к другим доменам (именам вроде themafia.org и mybank.com).

Это может мешать разработке систем, которым надо иметь доступ к разным доменам по уважительной причине. К счастью, сервер может включать в ответ следующий заголовок, поясняя браузерам, что запрос может прийти с других доменов:

Access-Control-Allow-Origin: *

##Абстрагируем запросы
В главе 10 в нашей реализации модульной системы AMD мы использовали гипотетическую функцию backgroundReadFile. Она принимала имя файла и функцию, и вызывала эту функцию после прочтения содержимого файла. Вот простая реализация этой функции:

```
function backgroundReadFile(url, callback) {
  var req = new XMLHttpRequest();
  req.open("GET", url, true);
  req.addEventListener("load", function() {
    if (req.status < 400)
      callback(req.responseText);
  });
  req.send(null);
}
```

Простая абстракция упрощает использование XMLHttpRequest для простых GET-запросов. Если вы пишете программу, которая делает HTTP-запросы, будет неплохо использовать вспомогательную функцию, чтобы вам не приходилось всё время повторять уродливый шаблон XMLHttpRequest.

Аргумент callback (обратный вызов) – термин, часто использующийся для описания подобных функций. Функция обратного вызова передаётся в другой код, чтобы он мог позвать нас обратно позже.

Несложно написать свою вспомогательную функцию HTTP, специально скроенную под вашу программу. Предыдущая делает только GET-запросы, и не даёт нам контроля над заголовками или телом запроса. Можно написать ещё один вариант для запроса POST, или более общий, поддерживающий разные запросы. Многие библиотеки JavaScript предлагают обёртки для XMLHttpRequest.

Основная проблема с приведённой обёрткой – обработка ошибок. Когда запрос возвращает код статуса, обозначающий ошибку (от 400 и выше), он ничего не делает. В некоторых случаях это нормально, но представьте, что мы поставили индикатор загрузки на странице, показывающий, что мы получаем информацию. Если запрос не удался, потому что сервер упал или соединение прервано, страница будет делать вид, что она чем-то занята. Пользователь подождёт немного, потом ему надоест и он решит, что сайт какой-то дурацкий.

Нам нужен вариант, в котором мы получаем предупреждение о неудачном запросе, чтобы мы могли принять меры. Например, мы можем убрать сообщение о загрузке и сообщить пользователю, что что-то пошло не так.

Обработка ошибок в асинхронном коде ещё сложнее, чем в синхронном. Поскольку нам часто приходится отделять часть работы и размещать её в функции обратного вызова, область видимости блока try теряет смысл. В следующем коде исключение не будет поймано, потому что вызов backgroundReadFile возвращается сразу же. Затем управление уходит из блока try, и функция из него не будет вызвана.

```
try {
  backgroundReadFile("example/data.txt", function(text) {
    if (text != "expected")
      throw new Error("That was unexpected");
  });
} catch (e) {
  console.log("Hello from the catch block");
}
```

Чтобы обрабатывать неудачные запросы, придётся передавать дополнительную функцию в нашу обёртку, и вызывать её в случае проблем. Другой вариант – использовать соглашение, что если запрос не удался, то в функцию обратного вызова передаётся дополнительный аргумент с описанием проблемы. Пример:

```
function getURL(url, callback) {
  var req = new XMLHttpRequest();
  req.open("GET", url, true);
  req.addEventListener("load", function() {
    if (req.status < 400)
      callback(req.responseText);
    else
      callback(null, new Error("Request failed: " +
                               req.statusText));
  });
  req.addEventListener("error", function() {
    callback(null, new Error("Network error"));
  });
  req.send(null);
}
```

Мы добавили обработчик события error, который сработает при проблеме с вызовом. Также мы вызываем функцию обратного вызова с аргументом error, когда запрос завершается со статусом, говорящим об ошибке.

Код, использующий getURL, должен проверять не возвращена ли ошибка, и обрабатывать её, если она есть.

```
getURL("data/nonsense.txt", function(content, error) {
  if (error != null)
    console.log("Failed to fetch nonsense.txt: " + error);
  else
    console.log("nonsense.txt: " + content);
});
```

С исключениями это не помогает. Когда мы совершаем последовательно несколько асинхронных действий, исключение в любой точке цепочки в любом случае (если только вы не обернули каждый обработчик в свой блок try/catch) вывалится на верхнем уровне и прервёт всю цепочку.

##Обещания
Тяжело писать асинхронный код для сложных проектов в виде простых обратных вызовов. Очень легко забыть проверку на ошибку или позволить неожиданному исключению резко прервать программу. Кроме того, организация правильной обработки ошибок и проход ошибки через несколько последовательных обратных вызовов очень утомительна.

Предпринималось множество попыток решить эту проблему дополнительными абстракциями. Одна из наиболее удачных попыток называется обещаниями (promises). Обещания оборачивают асинхронное действие в объект, который может передаваться и которому нужно сделать какие-то вещи, когда действие завершается или не удаётся. Такой интерфейс уже стал частью текущей версии JavaScript, а для старых версий его можно использовать в виде библиотеки.

Интерфейс обещаний не особенно интуитивно понятный, но мощный. В этой главе мы лишь частично опишем его. Больше информации можно найти на <a href="http://www.promisejs.org">www.promisejs.org</a>

Для создания объекта promises мы вызываем конструктор Promise, задавая ему функцию инициализации асинхронного действия. Конструктор вызывает эту функцию и передаёт ей два аргумента, которые сами также являются функциями. Первая должна вызываться в удачном случае, другая – в неудачном.

И вот наша обёртка для запросов GET, которая на этот раз возвращает обещание. Теперь мы просто назовём его get.

```
function get(url) {
  return new Promise(function(succeed, fail) {
    var req = new XMLHttpRequest();
    req.open("GET", url, true);
    req.addEventListener("load", function() {
      if (req.status < 400)
        succeed(req.responseText);
      else
        fail(new Error("Request failed: " + req.statusText));
    });
    req.addEventListener("error", function() {
      fail(new Error("Network error"));
    });
    req.send(null);
  });
}
```

Заметьте, что интерфейс к самой функции упростился. Мы передаём ей URL, а она возвращает обещание. Оно работает как обработчик для выходных данных запроса. У него есть метод then, который вызывается с двумя функциями: одной для обработки успеха, другой – для неудачи.

```
get("example/data.txt").then(function(text) {
  console.log("data.txt: " + text);
}, function(error) {
  console.log("Failed to fetch data.txt: " + error);
});
```

Пока это всё ещё один из способов выразить то же, что мы уже сделали. Только когда у вас появляется цепь событий, становится видна заметная разница.

Вызов then производит новое обещание, чей результат (значение, передающееся в обработчики успешных результатов) зависит от значения, возвращаемого первой переданной нами в then функцией. Эта функция может вернуть ещё одно обещание, обозначая что проводится дополнительная асинхронная работа. В этом случае обещание, возвращаемое then само по себе будет ждать обещания, возвращённого функцией-обработчиком, и успех или неудача произойдут с таким же значением. Когда функция-обработчик возвращает значение, не являющееся обещанием, обещание, возвращаемое then, становится успешным, в качестве результата используя это значение.

Значит, вы можете использовать then для изменения результата обещания. К примеру, следующая функция возвращает обещание, чей результат – содержимое с данного URL, разобранное как JSON:

```
function getJSON(url) {
  return get(url).then(JSON.parse);
}
```

Последний вызов then не обозначил обработчик неудач. Это допустимо. Ошибка будет передана в обещание, возвращаемое через then, а ведь это нам и надо – getJSON не знает, что делать, когда что-то идёт не так, но есть надежда, что вызывающий её код это знает.

В качестве примера, показывающего использование обещаний, мы напишем программу, получающую число JSON-файлов с сервера, и показывающую во время исполнения запроса слово «загрузка». Файлы содержат информацию о людях и ссылки на другие файлы с информацией о других людях в свойствах типа отец, мать, супруг.

Нам нужно получить имя матери супруга из example/bert.json. В случае проблем нам нужно убрать текст «загрузка» и показать сообщение об ошибке. Вот как это можно делать при помощи обещаний:

```
<script>
  function showMessage(msg) {
    var elt = document.createElement("div");
    elt.textContent = msg;
    return document.body.appendChild(elt);
  }

  var loading = showMessage("Загрузка...");
  getJSON("example/bert.json").then(function(bert) {
    return getJSON(bert.spouse);
  }).then(function(spouse) {
    return getJSON(spouse.mother);
  }).then(function(mother) {
    showMessage("Имя - " + mother.name);
  }).catch(function(error) {
    showMessage(String(error));
  }).then(function() {
    document.body.removeChild(loading);
  });
</script>
```

Итоговая программа относительно компактна и читаема. Метод catch схож с then, но он ожидает только обработчик неудачного результата и в случае успеха передаёт дальше неизменённый результат. Исполнение программы будет продолжено обычным путём после отлова исключения – так же, как в случае с try/catch. Таким образом, последний then, удаляющий сообщение о загрузке, выполняется в любом случае, даже в случае неудачи.

Можно представлять себе, что интерфейс обещаний – это отдельный язык для асинхронной обработки исполнения программы. Дополнительные вызовы методов и функций, которые нужны для его работы, придают коду несколько странный вид, но не настолько неудобный, как обработка всех ошибок вручную.

##Цените HTTP
При создании системы, в которой программа на JavaScript в браузере (клиентская) общается с серверной программой, можно использовать несколько вариантов моделирования такого общения.

Общепринятый метод – удалённые вызовы процедур. В этой модели общение идёт по шаблону обычных вызовов функций, только функции эти выполняются на другом компьютере. Вызов заключается в создании запроса на сервер, в который входят имя функции и аргументы. Ответ на запрос включает возвращаемое значение.

При использовании удалённых вызовов процедур HTTP служит лишь транспортом для общения, и вы, скорее всего, напишете слой абстракции, который спрячет его полностью.

Другой подход – построить свою систему общения на концепции ресурсов и методов HTTP. Вместо удалённого вызова процедуры по имени addUser вы делаете запрос PUT к /users/larry. Вместо кодирования свойств пользователя в аргументах функции вы определяете формат документа или используете существующий формат, который будет представлять пользователя. Тело PUT-запроса, создающего новый ресурс, будет просто документом этого формата. Ресурс получается через запрос GET к его URL (/user/larry), который возвращает представляющий этот ресурс документ.

Второй подход упрощает использование некоторых возможностей HTTP, например поддержки кеширования ресурсов (копия ресурса хранится на стороне клиента). Также он способствует созданию согласованного интерфейса, потому что думать в терминах ресурсов проще, чем в терминах функций.

##Безопасность и HTTPS
Данные путешествуют по интернету по длинному и опасному пути. Чтобы добраться до пункта назначения, им надо попрыгать через всякие места, начиная от Wi-Fi сети кофейни до сетей, контролируемых разными организациями и государствами. В любой точке пути их могут прочитать или даже поменять.

Если нужно хранить что-либо в секрете, например пароли к емейлу, или данным необходимо прийти в пункт назначения в неизменном виде — таким, например, как номер банковского счёта, на который вы переводите деньги,- простого HTTP недостаточно.

Безопасный протокол HTTP, URL которого начинаются с https://, оборачивает HTTP-трафик так, чтобы его было сложнее прочитать и поменять. Сначала клиент подтверждает, что сервер – тот, за кого себя выдаёт, требуя с сервера представить криптографический сертификат, выданный авторитетной стороной, которую признаёт браузер. Потом, все данные, проходящие через соединение, шифруются так, чтобы предотвратить прослушку и изменение.

Таким образом, когда всё работает правильно, HTTPS предотвращает как случаи, когда кто-то притворяется другим веб-сайтом, с которым вы общаетесь, так и случаи прослушки вашего общения. Он не идеален, и уже были случаи, когда HTTPS не справлялся с работой из-за поддельных или краденых сертификатов или сломанных программ. Тем не менее, с HTTP очень легко сделать что-то плохое, а взлом HTTPS требует таких усилий, которые могут прикладывать только государственные структуры или очень серьёзные криминальные организации (а между этими организациями иногда совсем нет различий).

##Итог
В этой главе мы увидели, что HTTP – это протокол доступа к ресурсам в интернете. Клиент отправляет запрос, содержащий метод (обычно GET), и путь, который определяет ресурс. Сервер решает, что ему делать с запросом и отвечает с кодом статуса и телом ответа. Запросы и ответы могут содержать заголовки, в которых передаётся дополнительная информация.

Браузеры делают GET-запросы для получения ресурсов, необходимых для показа страницы. Страница может содержать формы, которые позволяют информации, введённой пользователем, быть отправленной в запросе, который создаётся после отправки формы. Вы узнаете об этом больше в следующей главе.

Интерфейс, через который JavaScript делает HTTP-запросы из браузера, называется XMLHttpRequest. Можно игнорировать приставку “XML” (но писать её всё равно нужно). Использовать его можно двумя способами: синхронным, который блокирует всю работу до окончания выполнения запроса, и асинхронным, который требует установки обработчика событий, отслеживающего окончание запроса. Почти во всех случаях предпочтительным является асинхронный способ. Создание запроса выглядит так:

```
var req = new XMLHttpRequest();
req.open("GET", "example/data.txt", true);
req.addEventListener("load", function() {
  console.log(req.statusCode);
});
req.send(null);
```

Асинхронное программирование – непростая вещь. Обещания – интерфейс, который делает её проще, помогая направлять сообщения об ошибках и исключения к нужному обработчику, и абстрагируя некоторые повторяющиеся элементы, подверженные ошибкам.

##Упражнения
###Согласование содержания (content negotiation)
Одна из вещей, которые HTTP умеет делать, но которую мы не обсуждали, называется согласованием содержания. Заголовок Accept в запросе можно использовать для сообщения серверу того, какие типы документов клиент желает получить. Многие серверы его игнорируют, но когда сервер знает о разных способах кодирования ресурса, он может взглянуть на заголовок и отправить тот, который предпочитает клиент.

URL eloquentjavascript.net/author настроен на ответ как прямым текстом, так и HTML или JSON, в зависимости от запроса клиента. Эти форматы определяются стандартизированными типами содержимого text/plain, text/html, и application/json.

Отправьте запрос для получения всех трёх форматов этого ресурса. Используйте метод setRequestHeader объекта XMLHttpRequest для установки заголовка Accept в один из нужных типов содержимого. Убедитесь, что вы устанавливаете заголовок после open, но перед send.

Наконец, попробуйте запросить содержимое типа application/rainbows+unicorns и посмотрите, что произойдёт.

###Ожидание нескольких обещаний
У конструктора Promise есть метод all, который, получая массив обещаний, возвращает обещание, которое ждёт завершения всех указанных в массиве обещаний. Затем он выдаёт успешный результат и возвращает массив с результатами. Если какие-то из обещаний в массиве завершились неудачно, общее обещание также возвращает неудачу (со значением неудавшегося обещания из массива).

Попробуйте сделать что-либо подобное, написав функцию all.

Заметьте, что после завершения обещания (когда оно либо завершилось успешно, либо с ошибкой), оно не может заново выдать ошибку или успех, и дальнейшие вызовы функции игнорируются. Это может упростить обработку ошибок в вашем обещании.

```
function all(promises) {
  return new Promise(function(success, fail) {
    // Ваш код.
  });
}

// Проверочный код.
all([]).then(function(array) {
  console.log("Это должен быть []:", array);
});
function soon(val) {
  return new Promise(function(success) {
    setTimeout(function() { success(val); },
               Math.random() * 500);
  });
}
all([soon(1), soon(2), soon(3)]).then(function(array) {
  console.log("Это должен быть [1, 2, 3]:", array);
});
function fail() {
  return new Promise(function(success, fail) {
    fail(new Error("бабах"));
  });
}
all([soon(1), fail(), soon(3)]).then(function(array) {
  console.log("Сюда мы попасть не должны ");
}, function(error) {
  if (error.message != "бабах")
    console.log("Неожиданный облом:", error);
});
```
