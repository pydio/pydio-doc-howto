*This article assumes you have a running instance of Pydio. It will try to give some tips and tricks to make your Pydio run smoother.*

### 1. Picking the right server, configuring it correctly

This is out of the scope, but the first important parameter is of course to choose the right hardware (make sure to give plenty of CPU if you can), and the right software to run Pydio (Apache, Nginx, etc...). As the most used all around the world, Apache is generally a good choice, but if you have a good knowledge of Nginx + PHP FPM setup, it can be a better option. If you are not sure, stick to Apache.

### 2. PHP Configuration

#### Output_buffering

One important thing regarding the PHP config (it should be displayed in the diagnostic tool at Pydio install time): make sure to **disable output_buffering in your PHP configuration**. Pydio is trying whenever it's possible, to send data to the users even before the PHP request has finished processing. This enhances the responsiveness of the interface, but this can be done only if output_buffering is Off in your php.ini. 

Find the right php.ini, and set the parameter value: 
```Output_buffering Off```

#### Opcode Caching

Nowadays, it's a standard setup to have an opcode caching mechanism integrated in PHP. **Opcache** comes bundled by default since php5.4, but make sure it's enabled. Otherwise, you can use **XCache** or **APC** extension. This is critical, as Pydio framework has a lot of files that are heavy to load on each request. Opcode caching avoids this operation.


#### Command-line PHP

As described above regarding output_buffering and php execution after the http connection is closed, Pydio can push the "background-tasking" a step further by sending long-running operation directly on the server command-line. In that case, it's fully non-blocking and allows the user to carry on navigation. This is for example used for compressing/decompression operations, indexation, etc. 

This feature requires PHP to be available via the command line, and you must activate the Command Line option in the Pydio Main Options via the admin panel. 

[Note] *If you are using the Enterprise Distribution, make sure IonCube extension is correctly installed for the command line as well.*

### 4. In-memory caching: Pydio on steroids

Since v6.2.X, we introduced an in-memory caching system that allows Pydio to put some data in memory across http requests (see also [https://pydio.com/en/blog/how-we-improved-pydio-performances-using-blackfireio]()). The current implementation is relying on APC/APCu. Depending on your system, you should be able to activate this PHP Extension. For systems that don't support APC anymore, APCu should be available for emulating the legacy Key/Value Caching system that APC was providing. 

Once it is installed, just edit the *conf/boostrap\_context.php* (or */etc/pydio/bootstrap\_context.php*) file and set the **AJXP\_KVCACHE\_IGNORE** flag to false instead of true. The additional parameter **AJXP\_KVCACHE\_PREFIX** can be used to separate multiple pydio instances running on the same server: make sure to have each pydio using a specific prefix!

Sample: 

```
// KEY-VALUE-CACHE 
define("AJXP_KVCACHE_PREFIX", "my-pydio-instance"); 
define("AJXP_KVCACHE_IGNORE", false);
```


[Note] *We are currently working on a more generic interface that will be able to support other Key-Value Stores, like Memcache, Memcached, Redis, etc.*