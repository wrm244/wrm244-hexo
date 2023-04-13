---
title: Docker部署Overleaf包含中文字体与全套texlive镜像
categories:
  - web
tags:
  - 论文
date: 2023-04-12 16:09:48
---


::: tips
如今```Overleaf```已推出国内域名[访问](https://cn.overleaf.com/)，速度较之前有很大的提升。但考虑到有些同学为了私密与方便性，因此有了自己搭建开源Overleaf服务的打算。请注意[开源项目Overleaf](https://github.com/overleaf/overleaf)不支持开放注册（需管理员账号来申请注册[issue#461](https://github.com/overleaf/overleaf/issues/461)）与跟踪评论功能。该项目支持Docker容器化部署，安装过程比较容易。本文记录了在实验室内网环境下利用官方提供的Overleaf Toolkit的docker-compose搭建Overleaf服务的过程，==同时采用了基于官方开源搭建的镜像，包含了中文字体与全套texlive软件系统==。`该文章最后更新为2023年1月5日，注意技术文章的时效性。`
:::


# 准备工作
Overleaf 依赖于以下程序：
 - docker 
 - docker-compose

> 建议安装最新版本的 docker 和 docker-compose。

Docker环境的安装详见[官方文档](https://docs.docker.com/get-docker/)进行安装，然后继续安装[compose安装文档](https://docs.docker.com/compose/install/)安装docker-compose组件，这里不再赘述。[1]
 [1]: [Get Docker](https://docs.docker.com/get-docker/)

# 安装并配置 Overleaf
## Overleaf Toolkit 部署
### 拉取Overleaf Toolkit 工具包
首先，将这个工具包 git 存储库克隆到你的主机上（以下命令已重命名为overleaf-toolkit文件夹）：

```bash
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit
```
接下来，让我们进入此目录：

```bash
cd ./overleaf-toolkit
```

> 对于本文章的其余部分，我们将假定你将从此目录运行所有后续命令。[^2]

 [^2]: [overleaf-toolkit-quick-start-guide](https://github.com/overleaf/toolkit/blob/master/doc/quick-start-guide.md)

### 初始化配置
通过运行以下命令创建本地配置：

```bash
bin/init
```
> bin是该工具包提供的一组脚本，这些脚本封装起来，并为你处理大部分的细节。
> 
查看配置目录下的内容config/

```bash
ls config
```
config目录下会生成以下三个文件：
``overleaf.rc ``   
 ```variables.env ```
```version```

> 三个配置文件的作用：
> 
>  - overleaf.rc：顶级配置文件 
>  - variables.env：加载到 docker 容器中的环境变量 
>  - version：使用的 docker 镜像版本
>  
>  配置内容可根据后续需求修改

其中需要注意的是在``overleaf.rc ``  文件中，==可以修改其服务端口==：

```txt
# Sharelatex container
SHARELATEX_DATA_PATH=data/sharelatex
SERVER_PRO=false
SHARELATEX_LISTEN_IP=127.0.0.1
SHARELATEX_PORT=9000 #将该行修改为你所需服务端口，默认为80端口
```

### 修改启动镜像

> 考虑到官方提供的镜像是不完全texlive程序及不支持中文字体，在这里我基于官方开源的 overleaf 镜像搭建了自己的镜像，wrm244/sharelatex:with-texlive-full，如果对 docker 比较熟悉的同学可以跳过该步骤自行拉取官方镜像然后再进行配置，当然官方镜像不包含中文字体支持，可[参考文章](https://yxnchen.github.io/technique/Docker%E9%83%A8%E7%BD%B2ShareLaTeX%E5%B9%B6%E7%AE%80%E5%8D%95%E9%85%8D%E7%BD%AE%E4%B8%AD%E6%96%87%E7%8E%AF%E5%A2%83/#%E5%AE%89%E8%A3%85%E5%B9%B6%E9%85%8D%E7%BD%AEShareLaTeX)配置。

进入`overleaf-toolkit`文件夹下的 `lib` 目录
```bash
cd lib
```
修改```docker-compose.base.yml```文件以下内容

```bash
vim docker-compose.base.yml
```

> 将源文件的`image: "${IMAGE}"` 改为 `image: wrm244/sharelatex:with-texlive-full` 改这一行即可，以下为修改后文件内容

```yml
---
version: '2.2'
services:

    sharelatex:
        restart: always
        image: wrm244/sharelatex:with-texlive-full
        container_name: sharelatex
        volumes:
            - "${SHARELATEX_DATA_PATH}:/var/lib/sharelatex"
        ports:
            - "${SHARELATEX_LISTEN_IP:-127.0.0.1}:${SHARELATEX_PORT:-80}:80"
        environment:
          SHARELATEX_MONGO_URL: "${MONGO_URL}"
          SHARELATEX_REDIS_HOST: "${REDIS_HOST}"
          REDIS_HOST: "${REDIS_HOST}"
        env_file: ../config/variables.env
```

保存退出该文件，重新回到上一级`overleaf-toolkit`目录

```bash
cd ..
```

> 也可以修改`config`目录下`overleaf.rc`配置文件赋值为`SHARELATEX_IMAGE=wrm244/sharelatex:with-texlive-full`

## 启动服务

让我们启动 docker 服务：

```bash
bin/up
```

> bin是该工具包提供的一组脚本，这些脚本封装起来，并为你处理大部分的细节。

现在会看到来自 docker 容器的一些日志输出，表示正在拉取镜像，后续会自动运行容器。如果在终端上按下<kbd>Ctrl</kbd>+<kbd>c</kbd>，服务将关闭。您可以通过命令`bin/start`来重新启动它们（不附加到日志输出）。

## 创建管理员帐户
在浏览器中，打开 http://localhost:服务端口/launchpad 后会看到注册界面。 使用要用作管理员帐户的凭据填写，然后点击“注册”。

> 服务端口默认为80，即http://localhost/launchpad 缺省条件下即可访问。当然你也可以在`./config`目录下`overleaf.rc`文件中修改所需端口。

然后单击链接以转到登录页面（http://localhost:服务端口/login）。 登录后，你将被带到欢迎页面。

单击页面底部的绿色按钮以开始使用 Overleaf。

> 注意：在Overleaf实现中文输出需采用`XeLaTex`编译，在页面右上角可进行设置


# 反向代理域名服务

有些同学有域名访问需求，在`overleaf-toolkit`工具包中自然提供`nginx`服务，默认是关闭的。可访问该[指导文档](https://github.com/overleaf/toolkit/blob/master/doc/quick-start-guide.md)进行配置。当然你也可以自行搭建代理服务，这里不再赘述。

# 迁移与备份
如果是采用`overleaf-toolkit`工具包进行部署服务的话，在该`overleaf-toolkit`目录下的`data`文件夹会映射docker容器的文件，包括`sharelatex` `redis` `mongo` 文件夹，备份这几个文件夹即可，在迁移的时候，启动容器前先把文件复制到`data`目录下即可恢复数据。
# 更多
文章采用的docker镜像：`wrm244/sharelatex:with-texlive-full`包含了以下中文字体，具体在路径`/usr/share/fonts/chinese`下
```bash
AGENCYB.TTF                  FRAMDCN.TTF                        PER_____.TTF
AGENCYR.TTF                  framdit.ttf                        phagspab.ttf
ALGER.TTF                    framd.ttf                          phagspa.ttf
ANTQUABI.TTF                 FrederickatheGreat-Regular.ttf     PLAYBILL.TTF
ANTQUAB.TTF                  FredokaOne-Regular.ttf             PoiretOne-Regular.ttf
ANTQUAI.TTF                  FREESCPT.TTF                       POORICH.TTF
arialbd.ttf                  FRSCRIPT.TTF                       PRISTINA.TTF
arialbi.ttf                  FTLTLT.TTF                         RAGE.TTF
ariali.ttf                   FZSTK.TTF                          Raleway-Bold.ttf
ARIALNBI.TTF                 FZXBSJW.TTF                        Raleway-Regular.ttf
ARIALNB.TTF                  FZYTK.TTF                          RAVIE.TTF
ARIALNI.TTF                  Gabriola.ttf                       REFSAN.TTF
ARIALN.TTF                   gadugib.ttf                        REFSPCL.TTF
arial.ttf                    gadugi.ttf                         Roboto-BoldItalic.ttf
ariblk.ttf                   GARABD.TTF                         Roboto-Bold.ttf
ARLRDBD.TTF                  GARAIT.TTF                         RobotoCondensed-BoldItalic.ttf
Arvo-BoldItalic.ttf          GARA.TTF                           RobotoCondensed-Bold.ttf
Arvo-Bold.ttf                georgiab.ttf                       RobotoCondensed-Italic.ttf
Arvo-Italic.ttf              georgiai.ttf                       RobotoCondensed-Regular.ttf
Arvo-Regular.ttf             georgia.ttf                        Roboto-Italic.ttf
bahnschrift.ttf              georgiaz.ttf                       Roboto-Regular.ttf
BarlowCondensed-Regular.ttf  GIGI.TTF                           RobotoSlab-Bold.ttf
Barrio-Regular.ttf           GILBI___.TTF                       RobotoSlab-Regular.ttf
BASKVILL.TTF                 GILB____.TTF                       ROCCB___.TTF
BAUHS93.TTF                  GILC____.TTF                       ROCC____.TTF
BELLB.TTF                    GILI____.TTF                       ROCKBI.TTF
BELLI.TTF                    GILLUBCD.TTF                       ROCKB.TTF
BELL.TTF                     GILSANUB.TTF                       ROCKEB.TTF
BERNHC.TTF                   GIL_____.TTF                       ROCKI.TTF
BKANT.TTF                    GLECB.TTF                          ROCK.TTF
BOD_BI.TTF                   GlobalMonospace.CompositeFont      sarasa-mono-k-bolditalic.ttf
BOD_BLAI.TTF                 GlobalSansSerif.CompositeFont      sarasa-mono-k-bold.ttf
BOD_BLAR.TTF                 GlobalSerif.CompositeFont          sarasa-mono-k-extralightitalic.ttf
BOD_B.TTF                    GlobalUserInterface.CompositeFont  sarasa-mono-k-extralight.ttf
BOD_CBI.TTF                  GLSNECB.TTF                        sarasa-mono-k-italic.ttf
BOD_CB.TTF                   GOTHICBI.TTF                       sarasa-mono-k-lightitalic.ttf
BOD_CI.TTF                   GOTHICB.TTF                        sarasa-mono-k-light.ttf
BOD_CR.TTF                   GOTHICI.TTF                        sarasa-mono-k-regular.ttf
BOD_I.TTF                    GOTHIC.TTF                         sarasa-mono-k-semibolditalic.ttf
BOD_PSTC.TTF                 GOUDOSB.TTF                        sarasa-mono-k-semibold.ttf
BOD_R.TTF                    GOUDOSI.TTF                        SCHLBKBI.TTF
BOOKOSBI.TTF                 GOUDOS.TTF                         SCHLBKB.TTF
BOOKOSB.TTF                  GOUDYSTO.TTF                       SCHLBKI.TTF
BOOKOSI.TTF                  HARLOWSI.TTF                       SCRIPTBL.TTF
BOOKOS.TTF                   HARNGTON.TTF                       segmdl2.ttf
BRADHITC.TTF                 HATTEN.TTF                         segoeprb.ttf
BRITANIC.TTF                 himalaya.ttf                       segoepr.ttf
BRLNSB.TTF                   holomdl2.ttf                       segoescb.ttf
BRLNSDB.TTF                  HTOWERTI.TTF                       segoesc.ttf
BRLNSR.TTF                   HTOWERT.TTF                        segoeuib.ttf
BROADW.TTF                   impact.ttf                         segoeuii.ttf
BRUSHSCI.TTF                 IMPRISHA.TTF                       segoeuil.ttf
BSSYM7.TTF                   IndieFlower.ttf                    segoeuisl.ttf
BubblegumSans-Regular.ttf    INFROMAN.TTF                       segoeui.ttf
CabinSketch-Bold.ttf         Inkfree.ttf                        segoeuiz.ttf
CabinSketch-Regular.ttf      ITCBLKAD.TTF                       seguibli.ttf
calibrib.ttf                 ITCEDSCR.TTF                       seguibl.ttf
calibrii.ttf                 ITCKRIST.TTF                       seguiemj.ttf
calibrili.ttf                javatext.ttf                       seguihis.ttf
calibril.ttf                 JOKERMAN.TTF                       seguili.ttf
calibri.ttf                  JUICE___.TTF                       seguisbi.ttf
calibriz.ttf                 JuliusSansOne-Regular.ttf          seguisb.ttf
CALIFB.TTF                   KUNSTLER.TTF                       seguisli.ttf
CALIFI.TTF                   l_10646.ttf                        seguisym.ttf
CALIFR.TTF                   LATINWD.TTF                        ShadowsIntoLight.ttf
CALISTBI.TTF                 LBRITEDI.TTF                       SHOWG.TTF
CALISTB.TTF                  LBRITED.TTF                        simfang.ttf
CALISTI.TTF                  LBRITEI.TTF                        simhei.ttf
CALIST.TTF                   LBRITE.TTF                         simkai.ttf
cambriab.ttf                 LCALLIG.TTF                        SIMLI.TTF
cambriai.ttf                 LeelaUIb.ttf                       simsunb.ttf
cambria.ttc                  LEELAWAD.TTF                       simsun.ttc
cambriaz.ttf                 LEELAWDB.TTF                       SIMYOU.TTF
Candarab.ttf                 LeelawUI.ttf                       SitkaB.ttc
Candarai.ttf                 LeelUIsl.ttf                       SitkaI.ttc
Candarali.ttf                LFAXDI.TTF                         Sitka.ttc
Candaral.ttf                 LFAXD.TTF                          SitkaZ.ttc
Candara.ttf                  LFAXI.TTF                          SNAP____.TTF
Candaraz.ttf                 LFAX.TTF                           StaticCache.dat
CASTELAR.TTF                 LHANDW.TTF                         STCAIYUN.TTF
CENSCBK.TTF                  Lobster-Regular.ttf                STENCIL.TTF
CENTAUR.TTF                  LSANSDI.TTF                        STFANGSO.TTF
CENTURY.TTF                  LSANSD.TTF                         STHUPO.TTF
CHILLER.TTF                  LSANSI.TTF                         STKAITI.TTF
COLONNA.TTF                  LSANS.TTF                          STLITI.TTF
Comfortaa-Bold.ttf           LTYPEBO.TTF                        STSONG.TTF
Comfortaa-Regular.ttf        LTYPEB.TTF                         STXIHEI.TTF
comicbd.ttf                  LTYPEO.TTF                         STXINGKA.TTF
comici.ttf                   LTYPE.TTF                          STXINWEI.TTF
comic.ttf                    lucon.ttf                          STZHONGS.TTF
comicz.ttf                   MAGNETOB.TTF                       Swkeys1.ttf
consolab.ttf                 MAIAN.TTF                          sylfaen.ttf
consolai.ttf                 malgunbd.ttf                       symbol.ttf
consola.ttf                  malgunsl.ttf                       tahomabd.ttf
consolaz.ttf                 malgun.ttf                         tahoma.ttf
constanb.ttf                 marlett.ttf                        taileb.ttf
constani.ttf                 MATURASC.TTF                       taile.ttf
constan.ttf                  Megrim.ttf                         TCBI____.TTF
constanz.ttf                 micross.ttf                        TCB_____.TTF
COOPBL.TTF                   mingliub.ttc                       TCCB____.TTF
COPRGTB.TTF                  MISTRAL.TTF                        TCCEB.TTF
COPRGTL.TTF                  mmrtextb.ttf                       TCCM____.TTF
corbelb.ttf                  mmrtext.ttf                        TCMI____.TTF
corbeli.ttf                  MOD20.TTF                          TCM_____.TTF
corbelli.ttf                 monbaiti.ttf                       TEMPSITC.TTF
corbell.ttf                  Monoton-Regular.ttf                timesbd.ttf
corbel.ttf                   msgothic.ttc                       timesbi.ttf
corbelz.ttf                  msjhbd.ttc                         timesi.ttf
courbd.ttf                   msjhl.ttc                          times.ttf
courbi.ttf                   msjh.ttc                           trebucbd.ttf
couri.ttf                    MSUIGHUB.TTF                       trebucbi.ttf
cour.ttf                     MSUIGHUR.TTF                       trebucit.ttf
CURLZ___.TTF                 msyhbd.ttc                         trebuc.ttf
Delius-Regular.ttf           msyhl.ttc                          VastShadow-Regular.ttf
Dengb.ttf                    msyh.ttc                           verdanab.ttf
Dengl.ttf                    msyi.ttf                           verdanai.ttf
Deng.ttf                     MTCORSVA.TTF                       verdana.ttf
desktop.ini                  MTEXTRA.TTF                        verdanaz.ttf
Dosis-Regular.ttf            mvboli.ttf                         VINERITC.TTF
DroidSerif-BoldItalic.ttf    NanumPenScript-Regular.ttf         VIVALDII.TTF
DroidSerif-Bold.ttf          NIAGENG.TTF                        VLADIMIR.TTF
DroidSerif-Italic.ttf        NIAGSOL.TTF                        webdings.ttf
DroidSerif.ttf               NirmalaB.ttf                       wingding.ttf
DShirgy4.ttc                 NirmalaS.ttf                       WINGDNG2.TTF
DUBAI-BOLD.TTF               Nirmala.ttf                        WINGDNG3.TTF
DUBAI-LIGHT.TTF              ntailub.ttf                        YuGothB.ttc
DUBAI-MEDIUM.TTF             ntailu.ttf                         YuGothL.ttc
DUBAI-REGULAR.TTF            OCRAEXT.TTF                        YuGothM.ttc
ebrimabd.ttf                 OLDENGL.TTF                        YuGothR.ttc
ebrima.ttf                   ONYX.TTF                           ZillaSlab-Bold.ttf
ELEPHNTI.TTF                 OpenSans-BoldItalic.ttf            ZillaSlab-Regular.ttf
ELEPHNT.TTF                  OpenSans-Bold.ttf                  书法家行楷体.TTF
ENGR.TTF                     OpenSans-Italic.ttf                仿宋_GB2312.ttf
ERASBD.TTF                   OpenSans-Regular.ttf               南构周洋字体.ttf
ERASDEMI.TTF                 OpenSans-Semibold.ttf              南构无边.ttf
ERASLGHT.TTF                 OUTLOOK.TTF                        南构日系楷行.ttf
ERASMD.TTF                   palabi.ttf                         南构玄道硬笔.ttf
FELIXTI.TTF                  palab.ttf                          南构诗韵新隶.ttf
fms_metadata.xml             palai.ttf                          南构邱见行书.ttf
fonts.dir                    pala.ttf                           南构钟声行书.ttf
fonts.scale                  PALSCRI.TTF                        方正小标宋简体_0.ttf
FORTE.TTF                    Pangolin-Regular.ttf               日文毛笔行书.ttf
FRABKIT.TTF                  PAPYRUS.TTF                        楷体_GB2312_0.ttf
FRABK.TTF                    PARCHM.TTF                         楷体_GB2312.ttf
FRADMCN.TTF                  PERBI___.TTF                       特太行書.ttc
FRADMIT.TTF                  PERB____.TTF                       蒙纳简行书.otf
FRADM.TTF                    PERI____.TTF                       蒙纳简行楷.otf
FRAHVIT.TTF                  PERTIBD.TTF
FRAHV.TTF                    PERTILI.TTF
```
