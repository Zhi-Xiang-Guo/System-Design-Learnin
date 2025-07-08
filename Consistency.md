# BackGround

任何涉及共享状态和并发操作的系统都必须仔细考虑一致性问题。

解决这些问题的方法论通常涉及**原子性操作、隔离性保证、分布式锁、事务管理、幂等性设计以及鲁棒的错误处理和监控机制**。



寻找潜在的竞态条件、非原子操作、缺乏锁机制或事务管理的区域。

**关注点：** 共享资源的访问（如数据库记录、缓存条目）、异步操作、以及任何涉及多个步骤或多个服务交互的逻辑。

# issue cases

<details class="lake-collapse"><summary id="u63a28ce6"><span class="ne-text">Tinder Miss match</span></summary><h3 id="743d5c29" style="font-size: 20; line-height: 28px; margin: 16px 0 5px 0"><span class="ne-text">1. 错失匹配问题 (Miss Match Problem)</span></h3><p id="udf992a4d" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">这是并发“滑动”操作最直接的后果。考虑两个用户，用户A和用户B，他们都看到了对方的资料。</span></p><ul class="ne-ul" style="margin: 0; padding-left: 23px"><li id="u760b855d" data-lake-index-type="0"><strong><span class="ne-text">场景：</span></strong></li></ul><ol class="ne-list-wrap" style="margin: 0; padding-left: 23px; list-style: none"><ol ne-level="1" class="ne-ol" style="margin: 0; padding-left: 23px; list-style: lower-alpha"><li id="uc3ea52fb" data-lake-index-type="0"><span class="ne-text">用户A向用户B的资料“向右滑”（表示喜欢）。</span></li><li id="u7c81c0db" data-lake-index-type="0"><span class="ne-text">与此同时，用户B也向用户A的资料“向右滑”（表示喜欢）。</span></li></ol></ol><ul class="ne-ul" style="margin: 0; padding-left: 23px"><li id="u6351e099" data-lake-index-type="0"><strong><span class="ne-text">没有原子操作时（假设直接写入数据库或简单的应用层逻辑）：</span></strong></li></ul><ul class="ne-list-wrap" style="margin: 0; padding-left: 23px; list-style: none"><ul ne-level="1" class="ne-ul" style="margin: 0; padding-left: 23px; list-style: circle"><li id="u9ea37600" data-lake-index-type="0"><span class="ne-text">两个滑动请求几乎同时到达</span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">Swipe Service</span></code><span class="ne-text">。</span></li><li id="ue4fade69" data-lake-index-type="0"><span class="ne-text">每个服务实例独立地检查对方用户是否已经“向右滑过”。</span></li><li id="u7e48c24d" data-lake-index-type="0"><span class="ne-text">如果没有原子检查和更新，两者都可能发现对方尚未“向右滑”。</span></li><li id="ub35c917a" data-lake-index-type="0"><span class="ne-text">然后，两者都继续记录自己的“向右滑”操作，并可能触发“匹配”通知。</span></li><li id="u8980f79b" data-lake-index-type="0"><span class="ne-text">然而，如果“匹配”条件依赖于在检查的精确时刻，双方的滑动都已存在，则可能发生竞态条件：一个用户的滑动被记录，而另一个用户的检查发生在其提交之前，导致“错失匹配”。</span></li></ul></ul><p id="uebd1d82d" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><strong><span class="ne-text"></span></strong></p><p id="u5e0131a6" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><strong><span class="ne-text">方案：</span></strong></p><p id="u4798b537" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">在Match表 定时任务捞次</span></p><p id="ue20b34d8" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">利用Redis atomic保证</span></p><h3 id="5fbb3742" style="font-size: 20; line-height: 28px; margin: 16px 0 5px 0"><span class="ne-text">2. 状态不一致 (Inconsistent State)</span></h3><ul class="ne-ul" style="margin: 0; padding-left: 23px"><li id="ucfaca050" data-lake-index-type="0"><strong><span class="ne-text">计数器/聚合上的竞态条件：</span></strong><span class="ne-text"> 如果您正在跟踪用户收到的“喜欢”数量或剩余的滑动次数等信息，没有原子操作的并发更新可能会导致不正确的计数。例如，如果两个用户同时喜欢一个资料，并且系统读取当前计数，增加它，然后写回，其中一个更新可能会覆盖另一个，导致一个“喜欢”的丢失。</span></li><li id="u80107b57" data-lake-index-type="0"><strong><span class="ne-text">重复通知：</span></strong><span class="ne-text"> 如果没有原子操作来确保匹配只处理一次，用户可能会收到关于同一事件的多个“你已匹配！”通知，导致糟糕的用户体验。</span></li></ul><h3 id="a2d57939" style="font-size: 20; line-height: 28px; margin: 16px 0 5px 0"><span class="ne-text">3. 数据完整性问题 (Data Integrity Issues)</span></h3><ul class="ne-ul" style="margin: 0; padding-left: 23px"><li id="u7e61aa8b" data-lake-index-type="0"><strong><span class="ne-text">部分更新：</span></strong><span class="ne-text"> 在涉及多个数据库操作的复杂事务中（例如，记录滑动、更新用户的匹配计数、发送通知），如果没有适当的原子性（例如分布式事务管理器或带有幂等性的仔细应用层重试），这些操作中间的失败可能会使系统处于不一致状态。例如，滑动可能已被记录，但匹配通知可能未发送。</span></li><li id="u5550f7f6" data-lake-index-type="0"><strong><span class="ne-text">幻读/不可重复读：</span></strong><span class="ne-text"> 尽管这更多与数据库隔离级别相关，但如果系统依赖于读取滑动操作的当前状态来做出决策，并且服务之间没有强大的强一致性模型，某个服务可能读取到一个立即被另一个并发操作无效化的状态，从而导致逻辑错误。</span></li></ul></details>

- **Uber (或任何打车/外卖平台)：**

- **问题示例：**

- **订单匹配：** 多个司机同时接受同一订单。如果没有原子性操作（例如，通过分布式锁或数据库事务的行锁），可能导致多个司机都以为自己接到了单，甚至多个司机同时前往接乘客。
- **价格计算/扣款：** 在高峰期或促销活动期间，并发请求可能导致价格计算错误或重复扣款/退款。
- **车辆位置更新：** 大量车辆同时报告位置，如果处理不当，可能导致地图上的车辆位置更新不及时或不准确，影响匹配效率。

- **解决方案：** 类似Tinder，Uber会使用分布式事务、消息队列（用于异步处理和解耦）、分布式锁（如基于Redis或Zookeeper的锁），以及数据库的强一致性特性来确保核心业务逻辑的正确性。

- **社交媒体平台 (如Facebook, Twitter)：**

- **问题示例：**

- **点赞/评论计数：** 多个用户同时点赞或评论，可能导致计数不准确。
- **消息发布：** 多次发布同一条消息或消息顺序错乱。
- **好友关系管理：** 并发添加/删除好友，可能导致关系图谱不一致。

- **解决方案：** 最终一致性（对于非关键计数）、消息队列、乐观锁/悲观锁、以及利用Redis等缓存来处理高并发读写。