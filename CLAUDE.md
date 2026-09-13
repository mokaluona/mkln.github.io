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
8. **Windows 和 Ubuntu 两个克隆都能往 `main` 提交**（后者写文章兼预览）。所以**动手前必须先 `git pull`**，推之前先 `git pull --rebase`——详见下文「两台机器的分工与同步」。

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

## 正文段首缩进（自定义）

洛娜要的「段首空两格」实现在 `_sass/minimal-mistakes/_base.scss`：

```scss
.page__content p:not([class*="notice"]),
.archive > p,
.archive blockquote p { text-indent: 1.3em; }
```

1. **必须按正文容器限定，绝不能挂回裸 `p`。** 裸 `p` 会波及所有非正文段落：`.page__meta`（阅读时长、标签、分类、更新日期）、`.archive__item-excerpt`（列表摘要）、侧栏作者简介等。
2. **正文容器有两种**：`.page__content`（single/splash 布局）与 `.archive`（archive/home/posts/search 布局，about 页也在内）。`.archive` 用直接子选择器 `>` 是为了避开嵌套在列表项里的 `.archive__item-excerpt`。
3. **`.page__content i / .archive i { text-indent: 0 }` 这条守卫不能删。** `text-indent` 是**可继承**属性，而 Font Awesome 图标 `<i>` 是 `display:inline-block`，会把继承到的缩进量用在自身那 18.75px 宽的行框里，字形被挤出框外、压到后面的文字上——表现就是「图标和文字重叠」（如 `⏱ess than 1 minute read`）。删掉守卫，正文里一出现内联图标就会重演。
4. 当前缩进量 **1.3em**（沿用原值）。严格意义的「两格」是 `2em`，改这一个数字即可（注意别动 `$indent-var`，它同时控制段间距）。

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

## 两台机器的分工与同步

| 机器 | 路径 | 职责 |
|---|---|---|
| **Windows** | `D:\MY_WORKSPACE\mkln.github.io` | **改代码**——主题、样式、配置、文档。AI 在这里动手 |
| **Ubuntu** | `~/mkln-blog` | **写文章 + 发布前预览**（跑 `jekyll serve`） |
| **GitHub** | `origin` | 唯一的交汇点。只有 `main` 会被 Pages 发布 |

**两个克隆都能往 `main` 提交**（文章两边都可能改），所以同步只能靠纪律：

1. **动手前先 `git pull`。** 在陈旧的基础上提交，两边就会各走各的，之后再推就互相挡住。
2. **推之前先 `git pull --rebase` 再 `git push`。** 免得攒出一堆无谓的合并提交。
3. **`preview` 上永远不提交**（它只是 `main` 的镜像，见下一节）。
4. **改完尽快推，别让提交躺在本地过夜。** 未提交/未推送的本地状态是最危险的中间态——它会在 `git status` 里一直挂着，切分支还会被带着走。

> 踩过的坑：曾有一处 `M _posts/2026-4-26-练笔《吸烟有害健康》.md` 在 Ubuntu 侧漂了很久，两边状态不一致，最后只能人工判断该丢还是该留。

**不要在某一台单独建 `.gitignore`。** 它是被仓库跟踪的文件，两台共用一份；本地私建会让 `git pull` 直接报 `untracked working tree files would be overwritten by merge` 而卡死。

### Obsidian 配置：不进版本库

`_posts/` 是一个 Obsidian vault，但**只当查看器用**，两台的 Obsidian 配置各自保留即可。整个 `_posts/.obsidian/` 都在 `.gitignore` 里，**不要「顺手」把它加进版本库**。

> 2026-09-13 曾试过同步它，结果 `git push` 被 GitHub 的 push protection 直接拦下：`_posts/.obsidian/plugins/remotely-save/main.js` 里内嵌着 Google OAuth 的 Client ID / Client Secret，而本仓库是**公开**的。教训——公开仓库里第三方插件的目录默认当污染源看，整目录忽略，别只挑几个文件排除。

## 分支与发布

- **`main`** 是唯一的发布分支。GitHub Pages 只构建 `main`（分支根目录），**任何其他分支推上去都不会被发布**。
- **`preview`** 是洛娜"发布前先在 Ubuntu 里看一遍"用的预览分支，**它永远只是 `main` 的镜像，不要在它上面提交任何东西**。
- 远端另有一个历史遗留分支 `origin/master`（早期默认分支），与现行流程无关，**不要动**。

### 预览 → 发布流程

```bash
# 1. Windows 本地：把 main 推成远端的 preview，供 Ubuntu 预览
git push origin main:preview

# 2. Ubuntu 虚拟机：拉下来起服务查看
git fetch origin && git switch preview
bundle exec jekyll serve --livereload --host=0.0.0.0

# 3. 确认页面没问题后，Windows 本地正式发布（必须先确认在 main 上）
git branch --show-current      # 必须输出 main
git push

# 4. Ubuntu 里切回主干并同步
git switch main && git pull
```

刷新 preview 若报 non-fast-forward（说明 `main` 被改写而非新增提交），用 `git push --force origin main:preview`——preview 从不被直接修改，强制推是安全的。

删除预览分支：`git push origin --delete preview`，再 `git branch -d preview`。

### 给 AI 的硬性提醒

1. **提交前先 `git branch --show-current` 确认在 `main` 上**，不要往 `preview` 提交。
2. `preview` 与 `main` 内容一致时，"正式发布"就等于在 `main` 上 `git push`，不需要额外合并动作。
3. 洛娜可能长期保留 `preview` 分支，这是正常的，不要"顺手清理"。
4. 若发现提交误落在 `preview` 上，补救办法是先切回 `main`，再 `git cherry-pick <提交>`（或 `git merge preview`）。
