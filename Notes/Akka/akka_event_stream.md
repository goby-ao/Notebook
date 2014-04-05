Akka 学习之eventStream 
=====================


akka 是异步、非阻塞、高性能的事件驱动编程模型。akka的事件流是每一个actor系统的主要事件总线,主要用来订阅监听（subscribe）actor（每个actor可以是一个事件）。
下面以一个简答的例子来解释一下 akka 的事件流。

在akka中事件流只需要一行代码
```scala
    context.system.eventStream.subscribe(context.self, classOf[your actor class])
```
比如在一个系统中，你需要更新用户的账户信息，这个账户可能会频繁的被访问，所以你决定使用cache来提高系统性能。当账户信息被更新后，你也希望能检查账户余额是否超过临界值，如果是的话，就自动给用户发邮件提醒。但是你既不希望废弃cache也不希望余额检查直接影响到账户更新操作，这可能会降低响应速度。所以你可以这样来实现
```scala
    // 更新账户的actor
    class AccountManager extends Actor
         val dao = new AccountManagerDao
    
         def receive = {
            case UpdateAccountBalance(userId, amount) =>
                val res = for(result <- dao.updateBalance(userId, amount)) yield{
                    context.system.eventStream.publish(BalanceUpdated(userId))
                    result                
          }
          sender ! res
      }
    }
    
    // accoutn cache管理
    class AccountCacher extends Actor{
      val cache = new AccountCache
    
      override def preStart = {
        context.system.eventStream.subscribe(context.self, classOf[BalanceUpdated])
      }
    
      def receive = {
        case BalanceUpdated(userId) =>
          cache.remove(userId)
      }
    }
    
    // 检查账户余额
    class LowBalanceChecker extends Actor{
      val dao = new LowBalanceDao
    
      override def preStart = {
        context.system.eventStream.subscribe(context.self, classOf[BalanceUpdated])
      }
    
          def receive = {
            case BalanceUpdated(userId) =>
              for{
                balance <- dao.getBalance(userId)
                theshold <- dao.getBalanceThreshold(userId)
                if (balance < threshold)
              }{
                sendBalanceEmail(userId, balance)
              }
          }
        }
```
在这个例子当中， `AccoutCacher`和 `LowBalanceChecker`actor 都订阅了`BalanceUpdated`的事件流。如果发生了 **update** 的事件流，订阅者（监听者）都会收到消息。在`AccountManager`中，当更新操作完成时，它就会给用户发送一个`BalanceUpdated`的事件流。当事件流产生时，消息会并发的发送到`AccoutCacher`和 `LowBalanceChecker`的 mailbox 中，导致 balance 从 cache 中清除，并且账户余额能够得到检查。

subscribe event stream 是一个被动模式，监听事件流中自己订阅的事件。

