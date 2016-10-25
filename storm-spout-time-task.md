#spout tims task
Once I should read infos from DB every 6 hours in Spout, but timeTask is not supported in spout as bolt,  
so I just use the simple method **Thead.sleep**.
``` java
    @Override
    public void nextTuple() {
        AgentInfoDao agentInfo = new AgentInfoDao();
        List<AgentInfo> agentInfoList = agentInfo.queryAgentInfo();
        agentInfo.closeDBCon();
        
        _collector.emit(new Values(agentInfoList));
        try {
          /**
          * get infos from DB by every 6 hours
          */
          Thread.sleep(frequency);
        } catch (InterruptedException e) {
          log.error("Thread sleep error occures: " + e.getMessage());
        }
    }
```
