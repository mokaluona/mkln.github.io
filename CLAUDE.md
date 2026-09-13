# CLAUDE.md

本文件是给 AI 助手看的仓库维护说明。人类读者请看 [写作说明.md](写作说明.md)。

## 项目概况

洛娜（魔卡洛娜）的个人博客，线上地址 <https://www.mkln.space>。

- **仓库**：<https://github.com/mokaluona/mkln.github.io>（GitHub Pages 用户站）
- **技术栈**：Jekyll 3.10 + Minimal Mistakes 4.27.1，**整份 fork**——`_includes/`、`_layouts/`、`_sass/`、`docs/` 都是主题全量文件，改动时注意区分"主题自带"与"本项目自定义"
- **本地克隆位置**（洛娜的机器）：`D:\MY_WORKSPACE\mkln.github.io`

## 硬约束（改代码前必读）

1. **部署方式是 GitHub Pages 传统构建（legacy build）**：`main` 分支根目录直接发布，没有 GitHub Actions，没有自定义构建步骤。
2. **禁止自定义 Ruby 插件**。只能用 `_config.yml` 白名单里的插件（`jekyll-paginate`、`jekyll-sitemap`、`jekyll-gist`、`jekyll-feed`、`jekyll-include-cache`）。任何主题逻辑改动**必须写成纯 Liquid** 放进 `_includes/` 或 `_layouts/`。写 `.rb` 插件是无效的。
3. **`excerpt_separator` 必须保持 `"\n\n"`，不要改**。Jekyll 原生摘要 = 首段。改它会连带改变 feed 与所有文章的默认摘要。
4. **不要碰不按命名规则命名的 md 文件**（如 `Hello World.md`、`迁站的话.md`），那是洛娜的草稿/测试文件。
5. **`locale` 是 `"China"`**（非标准 BCP-47 标签）。这会导致 `<html lang="China">` 且 UI 文案回落到英文——**这是洛娜的明确选择，不要"顺手修正"**。
6. **提交身份**：`mokaluona <mokaluona@outlook.com>`。**只本地 commit，push 由洛娜本人执行**，不要尝试处理她的 GitHub 凭据。
7. 根目录 `index.md` 是唯一首页。曾存在重复的 `index.html`（含 `author_profile: true`，因与 `index.md` 抢同一目标路径而未生效），已删除——**不要重新加回**。

## 摘要机制（本项目唯一的核心自定义逻辑）

**实现文件**：`_includes/excerpt-custom.html`（纯 Liquid）。

**优先级**（严格按序）：

1. `<!--excerpt-->` 与 `<!--/excerpt-->` 之间的内容 —— 可出现在任意位置，**包括段落中间**
2. `<!--more-->` 之前的内容
3. 都没有 → 输出**空字符串**，由调用方回落到 Jekyll 原生首段

细节：一篇文章里写了多组 `<!--excerpt-->` 时**只有最后一组生效**（实现用的是 `split | last`）；标记内为空时输出空串，等同于"没有标记"。

**集成点**（四处，缺一不可）：

| 文件 | 作用域 | 有标记时 | 无标记时 |
|---|---|---|---|
| `_includes/archive-single.html` | 列表页摘要 | 原文，**不截断** | 首段 + `truncate: 160` |
| `_includes/seo.html` | `<meta name="description">`、`og:description`、`twitter:description` | 优先于首段，但**低于 front matter 的 `description`** | 首段 → `site.description` |
| `_layouts/single.html` | 文章页 `<meta itemprop="description">` | 优先 | 首段 |
| `_layouts/splash.html` | 同上（splash 布局） | 优先 | 首段 |

`archive-single.html` 被 `_layouts/home.html`、`_layouts/posts.html`、`_layouts/archive-taxonomy.html`、`_includes/posts-category.html`、`posts-tag.html`、`posts-taxonomy.html`、`page__related.html` 引用 → 列表摘要出现在首页、`/blog/`、`/categories/`、`/tags/`、分类/标签归档页以及文章页底部的相关文章区。

**未接管的地方**：`feed.xml`。jekyll-feed 用自带模板，`<summary>` 取 Jekyll 原生 excerpt，`<content>` 为全文，**不读自定义标记**。若要一并接管需另作处理。

### ⚠️ 空字符串陷阱（务必牢记）

Liquid 中**只有 `nil` 和 `false` 为假；空字符串 `""` 是 truthy**。

因此 `{% if captured %}` 在 capture 结果为空串时**仍然为真**，会静默走进错误分支。本项目已因此踩过一次坑：列表摘要的 160 字截断曾对所有未标记文章整体失效。

判定"有没有自定义摘要"必须显式比较：

```liquid
{% assign _has_custom = _custom_ex | strip %}
{% if _has_custom == "" %}   {# 正确 #}
{% if _custom_ex %}          {# 错误：空串为真 #}
```

`| default:` 过滤器**不受此影响**（按 `empty?` 判断，空串会正确回落），可以放心使用。

## 站点配置要点

- `url` 必须带协议（`https://www.mkln.space`）。缺协议会污染 canonical、`og:url`、sitemap、JSON-LD 与页脚链接。
- `repository` 须为 `mokaluona/mkln.github.io` 形式（用于生成仓库相关链接）。
- 站点域名由根目录 `CNAME` 决定；`robots.txt` 里的 Sitemap 域名须与之一致。
- 文章永久链接为 `/:categories/:title/`，正文摘要与 SEO 描述依赖各篇文章的 front matter `categories`。

## 改动后如何验证

本沙箱**跑不了真实构建**（无 Jekyll、无 Ruby 开发头文件、无 docker）。可行的取舍：

1. **YAML**：`python3 -c "import yaml;yaml.safe_load(open('_config.yml'))"`
2. **Liquid 逻辑**：`pip install python-liquid` 后渲染 `_includes/excerpt-custom.html` 与各分支逻辑，验证优先级与截断行为。两个注意点：
   - 它的 include 参数语法是 `{% include "x.html", key: value %}`，与 Jekyll 的 `{% include x.html key=value %}` 不同，需用正则把 include 标签改写后才能解析；
   - 它忠实实现了"空串 truthy"，因此可用于捕捉上面那个陷阱。反之，若某个引擎按 Python 语义把空串当假，测试会"通过"但线上仍然出错——验证时先确认引擎语义，别被绿色结果误导。
3. **最终结论以 push 后 GitHub Pages 的构建结果为准。**

## 提交与推送

```bash
git config user.name "mokaluona"
git config user.email "mokaluona@outlook.com"
git add -A && git commit -F <提交信息文件>
```

提交完成后，请洛娜在 `D:\MY_WORKSPACE\mkln.github.io` 自行执行 `git push`。
