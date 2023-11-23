> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [wp.gxnas.com](https://wp.gxnas.com/13825.html)

> 习惯使用 Emby 或者 Jellyfin 的 NAS 玩家都知道，刮削器默认使用的是 TMDb 官网信息的，但是由于 TMDb 是国外网站，因此演职人员列表显示的名字都是英文，如下图所示整体看起来有点不太协调，所以博主就......

       习惯使用 Emby 或者 Jellyfin 的 NAS 玩家都知道，刮削器默认使用的是 TMDb 官网信息的，但是由于 TMDb 是国外网站，因此演职人员列表显示的名字都是英文，如下图所示整体看起来有点不太协调，所以博主就想折腾一下让其显示汉字；

[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029712-1.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029712-1.jpg)

1、以下操作博主在 DSM7.21 的系统下安装和操作，打开套件中心，安装好 Container  Manager（如果你的 DSM 版本低于 7.2 的，请安装 Docker）；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029717-3.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029717-3.jpg)

2、在套件中心，安装好文本编辑器；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029719-4.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029719-4.jpg)

3、在 File Station，docker 文件夹，右键属性，查看，权限，检查一下 Everyone 是否有读取和写入的权限（如果没有就自己添加），在 “应用到这个文件夹、子文件夹及文件” 打勾，保存；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029720-5.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029720-5.jpg)

4、在 docker 文件夹里面建立一个子文件夹，取名 MediaServerTools；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029722-6.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029722-6.jpg)

5、在 MediaServerTools 子文件夹右键，属性，把位置这里的路径记一下，等下需要用；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029723-7.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029723-7.jpg)

6、打开 Container  Manager，注册表，在搜索栏输入 “mediaservertools”，在显示出来的列表中选中名字是“ddskerek/mediaservertools” 的这个容器，点下载；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029725-8.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029725-8.jpg)

7、应用；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029726-9.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029726-9.jpg)

8、耐心等待一会，直到 DSM 系统提示下载成功；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029728-10.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029728-10.jpg)

9、在 Container  Manager 左边菜单，映像，找到刚刚下载的 “ddskerek/mediaservertools”，点进去，再点 “运行”；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029729-11.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029729-11.jpg)

10、把容器名称改为 “MediaServerTools”，在“启用自动重新启动” 处打勾，下一步；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029731-12.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029731-12.jpg)

11、存储空间设置，添加文件夹；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029732-13.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029732-13.jpg)

12、选中 docker 里面的 MediaServerTools 子文件夹，填写上 “/config”；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029734-14.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029734-14.jpg)

13、向下翻页，找到 “REPO_URL” 和“REPO_RWA_URL”这两行，分别在前面的链接加上“https://ghproxy.com/”，下一步；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029736-15.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029736-15.jpg)

14、把 “向导完成后运行此容器” 的打勾去除，完成；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029737-16.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029737-16.jpg)

15、到 File Station，docker 文件夹，点菜单上的 “上传”，“上传 - 覆盖”；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029739-17.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029739-17.jpg)

16、把这个【[模版文件](https://wp.gxnas.com/wp-content/uploads/2023/10/config.yaml)】下载到电脑，再上传到群晖；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029740-18.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029740-18.jpg)

17、在 config.yaml 右键，用文本编辑器打开；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029742-19.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029742-19.jpg)

18、找到第 16 行到 31 行的地方，根据自己实际使用进行修改，博主使用的是 EMBY，因此博主修改的内容是第 24 行到 31 行，host 写局域网中访问 EMBY 的地址和端口（如果你用的是 Jellyfin，则修改的是 16 行到 23 行）；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029743-20.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029743-20.jpg)

19、打开 EMBY，登录管理员账号进入控制台，左边菜单 “用户”，找到当前管理员账号点进去，然后把浏览器地址栏显示的 userid = 后面的内容全部复制，粘贴到上一步骤第 29 行 userid 引号里面；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029745-21.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029745-21.jpg)

20、EMBY 菜单找到 “API 密钥”，新 API 密钥；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029746-22.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029746-22.jpg)

21、应用程序名称写 “MediaServerTools”，确定；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029748-23.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029748-23.jpg)

22、把显示出来的内容复制（注意只复制前面的值就行了，后面的 “- MediaServerTools” 不需要复制，如下图），到第 18 个步骤的模版文件，粘贴到第 29 行 key 引号里面；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029750-24.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029750-24.jpg)

23、修改后如下图所示；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029751-26.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029751-26.jpg)

24、TMDb 的 API 需要自己到【[TMDb 官网](https://www.themoviedb.org/)】注册账号申请，把申请到的 API 填写到下图第 34 行引号里面；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029753-27.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029753-27.jpg)

25、在 config.yaml 配置文件中继续向下翻页，找到第 59 行是否刷新人名改为 true，第 61 行是否更新概述改为 true，第 65 行刷新时间改为 1 小时，如下图；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029754-28.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029754-28.jpg)

26、修改无误后点菜单上保存；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029756-29.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029756-29.jpg)

27、关闭；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029758-30.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029758-30.jpg)

28、回到 Container  Manager 左边菜单 “容器”，在“MediaServerTools” 容器上点“启动”；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029759-31.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029759-31.jpg)

29、MediaServerTools 启动后会根据影片刮削信息自动到豆瓣查询相关的信息后进行更名，具体等待的时间根据你的视频多少来决定，影片多的话可能需要好几天甚至几个星期的时间，只需要耐心等待就行了，影片已经处理好了会在日志中显示如下图的 “处理完成” 字样；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029761-32.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029761-32.jpg)

30、由于豆瓣网站有反爬虫，所以进行到一段时间会提示搜索访问太频繁，然后系统会自动停止；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029762-33.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029762-33.jpg)

31、这时就需要设置一下定时重启 docker 容器了，在群晖控制面板，任务计划，新增，计划的任务，用户定义的脚本；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029764-34.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029764-34.jpg)

32、任务名称写 “MediaServerTools”，用户账号改为 root，在“已启动” 处打勾；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029766-35.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029766-35.jpg)

33、在 “计划” 标签里面找到开始时间设置为 “00:00”，在“同一天内继续运行” 处打勾，“重复”设置为 “第 4 小时”，“最后运行时间” 改成“20:00”；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029767-36.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029767-36.jpg)

34、把下面的命令复制，放到 “任务设置” 标签里面的 “用户定义的脚本” 中，确定，注意：脚本中的 “MediaServerTools” 必须跟容器名称保持一致（区分大小写）；

```
docker restart MediaServerTools

```

[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029769-37.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029769-37.jpg)

35、确定；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029770-38.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029770-38.jpg)

36、输入群晖当前账号的密码，提交；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029771-39.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029771-39.jpg)

37、来看一下使用后的效果，当前影片还在处理中，所以演员中只有一人的名字为中文，其他演员的名字还是英文；[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029773-40.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029773-40.jpg)

38、这是已经修改完成后的影片，演员名字已经全部改成中文了，看起来舒服多了。[![](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029775-41.jpg)](https://wp.gxnas.com/wp-content/uploads/2023/10/1698029775-41.jpg)

39、特别说明：该项目目前只对 Emby 和 Jellyfin 有效，虽然 config.yaml 配置文件有针对 Plex 的设置，但是设置后修改也是无效的，所以如果使用 Plex 的话就不要浪费时间了。