# Java编程思想

1.13 Java与Internet

​	如果Java仅仅只是众多的程序设计语言中的一种，你可能就会问：为什么它如此重要？为什 么它促使计算机编程语言向前迈进了革命性的一步？如果从传统的程序设计观点看，问题的答案似乎不太明显。尽管Java对于解决传统的单机程序设计问题非常有用，但同样重要的是，它解决了在万维网（WWW)上的程序设计问题。

1.13.1 Web是什么

​	Web一词乍一看有点神秘，就像”网上冲浪“ 、”表现“ 、”主页” 一样。回头审视它的真实面貌有助于对它的理解，但是要这么做就必须先理解客户/服务器系统，它是计算技术中另一个充满了诸多疑惑的话题。

​	1.客户/服务器计算技术

​	客户/服务器系统的核心思想是：系统具有一个中央信息存储池（central repository of information），用来存储魔种数据，它通常存在与数据库中，你可以根据需要将它分发给某些人员或机器集群。客户/服务器概念的关键在于信息存储池的位置集中于中央，这使得它可以被修改，并且这些修改将被传播给信息消费这，总之，信息存储池、用于分发信息的软件以及信息与软件所驻留的机器或机群被总称为服务器。驻留在用户机器上的软件与服务器进行通信，以获取信息、处理信息，然后将它们显示在被称为客户机的用户机器上。

​	客户/服务器计算技术的基本概念并不复杂。问题在于你只有单一的服务器，却要同时为多	个客户服务。通常，这会涉及数据库管理系统，因此设计者把数据“均衡” 分布于数据表中，以取得最优的使用效果。此外，系统通常允许客户在服务器中插入新的信息。这意味着必须保证一个客户插入的新数据不会覆盖另一个客户插入的新数据，也不会在将其添加到数据库的过程中丢失（这被称为事务处理）。如果客户端软件发生变化，那么它必须被重新编译、调试并安装到客户端机器上，事实证明这比想象的要更加复杂与费力。如果想支持多种不同类型的计算机和操作系统，问题将更麻烦。最后还有一个最重要的性能问题：可能在任意时刻都有成百上千的客户向服务器发出请求，所以任何小的的延迟都会产生重大影响。为了将延迟最小化，程序员努力减轻处理任务的负载，通常是分散给客户端机器处理，但有时也会使用所谓的中间件将负载分散给在服务器端的其他机器。（中间件也被用来提高可维护性。）

​	分发信息这个简单思想的复杂性实际上是由很多不同层次的，这使得整个问题可能看起来高深莫测。但是它仍然至关重要：算起来客户/服务器计算技术大概占了所有程序设计行为的一半，从制定订单、信用卡交易到包括股票市场、科学计算、政府、个人在内的任意类型的数据分发。过去我们所做的，都是正对某个问题发明一个单独的解决方案，所以每一次都要发明一个新的方案。这些方案难以开发且难以使用，而且用户对每一个方案都要学习新的接口。因此，整个客户/服务器问题需要彻底解决。

​	2.Web就是一台巨型服务器

​	Web实际上就是一个巨型客户/服务器系统，但稍微差一点，因为所有的服务器和客户机都同时共存与同一个网络中。你不需要了解这些，因为你所要关心的只是在某一时刻怎样连接到一台服务器上，并与之进行交互（即便你可能要满世界地查找你想要的服务器）。

​	最初只有一种很简单的单向过程：你对某个服务器产生一个请求，然后它返回给你一个文件，你的机器（也就是客户机）上的浏览器软件根据本地机器的格式来解读这个文件。但是很快人们就希望做得更多，而不仅仅是从服务器传递回页面。人们希望实现完整的客户/服务器能力，使得客户可以将信息反馈给服务器。例如，在服务器上进行数据库查找，将新信息添加到服务器以及下订单（这需要特殊的安全措施）。这些变革，正式我们在Web发展过程中所目睹的。

​	Web浏览器向前跨进了一大步，它包含了这样的概念：一段信息不经修改就可以在任意型号的计算机上显示。然而，最初的浏览器仍然相当原始，很快就因为加诸于其上的种种需要而陷入困境。浏览器并不具备显著的交互性，而且它趋向于使服务器和Internet阻塞，因为在任何时候，只要你需要完成通过编程来实现的任务，就必须将信息发回到服务器去处理。这使得即便时发现你的请求中的拼写错误也要花去数秒甚至是数分钟的时间。因为浏览器只是一个观察器，因此它甚至不能执行最简单的计算任务。（另一方面,它确实安全的，因为它在你的本地机器上不会执行任何程序，而这些人程序有可能包含bug和病毒。）

​	为了解决这个问题，人们采用了各种不同的方法。首先，图形标准的到了增强，使得在浏览器中可以 播放质量更好的动画和视频。剩下的问题通过引入在客户端浏览器中运行程序的能力就可以解决。这就被称为 ”客户端编程”。

1.13.2 客户端编程

​	Web最初的 "服务器-浏览器"设计是为了能够提供交互性的内容，但是其交互性完全由服务器提供。服务器产生静态页面，提供给只能解释并显示它们的客户端浏览器。基本的HTML（HyperText Markup Language，超文本标记语言）包含有简单的数据收集机制：文本输入框、复选框、单选框、列表和下拉式列表以及按钮——它只能被编程来实现复位表单上的数据或提交表单上的数据给服务器。这种提交动作通过所有的Web服务器都提供的通用网关接口（common gateway interface ，CGI） 传递。提交内容会告诉CGI应该如何处理它。最常见的动作就是运行一个在服务器中常被命名为“cgi-bin"的目录下的一个程序。（当点击了网页上的按钮时，如果观察浏览器窗口顶部的地址，有时可以看见 ”cgi-bin“ 的字样混迹在一串冗长和不知所云的字符中 。）几乎所有的语言都可以用来编写这些程序。Perl已经成为最常见的选择，因为它被设计用来处理文本，并且是解释型语言，因此无论服务器的处理器和擦欧哦系统如何，它都适于安装，然而，Python 已对其产生了重大的冲击，因为它更强大且更简单。

