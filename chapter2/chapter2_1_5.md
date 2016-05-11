# 2.1.5 删除任务

&#160; &#160; &#160; &#160;暂停任务在`PmService`中的实现方法如下：
```java
public void deleteRtPmTask(long taskId){
    PmTask pmTask = rtTaskMp.get(taskId);
    if(pmTask == null)
        return;
    
    sbiPmMgr.deleteRtTask(taskId);
    ServerContext.sendMessage(new EmsMessage(pmTask,MsgCode.DeleteRtTask));
    rtTaskMap.remove(taskId);
}
```
&#160; &#160; &#160; &#160;`sbiPmMgr.deleteRtTask(taskId)`方法的功能：根据`taskId`从`taskMap`集合中取出`BasicTask`并执行其`cancel(true)`方法取消任务执行；根据`taskId`从`PolledDataMap`集合中移除相应的`PolledData`；根据`taskId`从`taskMap`集合中移除相应的`BasicTask`。

&#160; &#160; &#160; &#160;发送一条`EmsMessage`消息，`PmTablePane`中的`onMessage`方法根据`EmsMessage`消息将该`pmTask`从任务列表中移除。`rtTaskMap`集合移除`taskId`对应的`Pmtask`。