# 2.2.2 暂停任务

&#160; &#160; &#160; &#160;暂停任务在`PmService`中的实现方法如下：
```java
public void pauseHtPmTask(long taskId){
    PmTask pmTask = rtTaskMp.get(taskId);
    if(pmTask == null)
        return;
    
    sbiPmMgr.pauseRtTask(taskId);
    pmTask.setTaskState(TaskTate.Paused);
    pmDao.updateEntity(pmTask);
    ServerContext.sendMessage(new EmsMessage(pmTask,MsgCode.UpdateRtTask));
}
```

&#160; &#160; &#160; &#160;根据`taskId`从`rtTaskMp`中取出`PmTask`。`sbiPmMgr.
pauseRtTask(taskId`)就是将`BasicTask`中的`paused`置为`true`，这样在`run()`中便不能执行`execute()`了。将pmTask的任务状态设置为`TaskTate.Paused`，并更新数据库中的数据，发送一条`MsgCode`为`MsgCode.UpdateRtTask`的`EmsMessage`消息，`PmHtTabView`中的`onMessage()`方法根据接收到的消息将更新列表的数据。