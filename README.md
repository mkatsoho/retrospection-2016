# Retrospection-2016

A retrospection of technical failures expericeced in 2016 (2016年技术回顾）

Written by Michael Chen， 1/1/2016

----

## many engineers or several engineers （是一帮人一块开发，还是几个人开发）

2016中经历了一个feature，需要众多开发人员参与，协同开发，数据流从component 1 to component N，非常之长，不复杂的feature，经历了整整一周的开发时间，终于搞定。感叹。。。

相比一群人开发（many engineers），我更倾向支持几个人开发。显然，后者沟通成本更少，架构更合理，代码质量更高，开发进度更可控。缺点也很明显，没有人员互备，找到senior的人难度增加，”老家伙“做事没有热情，好像也就这三个问题。

曾经约了一共3个开发人员，着手做一个ERP，3个月败了。可是我依然支持 several > many。实在是经历了太多，初级开发人员写了初级的代码，后续要重新re-arch，甚至推翻重写，feature实现不完整，性能／安全／等等质量低下。

上述机构问题，senior人也有类似的情况，相对少很多，这个是不可否认的。注：senior和junior的区别不是工作年限，而是写代码的行数和代码的效率。

更实际的现实是，在many中有譬如1个senior的人，无论在流水线的哪个环节，依然可以协调上下游，依然可以解决一定的沟通问题，也许架构可以合理一些，代码质量吗，就听天由命吧。

简而言之，相信”代码改变世界“的话，就选择”several engineers“吧。如果相信”奋斗才能成功“，”many engineers“也许是好选择中的一个。

----

## 不同的APK中使用公共的SDK模块，真的好难

最近看到有好多的APK，都在使用一个内部自定义的SDK模块。现象就是：“五花八门”。

每个APK，UI都不一致，各有特色，每个都要自己的演进和变化，都在不同的事件中有类似又不一致的需求。问题来了，这个SDK没法让“大家满意”。于是乎，Fork一个新的，自己写自己的，结果“同一个SDK规范”有不同的实现的版本，APK们纵横交错，reuse别人的SDK代码，如是乎，“SDK规范”有了各种实现，也就有了几倍的工作量，有了各种bugs，各种各种的问题。。。

UI和使用行为的不一致，这是合理的。那这个问题怎么解？！

不同人有不同的想法。私以为，“SDK规范”的实现，可以和UI解耦，用非可视化的model来实现所有的下行和上行数据流（下发和上报）。当然在给2个UI的参考实现。不同的APK fork UI的实现，把各种自己期望的UI事件和model的行为链接在一起。这样既保证了“SDK规范”的一致性，又保证了“APK的UI特色”。值得考量。

当然，在说一下困难。其实困难不是技术，是人。不同部门见的独立，互相的敬仰或不敬仰，任务别人挫或自己强。。。当自己搞不定的时候，显然原因是有更优先的工作，人员不够，需求不应该。。。最终还是人的“问题”。

----

## 说说后台服务的QPS （每秒可以处理的请求数）

8核32G内存的AWS服务器，能处理多少QPS？

答：不确定。不知道cpu型号。不知道服务是不是cpu密集型的。不知道服务是不是io密集型的。不知道io对应的后端资源的latency。不知道请求的协议。不知道请求的payload大小。。。

如果，服务既不是CPU密集型的，也不是IO密集型的，这样一台应该CPU还不差的服务器的QPS还不到600？甚至不到300？不可思议吧？8086的处理速度？不，应该是个i5笔记本的速度。

原因在哪呢？技术和人，不外乎这两件事。

技术上选择了同步IO，用多进程来解决并发的问题。10 QPS每进程 * 32 进程 = 320 QPS 处理能力也就这样喽。异步IO会增加代码的复杂度吗？这个不好回答。c++，python，java，nodejs都不一样。或许每个语言都有成本不高的异步IO方案，即，学习成本不高，架构不复杂，代码也不很复杂，debug也容易的方案。我相信它们都有。

人的事情永远比技术的复杂度大，此处省略500字。

----

## 写测试代码太耗时间

每每听到有人说“写测试代码太耗时间”，“太赶，没时间”。曾经认为最正确的回应是”有足够的测试代码才能敏捷开发“。现在已经不在这么想了（老了）。

还是只谈技术，不谈”政治“（技术）。

技术上讲，“测试代码太复杂”核心的原因是：设计架构和API的时候，仅仅想着功能，就没有想怎么测试。更直白的讲，设计能力不够。写功能的时候，要考虑好写个事儿呢！例如：

- 并发性能。怎么知道服务端的QPS呢？设计的时候已经考虑了怎么测试性能了。否者，out了
- 响应时间。有的API不会有很大的访问压力，比如说服务端ready这样的API，需要很快的响应时间。只考虑功能，对响应时间不感冒，就完成了那点功能，就有点low了。
- security。不要到处留下password／token的脚印，不要，千万不要。需要保存话，也是一个不可逆的数值。
- 其它，没记错的应该一共有7个

把上面都考虑了，你会发现测试就是一个顺理成章的事。例如，5行搞定：

```
  # Exmaple: load a new conf to server, wait it restart, and ensure the new conf works
  startTimestamp = Time.now()
  assert url(/loadNewConf).sendRequest() == 200                 # load new conf
  setTimer(timeout=10s, interval=10ms, funciton(){
      break if url('/isRunningAfterLoad').sendRequest().body().status == 'running'  # wait until running
  )
  assert url('/isRunningAfterLoad').sendRequest().body().status == 'running'        # running!
  assert url('/getConf').sendRequest().body().conf = 'new'      # new conf has been effective
  assert Time.now() - startTimestamp < 500ms                    # should be less than 0.5 sec
```
