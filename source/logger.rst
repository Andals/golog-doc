.. _logger:

logger
=============
本包实现统一的log接口，并实现了简单的同步logger和异步logger。

.. contents:: 目录

统一log级别、接口
-----------------------

LEVEL::

    const (
        LEVEL_DEBUG     = 1
        LEVEL_INFO      = 2
        LEVEL_NOTICE    = 3
        LEVEL_WARNING   = 4
        LEVEL_ERROR     = 5
        LEVEL_CRITICAL  = 6
        LEVEL_ALERT     = 7
        LEVEL_EMERGENCY = 8
    )

ILogger::

    type ILogger interface {
        Debug(msg []byte)
        Info(msg []byte)
        Notice(msg []byte)
        Warning(msg []byte)
        Error(msg []byte)
        Critical(msg []byte)
        Alert(msg []byte)
        Emergency(msg []byte)

        Log(level int, msg []byte) error

        Flush() error
        Free()
    }

简单同步logger
------------------

::

    func NewSimpleLogger(writer IWriter, globalLevel int, formater IFormater) (*simpleLogger, error)

改logger在执行log操作时会阻塞等待对应的writer返回

Demo::

    import (
        "andals/golog"
    )

	fw, _ := golog.NewFileWriter("/tmp/test_simple_buffer_logger.log")
	bw := golog.NewBuffer(fw, 1024)
	logger, _ := golog.NewSimpleLogger(bw, log.LEVEL_INFO, new(log.SimpleFormater))

	msg := []byte("test simple logger")

	logger.Debug(msg)
	logger.Info(msg)
	logger.Notice(msg)
	logger.Warning(msg)
	logger.Error(msg)
	logger.Critical(msg)
	logger.Alert(msg)
	logger.Emergency(msg)

	logger.Free()

异步logger
--------------

如果不想执行log操作时被阻塞，可以使用改对象，该对象会把记录log的操作放到指定的goroutine中去做。

实测使用异步logger对性能又极大提升。

异步logger对象是对ILogger的装饰，所以本质上还是ILogger。

特别注意
*********************

1. 先调用::

    func InitAsyncLogRoutine(msgQueueLen int)

2. 程序结束前调用::

   func FreeAsyncLogRoutine()

这个要在程序退出前做，这样避免log丢失::

Demo
*************

::

    import (
        "andals/golog"
    )

    golog.InitAsyncLogRoutine(4096)
    defer golog.FreeAsyncLogRoutine()

	fw, _ := golog.NewFileWriter("/tmp/test_async_web_logger.log")
	bw := golog.NewBuffer(fw, 1024)
	sl, _ := golog.NewSimpleLogger(bw, log.LEVEL_INFO, log.NewWebFormater([]byte("async_web")))
	logger := golog.NewAsyncLogger(sl)

	msg := []byte("test async web logger")

	logger.Debug(msg)
	logger.Info(msg)
	logger.Notice(msg)
	logger.Warning(msg)
	logger.Error(msg)
	logger.Critical(msg)
	logger.Alert(msg)
	logger.Emergency(msg)

	logger.Free()
