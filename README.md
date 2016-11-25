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


1. 问答的状态
    1. 草稿

        用户未提问（qa.rq<'1999-99'）

    1. 退回/收回稿back

1. 模块各流程

    1. 用户提问：

        1. 发起

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



<style><span class="ui-helper-hidden">{}  li{list-style-type:none;margin:0.5em 0 0.2em 0!important; line-height:1;font-size:0.9em;font-weight:500;} li h1,li h2,li h3,li h4,li h5,li>p:not(:first-child){font-size:14px;font-weight:500; line-height:1.33;margin:0.2em 0 0.2em 2em!important;color:#555;} li>p:first-child{display: inline;} body{counter-reset:h1;} body>ol>li{counter-reset:h2; font-size:26px;color:#00f; font-weight:900;} body>ol>li>ol>li{counter-reset:h3; color:#008;} body>ol>li>ol>li>ol>li{counter-reset:h4; color:#338;} body>ol>li>ol>li>ol>li>ol>li{counter-reset:h5; color:#558;} body>ol>li>ol>li>ol>li>ol>li>ol>li{counter-reset:h6; color:#555;} body>ol>li:before{ counter-increment:h1; content:counter(h1) ". ";} body>ol>li>ol>li:before{ counter-increment:h2; content:counter(h1) "." counter(h2) " ";} body>ol>li>ol>li>ol>li:before{ counter-increment:h3; content:counter(h1) "." counter(h2) "." counter(h3) " ";} body>ol>li>ol>li>ol>li>ol>li:before{ counter-increment:h4; content:counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) " ";} body>ol>li>ol>li>ol>li>ol>li>ol>li:before{ counter-increment:h5; content:counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) " ";}</span></style>




