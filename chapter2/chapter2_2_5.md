# 2.2.5 条件查询

&#160; &#160; &#160; &#160;选中历史性能任务列表中的数据，点击条件查询按钮，进行条件查询操作。根据列表中所选数据，设置`PmDataQueryCondition`对象的相关属性：
```java
PmDataQueryCondition.java
public class PmDataQueryCondition implements Serializable{
    ...
    private long startTime;
    private long endTime;
    private Granularity granularity;
    private List<PmPara> pmParas;
    private List<EqNaming> targets;
    ...
}

PmHtTabView.java
btnQuery.addActionListener(new ActionListener(){
    @Override
    public void actionPerformed(ActionEvent e){
        PmDataQueryCondition condition = new PmDataQueryCondition();
        PmTask task = pmTaskTable.getSelectedRowData();
        
        List<EqNaming> eqNames = new ArrayList<EqNaming>();
        eqNames.add(task.getEqName());
        
        condition.setPmParas(new ArrayList<PmPara>(task.getParaList()));
        condition.setTargets(eqNames);
        
        WindowManager.openDialog(new PmDataQueryView(PmHtTabView.this),condition);
    }
});
```
&#160; &#160; &#160; &#160;进入到`PmDataQueryView`页面，根据`condition`中的性能参数构建`paraTree`。在`PmDataQueryView`中通过在`paraTree`树中所选择的性能参数以及采集粒度、起始时间、终止时间重新构建一个`PmDataQueryCondition`对象，将该对象作为参数执行`getHtPmDatas(condition)`方法,该方法最终从数据库的`PmData`表中查询数据，并将结果数据添加到列表中：
```java
PmDataQueryView.java
private void doQuery(){
    SwingCaller.execute(new CallBack() {
        @Override
        public void process() throws Exception {
            List<CTreeNode> nodes = paraTree.getSelectedNodes();
            List<PmPara> paras = new ArrayList<PmPara>();
            for(CTreeNode node : nodes){
                paras.add((PmPara) node.getUserObject());
            }
            
            PmDataQueryCondition condition = new PmDataQueryCondition();
            condition.setPmParas(paras);
            List<EqNaming> targets = new ArrayList<EqNaming>();
            targets.add(target);
            
            condition.setGranularity(cmbGran.getSelectedItem());
            condition.setStartTime(dfStart.getTime());
            condition.setEndTime(dfEnd.getTime());
            condition.setTargets(targets);
            
            List<PmData> pmDatas = ClientContext.getPmFacade().getHtPmDatas(condition);
            table.clearAllRowDatas();
            table.addRowDatas(pmDatas);
        }
    }, ClientContext.getMainFrame());
}
```
&#160; &#160; &#160; &#160;创建图表。将列表中所有数据放入到`Map<PmPara,List<PmData>>`的集合中,每一个`PmPara`创建一个图表，图表根据`PmPara`对应的`List<PmData>`集合中的数据进行显示：
```java
PmHtTabView.java
@Override
public void initData(Object arg, OperType type){
    ...
    List<Pmdata> pmDatas = (List<Pmdata>)arg;
    Map<PmPara,List<PmData>> dataMap = new LinkedGashMap<PmPara,List<PmData>>();
    
    for(Pndata data : pmDatas) {
        List<PmData> datas = dataMap.get(data.getPara());
        if(datas == null){
            datas = new ArrayList<PmData>();
            dataMap.put(data.getPara(),datas);
        }
        datas.add(data);
    }
    
    tabPane.removeAll();
    for(List<PmData> datas : dataMap.values())
        addPmDatas(datas);
        
    return;
    ...
}

private void addPmdATAS(List<Pmdata> pmDatas) {
    PmPara pmPara = pmDatas.get(0).getPara();
    TimeLineChart chart = new TmeLineChart();
    String key = ClientContext.getMoFullNameByEqNaming(pmDatas.get(0)
                .getTarget()) + " " + pmPara.getString();
    
    chartt.setTitle(pmPara.getString() + "(" + pmPara.getUnit() + ")");
    for(PmData data : pmDatas)
        chart.appendSample(key,data.getValue(),data.getTime());
    
    tabPane.addTab(pmPara.getString(),chart);
}

```