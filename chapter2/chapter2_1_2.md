# 2.1.2创建图表

&#160; &#160; &#160; &#160; 选中任务列表中的数据，点击创建表格按钮，则会出现一个选择列的视图，实质就是选择视图上的性能参数，显示相应的图表，每个参数对应一个图表：
```java
PmTablPane.java
public List<Chart> getCharts() {
    ...
    List<PmPara> columns = selectView.getData();
    
    Vector<?> dataVector = model.getDataVector();
    List<Chart> charts = new ArrayList<Chart>(columns.size());
    
    for(PmPara para : columns) {
        TimeLineChart chart = new TimeLineChart();
        for(int j = 0; j< model.getRowCount(); j++){
            Vector<?> data = (Vector<?>) dataVector.get(j);
            EqNming eqNaming = ((PmTask) data.get(0)).getEqName();
            String seriesKey = ClientContext.getMoFullNameByEqNaming(eqNaming);
            if(seriesKey == null)
                continue;
                
            List<PmData> history = getHistory(eqNaming,para);
            if(history != null)
                for(Pmdata hd : history){
                    chart.appendSample(seriesKey,hd.getValue(),hd.getTime());
                }
            chartMap.put(para,chart);
        
        chart.setTitle(para.getString() + "(" +　para.getUnit + ")");
        charts.add(chart);
    }
    return charts;
}
```
&#160; &#160; &#160; &#160;遍历所选择的性能参数，分别创建一个`TimeLineChart`对象，根据所选任务列表的数据得到设备类型`EqNaming`，并根据`EqNaming`得到图表的`seriesKey`，通过`EqNaming`和性能参数`para`从`historyData`集合中得到`List<Pmdata>`，遍历`List<Pmdata>`将数据插入到图表中,设置图表的标题。