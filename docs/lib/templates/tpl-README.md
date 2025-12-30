# Templates

这些模板供仓库内的 `biz-owner` / `prd` / `tech` / `proj` / `dev` subagent（以及人类写文档）参考，用于统一文档结构与字段。

对应关系：

* `biz-owner` → `biz-overview.md`（模板文件：`tpl-biz-overview.md`）
* `prd` → `tpl-prd.md`、`tpl-story.md`
* `prd/tech` → `tpl-slice-spec.md`
* `ux` → `tpl-prototype-index.html`（原型 HTML 模板，建议复制到 `docs/{{EPIC_DIR}}/prototypes/index.html`）
* `tech` → `tpl-tech-epic.md`
* `tech`（只读评审提示）→ `tpl-code-review-tech.md`
* `dev` → `tpl-task.md`
* `proj` → `tpl-proj-epic.md`、`tpl-proj-roadmap.md`

约定（建议默认）：

* Story 文件名建议：`STORY-{{EPIC_ID}}-{{NNN}}-xxx.md`
* Task 文件名建议：`TASK-{{EPIC_ID}}-{{AREA}}-{{NNN}}.md`
* 本期纳入的 Task 必须能追溯到 `STORY_ID` 与 `SLICE_ID`（见 `tpl-proj-epic.md` 的对齐表）
