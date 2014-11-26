---
name: Cache Module
sort: 2
---

# Кэш модуль

Модуль кэша BeeGo используется для кэширования данных. Его создание было вдохновлено `database/sql`. Он поддерживает четыре провайдера данных: файл, оперативная память, memcache и redis. Установить модуль можно так:

	go get github.com/astaxie/beego/cache

>>>если вы используете memcache или redis провайдер, вы должны сначала установить модуль

	go get -u github.com/astaxie/beego/cache/memcache

>>>и потом импортировать его.

    import _ "github.com/astaxie/beego/cache/memcache"

## Базовое использование

Первый шаг это импорт пакета:

	import (
		"github.com/astaxie/beego/cache"
	)

Следом надо инициализировать глобальную переменную с объектом кэша:

	bm, err := cache.NewCache("memory", `{"interval":60}`)

Теперь мы можем использовать `bm` для работы с кэшом:

	bm.Put("astaxie", 1, 10)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")

## Настройки провайдера

Как настраиваются четыре поддерживаемых провайдера:

- memory

  `interval` устанавливает таймер для GC, который означает что каждые 60 секунд кэш будет очищаться:

  {"interval":60}

- file

		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}

- redis

	redis использует [redigo](http://github.com/garyburd/redigo/redis)

		{"conn":":6039"}

- memcache

  memcache использует [vitess](http://code.google.com/p/vitess/go/memcache)

		{"conn":"127.0.0.1:11211"}

## Создание вашего собственного провайдера

Модуль кэша имплементирует интерфейс Cache, поэтому вы можете создать ваш собственный кэш провайдер с помощью имплементации этого интерфейса.

	type Cache interface {
		Get(key string) interface{}
		Put(key string, val interface{}, timeout int64) error
		Delete(key string) error
		Incr(key string) error
		Decr(key string) error
		IsExist(key string) bool
		ClearAll() error
		StartAndGC(config string) error
	}

Не забудьте зарегестрировать ваш провайдер

	func init() {
		Register("myowncache", NewOwnCache())
	}
