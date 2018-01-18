*This article assumes that you have a running instance of Pydio. It will try to give you some tips and tricks to make your Pydio run smoothly.*

### 1. Picking the right server, configuring it correctly

This is out of the scope, but the first important parameter is of course to choose the right hardware ( make sure to give plenty of CPU if you can), and the right software to run Pydio (Apache, Nginx, etc...). As the most used all around the world, Apache is usually a good choice, but if you have a good knowledge about Nginx + PHP FPM setup, it can be a better option. If you are not sure, stick to Apache.

### 2. PHP Configuration

#### Output_buffering

One important thing regarding the PHP config (it should be displayed in the diagnostic tool at Pydio install time): make sure to **disable output_buffering in your PHP configuration**. Pydio is trying whenever it's possible, to send data to the users even before the PHP request has finished processing. This enhances the responsiveness of the interface, but this can be done only if output_buffering is Off in your php.ini. 

Find the right php.ini,
on linux based distributions you can use `php --ini` on your terminal and look at the "Loaded Configuration File" line
and set the parameter value:  `Output_buffering Off`

#### Opcode Caching

Nowadays, it's a standard setup to have an opcode caching mechanism integrated in PHP. **OpCache** comes bundled by default since php 5.4, but make sure it's enabled. Otherwise, you can use **XCache** or **APC** extension. This is critical, as Pydio framework has a lot of files that are heavy to load on each request. Opcode caching avoids this operation.
To know if you have **OpCache** enabled type `php -v`
```
PHP 7.0.22-0ubuntu0.16.04.1 (cli) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.22-0ubuntu0.16.04.1, Copyright (c) 1999-2017, by Zend Technologies
```
and look for a line that looks like this one : **with Zend OPcache v7.0.22-0ubuntu0.16.04.1, Copyright (c) 1999-2017, by Zend Technologies**

#### Command-line PHP

As described above regarding **output_buffering** and php execution after the http connection is closed, Pydio can push the "background-tasking" a step further by sending long-running operation directly on the server's command-line. In that case, it's fully non-blocking and allows the user to carry on navigation. This is for example used for compressing/decompression operations, indexation, etc. 

This feature requires PHP to be available via the command line, and you must activate the Command Line option in the Pydio Main Options via the admin panel. 

[Note] *If you are using the Enterprise Distribution, make sure **[IonCube](https://pydio.com/en/docs/v8/troubleshooting#content)** extension is correctly installed for the command line as well.*

### 4. In-memory caching: Pydio on steroids

**Pydio 6.4 and later**

In Pydio 6.4, we worked on implementing a more generic interface that will be able to support other Key-Value Stores, like Memcache, Memcached, Redis, etc. 

It is now as simple as going to your admin dashboard and enabling this feature. For the Community edition, see **Menu > Cache Server**. 
For Enterprise Distribution,  see Cache Server in the left-hand menu. Here you can pick the key-value store you want to use. It can still be APC, but we also support Memcache, MemcacheD and Redis. These are third party servers that must be run separately from Pydio.