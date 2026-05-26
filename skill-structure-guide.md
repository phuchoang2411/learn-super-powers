# Cấu trúc Skill tốt — Superpowers vs Progressive Disclosure

Nguồn tham khảo: https://goonnguyen.substack.com/p/toi-a-sai-ve-agent-skills-va-cach

---

## Vấn đề: Skills ≠ Tài liệu

Sai lầm phổ biến: nhồi tất cả thông tin vào một SKILL.md khổng lồ.

```
# Sai
skills/cloudflare/SKILL.md   → 1,131 dòng
skills/nextjs/SKILL.md       → 900 dòng
skills/shadcn-ui/SKILL.md    → 850 dòng
```

Hậu quả: kích hoạt 5-7 skill = 5,000-7,000 dòng tràn vào context ngay lập tức.
90% thông tin không liên quan đến task hiện tại.

**Mental model đúng:** Skills là khả năng quy trình làm việc, không phải tài liệu tham khảo.

- `devops` không phải "tài liệu Cloudflare" → khả năng triển khai serverless
- `ui-styling` không phải "tài liệu Tailwind" → khả năng thiết kế giao diện nhất quán
- `systematic-debugging` không phải hướng dẫn → phương pháp giải quyết vấn đề

---

## Kiến trúc Progressive Disclosure (3 tầng)

```
Tầng 1: YAML frontmatter metadata
  → Luôn được nạp
  → ~100 từ
  → Đủ để agent quyết định skill có liên quan không

Tầng 2: SKILL.md entry point
  → Nạp khi skill kích hoạt
  → < 200 dòng (cứng)
  → Tổng quan + bản đồ điều hướng → trỏ đến references
  → KHÔNG bao gồm nội dung của references

Tầng 3: references/ + scripts/
  → Nạp theo yêu cầu, khi cần
  → 200-300 dòng mỗi file
  → Chi tiết kỹ thuật cho từng tình huống cụ thể
```

**Kết quả thực tế (từ case study):**
- Trước: 870 dòng một file
- Sau: 181 dòng + 13 reference files
- Giảm 79%, hiệu quả token tốt hơn 4.8 lần

---

## Tầng 1 trong obra/superpowers nằm ở đâu?

Superpowers **không tách tầng 1 thành file riêng**. Tầng 1 nằm ngay đầu SKILL.md dưới dạng YAML frontmatter — tầng 1 và tầng 2 gộp chung một file:

```yaml
---
name: brainstorming
description: "You MUST use this before any creative work - creating features,
building components, adding functionality, or modifying behavior."
---

# Brainstorming Ideas Into Designs        ← Tầng 2 bắt đầu từ đây
...
```

Ví dụ thực tế từ các skill đã đọc:

| Skill | Frontmatter description |
|---|---|
| `brainstorming` | "You MUST use this before any creative work..." |
| `systematic-debugging` | "Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes" |
| `using-superpowers` | "Use when starting any conversation - establishes how to find and use skills..." |

**Tại sao superpowers gộp tầng 1 + 2?**
Superpowers có ~15 skill tập trung vào workflow. Goon có 36 skill tool-specific cần scan nhanh. Khi số lượng skill ít và focused, gộp chung vẫn ổn. Khi số lượng lớn, tách metadata ra giúp scan nhanh hơn mà không nạp toàn bộ entry point.

---

## "References" trong obra/superpowers — thực tế không nhất quán

Superpowers không enforce cấu trúc thống nhất. Mỗi skill tổ chức khác nhau:

```
using-superpowers/
├── SKILL.md                              ← entry point
└── references/                           ← có folder riêng
    ├── codex-tools.md
    ├── copilot-tools.md
    └── gemini-tools.md

systematic-debugging/
├── SKILL.md                              ← entry point
├── root-cause-tracing.md                 ← reference, để thẳng
├── defense-in-depth.md                   ← reference, để thẳng
├── condition-based-waiting.md            ← reference, để thẳng
└── find-polluter.sh                      ← script, để thẳng

brainstorming/
├── SKILL.md                              ← entry point
├── spec-document-reviewer-prompt.md      ← reference, để thẳng
├── visual-companion.md                   ← reference, để thẳng
└── scripts/                              ← scripts có folder riêng
```

**Pattern thực sự:**

| File | Vai trò |
|---|---|
| `SKILL.md` | Bản đồ — mô tả skill làm gì, trỏ đến file khác |
| `root-cause-tracing.md` | Reference — chi tiết kỹ thuật khi cần đào sâu |
| `spec-document-reviewer-prompt.md` | Reference — prompt template để dispatch subagent |
| `scripts/` | Executable — chạy được, không phải đọc |

Superpowers đúng về **tinh thần** nhưng không enforce **cấu trúc**.
Kết quả: mỗi skill tổ chức theo cách riêng — vẫn hoạt động, nhưng ít predictable hơn.

---

## Cấu trúc chuẩn khi viết skill mới

Kết hợp tinh thần superpowers + cấu trúc rõ ràng của progressive disclosure:

```
skills/
└── ten-skill/
    ├── SKILL.md              ← < 200 dòng, bản đồ, YAML frontmatter
    ├── references/
    │   ├── chi-tiet-a.md     ← nạp khi agent cần A
    │   └── chi-tiet-b.md     ← nạp khi agent cần B
    └── scripts/
        └── ten-script.sh     ← tool chạy được
```

**SKILL.md phải trả lời được:**
1. Skill này cho phép làm gì?
2. Kích hoạt khi nào?
3. Reference nào cần đọc cho tình huống nào?

**Test cold start:** Xóa context, kích hoạt skill, đo lường.
Nếu nạp hơn 500 dòng khi kích hoạt lần đầu → cần tái cấu trúc.

---

## Tổ chức theo workflow capability, không theo tool

```
# Không nên
skills/cloudflare/
skills/docker/
skills/gcloud/
skills/vercel/

# Nên
skills/deploy/
├── SKILL.md                  ← "khả năng triển khai hạ tầng"
└── references/
    ├── cloudflare.md
    ├── docker.md
    └── gcloud.md
```

---

## Superpowers vs Progressive Disclosure — hai tầng khác nhau

| | obra/superpowers | Progressive Disclosure (Goon) |
|---|---|---|
| Giải quyết | Nội dung skill tốt trông như thế nào | Cấu trúc skill tốt trông như thế nào |
| Tập trung | Workflow — khi nào invoke, thứ tự, consistency | Context engineering — token efficiency |
| Áp dụng | Dùng skills đúng cách | Viết skills đúng cách |

**Cần cả hai:** superpowers dạy cách *dùng* skill trong workflow, progressive disclosure dạy cách *viết* skill không bị nặng context.
