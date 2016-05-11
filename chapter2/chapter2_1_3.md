# 2.1.3 暂停任务

&#160; &#160; &#160; &#160;暂停任务在PmService中的实现方法如下：
```java
public void pauseRtPmTask(long taskId){
    PmTask pmTask = rtTaskMp.get(taskId);
    if(pmTask == null)
        return;
    
    sbiPmMgr.pauseRtTask(taskId);
    pmTask.setTaskState(TaskTate.Paused);
    ServerContext.sendMessage(new EmsMessage(pmTask,MsgCode.UpdateRtTask));
}
```

&#160; &#160; &#160; &#160;根据`taskId`从`rtTaskMp`中取出`PmTask`。`sbiPmMgr.
pauseRtTask(taskId)`就是将`BasicTask`中的`paused`置为`true`，这样在`run()`中便不能执行`execute()`了。
```java
BasicTask.java
@Override
public void run(){
    if(endTask()){
        cancel(true);
        return;
    }
    
    if(paused)
        return;
    
    try{
        execute();
    }catch(Exception ex){
        LogMan.log(ex);
    }
}
```

&#160; &#160; &#160; &#160;然后将`pmTask`的任务状态设置为`TaskTate.Paused`，并发送一条`EmsMessage`消息。