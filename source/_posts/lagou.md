---
title: 从拉勾招聘看互联网行业
date: 2016-03-05 20:11:08
tags: 
  - Python
  - Crawler
---

<script src="//cdn.bootcss.com/echarts/3.0.0/echarts.min.js"></script>

　　最近几年互联网行业以迅雷不及掩耳盗铃儿响叮当仁不让之势席卷了每个人生活的方方面面，大把的年轻人开始投身互联网。那么，互联网公司的发展情况如何，对从业者的要求如何，以及不同职能的薪水如何，哪些城市的互联网行业发展更为迅猛，身为互联网行业的一份子，你知道吗？
　　不识庐山真面目，只缘身在此山中。那么，让我们站在更高的角度，用数据的方式来对互联网行业的职场进行分析，来一起看看这个行业吧。

<img src="http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/03/05/3349814270.png" width="700px"/>

<!--more--> 


----------

## 人数、融资、地域、标签概况

　　不算不知道，一算吓一跳。互联网行业中，超过半数（约59%以上）的公司，人数都在50人以下。事实上，互联网是创业门槛和先期投入相对较低的行业，所以很多创业公司扎堆在这个领域。
<div id="companysize" style="width: 750px;height:550px;"></div>
<script type="text/javascript"> 
    var myChart = echarts.init(document.getElementById('companysize'));
    var option = {title:{text:"互联网公司人数分布",subtext:"",x:"center"},tooltip:{trigger:"item",formatter:"{b}:{c}%"},legend:{x:"center",y:"bottom",data:["少于15人","15-50人","50-150人","150-500人","500-2000人","2000人以上"]},calculable:true,series:[{name:"",type:"pie",radius:["20%",205],center:["50%","50%"],roseType:"radius",data:[{value:18.63,name:"少于15人"},{value:40.76,name:"15-50人"},{value:24.33,name:"50-150人"},{value:13.36,name:"150-500人"},{value:2.83,name:"500-2000人"},{value:0.09,name:"2000人以上"}],startAngle:131}]};
    myChart.setOption(option);
</script>


----------

　　除了公司人数集体偏低，互联网公司的另一大特点便是，很大一部分公司是没有融到资的。大约只有1.59%的公司出于D轮融资及以上。

<div id="funding" style="width: 750px;height:550px;"></div>
<script type="text/javascript"> 
    var myChart = echarts.init(document.getElementById('funding'));
    var option = {title:{text:"互联网公司融资情况",subtext:"",x:"center"},tooltip:{trigger:"item",formatter:"{b}:{c}%"},legend:{x:"center",y:"bottom",data:["未融资","天使轮","A轮","B轮","C轮","D轮及以上","上市公司","不需要融资"]},calculable:true,series:[{name:"",type:"pie",radius:[60,205],center:["50%","50%"],data:[{value:39.73,name:"未融资"},{value:17.51,name:"天使轮"},{value:9.45,name:"A轮"},{value:3,name:"B轮"},{value:1.19,name:"C轮"},{value:1.59,name:"D轮及以上"},{value:5.84,name:"上市公司"},{value:21.69,name:"不需要融资"}],roseType:"radius"}]};
    myChart.setOption(option);
</script>


----------

　　从地域分布来看，互联网公司大部分还是集中在北上广深杭，中西部内陆城市中，成都的互联网公司最多。

![2016-03-05 15:25:17屏幕截图.png][1]


----------

　　通过对约10万家互联网公司标签的统计，下图这些标签是出镜率最高——

![tagsss.png][2]

　　比较有意思的是，<u>**周末双休**</u>居然会小到你几乎会忽略掉它。而<u>**朝九晚五**</u>这个标签则因为更小而不幸地没有出现在这张图中，反倒是<u>**弹性工作**</u>占据了较大的一块。
　　(P.S.悄悄问一句，弹性工作不是说，你可以晚点来，但是不许早走的意思吗？)

----------

## 城市与薪水

　　在互联网公司密度最高TOP6城市中（北、上、深、广、杭、成），已融资及以上（抛开未融资部分不统计）公司的分布如下图所示。上海的非天使轮互联网公司比例最大，上市公司占比略低于成都（考虑到上海互联网公司整体数量较成都多，实际上上市公司数量是多于成都的）。这表明仅仅从这个角度看，上海的互联网公司发展地还不错，如果你打算创业，我不负责地建议你可以考虑以上海作为创业地。

<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016%2F03%2F04%2Fsc%2Ffunding_location.svg" width="750" height="550"></iframe>


----------

　　在上述六个城市中，分别的薪水分布情况如下，整体来看，帝都的高薪比例以很不明显的优势略胜一筹。

<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016%2F03%2F04%2Fsc%2Fsalary_city.svg" width="700" height="550"></iframe>
 

----------

## 职位与薪水

　　看完了互联网公司的整体情况，我们来了解下互联网公司职位的相关数据吧。下图统计了拉勾网上面对各种不同工种对职位的需求数量：

<div id="recuri" style="width: 750px;height:550px;"></div>
<script type="text/javascript"> 
    var myChart = echarts.init(document.getElementById('recuri'));
    var option = {title:{text:"互联网公司职位需求",subtext:"",x:"center"},tooltip:{trigger:"item",formatter:"{b}:{d}%"},legend:{x:"center",y:"bottom",data:["技术","设计","产品","运营","市场与销售","金融","职能"]},calculable:true,series:[{name:"",type:"pie",radius:[30,"60%"],center:["50%","50%"],roseType:"area",data:[{value:63311,name:"技术"},{value:11343,name:"设计"},{value:9434,name:"产品"},{value:25280,name:"运营"},{value:28598,name:"市场与销售"},{value:2769,name:"金融"},{value:9248,name:"职能"}]}]};
    myChart.setOption(option);
</script>

　　作为互联网公司的地基，整个互联网行业是建立在计算机技术开发的基础之上。技术岗的招聘需求明显远高于其他所有岗位的需求人数，从招聘需求来看，那些“只差一个程序猿”的公司还是不少的嘛~~~。那么，除了产品自身优秀外，市场和运营的作用就非常关键，可以说决定着产品的前途和命运。所以，技术、市场、运营成为了招聘岗位最多的互联网工种。

----------

　　然后，通过对拉勾网所有招聘的统计分析，得到了如下的职位与工资分布图。由图，大部分的技术岗位工资集中在6-20k之间。仅仅从图中来看，技术岗的高端职位（30k+）比例很低，但是考虑到技术岗高端职位大多会通过猎头或者内推，较少会在招聘网站直接招聘，所以，技术岗的整体走势在图中的反应是有上升空间的。

<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016/03/04/sc/salary_cate.svg" width="700" height="550"></iframe>


----------

## 学历与薪水

　　在知乎上经常会看到诸如“高中毕业参加培训机构”、“xxx转行搞技术”之类的问题，可以看出，在大家心目中，互联网是一个非常亲民的行业。然而，互联网的起点学历大部分还是本科。这并不以为着低学历的人无法从业，而是说，你需要付出更多的努力。

<div id="degree" style="width: 750px;height:550px;"></div>
<script type="text/javascript"> 
    var myChart = echarts.init(document.getElementById('degree'));
    var option = {title:{text:"职位学历要求",subtext:"",x:"center"},tooltip:{trigger:"item",formatter:"{b}:{c}%"},legend:{x:"center",y:"bottom",data:["学历不限","高中/中专","大专","本科","硕士","博士"]},calculable:true,series:[{name:"",type:"pie",radius:[30,"60%"],center:["50%","50%"],roseType:"area",data:[{value:14.55,name:"学历不限"},{value:0.01,name:"高中/中专"},{value:38.49,name:"大专"},{value:46.15,name:"本科"},{value:0.82,name:"硕士"},{value:0.03,name:"博士"}],startAngle:95,itemStyle:{normal:{borderWidth:1,borderColor:"rgb(255,204,0)"}}}]};
    myChart.setOption(option);
</script>

----------

　　不幸的是，虽然互联网行业比较包容学历因素，但是，学历依然是工资的主要影响因素之一。绝大部分的高中中专学历者薪资集中在5k以下，而博士学历者的招聘需求虽然少，但是更集中在高薪职位上。

<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016/03/04/sc/salary_degree.svg" width="700" height="550"></iframe>

----------

## 工作经验、企业发展阶段与薪水

　　另一个影响薪水的因素便是从业时间与工作经验了。虽然时常看到“谁谁谁毕业就去了哪里哪里，薪资多少多少”的新闻，让所有人都以为这是一个傻子都可以赚钱的行业，无数“抽烟、喝酒、纹身，但是知道自己是好女孩”的“好女孩”们，常常说“等自己玩够了，就找个‘老实人’、‘程序猿’嫁了”。但是事实是，大家看到的都只是这个行业的最顶端（幸存者偏差），大部分的毕业生（约96.34%），第一份工作的工资都在10k以下的，而且，约68.34%的毕业生第一份工作工资是在5k以下的。随着工作经验的增加，尤其是拥有10年以上工作经验的从业者们，接近半数都拿到了30k+的高薪。不过，这并不意味着经验多工资就高，还是有小部分（约5%）的十年以上工作经验的从业者依旧拿着10k以下薪水。


<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016/03/04/sc/salary_workingyears.svg" width="700" height="550"></iframe>


----------

　　另外再看一下企业的发展阶段对对工资的影响吧。由图可知，不同发展阶段的公司的薪水分布情况大体上呈相似走势。所以，如果仅仅从工资角度考虑，去小公司还是大公司真的影响不大，不过，考虑到创业公司的风险程度，大公司会更加安稳。所以，大部分创业公司会为员工提供股票期权等工资之外的福利，来减轻员工职业生涯风险以及大公司更完善的福利等因素造成的岗位竞争力不足的劣势。


<iframe src="http://7xkpi6.com1.z0.glb.clouddn.com/2016/03/04/sc/salary_entity.svg" width="700" height="550"></iframe>


----------

## 小结

　　通过互联网行业招聘数据，我们小窥了这个行业的整体薪资状况。


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/03/05/1137197199.png
  [2]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/03/05/3349814270.png
  [3]: https://socialcredits.cn
  [4]: http://lagou.com
