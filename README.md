# Claude Code Custom Skills / Claude Code 自定义技能库

> Personal skill library for [Claude Code](https://claude.ai/claude-code) — specialized AI capabilities for scientific research.
>
> 个人 Claude Code 技能库——专为科研工作定制的 AI 增强能力。

---

## Skills / 技能列表

### 🔬 [topas-rietveld.md](./topas-rietveld.md)

**TOPAS Rietveld Refinement / X射线衍射 Rietveld 精修**

Enables Claude to autonomously perform Rietveld refinement using TOPAS software.
让 Claude 能够自主完成 X 射线衍射数据的 Rietveld 精修。

**Capabilities / 功能：**
- Parse CIF structure files and XRD data (`.xy` `.raw` `.xdd`)
  解析 CIF 结构文件与 XRD 数据
- Auto-generate TOPAS `.inp` input files
  自动生成 TOPAS 精修输入文件
- Execute multi-round refinement strategy (6 rounds)
  执行 6 轮递进式精修策略
- Output refined lattice params, atomic positions, ESDs
  输出精修晶格参数、原子坐标、误差
- Generate publication-quality fit plots (Obs/Calc/Diff)
  生成发表级拟合对比图
- Supports: lab XRD (Cu Kα), synchrotron, neutron
  支持：实验室 X 射线、同步辐射、中子衍射

**Trigger words:** `精修`, `Rietveld`, `TOPAS`, `XRD fitting`, provide `.xy` + `.cif` files

---

## How to Use / 如何使用

1. Clone or download this repo
2. Copy skill files to `~/.claude/skills/`
3. Restart Claude Code — skills are automatically active

```bash
git clone https://github.com/louco999/claude-skills.git
cp claude-skills/*.md ~/.claude/skills/
```

> Skills are markdown files read by Claude Code at startup.
> Each file contains trigger conditions, knowledge, and workflows.

---

## Environment / 运行环境

| Software | Path |
|----------|------|
| TOPAS 6 | `C:\topas\tc.exe` |
| Python (numpy/matplotlib) | `C:\Users\hanyi\miniforge3\python.exe` |
| GSAS-II Python | `C:\Users\hanyi\gsas2main\python.exe` |

> Paths are Windows-specific. Modify skill files for your own system.

---

*Built with Claude Code · 由 Claude Code 构建*
