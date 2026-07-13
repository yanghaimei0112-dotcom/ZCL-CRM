# CRM-YG 空白系统 · 搭建指南

一套和原系统功能完全一样、但**数据空白且独立**的 CRM。数据存在你自己的
Supabase 项目里，支持多人 / 多设备共享。

项目文件：

| 文件 | 说明 |
|---|---|
| `index.html` | 主程序（登录、仪表盘、客户/拜访/公海等全部页面） |
| `yg-data.js` / `yg-visits.js` / `yg-pool.js` | 可选的样本数据导入源，**不会自动导入**，只有在「数据管理」页手动点按钮才会用到。想要纯空白可以删掉这三个文件。 |
| `supabase-schema.sql` | 一键建表脚本 |

---

## 一、创建你自己的 Supabase 项目（空白后端）

1. 打开 https://supabase.com → 注册 / 登录 → **New project**。
   - 记住数据库密码；地区选离你近的（如 Singapore）。
2. 项目建好后，左侧 **SQL Editor** → New query → 把
   `supabase-schema.sql` 全文粘进去 → **Run**。看到成功即建表完成（此时数据库是空的）。
3. 左侧 **Authentication → Sign In / Providers（或 Settings）** → 找到
   **Confirm email**，**关闭**它。
   > 因为登录邮箱是自定义的（如 `yhm@yg.crm`），不关掉邮件确认新账号无法登录。
4. 左侧 **Project Settings → API**，复制两项：
   - **Project URL**（形如 `https://xxxx.supabase.co`）
   - **Publishable / anon key**（`sb_publishable_...` 或 `eyJ...` 的 anon public key）

把这两项发给我，我会替你写进 `index.html`。或自己改：打开 `index.html`，
找到这两行改成你的值：

```js
const SB_URL="https://你的项目.supabase.co";
const SB_KEY="你的 publishable / anon key";
```

---

## 二、创建第一个管理员账号

1. 先把网站跑起来（见下方「三」），打开登录页。
2. 由于还没账号，先去 Supabase **Authentication → Users → Add user**，
   手动新建一个用户（邮箱如 `yhm@yg.crm`，设个密码，勾选 Auto Confirm）。
3. 回到 **SQL Editor**，把这个账号提升为主管：
   ```sql
   update public.profiles
   set role = 'admin', name = 'YG主管'
   where id = (select id from auth.users where email = 'yhm@yg.crm');
   ```
4. 用这个邮箱密码登录网站 → 进入后在「用户管理」页就能继续添加业务员账号了
   （业务员默认角色 sales，无需再手动改）。

---

## 三、把网站跑起来

**本地预览：**
```bash
cd CRM-YG
python3 -m http.server 8000
# 浏览器打开 http://localhost:8000
```

**发布到 GitHub Pages（和原站一样的公网地址）：**
1. 新建一个 GitHub 仓库，把 `index.html`、三个 `yg-*.js`（可选）传上去。
2. 仓库 Settings → Pages → Source 选 `main` 分支根目录 → Save。
3. 稍等出现 `https://<用户名>.github.io/<仓库名>/` 即可访问。

---

## 用户管理：删除账号

删除账号用一个数据库函数 `admin_delete_user`（无需 Edge Function / service_role）。
在 SQL Editor 运行 `supabase-delete-user.sql` 即可：它会（1）确认所有未确认的账号，
（2）创建删除函数。之后「用户管理」里的删除按钮即可正常使用（仅管理员可删、不能删自己）。

> 若新建账号登录报「Email not confirmed」，说明 Authentication → Providers → Email
> 里的 **Confirm email** 还开着；关掉它，新账号即可直接登录。
