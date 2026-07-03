---
name: agent-task-handoff
description: >
  เมื่อผู้ใช้วาง bug report, requirement ใหม่, หรือ feature request แล้วบอกว่า "ทำเป็นไฟล์ md เพื่อส่งต่อให้ agents"
  skill นี้จะ: (1) trace โค้ดจริงเทียบกับ input ทุกข้อก่อนเชื่อ (2) แยกว่าอะไรคือ bug / อะไรคือ feature ใหม่ / อะไรยังไม่มีใน schema
  (3) เขียน handoff .md ที่ถูกต้องตาม reality ไม่ใช่ copy input ตรงๆ
  Input ไม่ต้องเป็น bug report เสมอไป — requirement ใหม่, feature เพิ่มเติม, หรือ improvement ก็ใช้ได้
  Triggers: วาง bug/requirement/feature + "ทำเป็นไฟล์ md", "ส่งต่อให้ agent", "handoff ให้ agent", "เพิ่มเป็น file ส่งต่อ", "ทำ spec ให้ agent", "สรุปให้ agent"
---

# Agent Task → Handoff

เมื่อได้รับ bug report / requirement / feature request แล้วให้ทำ handoff สำหรับ agent
**Input ไม่ต้องเป็น bug เสมอไป** — requirement ใหม่หรือฟีเจอร์เพิ่มเติมก็ใช้ workflow เดียวกันได้

## หลักการ

**ห้าม copy input ลง handoff ตรงๆ** — bug report หรือ requirement อาจเขียนบน assumption ที่ไม่ตรงโค้ดจริง
(เช่น อ้างฟังก์ชันที่ไม่เคยสร้าง, field ที่ไม่มีใน schema, หรือ "random value" ที่จริงแล้วมาจาก logic ที่ถูกต้อง)

**ทุกข้อต้อง trace ก่อนเชื่อ** → แล้วค่อยแปลงเป็น task จริง

> สำหรับ requirement ใหม่ที่ยังไม่มีในโค้ด: ไม่ต้อง trace หา "สิ่งที่พัง" แต่ trace ว่า "ปัจจุบันมีอะไรอยู่แล้ว" เพื่อประเมิน gap และ prerequisite

**เกณฑ์ยอมรับงานของ agent ที่รับ handoff คือ "Build ผ่าน ไม่มี Error" เท่านั้น** — ห้ามเขียน task ย่อยที่สั่งให้ agent
ไปทดสอบการใช้งานจริงเอง (manual test / เปิดเบราว์เซอร์ลองกด / end-to-end test) ผู้ใช้จะเป็นคนทดสอบการใช้งานเองภายหลัง
จาก handoff เสร็จ — ไม่ต้องรอ agent ยืนยัน runtime behavior ก่อนส่งงานกลับ

---

## Workflow (ทำตามลำดับ)

### Step 1 — อ่าน report ทุกข้อ แล้ว identify claim
สำหรับแต่ละข้อใน bug report ให้ตั้งคำถาม:
- claim นี้คือ **bug** (ของมีอยู่แต่พัง), **missing feature** (ของไม่เคยมี), หรือ **schema gap** (DB/EDM ยังไม่รองรับ)?
- claim นี้ assume ว่ามีอะไรอยู่แล้วบ้าง? (ฟังก์ชัน, field, ตาราง, คอลัมน์)

### Step 2 — Trace โค้ดจริงทุกข้อ (อย่าข้าม)
สำหรับแต่ละ claim:
1. **grep หา** ฟังก์ชัน/field/class ที่ report อ้าง — ถ้าไม่เจอ = ไม่เคยมี
2. **อ่านโค้ดจริง** ของ controller/view/entity ที่เกี่ยวข้อง
3. **query DB** ถ้า report อ้างว่ามีคอลัมน์/ข้อมูลบางอย่าง (`sqlcmd` หรือ SSMS)
4. **บันทึก** ว่า claim ตรงหรือไม่ตรง พร้อม evidence (file:line หรือ query result)

### Step 3 — แยกประเภท task
จัด task ออกเป็น 3 กลุ่ม:
- 🔴 **แก้ bug** — ของมีอยู่แต่ทำงานผิด (trace แล้วยืนยัน)
- 🟡 **สร้างฟีเจอร์ใหม่** — ของไม่เคยมีในโค้ด
- 🔵 **schema ก่อน** — ต้อง ALTER TABLE + Update EDMX ก่อนจึงเขียนโค้ดได้

### Step 4 — เคลียร์ decision ที่ขาด
ถ้า report ระบุ output ที่ต้องการ แต่ไม่ได้ให้ข้อมูลพอจะคำนวณ → **ถามผู้ใช้ก่อน** อย่าให้ agent เดา
ตัวอย่าง: "แสดง 2 เส้น (≈200 เม็ด)" แต่ไม่บอกว่า 1 เส้น = กี่เม็ด

### Step 5 — เขียน handoff .md
บันทึกที่ **`.agents/Task.md`** ของโปรเจกต์ที่กำลังทำงานอยู่ (ไม่ใช่ `docs/`) — เพื่อให้ agent ตัวอื่น (Antigravity, Claude
Code เซสชันอื่น ฯลฯ) เข้าถึงเจอตามตำแหน่งเดียวกันเสมอ ถ้าโฟลเดอร์ `.agents/` ยังไม่มีในโปรเจกต์นั้น ให้สร้างขึ้นมาก่อน
ถ้ามี `.agents/Task.md` เดิมอยู่แล้ว (งานค้างจากรอบก่อน) ให้ถามผู้ใช้ก่อนว่าจะเขียนทับ/ต่อท้าย/หรือเก็บของเดิมไว้

ไฟล์ประกอบด้วย:

```markdown
# Handoff: <ชื่อ feature> — <suffix>

## ⚠️ สภาพปัจจุบัน (อ่านก่อน)
<อธิบายว่าโค้ดจริงเป็นยังไง ต่างจาก bug report ยังไง>

## ผลตรวจทีละข้อ
<claim vs สิ่งที่ trace จริง พร้อม file:line>

## งาน (Task 1, 2, 3, ...)
<task จริงที่ต้องทำ โดยมี prerequisite + dependency ชัดเจน — เกณฑ์จบงานของแต่ละ task คือ "Build ผ่าน ไม่มี Error"
เท่านั้น ห้ามใส่ task ย่อยที่สั่งให้ agent ทดสอบการใช้งานจริง/manual/end-to-end เอง ผู้ใช้จะทดสอบเองภายหลัง>

## ลำดับแนะนำ + dependency
<ตารางหรือลำดับ>

## กับดักโปรเจกต์
<copy จาก CLAUDE.md ที่เกี่ยวข้อง>
```

---

## กฎ

- **Cite or it didn't happen** — ทุก claim ต้องอ้าง file:line หรือ query result
- **ห้ามเขียน task ที่ยังมี decision ค้าง** — mark ว่า "ถามผู้ใช้ก่อน" แล้วรอ
- **schema gap = บล็อก task อื่น** — ใส่ prerequisite ชัดเจน (ALTER TABLE → Update EDMX ใน VS → เขียนโค้ด)
- **"Build สำเร็จ" ไม่ได้แปลว่า EDMX wiring ถูก** — EF6 EDMX ต้องมี CSDL/SSDL entry ครบถึงจะ query field ใหม่ได้จริง
  (compile ผ่านได้แม้ EDMX ยังไม่ sync กับคอลัมน์ที่เพิ่งเพิ่ม) → เตือนให้ agent เช็ค CSDL/SSDL/MSL ครบ 3 ชั้น + entity `.cs`
  ให้ตรงกันเอง แต่**ไม่ใช่การสั่งให้ทดสอบ runtime/manual** — เกณฑ์ยอมรับงานสุดท้ายยังคงเป็น **"Build ผ่าน ไม่มี Error"**
  ผู้ใช้จะทดสอบการใช้งานจริงเอง
- สำหรับ JewelryPrincess: EDMX แก้ใน VS เท่านั้น, `.cshtml` ต้องมี UTF-8 BOM, `JavaScriptSerializer` ตัด null key

---

## ตัวอย่าง pattern ที่เจอบ่อย

| bug report บอกว่า | trace แล้วพบว่า | action ใน handoff |
|---|---|---|
| "input โชว์ random ต้อง fix binding" | property เดียวกัน row เดียวกัน ค่าต้องเท่ากัน | verify ว่าดู build ถูกตัว ไม่ใช่ mock |
| "function X incomplete" | grep ไม่เจอ function X เลย | task = สร้าง function ใหม่ |
| "join VendorProductMapping หาย" | คอลัมน์ที่จะ join ไม่มีใน DB เลย | prerequisite = ALTER TABLE ก่อน |
| "dropdown ต้อง filter ตาม X" | ปัจจุบัน load ทั้งหมด by design | task = รื้อ design ไม่ใช่ fix bug |
| "ราคาคู่ incomplete" | ไม่เคยมี + อาจผิด stage | ถามผู้ใช้ก่อนว่าต้องการจริงไหม |
