<h3><strong>Hive 功能介绍</strong></h3>
<p>Hive 是基于 Hadoop 的一个外围数据仓库分析组件，可以把 Hive 理解为一个数据仓库，但这和传统的数据库是有差别的。</p>
<p>传统数据库是面向业务存储，比如 OA、ERP 等系统使用的数据库，而数据仓库是为分析数据而设计的。同时，数据仓库是在数据量巨大的情况下，为了进一步挖掘数据资源、为了企业决策需要而产生的，它不是所谓的“大型数据库”。</p>
<p>Hive 通过将结构化的数据文件映射到一张数据库表上，然后通过执行 SQL 语句实现查询功能。它将 SQL 语句转换为 Hadoop 上的 MapReduce 任务提交运行，这种类 SQL 语言也称为 <strong>HQL</strong>，通过这种方法，就可以使不熟悉 MapReduce 程序的用户可以很方便地利用 SQL 语句实现大数据的查询、分析和汇总。</p>
<p>因此，可以将 Hive 理解为将 HQL 语句转换为 MR 的语言翻译器。它的数据分析是基于 MapReduce 的，而数据存储使用的是 HDFS。Hive 适合对离线数据（批数据）分析处理。</p>
<h3>Hive 架构与应用场景</h3>
<h4>1. Hive 架构</h4>
<p>下图展示了 Hive 的运行和实现架构：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/10/FC/Ciqc1F7LdZGAR9tAAAC6e95LBXg140.png" alt="image1.png"></p>
<p>由图可知，Hive 主要由 Metastore、DB 和 Hiveserver2、Hive CLI 几部分组成。其中，Metastore 是 Hive 的核心，所有外围客户端比如 Beeline、Hue、Impala 最终都会连接到 Metastore，而 Metastore 再去访问 DB。</p>
<p>要访问 Hive，可以通过 Hive CLI（Command Line Interface）方式、程序连接方式(JDBC/ODBC)、Web UI 方式进行。例如，你登录 Hadoop 外围机后，可以通过执行 hive 命令（hive CLI）去访问 Hive；如果你开发了一个程序，想让程序自动连接 Hive 实现查询分析，那么你就需要通过 Hiveserver2 方式连接到 Hive 上来；而如果你要给客户提供一个傻瓜式的查询平台，那么你就应该选择 Hue 这个 Web 查询工具。</p>
<p>由此可知，不同的应用需求，对 Hive 的访问方式也各不相同。下面我就来分别介绍下 Hive 中各个组件的功能及应用场景。</p>
<p>为了便于理解，我们将 Hive CLI、Beeline CLI、Hue、Impala 统称为 <strong>Hive 客户端</strong>。</p>
<p><strong>（1）MetaStore</strong></p>
<p>MetaStore 表示元数据存储，所谓的元数据就是 Hive 创建的数据库、表等信息，这些元数据可以存储在关系型数据库 Derby、MySQL 中。</p>
<p>可以把 MetaStore 理解为后端数据库的代理层，Hive 客户端连接到 MetaStore 后，MetaStore 再去连接后端 MySQL 数据库来存取元数据。这样，就可以有多个 Hive 客户端同时连接到 MetaStore，而且这些客户端不需要知道 MySQL 数据库的用户名和密码，它们只需要连接 MetaStore 服务即可。</p>
<p>MetaStore 服务实际上就是一种 Thrift 服务，通过它我们可以获取到 Hive 元数据，并且通过 Thrift 获取元数据的方式，屏蔽了数据库访问需要的驱动、URL、用户名、密码等细节。由此可知，通过 MetaStore 服务，实现了对访问数据库的统一认证和验权。</p>
<p><strong>（2）HiveServer/ HiveServer2</strong></p>
<p>顾名思义，这是 Hive 上启动的一个服务，在早期的 Hive 版本中，启动的服务是 HiveServer。此服务启动后，Hive 客户端就可以通过 IP 加端口的方式对 Hive 进行访问，此服务主要用于远程客户端使用各种编程语言向 Hive 提交请求并查询结果的情况。远程客户端可以通过 jdbc、odbc 等开发接口访问 HiveServer 服务。</p>
<p>由此可知，HiveServer 是一种可选服务，当有程序需要连接 Hive 的时候，才需要它，这也是生产环境使用最多的一种方式。但 HiveServer 无法处理来自多个客户端的并发请求，因此，从 Hive 0.11.0 版本开始，HiveServer2 替代了 HiveServer。</p>
<p>HiveServer2 是 HiveServer1 的改进版，目前 1 已经被废弃，2 可以支持多客户端并发和身份认证。同时可以为开放的 API 客户端（如 JDBC 和 ODBC）提供更好的支持。</p>
<p>HiveServer2 服务是 Hive 推荐的使用模式，因为它更加安全并且不需要直接对用户使用的 HDFS、Metastore 进行赋权。</p>
<p><strong>（3）Hive CLI</strong></p>
<p>Hive CLI 表示命令行接口，也就是以命令行的形式输入 SQL 语句进行数据查询操作。例如，你直接登录到 Hive 所在的服务器，然后在命令行中执行 hive 命令，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/08/CgqCHl7LdaiAR0y9AACfQqms-MA564.png" alt="image2.png"></p>
<p>这种使用模式是 Hive CLI，这种客户端模式是最古老的一种 Hive 访问模式，它将 SQL 在本地编译，然后直接访问 MetaStore，属于<strong>重客户端模式</strong>。</p>
<p>目前，Hive CLI 已经被废弃，推荐使用 Beeline 模式。</p>
<p><strong>（4）Beeline</strong></p>
<p>Beeline 是一个新的 Hive CLI，它是一种基于 SQL 命令行的 JDBC 客户端，相比于 Hive CLI，它在安全性、稳定性、认证机制及界面使用上都有了很大提升。需要注意，使用 Beeline，需要依赖 Hiveserver2 服务，也就是 Hiveserver2 服务启动后，才能使用 Beeline 客户端。这从 Beeline 的运行流程上可以看出，Beeline 启动后，它首先会连接到 Hiveserver2 服务端口，接着再去请求 Metastore，而 Metastore 最后再去请求数据库，获取需要的元数据信息。</p>
<p>从 Hive 0.14 版本开始，Beeline 在通过 HiveServer2 工作时，会从 Hiveserver2 获取输出日志信息到标准错误输出（STDERR）。因此，我们在 beeline 命令行执行任务时，如果发生错误，会在屏幕输出错误信息。</p>
<p>Beeline 模式是将 SQL 提交到 Hiveserver2，然后由 Hiveserver2 负责编译，接着再去访问 Metastore，最后将分析任务提交到 Hadoop 上，相对于 Hive CLI，Beeline 是轻客户端模式。</p>
<h4>2. Hive 应用场景</h4>
<p>Hive 是目前企业使用最多的数据仓库工具，典型应用场景有日志数据分析、构建数据仓库及数据挖掘等。例如，要统计 App 应用一段时间的 PV、UV 数据，并将数据通过不同维度进行展示；又比如要统计一个气象数据，要求统计出来 2019 年全年排行前 10 的最高气温日期及具体的温度。这些都是 Hive 的专长，从这些应用场景中可以看出，这些需求都对时间没有特别要求，一般是按天、周、月、年来进行数据统计。这其实就是<strong>离线分析场景</strong>。</p>
<p>此外，通过 Hive 还可以构建统一标准的数据仓库，从而提供基础数据，供上层应用进行更细化的数据分析。</p>
<h3>Hive Metastore 三种运行模式</h3>
<p>Metastore 作为访问元数据库的代理层，它有三种运行模式，即内嵌模式（Embedded）、本地模式（Local）及远程模式（Remote Server），每种模式对应不同的使用场景。</p>
<h4>1. 内嵌模式</h4>
<p>内嵌模式使用的是 Hive 内嵌的 Derby 数据库来存储元数据，它不需要额外启动 Metastore 服务。数据库和 Metastore 服务都嵌入在启动的 Hive 进程中。这个是默认的模式，配置简单，但一个 Hive 进程一次只能连接一个客户端。使用此模式，只需要下载 Hive 安装包，解压后在命令行中执行 hive 命令，启动即可使用。</p>
<p>如果另一个客户端也要使用 Hive 的话，只需解压安装包启动 hive 命令即可。由此可以看出，不同客户端、不同路径启动的 hive，每个 hive 进程都拥有自己的一套元数据，这些元数据无法共享。</p>
<p>内嵌模式只适用于实验环境，不适用于生产环境。</p>
<h4>2. 本地模式</h4>
<p>本地模式不再使用内嵌的 Derby 作为元数据的存储介质，而是采用外部数据库来存储元数据。目前支持的外部数据库有 MySQL、PostgreSQL、Oracle 等，企业使用 MySQL 的比较多。</p>
<p>本地模式也不需要启动 Metastore 服务，当启动 Hive 服务后，Hive 进程里面会默认启动一个 Metastore 服务。如果我们采用的外部存储是 MySQL，那么 MySQL 可以和 Metastore 在一台机器上，也可以不在一台机器上。</p>
<p>Hive 在启动的时候会根据配置文件（hive-site.xml）中的 hive.metastore.uris 参数值来判断运行模式，如果没有配置此参数或者此参数为空，那么 Hive 将启动一个本地模式。</p>
<p>本地模式是一个多用户模式，多个客户端可以连接到同一个 MySQL 中，但每个客户端必须要有对 MySQL 的访问权限，也就是说每个连接到 Hive 的客户端都必须在 MySQL 库中进行授权。很显然，这种机制有很大问题，如果有几百个客户端需要连接到 Hive 的话，那么就要在 MySQL 中做几百个授权。此时，权限管理和数据安全都将面临极大考验。</p>
<p>这种模式可以作为公司内部测试、开发环境使用，不适用于生产环境。</p>
<h4>3. 远程模式</h4>
<p>远程模式仍然是采用外部数据库来存储元数据，同时需要单独启动 Metastore 服务，并且 Metastore 服务和 Hive 服务是两个独立不同的进程。由于启动了 Metastore 服务，Hive 客户端只需要在 hive-site.xml 中配置 hive.metastore.uris 参数来指定 Metastore 服务所在机器的 IP 和端口，即可快速连接到后端的元数据库中，无需对客户端在数据库中进行授权操作。</p>
<p>在生产环境中，建议使用远程模式，它更加高效和安全。</p>
<p>Hive Metastore 的三种配置模式，其实也就是 Hive 的三种运行方式，你可以根据使用场景来决定使用哪种模式。下面我将介绍 Hive Metastore 远程模式下的使用。</p>
<h3>Hive 安装以及与 Hadoop 整合</h3>
<p>了解完 Hive 的架构及各个组件的功能后，下面我将开始介绍如何安装 Hive 并与 Hadoop 进行整合。</p>
<h4>1. 部署环境介绍</h4>
<p>这里以 Apache Hadoop 发行版本为例，介绍如何实现 Hadoop 与 Hive 的整合，Hive 版本同样采用 Apache Hive 发行版本。我这里的部署环境如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/EB/Ciqc1F7MzxeAV5c6AAC5xf3Kkhs081.png" alt="1.png"></p>
<p>从这个规划中可知，我将 Hivemetastore 和 Hiveserver2 服务都运行在 HiveDB 主机上，另外单独一台主机用来运行 Hive 客户端服务，还有一台用来运行 MySQL 服务。</p>
<p>这三台主机中，HiveDB 和 Hiveclient 需要部署 Hadoop 基础环境，也就是将 Hadoop 集群的安装程序、JDK、配置文件、环境变量等信息复制到这两台主机对应的路径下即可，不需要启动任何 Hadoop 服务。</p>
<h4>2. Hive 的安装与配置</h4>
<p><strong>（1）下载并解压 Hive 安装包</strong></p>
<p>直接从 <a href="https://hive.apache.org/">Hive 官网</a>下载安装包，这里我选择的版本是 Hive-3.1.2。Hive2.x 版本与 Hadoop3.x 版本整合会出现兼容性问题，这个需要注意。</p>
<p><a href="https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-3.1.2/">点击这里下载 Hive 二进制版本</a>。Hive 需要在 HiveDB 和 Hiveclient 两个机器上安装，在安装前，需确保你已经在这两个机器上部署好了 Hadoop 基础环境，并且创建了 Hadoop 用户，Hive 的安装步骤如下：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@hivedb</span> ~]$ mkdir /opt/bigdata/hive
[hadoop<span class="hljs-meta">@hivedb</span> ~]$ tar -zxvf apache-hive-<span class="hljs-number">3.1</span><span class="hljs-number">.2</span>-bin.tar.gz -C /opt/bigdata/hive
[hadoop<span class="hljs-meta">@hivedb</span> ~]$ cd /opt/ bigdata/hive
[hadoop<span class="hljs-meta">@hivedb</span>  hive]$  ln -s apache-hive-<span class="hljs-number">3.1</span><span class="hljs-number">.2</span>-bin current
</code></pre>
<p>这里我将 Hive 安装到了 /opt/bigdata 目录下，跟 Hadoop 安装目录保持一致，便于维护。<br>
上面这个操作过程是通过 Hadoop 用户完成的，请确保你在 /opt/bigdata 目录下有足够的权限，后面的配置操作都是通过这个来完成的。</p>
<p><strong>（2）配置环境变量</strong></p>
<p>为了方便使用，我把 hive 命令加到环境变量中去，编辑 hadoop 用户下的 .bashrc 文件或者 .bash_profile 文件，添加如下内容：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">export</span> JAVA_HOME=<span class="hljs-regexp">/opt/</span>bigdata/jdk
<span class="hljs-keyword">export</span> CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
<span class="hljs-keyword">export</span> PATH=$JAVA_HOME/bin:$PATH
<span class="hljs-keyword">export</span> HADOOP_HOME=<span class="hljs-regexp">/opt/</span>bigdata/hadoop/current
<span class="hljs-keyword">export</span> HADOOP_MAPRED_HOME=${HADOOP_HOME}
<span class="hljs-keyword">export</span> HADOOP_COMMON_HOME=${HADOOP_HOME}
<span class="hljs-keyword">export</span> HADOOP_HDFS_HOME=${HADOOP_HOME}
<span class="hljs-keyword">export</span> HADOOP_YARN_HOME=${HADOOP_HOME}
<span class="hljs-keyword">export</span> HTTPFS_CATALINA_HOME=${HADOOP_HOME}/share/hadoop/httpfs/tomcat
<span class="hljs-keyword">export</span> CATALINA_BASE=${HTTPFS_CATALINA_HOME}
<span class="hljs-keyword">export</span> HADOOP_CONF_DIR=<span class="hljs-regexp">/etc/</span>hadoop/conf
<span class="hljs-keyword">export</span> HTTPFS_CONFIG=<span class="hljs-regexp">/etc/</span>hadoop/conf
<span class="hljs-keyword">export</span> PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
<span class="hljs-keyword">export</span> HIVE_HOME=<span class="hljs-regexp">/opt/</span>bigdata/hive/current
<span class="hljs-keyword">export</span> PATH=$PATH:$HIVE_HOME/bin
</code></pre>
<p>注意，这个环境变量信息一部分是 Hadoop 集群的，另外一部分是 Hive 的，请确保它们都存在。保存退出后，运行 source 命令使配置立即生效。</p>
<p><strong>（3）修改 Hive 配置文件 hive-site.xml</strong></p>
<p>新建一个文件 hive-site.xml，并在 hive-site.xml 中粘贴如下配置信息：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">configuration</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>javax.jdo.option.ConnectionURL<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>jdbc:mysql://172.16.213.239:3306/hive?createDatabaseIfNotExist=true<span class="hljs-symbol">&amp;amp;</span>useSSL=false<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>javax.jdo.option.ConnectionDriverName<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>com.mysql.cj.jdbc.Driver<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>javax.jdo.option.ConnectionUserName<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>hive<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>javax.jdo.option.ConnectionPassword<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>hivepasswd<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>hive.cli.print.header<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>hive.cli.print.current.db<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span>
</code></pre>
<p>由于使用的是 MySQL8.0 版本，默认 Hive 和数据库连接走的是 SSL 方式，因此这里需要加上“useSSL=false”关闭 SSL 连接，通过普通模式进行连接。</p>
<p>Hive 的配置文件是 XML 格式，而在 xml 文件中 &amp;amp；才表示 &amp;，因此 jdbc 连接数据库的方式应该这么写：</p>
<pre><code data-language="java" class="lang-java">jdbc:mysql:<span class="hljs-comment">//localhost:3306/hive?createDatabaseIfNotExist=true&amp;amp;useSSL=false</span>
</code></pre>
<h4>3. MySQL 的安装与配置</h4>
<p>这里我们采用 MySQL 作为 Hive Metastore 的存储介质，并且 MySQL 独立安装在一台机器上， 这里 MySQL 的安装过程略去，我安装的版本是 8.0，安装完成后，需要<a href="https://dev.mysql.com/downloads/connector/j/">点击这里下载一个 mysql jdbc 包</a>，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/10/FD/Ciqc1F7LdbqAEr64AAFM6XIDC7U776.png" alt="image3.png"></p>
<p>将下载下来的压缩包传到 HiveDB 和 Hiveclient 两个机器上，然后执行如下操作：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@hivedb</span> ~]$tar -zxvf  mysql-connector-java-<span class="hljs-number">8.0</span><span class="hljs-number">.20</span>.tar.gz
[hadoop<span class="hljs-meta">@hivedb</span> ~]$cd mysql-connector-java-<span class="hljs-number">8.0</span><span class="hljs-number">.20</span>
[hadoop<span class="hljs-meta">@hivedb</span> mysql-connector-java-<span class="hljs-number">8.0</span><span class="hljs-number">.20</span>]$cp mysql-connector-java-<span class="hljs-number">8.0</span><span class="hljs-number">.20</span>.jar  /opt/bigdata/hive/current/lib
</code></pre>
<p>这个操作过程是将 mysql-connector-java 驱动包复制到 Hive 的 lib 目录下。因为 Hive 要连接 MySQL 的话，需要驱动支持。</p>
<p>接着，启动 MySQL 数据库，创建一个 mysql 用户用于 hive 连接，操作过程如下：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@mysqldb ~]# systemctl start mysqld
[root@mysqldb ~]# mysql -u root –p
mysql&gt; create user 'hive'@'localhost' identified by 'hivepasswd';
mysql&gt;create user 'hive'@'172.16.213.11' identified by 'hivepasswd'; 
mysql&gt;grant all privileges on hive.* to 'hive'@'localhost'; 
mysql&gt;grant all privileges on hive.* to 'hive'@'172.16.213.11';
mysql&gt;flush privileges;
</code></pre>
<p>这里创建了一个 hive 用户，并授权允许主机 172.16.213.11（HiveDB）远程访问 MySQL 库，这个用户和密码在 hive-site.xml 配置文件中会用到。</p>
<h4>4. 启动 metastore 与 hiveserver2 服务</h4>
<p><strong>（1）初始化 Hive 元数据</strong></p>
<p>在启动 Metastore 服务之前，需要先初始化安装一下 hive metastore 的元数据，执行如下命令：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@hivedb</span> ~]$ schematool -dbType mysql  -initSchema
</code></pre>
<p>其中，schematool 工具用于初始化当前 Hive 版本的 Metastore 数据。</p>
<p>在这个过程中，可能会出现如下错误：</p>
<pre><code data-language="java" class="lang-java">Exception in thread <span class="hljs-string">"main"</span> java.lang.NoSuchMethodError: 
com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
</code></pre>
<p>这个错误是因为系统找不到这个类所在的 jar 包或者 jar 包的版本不一样，导致系统不知道使用哪个。</p>
<p>com.google.common.base.Preconditions.checkArgument 这个类所在的 jar 包为 guava.jar，下面分别查看 Hive 和 Hadoop 中 guava.jar 的版本信息。</p>
<p>Hive 的 lib 库路径为：/opt/bigdata/hive/current/lib，检查发现 Hive 的该 jar 版本为 guava-19.0.jar。而从 Hadoop 的 jar 包路径 /opt/bigdata/hadoop/current/share/hadoop/common/lib 下找到该 jar 文件版本为 guava-27.0-jre.jar。这就出现了 jar 包版本冲突问题。</p>
<p>要解决这个问题，需要将 Hive 中低版本的 guava-19.0.jar 文件删除，然后复制 Hadoop 的 lib 库中的 guava-27.0-jre.jar 到 Hive 的 lib 目录下即可。</p>
<p><strong>（2）启动 Metastore 服务</strong></p>
<p>万事俱备后，就可以在 HiveDB 服务器上启动 Metastore 服务了，执行如下命令：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@hivedb</span> ~]$ hive --service metastore
</code></pre>
<p>此服务默认启动在前台，要长期运行的话，可通过如下命令放到后台运行：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@hivedb</span>~]$nohup hive --service metastore <span class="hljs-number">1</span>&gt; /opt/bigdata/hive/current/metastore.log <span class="hljs-number">2</span>&gt;/opt/bigdata/hive/current/metastor_err.log  &amp;
</code></pre>
<p>通过这种方式将 Metastore 标准日志输出和错误日志分别输出到不同的文件中，以便后期排查问题。<br>
Metastore 启动后，就可以在 Hiveclient 主机上访问 hive 服务了，但需要在 Hiveclient 主机的 hive-site.xml 文件中添加如下参数：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>hive.metastore.uris<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>thrift://hivedb:9083<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
</code></pre>
<p>hive.metastore.uris 参数其实就是指定了 hiveclient 主机连接 Metastore 服务的地址和端口，Metastore 默认的服务端口是 9083。</p>
<p>最后，直接在 hiveclient 主机执行 hive 命令，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/10/FD/Ciqc1F7LdcWATtNmAACkNxQpQb8304.png" alt="image4.png"></p>
<p>由图可知，命令执行后，可以正常进入 hive 命令行，并且对数据库做查询，也能返回结果，说明 Hiveclient 主机通过 Metastore 服务正常查询到了元数据库的内容。</p>

---

### 精选评论

##### **波：
> 导入万sql，启动了Ambari server，web ui访问报错提示 members表不存在，单独执行建表语句报错，什么鬼这破项目企业真的有人用？<br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 确认mysql版本以及jdbc驱动是否匹配，课程里面是mysql5.7版本，不建议使用mysql8版本。

##### *强：
> 我们生产上使用的也是重客户端模式 5555555

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 事实上，现在很有很多都在使用这种模式。

