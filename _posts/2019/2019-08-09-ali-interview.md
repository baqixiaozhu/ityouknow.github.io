---
layout: post
title:  jmeter+nmon+crontab简单的执行接口定时压测.......
no-post-nav: true
category: it
tags: [it]
excerpt: 压测
---


<p style="background: #007699;"><span style="color: #ffffff;"><strong><span style="font-size: 15px;">一.概述</span></strong></span></p>
<p><span style="font-size: 14px; font-family: 宋体;">临时接到任务要对系统的接口进行压测，上面的要求就是：压测，并发2000</span></p>
<p><span style="font-size: 14px; font-family: 宋体;">在不熟悉系统的情况下，按目前的需求，需要做的步骤：</span></p>
<ol>
<li><span style="font-family: 宋体;">需要有接口脚本</span></li>
<li><span style="font-family: 宋体;">需要能监控系统性能</span></li>
<li><span style="font-family: 宋体;">需要能定时执行脚本</span></li>
</ol>
<p><span style="font-family: 宋体;">&nbsp;</span></p>
<p style="background: #007699;"><span style="color: #ffffff; font-size: 15px;"><strong>二.观察</strong></span></p>
<p><span style="font-family: 宋体;">&gt;针对第一点：接口脚本</span></p>
<blockquote>
<p><span style="font-size: 14px; font-family: 楷体;">需要观察系统接口的情况：</span></p>
<ul>
<li><span style="font-size: 14px; font-family: 楷体;">系统使用swagger文档编辑接口，这很好，可以直接引用</span></li>
<li><span style="font-size: 14px; font-family: 楷体;">系统内关联接口熟悉，将需要的参数设置成变量以便调用</span></li>
<li><span style="font-size: 14px; font-family: 楷体;">系统内的接口返回状态很规范，可以直接判断code&amp;message</span></li>
</ul>
<p><span style="font-size: 14px; font-family: 楷体;">综上，为了效率，选择现存的开源工具执行（针对该开源工具的要就是可以使用命令行执行：jmeter）【备注：因为要定时执行】</span></p>
</blockquote>
<p><span style="font-family: 宋体;">&gt;针对第二点：监控系统性能</span></p>
<blockquote>
<p><span style="font-size: 14px; font-family: 楷体;">观察系统服务器:</span></p>
<ul>
<li><span style="font-size: 14px; font-family: 楷体;">系统为Linux</span></li>
<li><span style="font-size: 14px; font-family: 楷体;">Linux上的监控工具很多，要求是可以输出到文件并可对该文件进行分析</span></li>
<li><span style="font-size: 14px; font-family: 楷体;">或者，可以自己编写shell脚本监控获取信息，比如：top</span><span style="font-family: 楷体;">【为了效率，选择一款自主搭配即可（当前选择：nmon）】</span></li>
</ul>
</blockquote>
<p><span style="font-family: 宋体;">&gt;针对第三点：定时执行脚本</span></p>
<blockquote>
<p><span style="font-family: 楷体; font-size: 14px;">&nbsp;观察脚本即将存放并执行的系统</span></p>
<ul>
<li><span style="font-family: 楷体; font-size: 14px;">Linux系统自带crontab命令可执行定时任务</span></li>
</ul>
</blockquote>
<p style="background: #007699;"><span style="font-size: 15px;"><strong><span style="color: #ffffff;">三.编写</span></strong></span></p>
<p><strong><span style="font-family: 宋体;">&gt; 编写步骤：</span></strong></p>
<p><span style="font-family: 宋体;">1.使用jmeter编写接口脚本，并增加压测线程数，并编写启动脚本：StartJmx.sh</span></p>
<div class="cnblogs_code">
<pre><span style="font-size: 12px; font-family: 仿宋;">source /etc/profile<br />rm -rf ****<span style="color: #000000;">.jtl
</span>/绝对路径/jmeter  -n -t /绝对路径/debugTest.jmx -l /绝对路径<span style="color: #008000;">/*</span><span style="color: #008000;">***.jtl<br />sleep 10<br />nmonpid=`ps -ef | grep nmon | awk '{print $2}'`<br />kill -9 ${nmonpid}<br /></span></span></pre>
</div>
<p><span style="font-family: 宋体;">2.服务器上安装nmon，并编写启动脚本：StartNmon.sh</span></p>
<div class="cnblogs_code">
<pre><span style="font-family: 仿宋;"><span style="color: #000000;">#每5秒采集一次，采集120次，共10分钟的数据
nohup nmon </span>-f -T -s <span style="color: #800080;">5</span> -c <span style="color: #800080;">120</span> -m /绝对路径文件夹  &amp; echo $! &gt; nmonpid</span></pre>
</div>
<p><span style="font-family: 宋体;">3.编写定时脚本</span></p>
<div class="cnblogs_code">
<pre><span style="color: #800080;">0</span> <span style="color: #800080;">15</span> * * * <span style="color: #000000;">sh /绝对路径/StartNmon.sh
</span><span style="color: #800080;">0</span> <span style="color: #800080;">15</span> * * * sh /绝对路径/StartJmx.sh</pre>
</div>
<p style="background: #007699;"><span style="color: #ffffff; font-size: 15px;"><strong>&nbsp;四.综述</strong></span></p>
<p><span style="font-family: 宋体;"><strong>&gt;</strong>以上除开jmeter脚本编写，其他编写时间不超过1小时</span></p>
<ul>
<li><span style="font-family: 仿宋;">当任务来临的时候，不要慌张不要拒绝，先和对接人沟通相应的事宜，明确需求</span></li>
<li><span style="font-family: 仿宋;">需求明确之后，请思考实现方式，方式总是多种多样的，或请教前辈或上网求解</span></li>
<li><span style="font-family: 仿宋;">临时任务的重点均在于效率，这个前置条件给出的宽裕就是：你不需要把方案做的很完美，能得出结论即可</span></li>
<li><span style="font-family: 仿宋;">方案可后续再改良~</span></li>
</ul>
