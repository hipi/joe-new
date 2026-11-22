# 更新日志

本项目（Joe 主题）的所有重要变更都记录在此文件中。

格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [7.8.0] - 2026-11-22

### 新增

- 新增数据库方言适配层（`core/function.php`）：
  - `_getDbAdapterName()`：读取当前数据库适配器名称，兼容不同 Typecho 版本；
  - `_isPgsql()` / `_isSqlite()`：判断数据库类型；
  - `_sqlTable()`：按数据库生成合法的表名引用（PostgreSQL 用双引号，MySQL/SQLite 用反引号）；
  - `_sqlTimeFormat()`：把 MySQL 的 `FROM_UNIXTIME` 格式串翻译为 PostgreSQL 的 `to_char` / SQLite 的 `strftime`，并显式指定时区以保证与 PHP `date()` 输出一致。
- 新增 `_joeApiDispatch()`（`core/route.php`）作为主题开放 API 的统一分发入口：路径匹配兼容 `/joe/api`、`/joe/api/`、`/index.php/joe/api`，未知 `routeType` 返回 JSON 错误信息而非静默 404。
- 新增 `_joeRequestParam()`：通过 `Request::get()` 读取请求参数，兼容 Typecho 1.3 起对魔术属性的废弃。
- 新增 `_joePermalink()`：为行数据（`$widget->stack` 元素）生成固定链接，兼容 Typecho 1.1 ~ 1.3。

### 修复

#### PostgreSQL 兼容

- 修复主题启用时 `ALTER TABLE ... ADD \`views\`` / `` `agree` `` 的反引号语法在 PostgreSQL 下报错的问题，改为按数据库类型生成标准 SQL，并避免空表时 `fetchRow` 返回 `false` 引发的 TypeError。
- 修复侧栏随机文章 SQL 中的 `RAND()`、`floor()` 与反引号，PostgreSQL 改用 `RANDOM()`，SQLite 改用 `ABS(RANDOM())`。
- 修复“那年今日”、评论图表、文章归档中 `FROM_UNIXTIME(...)` 无法在 PostgreSQL 执行的问题，统一改用 `_sqlTimeFormat()`。
- 修复文章归档 `GROUP BY date` 在 PostgreSQL 下不支持按输出别名分组的问题，改用序号分组（`GROUP BY 1`）。

#### Typecho 1.3 兼容

- 修复访问 `/joe/api` 时抛出 `Typecho\Router\Exception: Path '/joe/api' not found` 的问题：
  Typecho 1.3 起 `themeInit` 只在 `Widget\Archive` 初始化时执行，未匹配路由的请求不会再进入主题代码；
  现由 `themeInit` 与 `404.php` 双入口分发（`404.php` 增加兜底），并统一走 `_joeApiDispatch()`。
- 修复导航、分类、面包屑等链接 `href` 为空导致“页面打不开”的问题：
  Typecho 1.3 的 `Contents::filter()` 不再写入 `permalink`/`path`/`year`/`month`/`day` 等动态字段，
  改用 `_joePermalink()` 兜底生成链接。
  涉及 `public/header.php`、`public/batten.php`、`public/aside.php`、`post.php`、`core/function.php`。
- 修复侧栏“那年今日”日期为空的连带问题，改用 `date('Y', $created + $timezone)` 计算。

#### 后台样式

- 修复主题设置页在大屏幕下右侧留白的问题：
  覆盖 Typecho 后台栅格 `.col-tb-8` 的 `max-width: 66.66667%`、`.container` 的 `max-width` 以及 `.row` 的 `width: 100vw`，
  使设置页占满后台内容区（`assets/typecho/config/css/joe.config.min.scss` 与其编译产物同步修改）。

### 说明

- 上述改动均为按环境分派，MySQL 部署的行为与原版本保持一致。
- Typecho 1.3 在 debug 模式下不会注册 404 异常处理器，`/joe/api` 需在 `config.inc.php` 中关闭 `__TYPECHO_DEBUG__` 才能走主题兜底分发。
