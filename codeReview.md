###代码规范

####reviewer 检查

	交互回馈 聊天框的形式，每段的reviewCode都有3个状态，reviewing，waiting，
	signed off

####kpi 排行榜，code score

####代码测试

	每次review 少量代码（500行），高频次，review代替一次性很多的代码
	review 加入开发体系，没有review 代码不能开发下去。多人review后才能上传

###具体表现（概念）

	对方框模式，提醒coder和reviewer，进行对代码的修正
	初期目的为了提升代码质量，修正潜在代码，最终成为培养团队新人的工具，形成交流分享，每一个coder<=reviewer

	第一，确定codeRewrite规则
	第二，确定checklist（重点检查的list）e.g. 什么写法性能低下？那个接口慎用？哪些设计方式规避？
	引发内存泄漏？，第一次加入，后续补充，智能新增
	第三，总结，分析，说明问题
	第四，激励机制，kpi？成就感？
	phabricator、gerrit、gitlab
