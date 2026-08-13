# Independent Audit Kit

ชุดตั้งค่าสำหรับให้บัญชี GitHub หนึ่งบัญชีทำหน้าที่ **ผู้ตรวจอิสระ (independent auditor)**
บน repository ของผู้อื่น โดยยึดหลัก "ตรวจได้ แต่แก้ไม่ได้"

- ผู้ตรวจได้สิทธิ์ **Read เท่านั้น** — push / merge / แก้โค้ด / แก้กฎ ไม่ได้เลย
- แต่ยัง **บล็อกการ merge ได้จริง** ผ่าน GitHub Actions ที่ตั้งเป็น required status check
- approve ผูกกับ commit ล่าสุด — push ใหม่เมื่อไหร่ ต้องตรวจใหม่ทุกครั้ง
- ผู้ตรวจอนุมัติ PR ของตัวเองไม่ได้

บัญชีผู้ตรวจในชุดนี้: **MKOL-prog**

## ทำไมต้องทำแบบนี้

GitHub มีข้อจำกัดที่ทำให้ "ผู้ตรวจอิสระแบบ read-only" ตั้งตรง ๆ ไม่ได้:

| ข้อจำกัดของ GitHub | ผลกระทบ |
|---|---|
| repo ใต้บัญชีส่วนตัวให้ read-only ไม่ได้ — collaborator ได้ write เสมอ | ต้องใช้ organization ถึงจะมี role Read |
| required approvals นับเฉพาะ reviewer ที่มี **write** | approve จากผู้ตรวจ read-only ไม่มีผลต่อการ merge |
| CODEOWNERS ต้องมี **write** เท่านั้น | ใส่ผู้ตรวจ read-only ลง CODEOWNERS แล้วจะไม่ถูก assign |
| branch protection / ruleset ฟรีเฉพาะ repo **public** | repo private ต้องขึ้น GitHub Team หรือ Pro |

ทางออกของชุดนี้คือ **ไม่พึ่ง required approvals** แต่ใช้ workflow อ่าน review ผ่าน API
แล้วรายงานผลเป็น status check แทน — ผู้ตรวจจึงอยู่ที่ Read ได้ โดยยังมีอำนาจยับยั้ง

## โครงสร้าง

```
docs/SETUP.md               ขั้นตอนติดตั้งสำหรับเจ้าของ repo
docs/INDEPENDENCE.md        ขอบเขตงาน หลักความเป็นอิสระ และสิ่งที่ผู้ตรวจไม่ทำ
docs/AUDIT-CHECKLIST.md     เกณฑ์ที่ใช้ตรวจแต่ละ PR
install/workflows/          workflow ที่ต้องคัดลอกไปวางใน repo ที่ถูกตรวจ
install/rulesets/           ruleset JSON สำหรับ import — มีทั้งระดับ repo และระดับองค์กร
templates/audit-report.md   เทมเพลตรายงานผลตรวจ
templates/access-request.md ข้อความขอสิทธิ์ Read ส่งให้เจ้าของ repo
```

**ruleset มีสองแบบ**

| ไฟล์ | ตั้งที่ไหน | ใครแก้ได้ |
|---|---|---|
| `main-independent-audit.json` | Settings → Rules ของ repo | repo admin ปิดเองได้ |
| `org-independent-audit.json` | Settings → Rules ขององค์กร | เฉพาะ org owner — repo admin แก้ไม่ได้ |

ถ้าต้องการความเป็นอิสระที่ตรวจสอบย้อนหลังได้จริง ให้ใช้แบบระดับองค์กร
ไฟล์นั้นตั้ง `protected: true` ไว้ด้วย เพื่อไม่ให้เลี่ยงกฎด้วยการเปลี่ยนชื่อ repo

## เริ่มยังไง

เจ้าของ repo ที่ต้องการให้ตรวจ อ่าน [docs/SETUP.md](docs/SETUP.md) แล้วทำตาม 5 ขั้นตอน
ใช้เวลาประมาณ 10 นาที

หรือ [เปิด issue ขอให้ตรวจ](../../issues/new/choose) แล้วกรอกแบบฟอร์มมาได้เลย

## ข้อจำกัดที่ต้องรู้

ชุดนี้ให้อำนาจยับยั้งทางเทคนิค แต่ **ไม่ได้ป้องกันเจ้าของ repo ที่ตั้งใจข้าม** — admin
ยังปิด ruleset หรือแก้ workflow เองได้ ถ้าต้องการความเป็นอิสระระดับที่ตรวจสอบย้อนหลังได้
ให้ตั้ง ruleset ที่ **ระดับ organization** ด้วย `install/rulesets/org-independent-audit.json`
และดู [docs/INDEPENDENCE.md](docs/INDEPENDENCE.md) หัวข้อ "การป้องกันการข้ามด่าน"

## สัญญาอนุญาต

[MIT](LICENSE) — เอาไปใช้ ดัดแปลง หรือปรับให้เข้ากับองค์กรของคุณได้
