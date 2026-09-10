![[{7D1630D6-8C75-4B38-8011-2FE4D5D62B7D}.png]]


模型是怎么知道有tools的存在的?

:langchain在我们发起调用的时候 将Message和tools的一些信息打包起来发给模型
![[{59C5A5C0-90C3-4567-9EE1-872256ED4DD0}.png]]


langchain自动分析模型返回的结果 并根据结果调用相关的工具
![[{2541F7D4-5D82-4A60-826A-0E6BCC65CED2}.png]]