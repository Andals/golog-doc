.. _buffer:

buffer
=============
实现对writer添加buffer，提升效率。

buffer是writer的装饰，所以依旧是IWriter。

.. contents:: 目录

使用
------

demo::

    import (
        "andals/golog"
    )

    golog.InitBufferAutoFlushRoutine(1024, time.Second*7)

	path := "/tmp/test_buffer.log"
	bufsize := 4096

	fw, _ := golog.NewFileWriter(path)
	bw := golog.NewBuffer(fw, bufsize)

	bw.Write([]byte("test file writer with buffer and time interval\n"))
	time.Sleep(time.Second * 5)
	bw.Free()

    golog.FreeBuffers()

特别注意
----------

本buffer会自动启动一个goroutine，在后台自动定时将buffer中的信息刷到对应的write中调用Write方法。

1. 使用buffer前，一定要先进行初始化

调用::

    func InitBufferAutoFlushRoutine(maxBufNum int, timeInterval time.Duration)

2. 程序结束前，一定要释放所有buffer，这样才能不丢失log。

调用::

    func FreeBuffers()
