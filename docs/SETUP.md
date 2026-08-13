# ขั้นตอนติดตั้ง (สำหรับเจ้าของ repo)

ทำ 5 ขั้นตอนนี้ในองค์กร/repo ของคุณ ใช้เวลาประมาณ 10 นาที
ทุกขั้นตอนต้องใช้บัญชีที่มีสิทธิ์ **Owner ของ organization** หรือ **Admin ของ repo**

> **ต้องเป็น repo ใต้ organization** — repo ใต้บัญชีส่วนตัวให้สิทธิ์ read-only ไม่ได้
> **ถ้า repo เป็น private** ต้องใช้แผน GitHub Team ขึ้นไป (ruleset ใช้กับ private ในแผนฟรีไม่ได้)

## 1. สร้างทีมผู้ตรวจ และให้สิทธิ์ Read

1. ไปที่ `https://github.com/orgs/<ORG>/new-team`
2. ตั้งชื่อทีม: `independent-auditors` — visibility: Visible
3. เข้า repo ที่จะถูกตรวจ → **Settings → Collaborators and teams → Add teams**
4. เลือก `independent-auditors` แล้วตั้ง role เป็น **Read**

อย่าตั้งเป็น Write หรือ Triage — ความเป็นอิสระอยู่ที่ผู้ตรวจแก้อะไรไม่ได้เลย

## 2. เชิญบัญชีผู้ตรวจเข้าทีม

1. `https://github.com/orgs/<ORG>/teams/independent-auditors/members`
2. **Add a member** → ใส่ `MKOL-prog`
3. ตรวจว่า role ในองค์กรเป็น **Member** (ไม่ใช่ Owner)

## 3. วาง workflow ด่านตรวจ

คัดลอกไฟล์ `install/workflows/independent-audit-gate.yml` ไปวางที่

```
.github/workflows/independent-audit-gate.yml
```

ของ repo ที่ถูกตรวจ แล้ว commit เข้า default branch

ถ้ามีผู้ตรวจหลายคน แก้ค่า `AUDITORS` ในไฟล์ให้เป็นรายชื่อคั่นด้วยลูกน้ำ เช่น

```yaml
AUDITORS: MKOL-prog,another-auditor
MIN_APPROVALS: "1"
```

## 4. รัน workflow หนึ่งรอบให้ GitHub รู้จัก status check

เปิด PR ทดสอบสัก 1 อัน (แก้ README บรรทัดเดียวก็ได้) เพื่อให้ชื่อ check
`independent-audit-approval` ปรากฏในระบบ — ถ้าไม่ทำขั้นนี้ ขั้นที่ 5 จะค้นหาชื่อ check ไม่เจอ

## 5. ตั้ง ruleset ให้ check นี้เป็นเงื่อนไขบังคับ

**วิธี import (เร็วกว่า):**

1. `https://github.com/<ORG>/<REPO>/settings/rules` → **New ruleset → Import a ruleset**
2. อัปโหลด `install/rulesets/main-independent-audit.json`
3. กด **Create**

**หรือตั้งมือ** ที่ New branch ruleset:

- Target branches: **Default branch**
- Enforcement: **Active**
- Bypass list: **ปล่อยว่าง**
- ติ๊ก **Require a pull request before merging**
  - Required approvals: `0` (ด่านจริงคือ status check ข้างล่าง)
  - ติ๊ก Dismiss stale pull request approvals when new commits are pushed
- ติ๊ก **Require status checks to pass**
  - ติ๊ก Require branches to be up to date before merging
  - เพิ่ม check: `independent-audit-approval`
- ติ๊ก **Block force pushes** และ **Restrict deletions**

> **แนะนำ** ตั้งที่ระดับองค์กรแทน โดย import `install/rulesets/org-independent-audit.json`
> ที่ `https://github.com/organizations/<ORG>/settings/rules` — repo admin จะปิดด่านเองไม่ได้
> และไฟล์นั้นกันการเลี่ยงกฎด้วยการเปลี่ยนชื่อ repo ไว้ด้วย

## ตรวจว่าใช้งานได้จริง

เปิด PR ใหม่แล้วดูว่า:

- [ ] check `independent-audit-approval` ขึ้นสถานะ **failing** ตั้งแต่ยังไม่มีใคร approve
- [ ] ปุ่ม Merge ถูกล็อก
- [ ] เมื่อ `MKOL-prog` กด Approve → check กลายเป็น **passing** ภายใน ~1 นาที
- [ ] push commit ใหม่เข้า PR เดิม → check กลับไป **failing** อีกครั้ง

ถ้าข้อสุดท้ายไม่เป็นแบบนี้ แปลว่า ruleset ยังไม่ active หรือชื่อ check สะกดไม่ตรง

## ถอนการติดตั้ง

ลบ ruleset → ลบไฟล์ workflow → เอา `MKOL-prog` ออกจากทีม
ผู้ตรวจไม่มีสิทธิ์อะไรค้างอยู่ในระบบ เพราะไม่เคยมีสิทธิ์เกิน Read
