# AI 可观测 Schema 3.0

State：已评审

**Owner:** @顺岭

**Reviewer:** @古雷 @云净 @润谨 @曜晟 @墨祥 @付琴 @弈安 @春度 @只云 @元先 @霄途

**Revisions:**

+ v2.0 - 2026/1/22：AI 可观测 Schema 2.0 版本发布，修订[记录](https://yuque.alibaba-inc.com/lhubic/gdebsi/xesgguim8ne18sv3)
+ v1.0 - 2025/7/20：AI 可观测 Schema 1.0 版本发布，修订[记录](https://yuque.alibaba-inc.com/lhubic/gdebsi/qt0yvb7mpxdcon3n)

---

### 修订原则

+ **任何条目的新增和修改，每个版本需单独标出并评审，大版本修订发起评审会，小版本修订基于文档评审**
+ **任务条目如果涉及删除，需要在下一个版本中标记deprecated，在下下个版本中才能被删除**

---

## 一、SDK Schema

为Metric/Trace数据埋点时，自定义属性应遵循统一的语义约定标准（Schema），以确保可观测数据在采集上报、存储索引和控制台可视化渲染的全链路一致性和跨平台兼容性。讨论语义约定的总原则为：

1. 优先使用社区语义约定，再考虑使用阿里语义约定，[详情](https://code.alibaba-inc.com/alibaba/ali-semantic-conventions)
2. 若社区语义约定未发布，推荐使用阿里语义约定，若社区语义约定已发布，则推荐使用社区语义约定
3. 若希望新增语义约定，推荐直接提交讨论或提交CR，待评审通过后，各个语言的语义约定会相应更新，新包发布后即可埋点
4. 用户新埋点，推荐使用最新版本的语义约定，技术共建小组会持续推进已埋点服务的字段更新

AI应用的可观测数据格式全面拥抱OpenTelemetry，以确保AI时代下，基于主流开源框架的业务能便捷上报可观测数据。此外，可观测团队会主动跟进核心平台（Whale/idealab/AIGW）和主流Agent的可观测埋点，确保AI 业务核心场景可观测。

### Metric

| **字段名** | **说明** | **类型** | **级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| name | 名称 | STRING | 必须 | gen_ai.client.token.usage |
| description | 描述 | STRING | 必须 | 客户端token用量 |
| unit | 计量单位 | STRING | 必须 | token |
| resources | 资源 | STRING | 必须 | {<br/>"service.name": "agent-server",<br/>"telemetry.sdk.language=go",<br/>"telemetry.sdk.name=opentelemetry",<br/>"telemetry.sdk.version=1.35.0"<br/>} |
| attributes | 属性 | STRING | 必须 | {<br/>"gen_ai.operation.name": "chat"<br/>"gen_ai.token.type": "input",<br/>"gen_ai.request.model": "DeepSeek-R1",<br/>"gen_ai.system": "openai"<br/>} |
| events | 事件 | STRING | 必须 | **** |

#### 推理引擎服务端（INFERENCE）

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| prompt_tokens | 输入token用量，计算口径为记录推理引擎的输入Token用量 | COUNTER | 必须 | token |
| generation_tokens | 输出token用量，计算口径为记录推理引擎的输出Token用量 | COUNTER | 必须 | token |
| cached_tokens | 缓存token用量，计算口径为记录推理引擎的缓存Token用量 | HISTOGRAM | 必须 | token |
| num_requests_running | 模型批量执行的请求数 | GAUGE | 必须 | count |
| num_requests_waiting | 模型等待被执行的请求数 | GAUGE | 必须 | count |
| e2e_request_latency | 端到端请求耗时 | HISTOGRAM | 必须 | ms |

#### LLM

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| gen_ai.client.token.usage | 客户端token用量，计算口径为记录推理引擎的Token用量，含输入和输出 | HISTOGRAM | 必须 | token |
| gen_ai.client.operation.duration | 客户端调用耗时: clientRespTime - clientReqTime | HISTOGRAM | 必须 | ms |
| gen_ai.client.operation | 客户端调用量 | HISTOGRAM | 必须 | count |
| gen_ai.client.time_to_first_token | 客户端首Token响应耗时（TTFT），计算口径为记录生成第一个Token所需的时间 | HISTOGRAM | 必须 | ms |
| gen_ai.client.time_per_output_token | 客户端每token输出平均耗时（TPOT），计算口径为记录生成一个Token所需的平均时间 | HISTOGRAM | 必须 | ms |

##### 属性

| **字段名** | **说明** | **类型** | **级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| gen_ai.operation.name | GenAi调用类型 | STRING | 必须 | chat、generate_content、text_completion |
| gen_ai.request.model | 请求模型名 | STRING | 必须 | gpt-4、claude-3.7、deepseek-r1 |
| gen_ai.token.type | token类型（仅适用于token.usage） | STRING | 必须 | input、output、cached |
| error.type | 错误类型 | STRING | 必须 | java.net.UnknownHostException; server_certificate_invalid; 500 |

#### MCP

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| gen_ai.mcp.tool_call.request | 服务端请求量 | COUNTER | 必须 | count |
| gen_ai.mcp.tool_call.response | 服务端响应量 | COUNTER | 必须 | count |
| gen_ai.mcp.tool_call.response_duration | 服务端请求耗时：serverRespTime - serverReqTime | HISTOGRAM | 必须 | ms |
| mcp.client.operation.duration | 客户端调用耗时：clientRespTime - clientReqTime | HISTOGRAM | 必须 | ms |

##### 属性

| **字段名** | **说明** | **类型** | **级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| gen_ai.mcp.method_name | MCP方法名称 | STRING | 必须 | tools/call、tools/list、prompt/list |
| gen_ai.mcp.tool_name | 工具名称 | STRING | 必须 | get-weather、execute_command |
| error.type | 主要记录JSON-RPC错误类型；若JSON-RPC成功，但结果失败，也需记录 | STRING | 必须 | java.net.UnknownHostException; server_certificate_invalid; 500 |

#### AGENT

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| gen_ai.agent.duration | Agent客户端调用耗时：agentRespTime - agentReqTime | HISTOGRAM | 必须 | ms |

##### 属性

| **字段名** | **说明** | **类型** | **属性要求级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| gen_ai.operation.name | GenAi调用类型 | STRING | 必须 | create_agent、execute_tool、invoke_agent |
| error.type | 错误类型 | STRING | 必须 | timeout; server_certificate_invalid; 500 |
| gen_ai.agent.name | Agent名称 | STRING | 必须 | sre_agent |

#### WORKFLOW

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| gen_ai.workflow.duration | WORKFLOW调用耗时：workflowRespTime - workflowReqTime | HISTOGRAM | 必须 | ms |

##### 属性

| **字段名** | **说明** | **类型** | **属性要求级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| gen_ai.workflow.name | 工作流名称 | STRING | 必须 | multi_agent_rag |
| error.type | 错误类型 | STRING | 必须 | timeout; server_certificate_invalid; 500 |

#### TOOL

| **指标名** | **计算口径说明** | **类型** | **级别** | **计量单位** |
| --- | --- | --- | --- | --- |
| gen_ai.tool.duration | 工具执行耗时：toolRespTime - toolReqTime | HISTOGRAM | 必须 | ms |

##### 属性

| **字段名** | **说明** | **类型** | **属性要求级别** | **数据示例** |
| --- | --- | --- | --- | --- |
| gen_ai.tool.name | 工具名称 | STRING | 必须 | resarch_step、memory_retrieval_step |
| error.type | 错误类型 | STRING | 必须 | timeout; server_certificate_invalid; 500 |


#### SKILL

| **指标名**               | **计算口径说明** | **类型** | **级别** | **计量单位** |
|-----------------------|----------| --- | --- | --- |
| gen_ai.skill.duration | skill执行耗时| HISTOGRAM | 必须 | ms |

##### 属性

| **字段名**           | **说明**  | **类型** | **属性要求级别** | **数据示例** |
|-------------------|---------| --- | --- | --- |
| gen_ai.skill.name | skill名称 | STRING | 必须 | alert-analysis |
| error.type    | 错误类型    | STRING | 必须 | timeout; server_certificate_invalid; 500 |

### Span

Span数据格式全面拥抱OpenTelemetry，以确保AI时代下，基于开源框架和平台的业务能快捷上报可观测数据。此外，可观测团队会主动跟进核心平台（Whale/idealab/AIGW）和主流Agent的可观测埋点，确保AI 核心链路联通。最后，通过双协议转换方式，确保最小改造成本的鹰眼Span数据兼容。

| **字段名** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- |
| name | STRING | 必须 | + AGENT：名称格式为：操作名 Agent名，如：invoke_agent fiction_writer<br/>+ WORKFLOW：名称格式为：操作名 Workflow名，如：workflow multi_agent_rag<br/>+ LLM：名称格式为：操作名 模型名，如：chat Qwen3-Coder-Plus<br/>+ TOOL：名称格式为：操作名 工具名，如：execute_tool getWeather<br/>+ MCP：名称格式为： MCP方法名 工具名，如：TOOL/CALL Flights<br/>+ RETRIEVAL：名称格式为：操作名 文档ID，如：retrieval H7STPQYOND<br/>+ EMBEDDING：名称格式为： 操作名 模型名，如：embedding gpt-4 | 等值、前后缀 |
| trace_id | STRING | 必须 | ac12db2217495584652395793d0226 | 等值 |
| start_time | INT64 | 必须 | 1750229442704597 | 范围（时间） |
| duration | INT32 | 必须 | 17621640 | 范围 |
| status_code | INT32 | 必须 | + 0：Unset 表示默认状态，该操作已完成且没有错误<br/>+ 1：OK 表示操作已完成，并经过验证<br/>+ 2：Error 操作包含错误 | 等值 |
| host | STRING | 必须 | 11.23.200.106 | 等值 |
| resources | STRING | 必须 | {<br/>"container.id=5",<br/>"host.name=aone-agentic-session7aa312efa7e743739b-pod","net.host.ip=29.241.126.163","os.description=Ubuntu 24.04.2 LTS (Noble Numbat) (Linux aone-agentic-session7aa312efa7e743739b-pod 5.10.112-005.ali5000.al8.x86_64 #1 SMP Tue Jun 28 10:43:38 CST 2022 x86_64)",<br/>"os.type=linux",<br/>"process.executable.name=aone-copilot-agent",<br/>"process.executable.path=/usr/bin/aone-copilot-agent",<br/>"process.owner=admin",<br/>"process.pid=136",<br/>"process.runtime.description=go version go1.24.1 linux/amd64",<br/>"process.runtime.name=go",<br/>"process.runtime.version=go1.24.1",<br/>"service.name=agent-server",<br/>"telemetry.sdk.language=go",<br/>"telemetry.sdk.name=opentelemetry",<br/>"telemetry.sdk.version=1.35.0"<br/>} | 仅展示 |
| id | STRING | 必须 | 24c04018f9c7de67 | 仅展示 |
| parent_id | STRING | 必须 | 55db73b8bda8be50 | 仅展示 |
| kind | INT32 | 必须 | 1 | 等值 |
| attributes | STRING | 必须 | {<br/>"gen_ai.span_kind_name=LLM",<br/>"gen_ai.request.model=qwen2.5_72b_agent",<br/>"gen_ai.input.messages=[{\"content\":\" user task编写一个静态页面，使用HTML5和CSS3实现，页面居中展示纯文本 'hello world'。 /task output一个包含HTML和CSS文件的静态页面，能够在一个标准的现代浏览器中打开，显示居中的 'hello world' 文本。 /output notes1. 使用HTML5文档结构，确保页面兼容现代浏览器。\\n2. 使用CSS3的Flexbox或Grid布局实现文本的居中显示。\\n3. 确保页面简洁，没有多余的样式或脚本。 /notes /user\",\"role\":\"user\"},{\"content\":\" task_context\\n  user_request user task编写一个静态页面，使用HTML5和CSS3实现，页面居中展示纯文本 'hello world'。 /task output一个包含HTML和CSS文件的静态页面，能够在一个标准的现代浏览器中打开，显示居中的 'hello world' 文本。 /output notes1. 使用HTML5文档结构，确保页面兼容现代浏览器。\\n2. 使用CSS3的Flexbox或Grid布局实现文本的居中显示。\\n3. 确保页面简洁，没有多余的样式或脚本。 /notes /user /user_request\\n  current_task\\n  namecreate_html_structure /name\\n  description\\nCreate an HTML5 file with proper document structure to display 'hello world' text, ensuring it follows modern browser compatibility standards.\\n /description\\n  task_completion_criteria\\n completion_criteriaHTML5 document with proper DOCTYPE declaration is created /completion_criteria\\n completion_criteriaThe page contains the text 'hello world' /completion_criteria\\n completion_criteriaProper meta tags for charset and viewport are included /completion_criteria\\n completion_criteriaHTML file is valid and can be opened in a browser /completion_criteria\\n /task_completion_criteria\\n  how_tos_important\\n importantCreate an index.html file with HTML5 doctype declaration /important\\n importantAdd proper head section with meta tags for charset and viewport /important\\n importantInclude a title for the page /important\\n importantCreate a simple body with a div or paragraph containing 'hello world' /important\\n importantOpen the file in a browser to verify basic structure /important\\n /how_tos_important\\n  knowledges\\n /knowledges\\n /current_task\\n  plan\\n001 create_html_structure [CURRENT]\\n002 implement_css_centering\\n /plan\\n  task_rule\\n1.You one reply must use only one tools\\n2.you should follow `current_task`\\n3.Do not do over your current step \\n4.you must step by step \\n /task_rule\\n /task_context\",\"role\":\"assistant\"},{\"content\":\" task_completion task=\\\"创建HTML5结构\\\" timestamp=\\\"1750229414\\\"\\n ![CDATA[完成了基本HTML5文档结构的创建，包含适当的DOCTYPE声明、meta标签和'hello world'文本。文件已保存为index.html，满足了基本的HTML5标准和文档结构要求。]]\\n /task_completion\",\"role\":\"user\"},{\"content\":\" task_context\\n  user_request user task编写一个静态页面，使用HTML5和CSS3实现，页面居中展示纯文本 'hello world'。 /task output一个包含HTML和CSS文件的静态页面，能够在一个标准的现代浏览器中打开，显示居中的 'hello world' 文本。 /output notes1. 使用HTML5文档结构，确保页面兼容现代浏览器。\\n2. 使用CSS3的Flexbox或Grid布局实现文本的居中显示。\\n3. 确保页面简洁，没有多余的样式或脚本。 /notes /user /user_request\\n  current_task\\n  nameimplement_css_centering /name\\n  description\\nCreate a CSS file and implement modern centering techniques using Flexbox or Grid to position the 'hello world' text in the center of the page both horizontally and vertically.\\n /description\\n  task_completion_criteria\\n completion_criteriaCSS file is created and linked to the HTML file /completion_criteria\\n completion_criteriaText is centered both horizontally and vertically in the viewport /completion_criteria\\n completion_criteriaLayout uses either Flexbox or Grid as specified in requirements /completion_criteria\\n completion_criteriaPage maintains centering when browser window is resized /completion_criteria\\n completion_criteriaFinal page matches the requirement of showing only centered 'hello world' text /completion_criteria\\n /task_completion_criteria\\n  how_tos_important\\n importantCreate a styles.css file /important\\n importantLink the CSS file in the HTML head section /important\\n importantImplement CSS to make the body a flex or grid container /important\\n importantSet height to 100vh to use the full viewport height /important\\n importantConfigure flex/grid properties to center content both horizontally and vertically /important\\n importantAdd minimal additional styling to keep the design clean /important\\n importantPreview the page in a browser to verify proper centering /important\\n /how_tos_important\\n  knowledges\\n /knowledges\\n /current_task\\n  plan\\n=== NEW PROCEDURE TO FOLLOW ===\\n001 create_html_structure [complete]\\n002 implement_css_centering - [current]\\n=== END NEW PROCEDURE TO FOLLOW ===\\n /plan\\n  task_rule\\n1.You one reply must use only one tools\\n2.you should follow `current_task`\\n3.Do not do over your current step \\n4.you must step by step \\n /task_rule\\n /task_context\",\"role\":\"assistant\"},{\"content\":\" task_completion task=\\\"实现CSS居中效果\\\" timestamp=\\\"1750229460\\\"\\n ![CDATA[成功完成了居中布局的实现。创建了styles.css文件，使用Flexbox布局技术实现了'hello world'文本在页面中的水平和垂直居中。主要实现了以下功能：\\n1. 使用display: flex、justify-content: center和align-items: center实现完美居中\\n2. 设置min-height: 100vh确保全屏显示和居中效果\\n3. 添加了基本的样式设置（字体、颜色、大小）使页面美观简洁\\n4. 在HTML文件中添加了CSS链接\\n\\n页面现在可以在任何现代浏览器中显示，并会在各种屏幕尺寸下保持文本居中。]]\\n /task_completion\",\"role\":\"user\"},{\"content\":\"We are working on the following task:\\n\\n\\nWe have completed the task.\\n\\nThe above messages contain the conversation that took place to complete the task.\\n\\nBased on the information gathered, provide the final answer to the original request.\\nThe answer should be phrased as if you were speaking to the user.\",\"role\":\"user\"}]\",<br/>\"gen_ai.usage.input_tokens=2278\",<br/>\"gen_ai.usage.output_tokens=576\",<br/>\"gen_ai.output.messages={\"id\":115993120,\"context\":\"您好！我们已经完成了您的请求，创建了一个静态页面，使用HTML5和CSS3实现了页面居中展示纯文本 'hello world'。以下是具体的实现：\\n\\n### HTML (index.html)\\n```html\\n !DOCTYPE html\\n html lang=\\\"en\\\"\\n head\\n  meta charset=\\\"UTF-8\\\"\\n  meta name=\\\"viewport\\\" content=\\\"width=device-width, initial-scale=1.0\\\"\\n  titleHello World /title\\n  link rel=\\\"stylesheet\\\" href=\\\"styles.css\\\"\\n /head\\n body\\n  div class=\\\"centered-text\\\"\\n hello world\\n  /div\\n /body\\n /html\\n```\\n\\n### CSS (styles.css)\\n```css\\nbody, html {\\n height: 100%;\\n margin: 0;\\n display: flex;\\n justify-content: center;\\n align-items: center;\\n font-family: Arial, sans-serif;\\n background-color: #f0f0f0;\\n}\\n\\n.centered-text {\\n font-size: 24px;\\n color: #333;\\n}\\n```\\n\\n### 说明\\n1. **HTML文件** (`index.html`): 使用了HTML5的文档结构，包含必要的meta标签和一个链接到CSS文件的` link`标签。\\n2. **CSS文件** (`styles.css`): 使用了Flexbox布局技术，通过设置`display: flex`、`justify-content: center`和`align-items: center`实现了文本的水平和垂直居中。同时设置了基本的字体、颜色和背景样式，使页面简洁美观。\\n\\n您可以将这两个文件保存到您的项目目录中，并在现代浏览器中打开`index.html`文件，查看效果。希望这能满足您的需求！如果有任何其他问题或需要进一步的帮助，请随时告诉我。\"}\"} | 仅展示 |
| links | String | 必须 | | |

#### COMMON

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.span_kind_name | AI的调用类型 | STRING | 必须 | + AGENT：AI Agent 应用，包含一组LLM调用和工具调用的根Span（最外层容器）<br/>+ CHAT: AI CHAT 应用，包含一组LLM调用和工具调用的根Span（最外层容器）<br/>+ WORKFLOW: AI WORKFLOW应用，包含一组LLM调用和工具调用的跟Span（最外层容器）<br/>+ CHAIN：AI 处理链，用于记录应用步骤的节点和步骤间的连接点<br/>+ LLM：大模型服务，用于记录调用LLM生成文本补全或进行对话交互<br/>+ TOOL：工具调用，用于记录代表LLM执行的API或函数调用<br/>+ MCP：MCP 调用<br/>+ RETRIEVAL：检索调用，用于记录从数据存储中查询上下文信息的检索操作<br/>+ EMBEDDING：嵌入编码调用，对非结构化数据（如文本/图像）的向量化表示<br/>+ RERANK：重排序调用，用于记录基于相关性文档进行重新排序的操作<br/>+ GUARDRAIL：安全防护调用<br/>+ EVALUATOR：评估调用 | 等值 |
| error.type | 报错类型 | STRING | 必须 | timeout; java.net.UnknownHostException; server_certificate_invalid; 500 | 等值，模糊查询 |

#### CUSTOM

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.messages_ref | STRING | 必须 | refUrl链接 | http://{API_ENDPOINT}/api/ref/{id} | 仅展示 |
| alibaba.user.data | STRING | 必须 | 用户自定义数据 | {<br/>   \"prompt_cache_key\": 100,<br/>  \"reasoning\": {<br/>\"effort\": high,<br/>\"summary\": auto<br/>   }<br/>} | 仅展示 |

#### LLM

| **字段名** | **说明** | **类型** | **属性级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.request.model | 模型名称 | STRING | 必须 | Qwen-2.5 | 等值 |
| gen_ai.usage.input_tokens | 用量统计输入Token 量 | INT32 | 必须 | 188 | 等值 |
| gen_ai.usage.output_tokens | 用量统计之输出 Token 量 | INT32 | 必须 | 376 | 范围 |
| gen_ai.usage.cache_creation.input_tokens | 写入缓存的输入令牌数量 | INT32 | 必须 | 50 | 仅展示 |
| gen_ai.usage.cache_read.input_tokens | 从缓存中读出的输入令牌数量 | INT32 | 必须 | 100 | 仅展示 |
| gen_ai.client.time_to_first_token | 流式调用首令牌耗时 | INT32 | 必须 | 127ms | 仅展示 |
| gen_ai.client.time_per_output_token | 流式调用平均令牌耗时 | INT32 | 必须 | 3ms | 仅展示 |
| gen_ai.conversation.id | 会话的唯一标识符 | STRING | 必须 | 499F3AD1-89AA-4E8D-BB40-D39FEA215AA9 | 仅展示 |
| gen_ai.tool.definitions | 模型可用的源系统工具定义列表 | STRING | 必须 | [{"type": "function","name": "get_current_weather","description”: “Get the current weather in a given location","parameters": {"type”: “object","properties": {"location": {"type”: “string","description": "city and state, e.g. San Francisco, CA"},"unit": {"type": "string","enum": ["celsius","fahrenheit"]}},"required": ["location","unit"]}}] | 等值 |
| gen_ai.input.messages | 大模型输入 | STRING | 必须 | [{\"role\":\"system\",\"content\":\"You are a helpful assistant.\"},{\"role\":\"user\",\"content\":\"给顺岭买一个苹果，并返回结果\"}] | 仅展示、模糊查询 |
| gen_ai.output.messages | 大模型输出 | STRING | 必须 | [{\"role\":\"system\",\"content\":\"You are a helpful assistant.\"},{\"role\":\"user\",\"content\":\"给顺岭买一个苹果，并返回结果\"}] | 仅展示、模糊查询 |

#### MCP

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| mcp.method.name | MCP方法名 | STRING | 必须 | tools/call | 仅展示 |
| gen_ai.tool.name | MCP工具名 | STRING | 必须 | mstest_buysomethingforsomeone | 仅展示 |
| mcp.session.id | MCP session会话 | STRING | 必须 | e37e73f8-9023-4ddf-b46b-1219d4e04c91 | 仅展示 |
| gen_ai.tool.call.arguments | 工具调用请求 | STRING | 必须【Playground】 | {\"method\":\"tools/call\",\"params\":{\"name\":\"mstest_buysomethingforsomeone\",\"arguments\":{\"someoneName\":\"顺岭\",\"somethingNames\":[{\"name\":\"苹果\",\"count\":1}]}},\"jsonrpc\":\"2.0\",\"id\":3} | 仅展示 |
| gen_ai.tool.call.result | 工具调用响应 | STRING | 必须【Evaluator】 | {\"jsonrpc\":\"2.0\",\"id\":3,\"result\":{\"content\":[{\"type\":\"text\",\"text\":\"buy [{\\\"count\\\":1,\\\"name\\\":\\\"苹果\\\"}] for 顺岭 success\"}]}} | 仅展示 |

#### AGENT

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.operation.name | 操作名称 | STRING | 必须 | create_agent、invoke_agent | 等值、模糊查询 |
| gen_ai.agent.id | 应用内全局唯一的AgentId | STRING | 必须 | asst_5j66UpCpwteGg4YSxUnt7lPY | 仅展示 |
| gen_ai.agent.name | Agent名称 | STRING | 必须 | Math Tutor | 仅展示 |
| gen_ai.input.messages | Agent输入信息 | STRING | 必须 | [   {     \"role\": \"user\",     \"parts\": [       {         \"type\": \"text\",         \"content\": \"Weather in Paris?\"       }     ]   },   {     \"role\": \"assistant\",     \"parts\": [       {         \"type\": \"tool_call\",         \"id\": \"call_VSPygqKTWdrhaFErNvMV18Yl\",         \"name\": \"get_weather\",         \"arguments\": {           \"location\": \"Paris\"         }       }     ]   } ] | 仅展示 |
| gen_ai.output.messages | Agent输出信息 | STRING | 必须 | [   {     \"role\": \"assistant\",     \"parts\": [       {         \"type\": \"text\",         \"content\": \"The weather in Paris is currently rainy with a temperature of 57°F.\"       }     ],     \"finish_reason\": \"stop\"   } ] | 仅展示 |

#### WORKFLOW

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.workflow.name | 工作流名称 | STRING | 必须 | customer_support_flow、multi_agent_rag | 等值 |
| gen_ai.input.messages | 工作流输入 | STRING | 必须 | {   \"inputs\": {     \"messages\": [{       \"lc\": 1,       \"type\": \"constructor\",       \"id\": [         \"langchain\",         \"schema\",         \"messages\",         \"HumanMessage\"       ],\"kwargs\": {         \"content\": \"共有1台机器[tddl写成功率]异常，摘要：\\n* [33.6.228.229] tddl写成功率 [当前值为: 93.023%] 最近5分钟连续小于95%\\n##\\nalarmId:5f2e2bafdd6b42fca414fdbef01dc384;filterOp:<;filterThreshold:95;metric_name:middleware_tddl_write_success_rate;app_name:xixi-ikb;ip:33.6.228.229;alarm_id:5f2e2bafdd6b42fca414fdbef01dc384;start_time:2025-12-22 20:45:00;end_time:2025-12-22 20:55:00;\",         \"type\": \"human\"       }}]},   \"tags\": [],   \"metadata\": {     \"thread_id\": \"42014555-1096-4f16-820b-42d643aaea7c\",     \"recursion_limit\": 50   },   \"kwargs\": {     \"name\": \"LangGraph\"   }} | 仅展示 |
| gen_ai.output.messages | 工作流输出 | STRING | 必须 | {   \"outputs\": {     \"messages\": [{         \"lc\": 1,         \"type\": \"constructor\",         \"id\": [\"langchain\",\"schema\",\"messages\",           \"HumanMessage\"],         \"kwargs\": {           \"content\": \"共有1台机器[tddl写成功率]异常，摘要：\\n* [33.6.228.229] tddl写成功率 [当前值为: 93.023%] 最近5分钟连续小于95%\\n##\\nalarmId:5f2e2bafdd6b42fca414fdbef01dc384;filterOp:<;filterThreshold:95;metric_name:middleware_tddl_write_success_rate;app_name:xixi-ikb;ip:33.6.228.229;alarm_id:5f2e2bafdd6b42fca414fdbef01dc384;start_time:2025-12-22 20:45:00;end_time:2025-12-22 20:55:00;\",           \"type\": \"human\",           \"id\": \"094a1397-0f5c-425a-b9d5-d5a9775acdd6\"     }},{         \"lc\": 1,         \"type\": \"constructor\",         \"id\": [\"langchain\",\"schema\",\"messages\",\"AIMessage\"],         \"kwargs\": {           \"content\": \"我是RDS诊断专家，正在分析您遇到的tddl写成功率异常问题。\\n\\n根据您提供的信息，我看到：\\n- 机器IP：33.6.228.229\\n- 应用名：xixi-ikb\\n- tddl写成功率从正常值下降到93.023%\\n- 异常时间：2025-12-22 20:45:00至20:55:00\\n- 连续5分钟小于95%的阈值\\n\\n您提到这个问题可能与RDS相关，但当前报警是tddl写成功率异常，而非直接的RDS指标异常。为了准确诊断RDS是否存在问题，我需要获取更多关于该时间段内的RDS状态信息，特别是查看是否存在慢查询影响了数据库性能。\\n\\n不过，我注意到您提供的报警信息中并没有包含RDS实例ID (以rm-开头)，这是使用RDS慢查询工具所必需的参数。为了进行进一步的诊断，我需要您提供相关的RDS实例ID。\\n\\n如果您不清楚RDS实例ID，可以检查：\\n1. 应用配置文件中的数据库连接信息\\n2. 云控制台中该应用关联的RDS实例\\n3. 监控系统中相关数据库实例的连接信息\\n\\n一旦获得RDS实例ID，我就能帮您分析该时段是否存在慢查询导致的性能问题。\",           \"additional_kwargs\": {             \"refusal\": null           },           \"response_metadata\": {             \"token_usage\": {               \"completion_tokens\": 304,               \"prompt_tokens\": 632,               \"total_tokens\": 936,               \"completion_tokens_details\": null,               \"prompt_tokens_details\": null,               \"cacheReadInputTokensCompatible\": 0            },             \"model_provider\": \"openai\",             \"model_name\": null,             \"system_fingerprint\": null,             \"id\": \"chat-\",             \"finish_reason\": \"stop\",             \"logprobs\": null           },           \"type\": \"ai\",           \"id\": \"lc_run--eb4a6b8f-f575-4416-87ca-aafd3c1fd934-0\",           \"usage_metadata\": {             \"input_tokens\": 632,             \"output_tokens\": 304,             \"total_tokens\": 936,             \"input_token_details\": {},             \"output_token_details\": {}           },           \"tool_calls\": [],           \"invalid_tool_calls\": []   }}]},   \"kwargs\": {     \"tags\": []   }   } | 仅展示 |

#### SKILL

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.operation.name | 操作名称 | STRING | 必须 | invoke_skill | 等值 |
| gen_ai.skill.id | 技能调用唯一ID | STRING | 必须 | skill_29bbe8a7 | 仅展示 |
| gen_ai.skill.name | 技能名称 | STRING | 必须 | code_review、change_workitem | 等值、模糊查询 |
| gen_ai.skill.version | 技能版本号 | STRING | 必须 | 0.2.0, v2.1.0 | 仅展示 |
| gen_ai.skill.description | 技能功能描述 | STRING | 必须 | 执行在 github 中查询最近工作项、按迭代筛选工作项并查看详情。Use when the user wants to search github workitems, filter by sprint, view recent Req/Bug/Task, or query workitem detail. | 仅展示 |
| gen_ai.input.messages | 技能输入参数 | STRING | 必须 | {"intent": "review_code"} | 仅展示 |
| gen_ai.output.messages | 技能执行结果状态 | STRING | 必须 | success, fallback_to_human | 仅展示 |

#### TOOL

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.operation.name | 操作名称 | STRING | 必须 | execute_tool | 仅展示 |
| gen_ai.tool.call.id | 工具调用唯一ID | STRING | 必须 | call_mszuSIzqtI65i1wAUOE8w5H4 | 等值 |
| gen_ai.tool.name | 工具名 | STRING | 必须 | Flights | 等值、模糊查询 |
| gen_ai.tool.type | 工具类型 | STRING | 必须 | function、extension、datastore | 仅展示 |
| gen_ai.tool.call.arguments | 工具调用入参 | STRING | 必须 | {    \"location\": \"San Francisco\",    \"date\": \"2025-10-01\"} | 仅展示 |
| gen_ai.tool.call.result | 工具调用返回值 | STRING | 必须 | {   \"temperature_range\": {       \"high\": 75,       \"low\": 60   },   \"conditions\": \"sunny\"} | 仅展示 |

#### EMBEDDING

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.embedding.text | 嵌入文本 | STRING | 必须 | hello world | 仅展示 |
| gen_ai.embedding.vector | 嵌入向量 | STRING | 必须 | [      0.007293701171875,      0.01080322265625,      0.0040283203125,      -0.025390625] | 仅展示 |

#### RETRIEVAL

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.retrieval.documents | 检索文本列表 | STRING | 必须 | [{\"id\":\"doc_123\",\"score\": 0.95,\"content\": \"Paris is the capital\"}] | 仅展示 |
| gen_ai.retrieval.query.text | 检索内容 | STRING | 必须 | What is the capital of France? | 仅展示 |

#### RERANK

| **字段名** | **说明** | **类型** | **属性要求级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.rerank.input_documents | 重排序输入文本列表 | STRING | 必须 | [\"文本排序模型广泛用于搜索引擎和推荐系统中，它们根据文本相关性对候选文本进行排序\",      \"量子计算是计算科学的一个前沿领域\",      \"预训练语言模型的发展给文本排序模型带来了新的进展\"] | 仅展示 |
| gen_ai.rerank.output_documents | 重排序输出文本列表 | STRING | 必须 | {\"index\": 0,      \"relevance_score\": 0.7314485774089865,      \"document\": {            \"text\": \"文本排序模型广泛用于搜索引擎和推荐系统中，它们根据文本相关性对候选文本进行排序\"       }   },   {      \"index\": 2,      \"relevance_score\": 0.5831720487049298,      \"document\": {             \"text\": \"预训练语言模型的发展给文本排序模型带来了新的进展\"}   },   {       \"index\": 1,       \"relevance_score\": 0.04973238644524712,       \"document\": {                \"text\": \"量子计算是计算科学的一个前沿领域\"}   } | 仅展示 |
| gen_ai.rerank.query | 重排序查询语句 | STRING | 必须 | 什么是文本排序模型？ | 仅展示 |
| gen_ai.rerank.top_k | 重排序的topK入参 | DOUBLE | 必须 | 1.0 | 仅展示 |

#### EVALUATOR

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.evaluator.name | 评估器名称 | STRING | 必须 | hallucination_evaluator | 仅展示 |
| gen_ai.evaluator.score | 评估分数 | DOUBLE | 必须 | 5 | 仅展示 |

#### A2A

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| gen_ai.a2a.rpc.method | A2A JSON-RPC方法 | STRING | 必须 | message/send、tasks/get | 仅展示 |
| gen_ai.a2a.task.id | A2A 任务ID | INT32 | 必须 | wJU4XHw4 | 仅展示 |
| gen_ai.a2a.task.state | A2A 任务状态 | STRING | 必须 | submitted、working、input-required、completed、failed、canceled | 仅展示 |
| gen_ai.a2a.protocol.version | A2A 协议版本 | STRING | 必须 | 0.1 | 仅展示 |
| gen_ai.a2a.streaming | 标记响应是否使用SSE | BOOLEAN | 必须 | true / false | 仅展示 |
| gen_ai.a2a.request_body | 请求体 | STRING | 必须 | {   \"jsonrpc\": \"2.0\",} | 仅展示 |
| gen_ai.a2a.response_body | 响应体 | STRING | 必须 | {   \"id\": 1} | 仅展示 |
| gen_ai.a2a.response_status_code | 响应状态码 | STRING | 必须 | 200、400、404 | 仅展示 |

### RESOURCE

Resource用于记录可观测数据来源的实体信息，例如：容器POD、手机端APP、网页SPA码等。所有可观测数据都需要关联一个资源，从而实现同一资源的数据上下文关联。资源包含两个部分：

+ 实体的唯一标识，一般为service.name，对应阿里集团的统一身份概念
+ 有助于用户确定该实体在基础设施中的位置信息，例如：环境、机房、单元等

| **字段名** | **说明** | **类型** | **级别** | **数据示例** | **查询方式** |
| --- | --- | --- | --- | --- | --- |
| host.ip | 当前IP | STRING | 必须 | 127.0.0.0.1 | 等值 |
| service.name | 当前统一身份名，支持Aone、TPP等 | STRING | 必须 | whale-wave | 等值 |
| alibaba.base.env | 当前环境 | STRING | 必须 | DEV,PRE | 等值 |
| alibaba.base.idc | 当前机房 | STRING | 必须 | NA610 | 等值 |
| experiement.id | STRING | 必须 | RL实验ID唯一标识 | test-openclaw-metrics-b | 仅展示 | 
| group.id | STRING | 必须 | RL分组ID唯一标识 | default-group | 仅展示 |
| job.id | STRING | 必须 | RL任务ID唯一标识 | job-test-openclaw-0324 | 仅展示 |
| instance.id | STRING | 必须 | RL实例ID唯一标识 | swe-env.v1.tobymao__sqlglot-4451.composite__f5umdxdw | 仅展示 | 

---

## 二、Uniagent schema

### Span

```protobuf
syntax = "proto2";
package protocol.holo;
option java_package="com.alibaba.search.trace.protocol";
option java_outer_classname = "SpanProto";
option go_package="holoproto";

message SpanLink {
    optional string trace_id = 1;
    optional string span_id = 2;
    optional string trace_state = 3;
    repeated string attributes = 4;
}

message SpanLinks {
    repeated SpanLink links = 1;
}

message SpanEvent {
    optional int64 time_unix_micro = 1;
    optional string name = 2;
    repeated string attributes = 3;
}

message SpanEvents {
    repeated SpanEvent events = 1;
}

message Span {
    optional string host = 1;
    repeated string resources = 2;
    optional string name = 3;
    optional string id = 4;
    optional int32 kind = 5;
    optional string trace_id = 6;
    optional string parent_id = 7;
    repeated string attributes = 8;
    optional bytes links = 9;
    optional bytes events = 10;
    optional int64 start_time = 11;
    optional int64 duration = 12;
    optional int32 status_code = 13;
    optional string status_message = 14;
}


message Batch {
    repeated Span spans = 1;
}
```

代码：[https://code.alibaba-inc.com/monitor_service/uniagent/blob/master/core/component/exporter/holoexporter/bhclient/proto/span.proto](https://code.alibaba-inc.com/monitor_service/uniagent/blob/master/core/component/exporter/holoexporter/bhclient/proto/span.proto)

### Metric

```java
option csharp_namespace = "OpenTelemetry.Proto.Metrics.V1";
option java_multiple_files = true;
option java_package = "io.opentelemetry.proto.metrics.v1";
option java_outer_classname = "MetricsProto";
option go_package = "go.opentelemetry.io/proto/otlp/metrics/v1";

message ScopeMetrics {
  opentelemetry.proto.common.v1.InstrumentationScope scope = 1;
  repeated Metric metrics = 2;
  string schema_url = 3;
}

message Metric {
  reserved 4, 6, 8;
    
  string name = 1;
  string description = 2;
  string unit = 3;
  oneof data {
    Gauge gauge = 5;
    Sum sum = 7;
    Histogram histogram = 9;
    ExponentialHistogram exponential_histogram = 10;
    Summary summary = 11;
    Statistic statistic = 12;
  }
}

message Statistic {
  repeated StatisticDataPoint data_points = 1;
}

message Gauge {
  repeated NumberDataPoint data_points = 1;
}

message Sum {
  repeated NumberDataPoint data_points = 1;
  AggregationTemporality aggregation_temporality = 2;
  bool is_monotonic = 3;
}

message Histogram {
  repeated HistogramDataPoint data_points = 1;
  AggregationTemporality aggregation_temporality = 2;
}

message StatisticDataPoint {
  repeated opentelemetry.proto.common.v1.KeyValue attributes = 7;

  fixed64 start_time_unix_nano = 2;
  fixed64 time_unix_nano = 3;
  repeated ValueType values = 4;
  string extend = 5;
  repeated Exemplar exemplars = 11;
  uint32 flags = 12;
}

message NumberDataPoint {
  reserved 1;

  repeated opentelemetry.proto.common.v1.KeyValue attributes = 7;
  fixed64 start_time_unix_nano = 2;
  fixed64 time_unix_nano = 3;
  oneof value {
    double as_double = 4;
    sfixed64 as_int = 6;
  }
  repeated Exemplar exemplars = 5;
  uint32 flags = 8;
}

message HistogramDataPoint {
  reserved 1;

  repeated opentelemetry.proto.common.v1.KeyValue attributes = 9;
  fixed64 start_time_unix_nano = 2;
  fixed64 time_unix_nano = 3;
  fixed64 count = 4;
  double sum = 5;
  repeated fixed64 bucket_counts = 6;
  repeated double explicit_bounds = 7;
  repeated Exemplar exemplars = 8;
  uint32 flags = 10;
  double min = 11;
  double max = 12;
}
```

代码: [https://code.alibaba-inc.com/monitor_service/uniagent/blob/master/core/log_pb/opentelemetry/proto/metrics/v1/metrics.proto](https://code.alibaba-inc.com/monitor_service/uniagent/blob/master/core/log_pb/opentelemetry/proto/metrics/v1/metrics.proto)

---

## 三、Swift schema

含鹰眼和Opentelemetry字段的宽表

```json
[
  {
    "name": "name",
    "type": "string"
  },
  {
    "name": "trace_id",
    "type": "string"
  },
  {
    "name": "start_time",
    "type": "string"
  },
  {
    "name": "duration",
    "type": "string"
  },
  {
    "name": "status_code",
    "type": "string"
  },
  {
    "name": "host",
    "type": "string"
  },
  {
    "name": "rpc_id",
    "type": "string"
  },
  {
    "name": "rpc_type",
    "type": "string"
  },
  {
    "name": "client_name",
    "type": "string"
  },
  {
    "name": "server_name",
    "type": "string"
  },
  {
    "name": "client_ip",
    "type": "string"
  },
  {
    "name": "server_ip",
    "type": "string"
  },
  {
    "name": "client_idc",
    "type": "string"
  },
  {
    "name": "server_idc",
    "type": "string"
  },
  {
    "name": "client_unit",
    "type": "string"
  },
  {
    "name": "server_unit",
    "type": "string"
  },
  {
    "name": "client_region",
    "type": "string"
  },
  {
    "name": "server_region",
    "type": "string"
  },
  {
    "name": "span_client",
    "type": "string"
  },
  {
    "name": "span_server",
    "type": "string"
  },
  {
    "name": "span_format",
    "type": "string"
  },
  {
    "name": "result_code",
    "type": "string"
  },
  {
    "name": "cross_token",
    "type": "string"
  },
  {
    "name": "test_token",
    "type": "string"
  },
  {
    "name": "user_data",
    "type": "string"
  },
  {
    "name": "rule_id",
    "type": "string"
  },
  {
    "name": "request_size",
    "type": "string"
  },
  {
    "name": "response_size",
    "type": "string"
  },
  {
    "name": "cs",
    "type": "string"
  },
  {
    "name": "service",
    "type": "string"
  },
  {
    "name": "host_name",
    "type": "string"
  },
  {
    "name": "resources",
    "type": "string"
  },
  {
    "name": "id",
    "type": "string"
  },
  {
    "name": "parent_id",
    "type": "string"
  },
  {
    "name": "kind",
    "type": "string"
  },
  {
    "name": "attributes",
    "type": "string"
  },
  {
    "name": "links",
    "type": "string"
  },
  {
    "name": "events",
    "type": "string"
  },
  {
    "name": "status_message",
    "type": "string"
  }
]
```

---

## 四、存储 Schema

[otel，鹰眼schema统一](https://yuque.alibaba-inc.com/lhubic/gdebsi/iyb5appoq6nz910k#TtbVr)

---

## 参考：

[**OTLP数据端协议**](https://aliyuque.antfin.com/lhubic/gdebsi/txkgprqvkxwg0phs)

[**AI 可观测接入技术方案**](https://yuque.alibaba-inc.com/lhubic/gdebsi/pc4m8h12tngnwy05)

[**OpenTelemetry SDK兼容分析**](https://yuque.alibaba-inc.com/lhubic/gdebsi/sk89bbxu7yp2f1hb)

[**AI Trace 业链接务需求沟通**](https://yuque.alibaba-inc.com/lhubic/gdebsi/cu8tazbxhlnlqtb5)
