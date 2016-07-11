找到你，微软认知服务的商业性应用
===============================================
<h2><a class="anchor" id="user-content-table-of-contents" aria-hidden="true" href="#table-of-contents"><span class="octicon octicon-link" aria-hidden="true"></span></a>目录</h2>

<ul>
<li><a href="#1">前言</a></li>
<li><a href="#2">总体架构设计</a></li>
<li><a href="#3">基础工作和概念</a></li>
<li><a href="#3.1">—— 前期准备</a></li>
<li><a href="#3.2">—— ClientHelper设计实现</a></li>
<li><a href="#3.3">—— MongoDBHelper设计实现</a></li>
<li><a href="#AIServicesLibrary设计实现">AIServicesLibrary设计实现</a></li>
<li><a href="#认知服务的人脸鉴定理解">认知服务的人脸鉴定理解</a></li>
<li><a href="#PersonFace的设计和实现">PersonFace的设计和实现</a></li>
<li><a href="#扩展ResultModel">扩展ResultModel</a></li>
<li><a href="#Verify系成员">Verify系成员</a></li>
<li><a href="#Detect系成员">Detect系成员</a></li>
<li><a href="#Identify系成员">Identify系成员</a></li>
<li><a href="#FindSimilar系成员">FindSimilar系成员</a></li>
<li><a href="#AIServicesWebApi设计实现">AIServicesWebApi设计实现</a></li>
<li><a href="#PersonGroupController设计实现">PersonGroupController设计实现</a></li>
<li><a href="#PersonController设计实现">PersonController设计实现</a></li>
<li><a href="#PersonFaceController设计实现">PersonFaceController设计实现</a></li>
<li><a href="#最后再说几句">最后再说几句</a></li>
</ul>


<h2><a class="anchor" id="user-content-1" aria-hidden="true" href="#1"><span class="octicon octicon-link" aria-hidden="true"></span></a><a name="user-content-1"></a>前言</h2>

“一个伟大的项目需要有合适的土壤：一个惊艳实用的创意，一个优秀的技术基础平台，一个不懈努力的践行者。”
——天眼四骑士

当“微软认知服务”还是“牛津计划”的时候，当how-old.net作为验证微软也能娱乐化的时候，我们就开始琢磨这套未来无限的平台可以做些什么了。要做一个伟大的项目，我们要有一个很帅气的团队名字，恰好这个时候，我们都看过《NOW YOU SEE ME》，我们觉得天眼这个组织非常酷，强大聪明无所不在的天眼岂不正是AI嘛，所以我们的团队就叫天眼四骑士。

我们团队在年初就开始研究“微软认知服务”的开发的检验，从构思产品到学习“微软认知服务”的API，断断续续用了不少时间，但是非常有收获，最终内部完成了一个简单可用案例。当完成了技术的初始积累，我们就开始着手去实践我们构思的那个伟大的项目（好像太自夸了）。
这次分享的内容，是我们这个项目中的一个分支，虽然是一个分支，但是我们也将这个分支包装成了一个可实践的商业解决方案。
我们设计的场景是一家高端产品销售公司，他们管理着一百多家门店。如何找到和判断潜在客户是他们一直在孜孜不倦研究的方案，但要精确的判断出潜客不可能一个数据源或一个解决方案就可以完成，而如何收集到科学的数据源是我们一直在研究的工作。销售公司的品牌经理、CRM经理、销售经理、门店经理等角色都会关心以下问题：
	拜访门店的人流量数据，人流量中不同性别的分类数据
	拜访门店人群中是否有人在其他时间拜访了同品牌的其他门店门店
	拜访门店人群中是否有人以不同的身份和不同的销售进行了接触
	拜访者对门店的哪些展品比较感兴趣
	拜访者是否还参加过同品牌的其他活动
	是否可以将CRM数据和拜访者做识别关联管理
我这个方案就是用来解决这些场景的:在门店和活动现场布置多个摄像头，将拍摄的照片使用“微软认知服务”的人脸识别、分组、鉴定等能力，我们可以计算出人流量、不同门店是否出现过同一个人、指定的人是否在不同活动出现过等等。配合BI的能力，我们提供用户的完美的数据可视化报表：
 

<h2><a class="anchor" id="user-content-2" aria-hidden="true" href="#2"><span class="octicon octicon-link" aria-hidden="true"></span></a><a name="user-content-2"></a>总体架构设计</h2>


这次的应用需要涉及的面比较多，最重要的是Services和WebApi，我绘制了一个组件图提供大家比较直观的了解总体架构中的组件关系。

 

Services提供了完善的“微软认知服务”封装，同时我将这个封装设计为一个可移植的组件，这个组建提供了对“微软认知服务”的调用、持久化管理、更强大的业务能力。WebApi是具体的业务应用，但是针对“微软认知服务”WebApi也抽象出了一套业务规则，可以方便的和其他业务应用进行整合。
从上图中可以看到，Services和WebApi有各自的数据存储组件，它们各自维护了自己的数据持久化，确保了组件的内聚能力，当开发人员使用这些组件的时候可以完成透明的开箱即用。


<h2><a class="anchor" id="user-content-3" aria-hidden="true" href="#3"><span class="octicon octicon-link" aria-hidden="true"></span></a><a name="user-content-3">基础工作和概念</a></h2>
