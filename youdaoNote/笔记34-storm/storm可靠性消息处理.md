

参考 http://www.cnblogs.com/Jack47/p/guaranteeing-message-processing-in-storm.html
- 描述  
  storm可以保证spout发出的每个消息都被处理。
-  如何保证可靠性。
- 一条消息被完整处理。
> 指的是从Spout发出的元组在正给消息树上的节点都被storm处理，如果在指定时间（默认30秒）未被处理，则这个拓扑宣告失败。




#### 消息被完整处理或者处理失败

- Storm通过调用Spout的nextTuple函数来从Spout请求一个元组。Spout任务使用open函数入参中提供的SpoutOutputCollector来给Spout任务的某个输出流发射一个新元组。当发射一个元组时，Spout提供了一个"消息标识"(message-id)，用来后续识别这个元组。
- 接下来，元组就被发送到下游的Bolt进行消费，Storm会负责跟踪这个Spout元组创建的消息树。如果Storm检测到一个元组被完整地处理了，Storm会调用产生这个元组的Spout任务(Spout Bolt有多个任务来运行)的ack函数，参数是Spout之前发送这个消息时提供给Storm的message-id。类似的，当元组处理超时或处理失败时，Storm会在元组对应的Spout任务上调用fail函数，参数是之前Spout发送这个消息时提供给Storm的message-id。这样应用程序通过实现Spout Bolt中的ack接口和fail接口来处理消息处理成功和失败的情况。例如当消息处理成功时记录当前处理的进度，当处理失败时，重新发送消息来对这个消息进行重新处理。但在本文的例子里fail函数中不需要做任何处理，因为这些元组不会从Kestrel队列中去掉，下次从队列取消息，仍然会取到这些消息，只有处理成功后，才会从Kestrel队列中摘除这些消息。
- 


#### Storm的可靠性API

作为Storm用户，如果想利用Storm的可靠性，需要做两件事：


```
1. 创建一个元组时(消息树上创建一个新节点)需要通知Storm
2. 处理完一个元组，需要通知Storm
```

通过这两个操作，当消息树被完全处理完，Storm就可以立即检测到，从而可以正确地确认这个Spout元组处理成功或者失败。Storm的API提供了一套简洁地处理这些操作的方法。
元组创建时通知Storm

在Storm消息树(元组树)中添加一个子结点的操作叫做锚定(anchoring)。在应用程序发送一个新元组时候，Storm会在幕后做锚定。还是之前的流式计算单词个数的例子，请看如下的代码片段：


```
public class SplitSentence extends BaseRichBolt {
    OutputCollector _collector;
    public void prepare(Map conf, TopologyContext context, OutputCollector collector){
        _collector = collector;
    }
    
    public void execute(Tuple tuple) {
        String sentence = tuple.getString(0);
        for(String word: sentence.split(" ")) {
            _collector.emit(tuple, new Values(word));
        }   
        _collector.ack(tuple);
    }   
    
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
    }   
}
```


每个单词元组是通过把输入的元组作为emit函数中的第一个参数来做锚定的。通过锚定，Storm就能够得到元组之间的关联关系(输入元组触发了新的元组)，继而构建出Spout元组触发的整个消息树。所以当下游处理失败时，就可以通知Spout当前消息树根节点的Spout元组处理失败，让Spout重新处理。相反，如果在emit的时候没有指定输入的元组，叫做不锚定：


```
_collector.emit(new Values(word));
```

这样发射单词元组，会导致这个元组不被锚定(unanchored)，这样Storm就不能得到这个元组的消息树，继而不能跟踪消息树是否被完整处理。这样下游处理失败，不能通知到上游的Spout任务。不同的应用的有不同的容错处理方式，有时候需要这样不锚定的场景。

一个输出的元组可以被锚定到多个输入元组上，叫做多锚定(multi-anchoring)。这在做流的合并或者聚合的时候非常有用。一个多锚定的元组处理失败，会导致Spout上重新处理对应的多个输入元组。多锚定是通过指定一个多个输入元组的列表而不是单个元组来完成的。例如：


```
List<Tuple> anchors = new ArrayList<Tuple>();
anchors.add(tuple1);  
anchors.add(tuple2);
_collector.emit(anchors, new Values(word));
```

多锚定会把这个新输出的元组添加到多棵消息树上。注意多锚定可能会打破消息的树形结构，变成有向无环图(DAG)，Storm的实现既支持树形结构，也支持有向无环图(DAG)。在本文中，提到的消息树跟有向无环图是等价的。消息之间的关系是有向无环图的例子见下图：

tuple_dag

消息形成的有向无环图

![image](http://img4.tbcdn.cn/L1/461/1/05c11d575398eb2dee7d54f43c74e9599f3c0945)

Spout元组A触发了B和C两个元组，而这两个元组作为输入，共同作用后触发D元组。
元组处理完后通知Storm

> 锚定的作用就是指定元组树的结构--下一步是当元组树中某个元组已经处理完成时，通知Storm。通知是通过OutputCollector中的ack和fail函数来完成的。例如上面流式计算单词个数例子中的split Bolt的实现SplitSentence类，可以看到句子被切分成单词后，当所有的单词元组都被发射后，会确认(ack)输入的元组处理完成。

可以利用OutputCollector的fail函数来立即通知Storm，当前消息树的根元组处理失败了。例如，应用程序可能捕捉到了数据库客户端的一个异常，就显示地通知Storm输入元组处理失败。通过显示地通知Storm元组处理失败，这个Spout元组就不用等待超时而能更快地被重新处理。

Storm需要占用内存来跟踪每个元组，所以每个被处理的元组都必须被确认。因为如果不对每个元组进行确认，任务最终会耗光可用的内存。

做聚合或者合并操作的Bolt可能会延迟确认一个元组，直到根据一堆元组计算出了一个结果后，才会确认。聚合或者合并操作的Bolt，通常也会对他们的输出元组进行多锚定。