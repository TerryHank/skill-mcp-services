# 魔搭部署说明

两个目录分别是独立 Python 分发包，包含代码和原 Skill 资源：
- mineru-book-mcp：MinerU 解析、本地回退、知识 Skill 校验与 ZIP 打包。
- fireworks-graph-mcp：图 JSON 转 SVG、PNG、HTML，返回布局报告。
两个 wheel 使用相同的内部模块名 skill_service，因此部署在各自 uvx 隔离环境，
不要安装进同一个 Python 环境。服务启动使用 stdio，魔搭负责对外代理 MCP。

## 本地安装启动
下载 dist 内 wheel 后：
    uvx --from ./mineru_book_mcp-0.1.0-py3-none-any.whl mineru-book-mcp
    uvx --from ./fireworks_graph_mcp-0.1.0-py3-none-any.whl fireworks-graph-mcp

服务等候 JSON-RPC 输入是正常启动状态。verify_mcp.py 用真正的 MCP 客户端进行握手、
tools/list、上传、任务调用、下载和 SHA-256 对比。

## 魔搭自定义 MCP
1. 把每个 wheel 上传至魔搭容器可下载的 HTTPS 文件存储，或发布到你控制的包仓库。
   本次未上传公共仓库、未发布 PyPI，也未创建魔搭实例。
2. 分别新建两个自定义 MCP 服务，安装命令选 uvx。
3. 使用对应 modelscope-*.json，把 REPLACE_WITH_WHEEL_HTTPS_URL 替换为该 wheel
   的实际可下载 URL。不要用 GitHub 网页地址代替文件下载地址。
4. 先不填 MinerU Token，可测试免费 Agent API。需要 Standard API 时，在魔搭
   服务环境变量中添加 MINERU_TOKEN。不要将密钥写入源码/ZIP/对话。
5. 服务启动后确认工具列表；连接地址和鉴权方式以魔搭实际生成的实例信息为准，
   不假设 ChatGPT 能使用任意 Bearer/OAuth 认证方式。
6. 对外暴露的服务应限制为个人或授权客户端访问；此包依赖 MCP 宿主的认证和隔离，
   没有独立账号体系。每个用户应使用独立部署，不适合共用的匿名多租户服务。

## 上传和下载
工具流程：
create_upload(filename,size,sha256) → upload_chunk(upload_id,offset,data_base64)
→ finish_upload → start_parse / start_render → job_status → read_artifact。

- 单文件上限 32 MiB，分块最多 256 KiB（解码后）。
- 使用 ASCII 文件名上传；文件内容可含中文。
- job_status 返回文件大小和 SHA-256；read_artifact 分块返回 base64，客户端
  拼接为文件并校验摘要。返回的不是自动公开的下载 URL。
- 每个实例最多两个活动任务；数据保存在 SKILL_SERVICE_DATA，默认 ./service-data。
- 任务和上传索引在内存中，重启失效；文件需由管理员定期清理，不保证魔搭临时盘持久性。
- ChatGPT 附件不会因部署 MCP 自动传送到服务。需要宿主能读取附件字节并调用上传工具，
  或使用具有文件访问能力的 MCP 客户端。不要让模型靠猜测生成文件 base64。
- 本次没有提供额外公网网页上传表单，以免假设魔搭 stdio 实例能暴露第二个 HTTP 端口。

## 文档流程
MD/TXT 直接本地解析；其他格式需 allow_cloud=true 才尝试 MinerU。
allow_local_fallback=true 时 MinerU 失败后调用 book-to-skill 本地解析；
结果 detail.engine 和 mineru_failed 明确记录实际路线。
api=agent 为免 Token；standard/auto 从服务环境读取 MINERU_TOKEN。
没有可用本地解析器或源文件损坏时会返回失败，不把失败包装成成功。

read_workflow 可分段读取原 SKILL.md 及 Markdown 参考，供宿主模型完成章节、
术语表、模式库和速查表。宿主提交 files 给 package_skill，它校验前置元数据、
内部链接及注入风险后打包。服务没有额外调用 LLM，不宣称 Python 自动生成完整知识。

## 绘图流程
上传符合 Fireworks schema 的 JSON，调用 start_render。
默认 architecture，返回 diagram.svg / diagram.png / diagram.html / layout.json。
源图限制 100 个节点、最大画布 4096；PNG 用 resvg 渲染为 1920 像素宽。
示例在服务内 vendor/fixtures/api-flow-style7.json。
需宿主按 Fireworks 工作流生成结构化数据；本包不自动把自然语言交给云 LLM。
GIF 涉及浏览器/FFmpeg 等运行依赖，当前服务没有暴露该功能。

## 实际验收
两份 wheel 已在独立 uv 环境安装，用 MCP Python SDK 客户端进行真实测试：
- 文档：上传合成 MD → 本地语料；上传合成 PDF → MinerU Agent 实际云解析，
  不允许本地回退，返回 engine=mineru-agent / mineru_failed=false。
- 生成包：宿主测试草稿 → 前置元数据/链接/注入扫描 → demo.zip，5 个 Markdown 文件。
- 图：上传标准图 JSON → SVG、1920×1400 PNG、HTML 和布局报告。
- 所有返回产物经 read_artifact 下载并逐个核对 SHA-256。
- 两个服务均拒绝越界读取请求。截图已人工查看，图像可读。
- 未验证魔搭线上冷启动、持久存储或多用户隔离；未验证 Standard Token 额度。

查看 evidence/doc.json、evidence/graph.json 和 examples 中的实际下载产物。

