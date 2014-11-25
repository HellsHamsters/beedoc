---
name: Config Module
sort: 7
---

# Разбор конфигурационных файлов

Этот модуль используется для разбора конфигов. При его создании мы вдохновлялись `database/sql`.
Он поддерживает ini, json, xml, yaml файлы. Что бы его установить используейте команду:

	go get github.com/astaxie/beego/config

>>>если вы хотите парсить xml или yaml, то вы сначала должны установить

	go get -u github.com/astaxie/beego/config/xml

>>>и подключить XML модуль

    import _ "github.com/astaxie/beego/config/xml"

## Простое использование

Создаем объект парсера:

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}

Получаем данные из файла:

	iniconf.String("appname")

Вот список всех доступных методов парсера:

* Set(key, val string) error
* String(key string) string
* Int(key string) (int, error)
* Int64(key string) (int64, error)
* Bool(key string) (bool, error)
* Float(key string) (float64, error)
* DIY(key string) (interface{}, error)

Конфигурационные файлы в формате ini поддерживают секции. Вы можете получить значение с помощью подобного выражения `section::key`.

Для примера:

  [demo]
  key1="asta"
  key2 = "xie"

Вы можете использовать `iniconf.String("demo::key2")` что бы получить значение
