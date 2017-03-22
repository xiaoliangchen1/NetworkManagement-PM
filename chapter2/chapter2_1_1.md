### 2.1.1 创建任务

在进行性能查询时，有两个查询分类，一种是网元性能查询，一种是端口性能查询。网元性能查询的采集对象是网元，采集参数有CPU利用率与存储利用率两种。端口性能查询的采集对象是具体到指定的端口，其采集参数有端口接收字数、端口发送字数、端口接收包数、端口发送包数、端口接收速率和端口发送速率。

   将所选择的采集对象和采集参数封装成\`PmTask\`，每一个采集对象封装成一个\`PmTask\`，并设置采集间隔时间。

```java
PmRtTabView.java
...
for(EqNaming eqNaming : pmView.getSelectedMoList()) {
    PmTask pmTask = new PmTask();
    pmTask.setEqName(eqNaming);
    pmTask.setParaList(pmView.getSelectedPmParas());
    pmTask.setCollectInterval(ClientContext.getIntProperty(
           ProfileKey.PmRtCollectInterval));
    pmTable.updateColumn(pmTask);

    ClientContext.getPmFacade().createRtPmTask(pmTask);
}
...

PmTask.java
@Entity
public class Pmtask extends Bo {
    ...
    @Id
    private long taskId;
    private EqNaming eqName;
    private final Set<PmPara> paraList = new HashSet<PmPara>();
    private CollectType collectType = CollectType.RealTme; 
    private String taskName;
    private long createTime;
    private long endTime;
    private Granularity granularity;
    private int collectInterval = 30;
    private int saveInterval = 3600;
    private long time;
    ...
}
```

       `pmTable`将根据封装好的`PmTask`形成列表的结构。将`PmTask`作为参数执行`createRtPmTask(pmTask)`方法，通过RMI，最终调用的是`PmService`中的`createRtPmTask(PmTask pmTask)`方法。

```java
PmService.java
public PmTsk createRtPmTask(PmTask pmTask) {
    pmTask.setSessionId(sessionService.getSession().getSessionId());

    if(rtTaskMap.containsValue(pmTsk)){
        PmTask oldPmTask = getPmTask(pmTask);
        oldPmTask.merge(pmTask);
        sbiPmMgr.resetPmTask(PmUtils.toPolledData(oldPmTask));
        rtTaskMap.put(oldPmTask.getTaskId(),oldPmTask);
        return oldPmTask;
    }

    pmTask.setTaskId(ArrayUtils.generateId());
    pmTask.setCollectType(CollectType.RealTme);
    pmTask.setCreateTime(System.currentTimeMillis());
    pmTask.setStartTime(System.currentTimeMillis());
    if(pmTask.getEndTime() == 0)
        pmTask.setEndTime(System.currentTimeMillis() + ArrayUtils.OneHourMillis);

    sbiPmMgr.createPmTask(PmUtils.toPolledData(oldPmTask));
    ServerContext.senMwssage(new EmsMessage(pmTask,MsgCode.CreateRtTask));
    rtTaskMap.put(pmTask.getTaskId(),pmTask);
}
```

       设置`pmTask`的任务Id、采集类型、任务创建时间、采集开始时间、采集结束时间。将设置后的`pmTask`转换成`PolledData`类型，并将之作为参数通过`SbiPmMgr`的`createPmatask`方法创建任务。此时会发送一条JMS消息，该消息将`pmTask`作为消息源。将`pmTask`放入`rtTaskMap`集合中。

       若是`rtTaskMap`的value中包含`pmTask`,则将集合中的value值`oldPmTask`取出，把`pmTask`中的采集参数全部加入到`oldPmTsk`中，并通过`SbiPmMgr`重置任务。

```java
SbiPmMgr.java
public long createPmTask(final PolledData data){
    final long taskId = data.getTaskId();
    VasicTask task = new BasicTask( new Date(data.getStartTime()),
                    new Date(data.getEndTime()),data.getInterval()) {

        SnmpCollector collector = new SnmpCollector();

        @Override
        protected void execute(){
            PolledData polledData = PolledDataMap.get(taskId);
            Object result = collector.collect(polledData);
            if(result == null)
                return;

            polledData.setTaskId(taskId);

            if(data.getCollectType() == CollectType.RealTime)
                rtProcessor.onPolledData(polledData);

            if(data.getCollectType() == CollectType.History)
                htProcessor.onPolledData(polledData);
        }
    };

    TaskManager.addSchedulerTask(task);
    PolledDataMap.put(taskId,data);
    taskMap.put(TaskId,task);

    return taskId; 
}
```

       根据`data`中的开始时间、结束时间、采集间隔创建一个`BasicTask`任务，该任务实现了`runnable`接口，是一个线程。`TaskManager`将该任务放入到线程池中执行。分别将`PolledData`和`BasicTask`放入`PolledDataMap`和`taskMap`集合中。`execute()`是采集的过程，`collector.collect(polledData)`就是从设备中将`polledData`中采集参数对应的值取出。`onPolledData(polledData)`是将`collect`后的数据封装到`PmTask`中，将`PmTak`放到`EmsMessage`中，并发送，在`PmRtTableView`中监听此条消息：

```java
PmRtTableView.java
public void onMessage(EmsMessage message){
    ...
    Pmtask pmTask = (PmTask) message.getSource();
    ...
    else if(message.getMsgCode() == MsgCode.UpdateRtTask){
        if(!pmTask.isFinished()){
            pmTable.updateRow(pmTask);
            monitorPane.update(pmTask);
        }
        eqMap.put(pmTask.getEqName(),pmTask);
    }
}
```

        `pmTable.updateRow(pmTask)`方法主要由三个作用：更新任务列表中的数据；将`pmTask`中的数据存入到`Map<EqNaming,Map<PmPara,LinkedList<PmData>>> historyData`集合中（会在导出表中数据时用到）；为图表显示`chart.appendSample()`提供数据。`monitorPane.update(pmTask)`是用于更新`MonitorPane`中的数据显示。

