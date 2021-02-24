# Java IO：阻塞/非阻塞式IO、同步/异步IO

同步（synchronous） IO和异步（asynchronous） IO，阻塞（blocking） IO和非阻塞（non-blocking）IO分别是什么，到底有什么区别？这个问题其实不同的人给出的答案都可能不同，比如wiki，就认为asynchronous IO和non-blocking IO是一个东西。这其实是因为不同的人的知识背景不同，并且在讨论这个问题的时候上下文(context)也不相同。所以，为了更好的回答这个问题，我先限定一下本文的上下文。本文讨论的背景是Linux环境下的network IO。

## IO操作两个阶段

再说一下IO发生时涉及的对象和步骤。以read函数举例，对于一个networkIO会涉及到两个系统对象，一个是调用这个IO的process (or thread)，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：

**（1）等待数据准备(Waitingfor the data to be ready)**

**（2）将数据从内核拷贝到进程中(Copyingthe data from the kernel to the process)**

**记住这两点很重要，因为这些IO Model的区别就是在两个阶段上各有不同的情况。是否阻塞说的是第一个阶段，即等待数据准备阶段是否会阻塞，而是否同步说的是第二阶段，即将数据从内核拷贝到进程这个真实的IO Operation操作阶段是否阻塞。**

![](img\阻塞与非阻塞，同步与异步理解.png)



同步与异步阻塞

老张爱喝茶，废话不说，煮开水。

出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。

1 老张把水壶放到火上，立等水开。（同步阻塞）

老张觉得自己有点傻

2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）

老张还是觉得自己有点傻，于是变高端了，买了把会响笛的那种水壶。水开之后，能大声发出嘀~~~~的噪音。

3 老张把响水壶放到火上，立等水开。（异步阻塞）

**老张觉得这样傻等意义不大**   (阻塞io一般都是同步的，异步阻塞没有意义！)

4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）

老张觉得自己聪明了。

所谓同步异步，只是对于水壶而言。

普通水壶，同步；响水壶，异步。

虽然都能干活，但响水壶可以在自己完工之后，提示老张水开了。这是普通水壶所不能及的。

同步只能让调用者去轮询自己（情况2中），造成老张效率的低下。

所谓阻塞非阻塞，仅仅对于老张而言。

立等的老张，阻塞；看电视的老张，非阻塞。

情况1和情况3中老张就是阻塞的，媳妇喊他都不知道。**虽然3中响水壶是异步的，可对于立等的老张没有太大的意义**。所以一般异步是配合非阻塞使用的，这样才能发挥异步的效用。



推荐文章：https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect

<script type="text/javascript">
window.addEventListener("load", function() {
  var click_handle = function() {
    if (this.href.substr(-5) == ".html") {
      location.href = this.href;
    } else {
      location.href = "./index.html";
    }
  };
  var as = document.querySelectorAll(".chapter a, .navigation-prev, .navigation-next");
  for (var i = 0; i < as.length; i++) {
    as[i].addEventListener("click", click_handle, true);
    as[i].title = as[i].innerText;
  }
});
</script>

