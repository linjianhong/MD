问答模块设计
============
*2015/12/26*



1. 问答模块组成

    用户提问
    专家回答
    用户评价
    编导归档
1. 数据库
    1. 一个完全的问答
        1. 提问

            一行

            <pre class=”brush: java; gutter: true;”>
            // Java代码
            class HelloImportnew {

            }
            </pre>

        1. 不满意（若干组）

            每组由若干行回答和一行不满意组成

        1. 满意（一组）

            由若干行回答和一行不满意组成

    1. 概述

        提问、回答、评价等信息在同一数据表中；
        提问：每个一行；
        回答：每个一行；
        评价：每个一行；
        归档：与提问同一行，字段state为归档时间，有则表示已归档；
        修改、删除、收回、退回：在各行的attr字段中用json格式数据保存；

        注：每次增加一行，都要求满足条件。所以，按id顺序可显示整个流程。

1. 问答的状态
    1. 草稿

        用户未提问（qa.rq<'1999-99'）

    1. 退回/收回稿back

        用户收回提问或被退回（qa.rq='1999-99'）

    1. 准提问asking

        用户发出提问（qa.rq<=now - 10分钟）
        从未指定专家回答（没有parentid=qa.id的行）

    1. 新提问newask

        用户发出提问（qa.rq>now - 10分钟）
        从未指定专家回答（没有parentid=qa.id的行）

    1. 刚刚不满意unsatisfing

        最后一行为评价不满意
        评价时间未超过10分钟

    1. 不满意unsatisfy

        最后一行为评价不满意
        在10分钟前已评价

    1. 正在回答toanswer

        有state='待回答'的行

    1. 已全部放弃回答等待触发giveuping

        最后一行是放弃回答（state='放弃回答'）
        没有待评价的回答（state='待回答/已回答'）

    1. 已全部放弃回答可触发giveup

        最后一行是放弃回答（state='放弃回答'）
        没有待评价的回答（state='待回答/已回答'）

    1. 已全部回答等待触发评价answereding

        最后一行是回答或放弃回答（state IN ('放弃回答/已回答')）
        有已完成回答（state='已回答' AND rq>'2015-00'）
        无未完成回答（state='回答' AND rq<'2015-00'）

    1. 已全部回答可触发answered

        最后一行是回答或放弃回答（state IN ('放弃回答/已回答')）
        有已完成回答（state='已回答' AND rq>'2015-00'）
        无未完成回答（state='回答' AND rq<'2015-00'）
        触发后增加一评价行，转到状态“等待评价”

    1. 等待评价toassess

        最后一行是待评价（state='待评价'）

    1. 已评价待触发

        最后一行是已评价（state='评价满意/不满意'）
        用户可修改、撤消评价
        定时器时间到达后，触发：改写已回答行（state='用户满意/不满意'）

    1. 待归档tofile

        含有（state='用户满意'）的行

    1. 已归档filed

        qa.state>'2015-00'

1. 问答列表

    1. 用户

        1. 全部

            按rq排序，这样退/收回稿在最前
            qa.userid = me.id
            所有qa.rq>='1999-99'

        1. 未评价

            按id降序
            qa.userid = me.id
            最后一行是“待评价”

        1. 已评价

            按id降序
            qa.userid = me.id
            有“已评价”的行

    1. 专家

        1. 待回答

            按answer.rq_send降序
            answer.userid = me.id
            answer.state = '待回答'

        1. 我的回答

            按answer.rq降序
            answer.userid = me.id
            answer.state IN ('已回答','放弃回答','用户满意','用户不满意')

    1. 编导

        1. 全部

            按qa.id降序
            所有qa.rq>='1999-99'

        1. 新提问（包括不满意、全部放弃）

            没有任何回答，或
            最后一行是“用户不满意”且过了10分钟，或
            最后一行是“放弃”且没有“待回答”和“已回答”且所有放弃回答已过了10分钟

        1. 未回答

            有“待回答”

        1. 已回答

            最后一行是“待评价”

        1. 已评价

            最后一行是“评价满意”或“评价不满意”

        1. 已归档

            qa.state>'2015-00'

1. 定时器

    1. 触发

        专用定时器每5秒触发一次，目前使用在服务器上运行网页方式触发
        系统中每个提交API查询触发定时器
        每次触发时，判断上次触发时间，未超过5秒的，不处理
        每次触发最多处理问答20个

    1. 读取数据

        读出所有定时器时间小于当前时间的行

    1. 策略

        回答（parentid>0）定时优先处理
        处理后，消去本行定时器。若要后续定时的，可重新设定定时器

    1. 响应

        提问：新提问、时效过半未回答、到时了有回答
        回答：全部回答完成、全部放弃、部分回答
        评价：满意、不满意

    1. 定时器种类

        1. 新提问

        1. 全部放弃回答

            条件：没有未到定时器的，且没有“已回答”，且有“放弃回答”
            本行定时 → 9999-99
            推送消息给编导，重新找专家

        1. 全部回答完成

            条件：没有未到定时器的，且有“已回答”
            定时 → 9999-99
            推送消息给用户已回答完成
            生成评价行

        1. 问答时限到了，有回答

            策略：先处理回答定时器，再处理提问时效
            按照定时器策略，此问答此时不会有其它的定时器
            条件：有“已回答”的行
            提问定时 → 9999-99
            推送消息给用户已回答完成
            生成评价行
            未回答的，改为“回答超时”

        1. 问答时限到了，没回答 ？？？

            推送消息给编导，重新找专家
            推送消息给专家：收到积分

1. 模块各流程

    1. 用户提问：

        1. 发起

            用户发出提问
            积分足够
            提问定时 → 10分钟


        1. 可用操作：

            1. 【提交回答】

                回答定时→ 10分钟
                state = '已回答'

            1. 【专家修改】

                保存修改记录
                回答定时→ 重设10分钟 ？可以不管呀！
            1.【不回答】

                回答后，或最后一次修改未超过10分钟之内有效，或
                尚未提交回答
                state = '放弃问答'
                回答定时→ 10分钟
            1. 【编导修改】

                保存修改记录

            1. 【编导删除】

                [保留]
                保存修改记录

        1. 显示

            专家信息
            回答详情/正在回答/未回答/不回答

        1. 编导操作
            1. 找专家
            1. 匹配知识库

                回答定时→ 10分钟

            1.设置标签

    1. 用户评价：

        1. 发起

            1. 定时器：到了问答时限，有回答
            1. 定时器：10分钟前全部专家已提交，有回答
            1. 发起后，增加一个用户评价行

                state = '未评价'

        1. 消息

            推送给用户
            推送时间[2016-01-05 10:50]
            查看消息时间（用户点击链接）[2016-01-05 10:50]

        1. 进行评价
            1. 满意/不满意

                评价定时→ 10分钟
                state = '评价满意/评价不满意'
                state = '用户满意/用户不满意'
                若满意，积分“预授权”改为“评价满意”扣积分

        1. 显示

            满意/不满意/未评价

    1. 编导归档

        1. 发起
            1. 定时器：10分钟前，用户已评价满意
        1. 操作
            1. 标签
            1. 编导点击归档

1. 界面：

    1. 用户提问：

        1. 显示，同输入

        1. 输入

            图文输入，正文至少3个字符
            时效，1~30天
            出价，10~1000000元
            标签，可选

        1. 操作

            发出提问

    1. 编导界面

        1. 显示
            1. 提问

                用户头像
                用户id
                用户呢称
                图文详情
                所有修改删除记录
                修改和删除按钮

            1. 历史回答（可选，或多次，每次多个回答）

                用户头像
                用户id
                用户呢称
                图文详情
                所有修改删除记录

            1. 用户评价：不满意（可选，或多次，与回答对应）
            1. 回答（可以多个回答）

                用户头像
                用户id
                用户呢称
                图文详情
                本回答所有修改删除记录
                修改和删除按钮

            1. 用户评价：满意/未评价
            1. 归档

                标签输入，用于以后更快匹配
                归档按钮

        1. 退回，在未找专家之前

            图文输入，正文至少3个字符
            退回按钮

        1. 添加标签

            便于匹配知识库

        1. 匹配知识库

            知识库列表，按匹配后的优先权排序
            各行匹配按钮




<style><span class="ui-helper-hidden">{}  li{list-style-type:none;margin:0.5em 0 0.2em 0!important; line-height:1;font-size:0.9em;font-weight:500;} li h1,li h2,li h3,li h4,li h5,li>p:not(:first-child){font-size:14px;font-weight:500; line-height:1.33;margin:0.2em 0 0.2em 2em!important;color:#555;} li>p:first-child{display: inline;} body{counter-reset:h1;} body>ol>li{counter-reset:h2; font-size:26px;color:#00f; font-weight:900;} body>ol>li>ol>li{counter-reset:h3; color:#008;} body>ol>li>ol>li>ol>li{counter-reset:h4; color:#338;} body>ol>li>ol>li>ol>li>ol>li{counter-reset:h5; color:#558;} body>ol>li>ol>li>ol>li>ol>li>ol>li{counter-reset:h6; color:#555;} body>ol>li:before{ counter-increment:h1; content:counter(h1) ". ";} body>ol>li>ol>li:before{ counter-increment:h2; content:counter(h1) "." counter(h2) " ";} body>ol>li>ol>li>ol>li:before{ counter-increment:h3; content:counter(h1) "." counter(h2) "." counter(h3) " ";} body>ol>li>ol>li>ol>li>ol>li:before{ counter-increment:h4; content:counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) " ";} body>ol>li>ol>li>ol>li>ol>li>ol>li:before{ counter-increment:h5; content:counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) " ";}</span></style>




