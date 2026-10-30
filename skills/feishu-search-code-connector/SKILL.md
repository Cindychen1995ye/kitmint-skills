---
name: feishu-search-code-connector
description: Connect a user-selected Feishu Bitable to Kitmint's search-code QR generator, including a row button, scoped authentication and PNG attachment writeback.
---
# Kitmint · Feishu Search Code Connector

用户授权你将指定飞书多维表接入 Kitmint 时执行。只操作用户选定的数据表。

## 确认目标
复用用户已经提供的多维表链接；未提供时询问，不能自行选择。链接必须包含 base 与具体 table 参数。让用户确认表中用于搜索口令的字段；最终接口字段名为 `search_code`、字段类型为文本，不要求在首列。已有其他名称的口令列时，先确认重命名；不要覆盖内容、拿 SKU 当口令或擅自重命名首列。

## 字段、权限及按钮
1. 检查存在 `search_code` 文本字段，缺少时在获得用户确认后新增。检查「电子指南二维码图片」为附件字段，缺少时自动创建，不删除已有附件列。
2. 确认当前 Kitmint 飞书应用有目标表的读取、修改和附件上传权限；没有权限时指导用户在表格协作/应用权限处授权。不要要求用户交出 App Secret。
3. 添加「生成电子指南二维码」按钮。设置点击当前行按钮触发的自动化：校验当前行 `search_code` 非空，将当前行值作为动态字段引用；禁止写死示例口令。保留自动化草稿，待授权配置和验收完成后启用。
4. HTTP 动作：`POST https://ops.kitmint.io/v1/feishu/qr-by-search-code`，请求头 `Content-Type: application/json`，`Authorization: Bearer <表格专属 Key>`，JSON `{ "search_code": "当前行搜索口令的动态引用" }`。不得额外发送 SKU、品牌、设计尺寸、用户 ID 或表格 ID。Key 已绑定表格和生成账号。

## 安全配置 Key
打开 `https://ops.kitmint.io/skills?table_url=<已确认表链接的 URL 编码>`。用户通过飞书登录后只需点击生成按钮，无需重复填写表格链接。生成前后端验证字段和应用权限。

若你的环境支持不向模型暴露输入值、不会记录或截图密钥的安全凭证窗口，打开该窗口让用户输入，并直接绑定到自动化 Authorization 配置。否则先打开飞书 HTTP 动作的 Authorization 输入框，指导用户亲自粘贴；不要假装存在安全窗口。不要将 Key 发到聊天、脚本日志、GitHub 或普通表格单元格。轮换旧 Key 会使旧自动化失效，先告知用户。

## 回写与验收
API 自行生成带口令文字的默认 PNG，并上传、写回 Key 绑定表中同一口令的唯一行。「电子指南二维码图片」该行字段将被替换成最新一张 PNG；其他字段保留。默认二维码区30×30mm，标签总尺寸约30×35mm、600 DPI，长口令增加文字区高度。API 返回业务 JSON，例如 `{"ok":true,"search_code":"EXAMPLE","updated_field":"电子指南二维码图片","width":709,"height":827}`。不要另把 JSON 当图片，不再建立重复上传动作。

询问一个真实测试口令，记录点击前附件 token/更新时间；点击一次，检查自动化日志返回 JSON、新附件 token/更新时间及 PNG 实际可下载。重复口令会报错，不任选一行。扫码验证解码链接和口令一致，并确认消费者展示页正常；消费者主站未上线时明确记录“扫码跳转未通过”，不得宣布可印刷。旧附件不能作为本次成功证据。

401：Key 无效或已轮换；403：权限或组织限制；422：字段缺失/类型错误；429：限流，稍后重试。收到 HTML/Cloudflare 页面或域名无法解析，保存状态码、时间和 Ray ID，交管理员排查 `ops.kitmint.io` 到独立 Kitmint Docker 服务的路由，不要求关闭全站防护。用户无需提供服务器密码或项目源码来使用 API。
