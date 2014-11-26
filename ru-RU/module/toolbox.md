---
name: Toolbox Module
sort: 6
---

# Модуль ToolBox

Этот модуль похож на [Dropwizard framework](https://github.com/dropwizard/dropwizard). Один из пользователей рассказал мне об этом фреймворке и он действительно крутой. [issue 128](https://github.com/astaxie/beego/issues/128) рассказывает об этой фиче. Я добавил много различных возможностей: healthcheck, profile, statistics и task.

## Установка

	go get github.com/astaxie/beego/toolbox

## Healthcheck

Может проверять состояние здоровья вашего приложения. Для примера: проверяет доступна ли база данных:

```
type DatabaseCheck struct {
}

func (dc *DatabaseCheck) Check() error {
	if dc.isConnected() {
		return nil
	} else {
		return errors.New("can't connect database")
	}
}
```
После это вы можете добавить этот объект как проверяемый:

```
toolbox.AddHealthCheck("database",&DatabaseCheck{})
```

После этого вы можете послать GET запрос на `/healthcheck`:

	$ curl http://beego.me:8088/healthcheck
	* deadlocks: OK
	* database: OK

Этот запрос вернет данные о состоянии базы данных.

## profile

Мониторинг производительности запущенных процессов это хороший путь для оптимизации производительности и наблюдением за ошибками вашего приложения. Для примера: информация о GC и горутинах. Профайлер предоставляет простой путь для отладки вашего приложения. Он использует `ProcessInput` функцию для исполнения запросов. Вот некоторые примеры:

- lookup goroutine

	Выводит все запущенные задачи и все горутины. Простой способ посмотреть что делают ваши горутины.

		goroutine 3 [running]:
		runtime/pprof.writeGoroutineStacks(0x634238, 0xc210000008, 0x62b000, 0xd200000000000000)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:511 +0x7c
		runtime/pprof.writeGoroutine(0x634238, 0xc210000008, 0x2, 0xd2676410957b30fd, 0xae98)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:500 +0x3c
		runtime/pprof.(*Profile).WriteTo(0x52ebe0, 0x634238, 0xc210000008, 0x2, 0x1, ...)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:229 +0xb4
		_/Users/astaxie/github/beego/toolbox.ProcessInput(0x2c89f0, 0x10, 0x634238, 0xc210000008)
			/Users/astaxie/github/beego/toolbox/profile.go:26 +0x256
		_/Users/astaxie/github/beego/toolbox.TestProcessInput(0xc21004e090)
			/Users/astaxie/github/beego/toolbox/profile_test.go:9 +0x5a
		testing.tRunner(0xc21004e090, 0x532320)
			/Users/astaxie/go/src/pkg/testing/testing.go:391 +0x8b
		created by testing.RunTests
			/Users/astaxie/go/src/pkg/testing/testing.go:471 +0x8b2

		goroutine 1 [chan receive]:
		testing.RunTests(0x315668, 0x532320, 0x4, 0x4, 0x1)
			/Users/astaxie/go/src/pkg/testing/testing.go:472 +0x8d5
		testing.Main(0x315668, 0x532320, 0x4, 0x4, 0x537700, ...)
			/Users/astaxie/go/src/pkg/testing/testing.go:403 +0x84
		main.main()
			_/Users/astaxie/github/beego/toolbox/_test/_testmain.go:53 +0x9c

- lookup heap

	Выводит информацию о текущем использовании ресурсов:


		heap profile: 1: 288 [2: 296] @ heap/1048576
		1: 288 [2: 296] @


		# runtime.MemStats
		# Alloc = 275504
		# TotalAlloc = 275512
		# Sys = 4069608
		# Lookups = 5
		# Mallocs = 469
		# Frees = 1
		# HeapAlloc = 275504
		# HeapSys = 1048576
		# HeapIdle = 647168
		# HeapInuse = 401408
		# HeapReleased = 0
		# HeapObjects = 468
		# Stack = 24576 / 131072
		# MSpan = 4472 / 16384
		# MCache = 1504 / 16384
		# BuckHashSys = 1476472
		# NextGC = 342976
		# PauseNs = [370712 77378 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
		# NumGC = 2
		# EnableGC = true
		# DebugGC = false

- lookup threadcreate

	Выводит информацию о тредах:

		threadcreate profile: total 4
		1 @ 0x17f68 0x183c7 0x186a8 0x188cc 0x19ca9 0xcf41 0x139a3 0x196c0
		#	0x183c7	newm+0x27			/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x186a8	startm+0xb8			/Users/astaxie/go/src/pkg/runtime/proc.c:974
		#	0x188cc	handoffp+0x1ac			/Users/astaxie/go/src/pkg/runtime/proc.c:992
		#	0x19ca9	runtime.entersyscallblock+0x129	/Users/astaxie/go/src/pkg/runtime/proc.c:1514
		#	0xcf41	runtime.notetsleepg+0x71	/Users/astaxie/go/src/pkg/runtime/lock_sema.c:253
		#	0x139a3	runtime.MHeap_Scavenger+0xa3	/Users/astaxie/go/src/pkg/runtime/mheap.c:463

		1 @ 0x17f68 0x183c7 0x186a8 0x188cc 0x189c3 0x1969b 0x2618b
		#	0x183c7	newm+0x27		/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x186a8	startm+0xb8		/Users/astaxie/go/src/pkg/runtime/proc.c:974
		#	0x188cc	handoffp+0x1ac		/Users/astaxie/go/src/pkg/runtime/proc.c:992
		#	0x189c3	stoplockedm+0x83	/Users/astaxie/go/src/pkg/runtime/proc.c:1049
		#	0x1969b	runtime.gosched0+0x8b	/Users/astaxie/go/src/pkg/runtime/proc.c:1382
		#	0x2618b	runtime.mcall+0x4b	/Users/astaxie/go/src/pkg/runtime/asm_amd64.s:178

		1 @ 0x17f68 0x183c7 0x170bc 0x196c0
		#	0x183c7	newm+0x27		/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x170bc	runtime.main+0x3c	/Users/astaxie/go/src/pkg/runtime/proc.c:191

		1 @

- lookup block

	Выводит информацию о блоках:

		--- contention:
		cycles/second=2294781025

- start cpuprof

	Запускает запись cpuprof информации в файл cpu-pid.pprof.

- stop cpuprof

  Останавливает запись.

- get memprof

	Запускает запись memprof в файл mem-pid.memprof

- gc summary

  Проверяет статус GC:

		NumGC:2 Pause:54.54us Pause(Avg):170.82us Overhead:177.49% Alloc:248.97K Sys:3.88M Alloc(Rate):1.23G/s Histogram:287.09us 287.09us 287.09us

## Статистика

Посмотрите на картинку, что там происходит? Это круто, верно? Модуль toolbox это поддерживает.

![](../images/toolbox.jpg)

Как можно использовать возможности статистики? Добавьте её в ваше приложение вот так:

	toolbox.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(2000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(120000))
	toolbox.StatisticsMap.AddStatistics("GET", "/api/user", "&admin.user", time.Duration(13000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/admin", "&admin.user", time.Duration(14000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user/astaxie", "&admin.user", time.Duration(12000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user/xiemengjun", "&admin.user", time.Duration(13000))
	toolbox.StatisticsMap.AddStatistics("DELETE", "/api/user", "&admin.user", time.Duration(1400))

Получите статистическую информацию:

	toolbox.StatisticsMap.GetMap(os.Stdout)

Пример вывода:

	| requestUrl                                        | method     | times            | used             | max used         | min used         | avg used         |
	| /api/user                                         | POST       |  2               | 122.00us         | 120.00us         | 2.00us           | 61.00us          |
	| /api/user                                         | GET        |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |
	| /api/user                                         | DELETE     |  1               | 1.40us           | 1.40us           | 1.40us           | 1.40us           |
	| /api/admin                                        | POST       |  1               | 14.00us          | 14.00us          | 14.00us          | 14.00us          |
	| /api/user/astaxie                                 | POST       |  1               | 12.00us          | 12.00us          | 12.00us          | 12.00us          |
	| /api/user/xiemengjun                              | POST       |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |

## Задачи

Задачи работают очень похоже на то как работает crontab. Задачи используют цикл запросов и ответов для запуска. Но они так же могут быть вызваны по ситуации или запланированы по-расписанию. Для примера: для составления отчета по использованию памяти и состоянии горутин или для очистки лог файлов по таймеру.

Для запуска новой задачи [https://godoc.org/github.com/astaxie/beego/toolbox#NewTask](toolbox.NewTask):

	tk1 := toolbox.NewTask("tk1", "0 12 * * * *", func() error {
		fmt.Println("tk1")
		return nil
	})


Сигнатура объекта NewTask:

	NewTask(tname string, spec string, f TaskFunc) *Task
	- tname: task name
	- spec: task format. See details below.
	- f: the function will be run as the task

Для реализации этой задачи добавьте её в глобальный список задач и запустите.

	toolbox.AddTask("tk1", tk1)
	toolbox.StartTask()
	defer toolbox.StopTask()

Тестирование TaskFunc

Используйте код ниже для теста того что TaskFunc работает корректно.

	err := tk.Run()
	if err != nil {
		t.Fatal(err)
	}


### spec в деталях

Формат spec повторяет формат crontab.

```
// Первые шесть частей:
//       second: 0-59
//       minute: 0-59
//       hour: 1-23
//       day: 1-31
//       month: 1-12
//       weekdays: 0-6（0 is Sunday）

// Специальные символы:
//       *: any time
//       ,: separator. E.g.: 2,4 означают что задача будет запущена в 2 и 4 часа
//　　    －: range. E.g.: 1-5 означает что задача будет запущена между 1 и 5 часами
//       /n : run once every n time. E.g.: */1 означает что задача запускается каждый час. Тоже самое с 1-23/1
/////////////////////////////////////////////////////////
//	0/30 * * * * *                        запуск каждые 30 секунд
//	0 43 21 * * *                         запуск в 21:43
//	0 15 05 * * *                         запуск в 05:15
//	0 0 17 * * *                          запуск в 17:00
//	0 0 17 * * 1                          запуск в 17:00 каждый понедельник
//	0 0,10 17 * * 0,2,3                   запуск в 17:00 и 17:10 каждые воскресенье, вторник и среду
//	0 0-10 17 1 * *                       запускается каждую минуту с 17:00 до 7:10 в первый день каждого месяца
//	0 0 0 1,15 * 1                        запуск в 0:00 1-ого и 15-ого числа каждого месяца и каждый понедельник
//	0 42 4 1 * *                          запуск в 4:42 1-ого числа каждого месяца
//	0 0 21 * * 1-6                        запуск в 21:00 с понедельника по субботу
//	0 0,10,20,30,40,50 * * * *            запуск каждые 10 минут
//	0 */10 * * * *                        запуск каждые 10 минут
//	0 * 1 * * *                           запуск каждую минуту между 1:00 и 1:59
//	0 0 1 * * *                           запуск в 1:00
//	0 0 */1 * * *                         запуск в :00 каждого часа
//	0 0 * * * *                           запуск в :00 каждого часа
//	0 2 8-20/3 * * *                      запуск в 8:02, 11:02, 14:02, 17:02 и 20:02
//	0 30 5 1,15 * *                       запуск в 5:30 1-ого и 15-ого числа каждого месяца
```
