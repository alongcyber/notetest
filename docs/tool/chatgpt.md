# chatgpt prompt

撰写Prompt的两个关键原则：
1. 给出清晰和明确的指令    
（1）使用分隔符清晰地将Prompt输入的不同部分分开    
（2）要求结构化的输出，如输出为`json`、`html`、`xml`等格式    
（3）检查内容是否满足任务所需的条件，拒绝无效或非法的内容    
（4）给出几个任务的示例（`Few-shot`）
2. 给模型一些思考的时间    
（1）告知模型完成任务所需的步骤    
（2）指导模型在得出结论之前先自行解决问题具体内容继续往下看使用分隔符清晰地将`Prompt`输入的不同部分分开
`Prompt`通常包含一些指令和待处理的内容，可以用分隔符将指令和内容明确分开.
这里的分隔符可以是
三个引号(""")、
三个反引号(```)、
三条短线(---)、
尖括号(<>)
xml tag(`<tab>`  `</tag>`)等。 

下面举例说明： 
我们可以写一个这样的`Prompt`，来为一段文本写摘要： 
'''
将下面用三个反引号括起来的内容总结为一句话： 
需要总结的文本是： 
\`\`\` 
你应该提供尽可能清晰和具体的指令来表达你想让模型做什么。这将引导模型朝着期望的输出方向发展，并减少收到无关或不正确响应的可能性。不要混淆写一个清晰的提示和写一个简短的提示。在许多情况下，更长的提示提供更多的清晰度和上下文，这可以导致更详细和相关的输出.
 \`\`\` 
 ChatGPT输出如下： 提供清晰具体的指令可以引导模型朝向期望的输出方向发展，减少不正确响应的可能性。提示不要简短，更长的提示提供更多清晰度和上下文，可以导致更详细相关的输出。
'''
要求结构化的输出，如输出为json、html、xml等格式

3. 扮演角色

'''
    你是一名 [XXX领域] 的专家,下面请对我给出的XXXX进行同行评议.

'''


