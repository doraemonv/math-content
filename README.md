# 数学内容仓库

文章正文的唯一来源。与 `/var/www/math` 程序仓库独立管理。
文章规范请阅读程序仓库 docs/CONTENT_FORMAT.md。

新增或修改后：`docker compose -f /var/www/math/compose.yml exec -T php php artisan math:content:sync --changed-only`。

示例文章为第一阶段入门内容，可自行持续补充与校订。
