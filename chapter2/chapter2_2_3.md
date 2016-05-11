# 2.2.3 恢复任务

&#160; &#160; &#160; &#160;恢复任务的处理流程与暂停任务大致相同，只不过是`sbiPmMgr.resumeRtTask(taskId)`是将`BasicTask`中的`paused`置为`false`，这样在`run()`方法就能执行`execute()`了。