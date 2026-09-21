# Skill 验收流水线 — 常见坑（维护者视角）

> 坑表是给修改者看的，不是给使用者看的——SKILL.md 正文只留引用，不复制本表。

## 验收后的修复环节

### 修复 push ≠ 发布完成：元数据五件套必同步

验收修复常伴随版本 bump（frontmatter version / README 徽章），但执行者容易只 push SKILL.md 就收工——tag、release、About description、README 徽章全部留在旧版本。实测：2026-09-10 验收三 skill（path-simulation / skill-health-audit / skill-acceptance）修复后仅推代码，release/About/徽章全漏，被用户一句「release about 版本号这些你就不改了是吧」抓包。

**防法**：修复涉及版本变更时，收尾必跑 github-skill-publishing 的 `scripts/publish_verify.sh` 五项终检（tag / Release Latest / About / README 徽章 / 本地远程一致性——清单以该脚本为唯一权威，本处不另列），exit 0 才算过，不能只看 push exit code。

### 版本号类 patch 必须 grep 落点值验证

`replace('version: 1.1.2', 'version: 1.1.3')` 这类改版本号操作，**目标不存在时静默滑过不报错**——执行者预设「仓库已是 1.1.2」实际是 1.1.0，replace 不命中、不抛异常、脚本继续跑，最后五件套终检才发现 frontmatter 停在旧版本。同族坑：README 徽章基线传错（仓库徽章停在 1.1.0，bump 脚本却按 1.3.0→1.3.2 找）。

**防法**：①改版本号前先 grep 当前实际值，不凭记忆/上轮报告预设；②replace 后立即 grep 新值确认落盘；③批量改版本时用断言（`assert old in s`）代替静默 replace。

### 双现场改动互为基线，收尾对账版本

本地 `~/.hermes/skills/` 与 GitHub 仓库是两个独立演化的现场——验收修复常两边各改各的（本地 patch description、仓库 bump 版本），结束后版本号必然漂移。实测：三 skill 验收后本地 2.7.0/1.1.1/1.3.0 vs 仓库 2.7.1/1.1.0/1.2.0，三个全部错位。

**防法**：①动第一个现场前先读另一个现场的当前值（版本/内容），把两边增量列进同一张清单；②修复收尾以「两边版本号一致 + 内容 diff 为空或有解释」为完成标准；③本地增量（实战沉淀的新条目）要主动推上仓库，仓库增量（其他会话的迭代）要主动拉回本地——双向都要走，单向同步等于另一半丢失。

### 验收结论的正确性依赖「被测对象版本」锚定

验收报告必须记录被测 skill 的 version 与「本地/仓库」哪一份——三 skill 验收同一天，仓库版可能含本地没有的增量（如 skill-acceptance 仓库的「批量」tag），不锚定版本，修复时会修错底稿。

**防法**：Phase 0 条 5 已落地（定界时记本地 + 仓库两份 frontmatter，报告问题清单每条标注「在哪一份上成立」）。
