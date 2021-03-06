RxJavaDemo基本用法：
###RxJava(函数式编程): #####源码解释:a library for composing asynchronous and event-based programs using observable sequences for the Java VM

释义:一个java的VM上使用可观测的序列来组成异步的，基于事件的程序的库

涉及到的核心名词:

1.Observables(被观察者，事件源):Observables发出一系列(0个或者多个)事件
2.Subscribers(观察者):Subscribers处理这些事件(执行Subscriber.onNext()或者Subscriber.onError())；

 备注:与观察者的区别是：Observerable没有任何的Subscriber,那么这个Observerable是不会发出任何事件的；
使用准备工作:配置－在Module下build.gradle的dependencies中添加如下话语：

 compile 'io.reactivex:rxjava:1.2.1'
 compile 'io.reactivex:rxandroid:1.2.1'
备注:想要使用函数式编程范式需要配置lambda，配置如下说明：

《1》. 增加classpath
 在Project的buildscript->dependencies中增加classpath
 classpath 'me.tatarka:gradle-retrolambda:3.2.4'
《2》. 增加plugin
  在module的build.gradle中增加plugin
   apply plugin: 'me.tatarka.retrolambda'
《3》.增加compileOptions
  在module的build.gradle->android中增加compileOptions
  compileOptions {
  sourceCompatibility JavaVersion.VERSION_1_8
  targetCompatibility JavaVersion.VERSION_1_8
  }
使用介绍说明如下：
1.基本方式1:
1》使用Observable的create()创建被观察者，重写call(),在该方法中调用观察者的onNext(),onCompleted(),onError()方法
2》new Subscriber(){}创建观察者，重写onNext()(输出信息),onCompleted(),onError()
3》被观察者Observerable对象调用subscribe(Subscriber对象);完成subscriber对observable的订阅

案例:请看项目中方法baseUseRxjava1()
基本方式2: 1》使用Observable的just()创建被观察者返回被观察者对象；
2》被观察者对象调用subscribe(new Action(){publid void call(T t){}})

       备注：使用lambda范式为：
          Observable.just("哈哈哈")
                    .subscribe(s ->
               Log.e("=====", "====callfff输出内容为==" + s));
案例:请看项目中方法baseUseRxjava2()
基本方式3:
1》使用Observable的from(T[] array)创建被观察者;(接收一个集合作为输入，然后每次输出一个元素给subscriber)
2》被观察者对象调用subscribe(new Action(){publid void call(T t){}})

 案例:请看项目中方法baseUseRxjava3()
###2. RxJava的常用操作符的介绍：
1》map()：把一个事件转换为另一个事件(用于变换Observable对象的，实现链式调用，最终将最简洁的数据传递给Subscriber对象)
示例：

     //刚创建的Observable是String类型的
      Observable.just("Hellp Map Operator")
              .map(new Func1<String, Integer>() {
                  @Override
                  public Integer call(String s) {
                      Log.e("=====", "=第一个map转化==");
                      return 2016;
                  }
              }).map(new Func1<Integer, String>() {
          @Override
          public String call(Integer integer) {
              Log.e("=====", "=第二个map转化=");
              return String.valueOf(integer);
          }
      }).subscribe(new Action1<String>() {
          @Override
          public void call(String s) {
              Log.e("=====", "=最终结果="+s);

          }
      });
使用范式形式：

           //范式形式如下：
           Observable.just("Hellp Map Operator")
                   //第一次转化
                   .map(s -> "2016")
                   //第二次转化
                   .map(m -> 2016)
                   //最终结果发给观察者
                   .subscribe(k -> Log.e("=====", "=最终结果=" + k));
       }
2》flatMap():
1. 使用传入的事件对象创建一个 Observable 对象；
2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；
3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable, 而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。
这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。

案例:请看项目中方法useFlatMapOperation()
备注：lift :针对事件序列的处理和再发送

尽量使用已有的 lift() 包装方法（如 map() flatMap() 等）进行组合来实现需求，

因为直接使用 lift() 非常容易发生一些难以发现的错误
###3.RxJava线程调度：
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，
即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。
如果需要切换线程，就需要用到 Scheduler（调度器:线程控制器：RxJava 通过它来指定每一段代码应该运行在什么样的线程）。

RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景： 1.Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。 2.Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。 3.Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多， 区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。 4.Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算， 即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池， 大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。 另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制

1.subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。 或者叫做事件产生的线程。subscribeOn() 的位置放在哪里都可以，但它是只能调用一次的 2.observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。observeOn() 指定的是它之后的操作所在的线程。 因此如果有多次切换线程的需求，只要在每个想要切换线程的位置调用一次 observeOn() 即可

案例:请看项目中方法threadSchedulers()

备注:在不使用RxJava中的观察者和被观察者的时候可以进行移除：

 //1.使用被观察者的对象(Observable)进行安全移除观察者Subscribe  
Subscription subscription = obs.unsafeSubscribe(Subscribe对象);  
subscription.unsubscribe();
额外扩充的操作符：
1》merge：合并多个数据源：
1.提示：Observerable.merge(Observerable o1,Observerable o2....);
备注：合并的结果是依次输出的，如果只想要最后一次的结果，根据合并的个数找出最后一次的就是最终的结果，
2.具体示例请看项目中mergeData()方法
2》timer：定时操作。表示“指定秒数后执行指定操作”,
1.提示： Observable.timer(2, TimeUnit.SECONDS) 表示指定2秒后执行该动作；
2.具体示例请看项目中timer()方法
3》interval：做周期性操作,表示每隔指定数执行指定的操作
1.提示：Observable.interval(2, TimeUnit.SECONDS)表示每隔2秒执行指定动作；
2.具体示例请看项目中intervalDeliver()方法
4》轮询操作：
1.提示：
//直接进行网络请求，本身就是异步的，已经开启工作线程，不需要自己手动再去写一个线程
Schedulers.newThread().createWorker()
//第一次在initialDelay时间后执行Action0,之后每次间隔period时间执行Action0
.schedulePeriodically(final Action0 action, long initialDelay, long period, TimeUnit unit)
主要应用是隔段时间重新请求网络刷新本地数据
2.具体示例请看项目中scheduleTask()方法 5》使用concat和first依次查询(案例是做三级缓存): 三级缓存：内存(LruCache)，磁盘(自定义存储路径)，网络 1.提示：依次检查memory、disk和network中是否存在数据，任何一步一旦发现数据后面的操作都不执行。

 Observable.concat(memory, disk, network)
         .first()
         //事件发生在子线程
         .subscribeOn(Schedulers.newThread())
         //事件消费发生在主线程
         .observeOn(AndroidSchedulers.mainThread())
         .subscribe(s);
2.具体示例请看项目中concatAndFirstOperation()方法
