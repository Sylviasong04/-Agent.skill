# 知页接入

优先使用用户指定的知识库，否则使用当前工作区的 `study-library`。项目不存在时询问现有项目位置，不另建网站。项目 README 有自定义约定时先确认相关部分。

## 登记课件

在知识库根目录执行：

```sh
python3 scripts/import_document.py '/课件绝对路径/lecture.pdf' --title '标题' --course '课程'
```

读取返回的真实 id，再读取 `data/documents/<id>/meta.json` 和已有 `notes.md`。重复导入时复用已有文档，不覆盖用户写好的讲解。PDF 保留在原目录，以 `sourcePath` 关联；PPT/PPTX 按导入器转换为阅读 PDF，不批量生成图片。

## 写入唯一讲解文件

`data/documents/<id>/notes.md` 示例：

```markdown
# 课件标题

<!-- page: 1 -->
## 第 1 页：主题

中文讲解……

<!-- page: 2 -->
## 第 2 页：概念

中文讲解……
```

页码从 1 开始，按原 PDF 顺序覆盖全部页面。Markdown 标题自动构成目录；`meta.json` 的 `chapters` 可按主题设置，每项包含 `id`、`title`、`start`、`end`，首尾页码均包含，范围连续且不重叠。保留用户的重命名、其他元数据和 `history.json`。

服务运行时优先使用已有笔记接口：`GET /api/bootstrap` 获取会话 token，`GET /api/documents/<id>/notes` 获取 revision，`PUT` 同一路径提交 `{markdown, revision}`，写请求携带 `X-Study-Token`。版本冲突时不强行覆盖。服务不可用时，确认文件自读取后未变化再原子写入；不为生成讲解启动或重启服务。

仅核对总页数、分页标记和保存位置。页面入口默认 `http://127.0.0.1:4317`，以项目配置为准；外部写文件后刷新页面读取。无需额外完成状态文件、目录扫描或浏览器全站检查。
