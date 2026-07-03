# Matt Pocock Skills (mattpocock/skills)

ติดตั้งวิธีต่างจาก 2 อันบน — repo นี้ไม่ใช้ `/plugin marketplace add` แต่ใช้ CLI ตัวเอง
ให้เลือก skill ที่ต้องการทีละตัวตอนติดตั้ง:

```
npx skills@latest add mattpocock/skills
```

รันแล้วจะมี prompt ให้เลือก skills และเลือก coding agent ที่จะติดตั้งลง (เลือก Claude Code)

Skill ที่ต้องใช้: **`setup-matt-pocock-skills`** (อยู่ใน `skills/engineering/setup-matt-pocock-skills`)

| Command/Skill | ใช้เมื่อ |
|---|---|
| `setup-matt-pocock-skills` | รัน**ครั้งเดียว**ก่อนใช้ engineering skills อื่นของ Matt Pocock ในแต่ละ repo — เป็น setup แบบ interactive (ถาม-ตอบ ไม่ใช่ script อัตโนมัติ) ให้เลือก 3 อย่างตามลำดับ: ที่เก็บ issue tracker → label vocabulary สำหรับ triage → โครงสร้าง domain docs แล้วเขียนผลลงไฟล์ `CLAUDE.md`/`AGENTS.md` (เพิ่ม block "Agent skills") + สร้าง `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, `docs/agents/domain.md` |

## แหล่งที่มา

- https://github.com/mattpocock/skills
