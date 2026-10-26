# Docs

基于 Sphinx + Read the Docs 的个人技术文档站点。

在线地址：<https://docscook.readthedocs.io/zh-cn/latest/>

## 目录结构

```
Docs/
├── source/                # 文档源文件
│   ├── index.rst          # 根目录（toctree 总入口）
│   ├── conf.py            # Sphinx 配置（Markdown 用 myst-parser 解析）
│   ├── readthedocs/       # Read the Docs 环境搭建笔记
│   ├── tiny200/           # tiny200 开发板学习笔记
│   └── Linux学习/          # Linux 学习笔记（按主题扩展）
│       └── index.rst      # 目录页：新增笔记后在此 toctree 追加
├── requirements.txt       # 在线构建依赖
├── .readthedocs.yaml      # Read the Docs 构建配置
└── Makefile / make.bat    # 本地构建脚本
```

新增一个主题文档时：
1. 在对应目录（如 `Linux学习/`）新建 `N.主题.md`（编号命名，仿照既有文件）。
2. 把文件名（不含后缀）追加到该目录 `index.rst` 的 `toctree` 中。
3. 若是全新目录，还要在 `source/index.rst` 的根 `toctree` 注册它的 `index`。

## 本地构建预览

```bash
# 首次：安装构建依赖
pip install -r requirements.txt

# 构建 HTML 到 build/ 目录
python -m sphinx -b html source build

# 或用 Makefile
make html          # Linux/macOS
make.bat html      # Windows
```

构建成功后，直接用浏览器打开 `build/index.html` 即可预览。

## 如何更新网站内容

Read the Docs 靠 GitHub 的 **webhook** 在 `git push` 时自动触发重新构建；若未配置 webhook，则需要在 RTD 后台手动重建。步骤：

1. **提交并推送源码到 GitHub**

   ```bash
   git add -A
   git commit -m "docs: 更新说明"
   git push origin master
   ```

2. **触发线上重新构建（二选一）**

   - **自动**：确认已配置 GitHub 集成 webhook —— 在 readthedocs.org 项目 → **Admin → Integrations** 查看是否存在 *GitHub incoming webhook*。没有则点 **Add integration → GitHub incoming webhook**，得到 URL 后到 GitHub 仓库 **Settings → Webhooks** 添加。配置好后每次 `git push` 自动重建。
   - **手动**：登录 [readthedocs.org](https://readthedocs.org) → 进入项目 `docscook` → **Builds** → 对 `latest` 版本点 **Rebuild**。

3. **验证**：等待 1~2 分钟构建完成，刷新在线页面确认新内容已出现；若构建失败，进入该次构建查看日志定位问题。

> 依赖变更注意：若修改了 `conf.py` 或新增 Python 依赖，请同步更新 `requirements.txt`，否则线上构建可能因缺少依赖而失败。
