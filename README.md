# 学生邮箱自助开通平台 v1.2

[![PHP Version](https://img.shields.io/badge/php-%3E%3D7.4-8892BF.svg)](https://php.net/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE) 本项目是一个基于 PHP 和 SQLite 的 Web 应用程序，旨在为学生提供一个通过卡密自助开通邮箱账户的平台。管理员拥有功能完善的后台，可对卡密、邮箱账户、系统设置等进行管理。

## ✨ 主要功能

### 👤 前台 (面向用户)

* **卡密验证:** 用户输入卡密进行有效性验证。
* **邮箱前缀检查:** 实时检查用户期望的邮箱前缀是否可用。
* **自助开通:** 验证通过后，自动通过 DirectAdmin API 创建邮箱账户。
* **结果展示:** 显示新邮箱账户信息（地址、初始密码、登录链接）。

### 🛠️ 后台 (面向管理员)

* **📊 仪表盘:** 实时概览卡密使用统计。
* **🔑 卡密管理:**
    * 按状态（未使用/已分发/已兑换）查看卡密。
    * 批量生成、导入、提取卡密。
    * “一键复制”分发卡密（自动更新状态）。
    * “收回”已分发但未使用的卡密。
* **📧 邮箱管理:**
    * 查看 DirectAdmin 上的邮箱列表（带缓存加速）。
    * 搜索、分页浏览邮箱。
    * 强制修改邮箱密码。
    * 删除邮箱账户。
    * 模拟登录邮箱 (需 Webmail 支持，如 RoundCube)。
    * 强制刷新邮箱列表缓存。
* **➕ 手动创建邮箱:** 管理员可直接为用户创建邮箱。
* **📬 邮件群发:**
    * 创建邮件任务队列，可发送给所有已兑换用户或指定列表。
    * 支持 PHP `mail()` 或 SMTP 两种发送方式（后台可配置）。
    * 由 Cron Job 驱动的真后台异步发送。
* **🤖 Cron 控制面板:**
    * 提供 Cron Job 设置助手，自动生成配置命令。
    * 手动触发邮件队列处理并查看实时日志。
    * 监控队列状态（待处理/已发送/已失败）。
    * 查看发送失败的邮件日志。
    * 提供常见 Cron Job 错误排查指南。
* **⚙️ Settings 表编辑器:** 提供图形界面查看、添加、修改、删除数据库 `settings` 表中的所有配置项。⚠️ 操作需谨慎！
* **📚 项目文档:** 内置后台使用说明页面。
* **📱 PWA 支持:** 可将后台管理界面添加到手机主屏幕。

## 📋 环境要求

* PHP >= 7.4
* PHP PDO SQLite 扩展已启用
* Web 服务器 (如 Apache, Nginx)
* 项目根目录 (`public_html`) 具有写入权限 (用于安装时创建 `database` 和 `cache` 目录)
* `database/` 目录需要写入权限 (用于 SQLite 数据库)
* `cache/` 目录需要写入权限 (用于 API 缓存)
* 拥有 API 访问权限的 DirectAdmin 账户 (强烈建议使用**仅有 `CMD_API_POP` 权限**的登录密钥 Login Key)
* (推荐) 服务器支持 Cron Job (用于邮件队列后台自动发送)
* (推荐) 服务器支持 PHP cURL 扩展 (用于后台手动触发 Cron)
* (可选) Composer (用于安装 PHPMailer 依赖) 或手动放置 PHPMailer 文件。

## 🚀 安装步骤

1.  将项目所有文件上传到服务器的 `public_html` (或相应网站根目录)。
2.  **手动创建 `cache` 目录：** 在 `public_html` 目录下创建一个名为 `cache` 的空目录，并确保 Web 服务器对其有写入权限 (例如 755 或 775)。
3.  **运行安装向导：** 通过浏览器访问 `https://your.domain.com/setup.php`。
    * 检查环境是否满足要求。
    * 点击 "创建/更新数据库" 来初始化 `database/main.db` 文件和表结构。
    * 填写管理员账户信息、DirectAdmin API 凭据和目标邮箱域名，然后保存配置。
4.  **获取初始密码：** 安装完成后，首次访问 `https://your.domain.com/admin/reset_admin_pass.php`。页面会显示一个一次性的管理员密码。复制此密码。
5.  **🚨 安全：删除安装文件：** **立即**从服务器上删除以下文件：
    * `setup.php`
    * `admin/reset_admin_pass.php`
    * (可选) `admin/debug_*.php` 文件
6.  **登录并修改密码：** 访问 `https://your.domain.com/admin/`，使用默认用户名 (`admin` 或您安装时设置的) 和第 4 步获取的密码登录。登录后，立即前往 "核心站点设置" -> "安全中心" 修改为您自己的强密码。

## ⚙️ 配置说明

系统的大部分配置可以通过后台管理界面进行。

1.  **`config.php` 文件:**
    * `DB_PATH`: 通常无需修改，指向 `database/main.db`。
    * `CRON_SECRET_KEY`: **必须手动编辑此文件**，将其值 `'YOUR_STRONG_SECRET_KEY_HERE'` 替换为一个长且随机的安全密钥。此密钥用于保护 Cron Job 脚本。您可以在后台 <a href="admin/cron_manager.php">Cron Job 控制面板</a> 查看当前设置并获取 Cron 命令。
2.  **后台 "核心站点设置" (`admin/settings.php`):**
    * **DirectAdmin API:** 配置连接 DA 服务器所需的主机、端口、用户名和登录密钥。
    * **邮件发送:** 选择使用 `PHP mail()` 还是 `SMTP`。如果选择 SMTP，需要配置详细的 SMTP 服务器信息（推荐使用第三方邮件服务如 Mailgun, SendGrid）。
    * **外观内容:** 配置前台页面的页脚信息、联系方式、Webmail 登录链接等。

## 🚀 使用方法

* **用户:** 访问 `https://your.domain.com/index.php` 进行自助开通。
* **管理员:** 访问 `https://your.domain.com/admin/` 登录后台进行管理。
* 详细的后台功能使用说明，请参考后台管理界面中的 **"📚 项目使用文档"** 页面 (`admin/documentation.php`)。

## 🛡️ 安全注意事项

* **🚨 安装后务必删除 `setup.php` 和 `admin/reset_admin_pass.php`！**
* 使用强密码保护您的管理员账户，并定期更换。
* 在 `config.php` 中设置一个强随机的 `CRON_SECRET_KEY`。
* 为 DirectAdmin API 使用**低权限**的专用登录密钥 (Login Key)。
* 确保服务器文件权限设置正确，仅 `database/` 和 `cache/` 目录需要写入权限。
* 强烈建议为您的网站启用 **HTTPS**。


## 📄 License

本项目采用 MIT License。您可以自由使用、修改和分发。（您可以根据需要更改此部分）

---
