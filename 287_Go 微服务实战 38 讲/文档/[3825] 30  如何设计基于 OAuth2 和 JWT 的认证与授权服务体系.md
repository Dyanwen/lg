<p data-nodeid="40513">在上一课时中，我们介绍了<strong data-nodeid="40584">统一认证与授权</strong>在微服务架构中存在的一些必要性和挑战，并介绍了 3 种主流的统一认证与授权方案，包括 OAuth2、分布式 Session 和 JWT。在本课时，我们将基于 OAuth2 和 JWT 设计一个认证与授权服务，让其<strong data-nodeid="40585">为我们微服务的信息安全保驾护航</strong>。</p>
<p data-nodeid="40514">微服务架构的演进使得服务体系变得分散，如果让每个微服务独立管理自身的用户信息容易造成信息隔离，阻碍应用的发展，因此将分散的认证和授权进行统一管理显得尤为必要。我们接下来就按照权限模型设计、OAuth2 与 JWT 结合、Token 中继和整体结构设计这个思路来阐述统一认证与授权服务体系的设计。</p>
<h3 data-nodeid="41623" class="">RBAC 和 ACL</h3>

<p data-nodeid="40516">认证和授权本质上是对用户的访问权限进行控制，关于权限系统的设计就不得不提及经典的<strong data-nodeid="40597">RBAC（Role-Based Access Control）模型</strong>，即<strong data-nodeid="40598">基于角色的访问控制模型</strong>。</p>
<p data-nodeid="40517">RBAC 模型将用户的操作过程抽象为 Who、What 和 How 这三者之间的关系，换句话说，就是 Who 可以对 What 进行 How 的操作。RBAC 模型<strong data-nodeid="40604">将用户通过角色与权限进行关联</strong>，用户通过成为某一角色而获得该角色的权限；一个用户可以有多个角色，而每个角色又拥有多种权限。通过“用户-角色-权限”的授权模型，大大简化了权限的管理。</p>
<p data-nodeid="40518">根据 LIST 提出的 RBAC96 模型，RBAC 由 4 个概念性模型组成，分别为：</p>
<ul data-nodeid="40519">
<li data-nodeid="40520">
<p data-nodeid="40521">RBAC0，作为<strong data-nodeid="40611">基本模型</strong>，定义了完全实现 RBAC 模型的系统的最低需求，它是 RBAC 的核心所在；</p>
</li>
<li data-nodeid="40522">
<p data-nodeid="40523">RBAC1，在包含 RBAC0 的基础上，增加了<strong data-nodeid="40617">角色分层</strong>的概念，一个角色可以从另一个角色中继承权限；</p>
</li>
<li data-nodeid="40524">
<p data-nodeid="40525">RBAC2，同样在 RBAC0 的基础上，引入<strong data-nodeid="40623">静态职责分离和动态职责分离</strong>，它是 RBAC 的约束模型；</p>
</li>
<li data-nodeid="40526">
<p data-nodeid="40527">RBAC3，作为<strong data-nodeid="40629">统一模型</strong>，在基于 RBAC0 的基础上，整合了 RBAC1 和 RBAC2。</p>
</li>
</ul>
<p data-nodeid="42945">RBAC96 模型族关系图如下所示：</p>
<p data-nodeid="44291"><img src="https://s0.lgstatic.com/i/image/M00/5C/7E/CgqCHl-BjmSASE66AAAcST7W53k950.png" alt="Drawing 0.png" data-nodeid="44295"></p>
<div data-nodeid="45139"><p style="text-align:center">RBAC96 模型族关系图</p></div>
<p data-nodeid="45140">除了 RBAC 外，还有一种比较流行的权限设计方案——<strong data-nodeid="45154">ACL（Access Control Lists）模型</strong>，即<strong data-nodeid="45155">访问控制列表</strong>。ACL 的核心思想是<strong data-nodeid="45156">将用户直接和权限关联</strong>，这样就可以非常简单地完成访问控制。但是在权限过多时，却又很容易造成授予权限时的复杂性，访问控制列表的管理最终会演变成极为烦琐的工作。</p>











<p data-nodeid="40532">我们的实践将基于 RBAC 模型进行权限设计。不过，在权限设计实践的过程中，考虑到需求的差异性，完全遵循 RBAC 模型也许并不可取。我们应该根据自身业务需要，适当地调整 RBAC 模型，使其更加契合业务。</p>
<p data-nodeid="46453">为了简化权限设计方案，便于实践的开展，我们就基于 RBAC0 模型描述的用户、角色、权限和会话关系，设计如下的权限模型：</p>
<p data-nodeid="46454" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/73/Ciqc1F-BjoeAOEb7AAC052s4Czk102.png" alt="Drawing 2.png" data-nodeid="46459"></p>
<div data-nodeid="46455"><p style="text-align:center">简易角色权限模型图</p></div>






<p data-nodeid="40537">上图中分别有用户、角色和权限 3 个实体，每个用户可以拥有多种角色，每一种角色都是一定数量权限的集合，它们之间通过关联表关联起来。当要授予某个用户部分权限时，可以将对应的角色赋予用户，比如在一个论坛系统中，帖子管理员具备审核帖子、删除帖子等权限，当有新的用户希望管理帖子时，可以将帖子管理员的角色授予该用户，这样他就具备了帖子管理的权限。</p>
<h3 data-nodeid="46888" class="">OAuth2 和 JWT</h3>

<p data-nodeid="40539">OAuth2 作为当前授权行业的标准，允许用户授权客户端访问它们存储在资源服务器上的信息。OAuth2 中默认支持 4 种客户端类型，分别为授权码类型、密码类型、简化类型和客户端类型，对它们之间的区别了解不多的同学可以参考上一课时中对 OAuth2 的详细介绍。</p>
<p data-nodeid="40540">在微服务架构中，会有多种多样的客户端通过网关接入系统中请求服务，我们可以对这些客户端进行简单的分类，包括第三方客户端应用和自家客户端应用两类。</p>
<p data-nodeid="48164"><strong data-nodeid="48175">第三方客户端应用</strong>基本为外部系统，希望请求本系统内的资源或者服务，为了避免将系统内的用户信息泄漏给第三方客户端，系统将通过授权码类型给这类客户端提供访问令牌。而对于<strong data-nodeid="48176">自家客户端应用</strong>，因为是可信任的客户端，所以可以允许它们通过密码类型获取访问令牌，即它们能够直接接触和保存用户的密码凭证等信息。</p>
<p data-nodeid="48165" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BjpWAOslFAAA3GP2cSQQ804.png" alt="Drawing 3.png" data-nodeid="48179"></p>
<div data-nodeid="48166"><p style="text-align:center">多客户端访问模型图</p></div>





<p data-nodeid="49024">JWT 是高度紧凑和自包含的安全传输对象，关于 JWT 头部、有效负载和签名的介绍可以参考上一课时。我们可以将 JWT 用作访问令牌和刷新令牌的载体，将签发 JWT 的 secret 保存在授权服务器，由授权服务器签发 JWT 样式的访问令牌。同时，我们还可以将用户的相关信息，比如用户 ID、用户角色码和权限码等信息放到有效负载中，当下游资源服务器使用用户的信息时，可以直接从访问令牌中解析获取，而无须重复查询数据库，这就减少了请求消耗，提升了访问效率。</p>
<p data-nodeid="49456"><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BjqKAJH-jAAAqGAJDXv8651.png" alt="Drawing 4.png" data-nodeid="49460"></p>
<div data-nodeid="49457" class=""><p style="text-align:center">携带用户信息的 JWT 样式访问令牌</p></div>




<h3 data-nodeid="50733" class="">Token 中继</h3>




<p data-nodeid="40549">在微服务架构中，各个微服务之间存在错综复杂的调用关系，而且在调用过程中很可能需要获取用户的身份信息，甚至部分接口还需要对用户拥有的权限进行鉴权。虽然我们可以在业务请求体中要求上游服务携带用户的相关信息，然后在业务实现中根据用户信息，去授权服务器查找用户对应的权限信息并进行鉴权操作，但是这种方式对业务开发人员不友好，需要他们在业务实现代码中频繁传递用户信息。同时这其中的部分通用鉴权行为，完全可以通过资源服务器的方式实现，从而减轻业务开发工作。</p>
<p data-nodeid="51983">我们可以<strong data-nodeid="51995">将访问令牌在多个请求中进行传递</strong>，也就是<strong data-nodeid="51996">Token 中继</strong>，由资源服务器根据自身的认证和鉴权配置，对访问令牌中用户身份和权限进行鉴权，如下图所示：</p>
<p data-nodeid="51984" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/73/Ciqc1F-BjrSAFuiKAACSXWInd6I057.png" alt="Drawing 6.png" data-nodeid="51999"></p>
<div data-nodeid="51985"><p style="text-align:center">在资源服务器中传递访问令牌</p></div>






<p data-nodeid="40554">如果访问令牌是 JWT 样式，并且其中包含了用户身份信息和权限信息，那么资源服务器就可以直接根据其内的信息进行鉴权操作，从而避免频繁向权限系统请求用户的角色和权限信息。同时，资源服务器应该具备 Token 中继的能力，自动将请求中的访问令牌携带到下游，让鉴权操作对业务开发人员透明。</p>
<h3 data-nodeid="52412" class="">整体结构设计</h3>

<p data-nodeid="53232">基于上面的介绍，我们可以将微服务统一认证与授权服务体系设计为如下：</p>
<p data-nodeid="53652"><img src="https://s0.lgstatic.com/i/image/M00/5C/73/Ciqc1F-Bjr2AVEhFAABGPArWoOo510.png" alt="Drawing 7.png" data-nodeid="53656"></p>
<div data-nodeid="53653" class=""><p style="text-align:center">统一认证与授权服务整体结构设计</p></div>





<p data-nodeid="40559">上述这个设计图的整体结构可描述为如下：</p>
<ul data-nodeid="40560">
<li data-nodeid="40561">
<p data-nodeid="40562"><strong data-nodeid="40715">客户端</strong>作为用户的代理，通过授权码类型或者密码类型向授权服务器请求访问令牌。客户端获取到对应的访问令牌后，即可通过网关请求对应的微服务访问用户存储在系统中的资源。</p>
</li>
<li data-nodeid="40563">
<p data-nodeid="40564"><strong data-nodeid="40720">网关服务</strong>不仅仅起到为下游微服务做反向代理的作用，还作为资源服务器去验证请求携带的访问令牌的有效性。JWT 样式的访问令牌在颁发之后，在 JWT 注册声明的有效时间到达之前，访问令牌都会被认为是有效的。考虑到这个问题，我们就需要在授权服务器中维护一个用户与有效访问令牌的映射，在令牌主动失效时，将该条映射关系主动删除，以保证失效后的访问令牌不可用。因此，网关在检验访问令牌的有效性时，是需要请求授权服务器进行验证，验证不通过的请求将被直接拒绝。</p>
</li>
<li data-nodeid="40565">
<p data-nodeid="40566"><strong data-nodeid="40725">授权服务器</strong>持有签发 JWT 样式访问令牌的 sercet，在请求访问令牌时，它会验证客户端的身份和携带的用户凭证或者授权码信息，给通过验证的客户端颁发合法的访问令牌。同时，授权服务器会接收网关关于令牌验证的请求，通过合法的访问令牌的请求，拒绝非法请求。除此之外，授权服务器还具备权限管理的能力。</p>
</li>
<li data-nodeid="40567">
<p data-nodeid="40568"><strong data-nodeid="40730">业务服务</strong>在处理业务时，可能需要获取当前访问用户的身份信息，在进行敏感操作时还会对用户的角色和权限进行鉴权。我们可以通过 Token 中继机制，在多个资源服务器之间传递访问令牌。由于访问令牌为 JWT 样式，资源服务器可以直接从访问令牌中解析出用户相关的角色和权限信息进行鉴权。</p>
</li>
</ul>
<h3 data-nodeid="54067" class="">小结</h3>

<p data-nodeid="40570">统一认证与授权服务体系对于微服务架构来说是重要的基础设施，它能够保护系统中的信息安全，为系统带来统一的身份认证、第三方用户授权等基础能力，提高系统的开放能力，有利于构建业务生态体系。</p>
<p data-nodeid="40571">在本课时，我们基于 OAuth2 和 JWT 设计了微服务架构下的认证与授权服务体系，借助 RBAC 设计了系统的权限模型，还讲解了 Token 中继机制如何在诸多资源服务器中简易地传递用户角色和权限信息。希望通过本课时的学习，能够加深你对微服务架构下统一认证与授权的设计与认知。在下一课时，我们将根据本课时中的设计，实践如何通过 Go 实现授权服务器。</p>
<p data-nodeid="40572">最后，你对统一认证与授权的设计有什么独到的想法或者见解，欢迎在留言区分享！</p>

---

### 精选评论


