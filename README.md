Hostkey Nginx web-server role
=============================

Стандартная роль для Nginx web-сервера
Роль устанавливает и конфигурирует Nginx-сервер (используется стандартный репозиторий)

Специфика исполнения
--------------------

Роль не устанавливает сертификаты, только проверяет их наличие и логгирует результат.
Проверка наличия/отсутствия сертификатов и ключей для конфигурационных файлов conf.d завернута в тег **check_ssl_certificates**

Global Variable Defaults
------------------------

Обязательные параметры. Умолчание задано.
* `hk_nginx_user: nginx`
  устанавливает пользователя Nginx
* `hk_nginx_connections: 1024`
  определяет параметр worker_connections
* `hk_nginx_gzip: false`
  вставляет в контекст http стандартный блок настроек gzip
* `hk_nginx_directory: /etc/nginx`
  корневая директория конфигурации Nginx
* `hk_nginx_ssl_directory: "{{ hk_nginx_directory }}/ssl"`
  директория для хранения ssl-ключей и сертификатов
* `hk_nginx_log_directory: /var/log/nginx`
  стандартная директория для хранения текстовых логов
* `hk_nginx_default_server_config: true`
  выключение параметра приводит к запрету установки конфигурации Default HTTP server (см. files), в случае отсутствия custom-конфигурации. Если конфигурационный файл default.conf создан, при установлке параметра в false - он будет удален.

Nginx HTTP Generic Variables
----------------------------

Необязательные параметры, умолчание не задано.
В контекст http возможно добавить произвольные параметры, с помощью Generic Variables.
Пример:

```
hk_nginx_http_generics:
  param1:
    name: proxy_connect_timeout
    value: 100
  param2:
    name: client_max_body_size
    value: 100m
```

Nginx Server Template (variables)
---------------------------------

Задание параметров запускает добавление конфигурации для сервисов, не обязательно к использованию, умолчание не задано.

### Обязательные Параметры (Server Context):

* `name`
  используется для задания стандартных имен для конфигурации сервера и log-файлов.
* `ssl`
  _true/false_ используется для указания типа сервера.
* `listen`
  задает директиву listen в контексте сервера. Примеры значений:
    80
    127.0.0.1:80
    {{ ansible_default_ipv4.address }}:443
* `server_name`
  задает директиву server_name в контексте сервера. Допустимо указание нескольких server_name в строку, через пробел, в кавычках.
* `ssl_required.certificate`
  обязательный параметр, если параметр **ssl** установлен в _true_. Задает путь к ssl-сертификату.
* `ssl_required.certificate_key`
  обязательный параметр, если параметр **ssl** установлен в _true_. Задает путь к приватному ключу для ssl-сертификата.

### Необязательные Параметры (Server Context):

* `listen_params`
  Дополнительные параметры для директивы listen http://nginx.org/ru/docs/http/ngx_http_core_module.html#listen
* `http2https`
  Если задан, вставляет стандартный блок редиректа http -> https (использовать для не ssl-сервера)
* `robotsdisallow`
  Если задан, выставляет стандартный блок запрета чтения / роботами
* `root`
  Задает значение для параметра root
* `index`
  Задает значение для параметра index
* `access_log`
  _true/false_ используется для включения/отключения стандартного лога в контексте сервера.
* `error_log`
  _true/false_ используется для включения/отключения стандартного лога в контексте сервера.
* `ssl_optional`
  произвольные опциональные параметры ssl, используются если **ssl** установлен в _true_.
  Пример:

```
hk_nginx_server:
  - name: {{ ansible_fqdn }}
    ssl: true
    ssl_optional:
      param1:
        name: ssl_protocols
        value: TLSv1 TLSv1.1 TLSv1.2
      param2:
        name: ssl_ciphers
        value: AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5
```


* `generics`
  произвольные опциональные параметры серверного контекста.
  Пример:

```
hk_nginx_server:
  - name: {{ ansible_fqdn }}
    generics:
      param1:
        name: keepalive_timeout
        value: 10s
      param2:
        name: ignore_invalid_headers
        value: 'on'
```


### Обязательные параметры (Location Context)

* `name`
  обязателен, если включено логгирование на уровне локации. Используется для задания имени log-файла.
* `path`
  обязательная директива, задающая локацию.
* `fastcgi_pass`
  обязателен, если **standart_fastcgi** установлен в _true_. Используется для указания параметра fastcgi_pass.

### Не обязательные параметры (Location Context)

* `alias`
  Задает значение для параметра alias.
* `root`
  Задает значение для параметра root.
* `access_log`
  _true/false_ используется для включения/отключения стандартного лога в контексте локации.
* `error_log`
  _true/false_ используется для включения/отключения стандартного лога в контексте локации.
* `standart_proxy_headers`
  включает стандартный блок с пробросом заголовоков на backend.
* `standart_fastcgi`
  включает стандартный блок с минимальными настройками fastcgi.
* `standart_websocket`
  включает стандартный блок с настройками для websocket.
* `generics`
  произвольные опциональные параметры контекста локации.
  Пример:

```
hk_nginx_server:
  - name: {{ ansible_fqdn }}
    locations:
      location1:
        generics:
          custom_rule1:
            name: try_files
            value: '$uri /images/default.gif'
          custom_rule2:
            name: types_hash_bucket_size
            value: 128
```

Nginx Rewrite Conditions
----------------------------

Необязательные параметры, умолчание не задано.
В контекст http возможно добавить условия для проверки.
Пример:

```
html:
  path: ~ ^/.+\.html$
  root: "{{ hk_document_root }}"
  generics_if:
    rewrite_1:
      condition: '!-e $request_filename'
      name: rewrite
      value: ^(.*)$ /index.php?$1 last

Ниже показан пример как это будет отображено в nginx.conf файле

location ~ ^/.+\.html$ {
        root {{ hk_document_root }};
        if ( !-e $request_filename ) {
                 rewrite ^(.*)$ /index.php?$1 last;
        }
}

```
