# AI 做的弹窗截图看着对，只用键盘还能完成任务吗？

AI 产品经理精选 · 2026-11-25 · 界面验收与键盘可访问性

AI 可以很快生成一个像样的编辑器。但当用户准备离开时，放弃草稿的弹窗能不能用键盘操作？按 Escape 到底保留了草稿，还是只把弹窗关上？这组问题可以在查看截图之外独立验收。

这是独立原创中文学习笔记，不是 Playwright 官方教程、完整源码或实测报告。编辑器、草稿和缺陷都是虚构的纸面用例，未运行浏览器或读屏测试。

## 为什么读 Playwright

[Playwright](https://github.com/microsoft/playwright) 是微软维护的浏览器自动化和端到端测试工具。[v1.65.0 发布说明](https://github.com/microsoft/playwright/releases/tag/v1.65.0)（2026-11-05 正式发布）记录了可访问名称和 ARIA 快照对未渲染文本的处理变化，以及部分定位器操作会等待控件不再处于 inert（不可交互）状态。这些变化不是 ARIA 快照首次上线，也不替你测试焦点是否被正确限制。研究时的后续稳定版为 [v1.65.1](https://github.com/microsoft/playwright/releases/tag/v1.65.1)，已核读该版本的 [README](https://github.com/microsoft/playwright/blob/v1.65.1/README.md) 和 [Apache-2.0 许可证](https://github.com/microsoft/playwright/blob/v1.65.1/LICENSE)。

2026-11-25 北京时间核查：[GitHub API](https://api.github.com/repos/microsoft/playwright) 显示 **101,038 Stars**，仓库公开、未归档，当日仍有推送。Playwright 不是新项目，本次关注的是 11 月的交互测试变化。

## 给评审三层证据

1. **能被识别。** 在弹窗范围内查找可访问名称为「放弃未保存的修改」的对话框，核对「继续编辑」和「放弃修改」两个操作。可以用 [ARIA 快照](https://playwright.dev/docs/aria-snapshots) 保存这一小块语义结构。默认子项匹配可以容许额外项目；如果多出的操作也会造成误触，就明确断言数量或用严格匹配。
2. **确实能操作。** 通过页面级 [keyboard.press](https://playwright.dev/docs/api/class-keyboard#keyboard-press) 送出真正的按键事件，以 [toBeFocused](https://playwright.dev/docs/api/class-locatorassertions#locator-assertions-to-be-focused) 检查实际焦点。不要用定位到按钮后的 [locator.press](https://playwright.dev/docs/api/class-locator#locator-press) 来证明按 Tab 能找到它，因为该方法会先给目标元素焦点。
3. **结果符合承诺。** 取消要保留编辑的最新内容，确认只应放弃当前草稿。不能只验弹窗消失，也不能用截图代替草稿状态。

## 练习：用两份假草稿验收「放弃修改」

每条路径都恢复相同的起始数据：草稿 A 有一段未保存的修改，草稿 B 是不会被操作的对照。用 Tab 到达 A 的离开入口，再按 Enter。按 [W3C 模态对话框模式](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)，这个不可轻易撤销的场景可以把初始焦点放在较安全的「继续编辑」。这是对本场景的产品选择，而非每个弹窗的通用规则。

| 操作 | 通过条件 |
| --- | --- |
| 键盘打开对话框 | 对话框和两个按钮都有正确的可访问名称；焦点落在「继续编辑」 |
| 分别按 Tab 和 Shift+Tab | 两个按钮间按预定顺序双向循环；背景控件不获得焦点、也未执行动作 |
| 按 Escape | 对话框关闭，焦点回到原触发入口，返回编辑后仍能读到 A 的最新内容；B 不变 |
| 重新以键盘打开、切到「放弃修改」并按 Enter | 对话框关闭，A 的本次修改按产品契约被放弃，B 不变；焦点落在关闭后合理、可用的位置 |
| 用键盘激活「继续编辑」 | 与 Escape 一样保留 A 与 B，且焦点回到合理的原入口 |

把前三种缺陷逐个放进测试用例：去掉可访问名称、允许 Tab 跑到背景、让 Escape 错误放弃 A。预期语义、键盘、结果三组检查分别至少拦住一种。再改一处无关配色，关键路径应全部继续通过。如果触发器在确认后已不存在，不要求回到消失的按钮，事先指定合理的后继位置。

**示例发布门槛：** 两条取消路径 2/2 保留草稿；确认路径 1/1 只影响 A；正反向遍历中焦点逃出 0 次，三个注入缺陷 3/3 被检出，无关配色调整不阻断发布。这些是纸面目标，不是实测结果，也不表示整体符合无障碍标准。上线前还需要人工验证可见焦点和代表性的读屏组合；有浏览器回退路径的，单独验收。自动化优先锁住会丢数据、卡住用户的行为，不把整个页面变成一张难以维护的快照。[官方无障碍测试说明](https://playwright.dev/docs/accessibility-testing)

**面试怎么说：**“我会把 AI 原型验收拆成控件语义、键盘操作和真实结果。截图看不出 Escape 是否丢草稿，所以我会亲自走一次键盘流程，并把焦点恢复和草稿是否保留写进上线标准。”
