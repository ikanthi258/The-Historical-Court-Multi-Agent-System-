# Historical Court Agent System

## 📋 Concept

**Historical Court** เป็นระบบ Multi-Agent ที่จำลองกระบวนการพิจารณาคดีในศาล โดยนำบุคคลหรือเหตุการณ์ทางประวัติศาสตร์มา "ตั้งโต๊ะ" ผ่านการค้นคว้าข้อมูลทั้งด้านบวกและด้านลบอย่างเป็นระบบ จากนั้นสรุปเป็นรายงานที่เป็นกลางและสมดุล

ระบบนี้ออกแบบมาเพื่อให้ AI ทำงานร่วมกันในบทบาทที่แตกต่างกัน เหมือนกับการทำงานของศาลจริง - มีทั้งฝ่ายจำเลย ฝ่ายโจทก์ ผู้พิพากษา และเสมียนศาล

---

## 🏛️ System Architecture

```
User Input
    ↓
┌─────────────────────────────────────────┐
│ Root Agent (Court Clerk)                │
│ - รับชื่อบุคคล/เหตุการณ์จาก User        │
│ - เริ่มกระบวนการพิจารณา                 │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ Sequential Agent (Court System)          │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │ Loop Agent (Trial Process)         │ │
│  │ ⟳ Max 4 iterations                 │ │
│  │                                    │ │
│  │  ┌──────────────────────────────┐ │ │
│  │  │ Parallel Agent (Investigation)│ │ │
│  │  │                               │ │ │
│  │  │  ┌────────────────────────┐  │ │ │
│  │  │  │ Admirer Agent          │  │ │ │
│  │  │  │ (หาข้อมูลด้านบวก)      │  │ │ │
│  │  │  └────────────────────────┘  │ │ │
│  │  │           ∥                  │ │ │
│  │  │  ┌────────────────────────┐  │ │ │
│  │  │  │ Critic Agent           │  │ │ │
│  │  │  │ (หาข้อมูลด้านลบ)       │  │ │ │
│  │  │  └────────────────────────┘  │ │ │
│  │  └──────────────────────────────┘ │ │
│  │                ↓                   │ │
│  │  ┌──────────────────────────────┐ │ │
│  │  │ Judge Agent                  │ │ │
│  │  │ - ตรวจสอบความสมดุล           │ │ │
│  │  │ - ตัดสินใจว่าจะค้นหาต่อหรือไม่│ │ │
│  │  └──────────────────────────────┘ │ │
│  └────────────────────────────────────┘ │
│                ↓                         │
│  ┌────────────────────────────────────┐ │
│  │ Verdict Writer Agent               │ │
│  │ - เขียนรายงานสรุปที่เป็นกลาง       │ │
│  │ - บันทึกเป็นไฟล์                   │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
    ↓
Final Report (court_records/{subject}.txt)
```

---

## 🎭 Agent Roles & Responsibilities

### 1. **Root Agent: Historical Court Clerk**
- **บทบาท**: เสมียนศาลที่ต้อนรับและเริ่มกระบวนการพิจารณา
- **หน้าที่**:
  - ทักทายผู้ใช้และอธิบายระบบ
  - รับชื่อบุคคล/เหตุการณ์ที่ต้องการตรวจสอบ
  - บันทึกชื่อลงใน state (`PROMPT`)
  - ส่งต่อให้ Court System ดำเนินการ
- **Tools**: `append_to_state`

---

### 2. **Sequential Agent: Court System**
ทำหน้าที่เป็น orchestrator ที่ควบคุมลำดับการทำงาน:
1. Trial Process (Loop) → ค้นคว้าและตรวจสอบ
2. Verdict Writer → เขียนรายงาน

---

### 3. **Loop Agent: Trial Process**
- **บทบาท**: กระบวนการพิจารณาคดีที่ซ้ำได้
- **โครงสร้าง**:
  - วนลูปสูงสุด 4 รอบ
  - แต่ละรอบประกอบด้วย: Investigation (Parallel) → Judge Review
- **หยุดเมื่อ**: Judge Agent เรียกใช้ `exit_loop` tool

---

### 4. **Parallel Agent: Investigation Team**
ประกอบด้วย 2 agents ที่ทำงานพร้อมกัน:

#### **4.1 Admirer Agent (ฝ่ายจำเลย)**
- **บทบาท**: นักประวัติศาสตร์ที่เน้นค้นหาผลงานและความดี
- **หน้าที่**:
  - ค้นหาความสำเร็จ ผลงาน รางวัล และมรดกทางบวก
  - ตอบสนองต่อคำสั่งเฉพาะจาก Judge (ถ้ามี)
  - บันทึกผลลงใน `pos_data`
- **Tools**: `wikipedia`, `append_to_state`
- **Output**: 3-5 จุดสำคัญพร้อมหลักฐานและตัวอย่างเฉพาะ

#### **4.2 Critic Agent (ฝ่ายโจทก์)**
- **บทบาท**: นักสืบที่เน้นค้นหาข้อถกเถียงและข้อวิพากษ์วิจารณ์
- **หน้าที่**:
  - ค้นหาข้อขัดแย้ง ความล้มเหลว การวิพากษ์วิจารณ์ และผลกระทบด้านลบ
  - ตอบสนองต่อคำสั่งเฉพาะจาก Judge (ถ้ามี)
  - บันทึกผลลงใน `neg_data`
- **Tools**: `wikipedia`, `append_to_state`
- **Output**: 3-5 จุดสำคัญพร้อมหลักฐานและตัวอย่างเฉพาะ

---

### 5. **Judge Agent (ผู้พิพากษา)**
- **บทบาท**: ผู้ตัดสินที่เป็นกลาง ตรวจสอบความสมดุลของข้อมูล
- **หน้าที่**:
  - ประเมินคุณภาพและปริมาณของข้อมูลจาก `pos_data` และ `neg_data`
  - **ตัดสินใจ**:
    - **ถ้าข้อมูลยังไม่เพียงพอ**: ส่งคำสั่งเฉพาะไปยัง Admirer/Critic ผ่าน `judge_feedback`
    - **ถ้าข้อมูลเพียงพอแล้ว** (3+ จุดที่มีคุณภาพต่อฝ่าย): เรียก `exit_loop` เพื่อจบการสืบสวน
- **เกณฑ์การตัดสิน**:
  - แต่ละฝ่ายมีข้อมูลอย่างน้อย 3-5 จุดที่ชัดเจน
  - ข้อมูลมีหลักฐานและตัวอย่างเฉพาะ
  - ทั้งสองมุมมองได้รับการนำเสนออย่างเหมาะสม
- **Tools**: `append_to_state`, `exit_loop`

---

### 6. **Verdict Writer Agent (เสมียนศาลผู้เขียนคำพิพากษา)**
- **บทบาท**: ผู้จดบันทึกและเขียนรายงานสรุปขั้นสุดท้าย
- **หน้าที่**: เขียนรายงานประวัติศาสตร์ที่เป็นกลางและครบถ้วน
- **โครงสร้างรายงาน**:
  1. **Introduction** - แนะนำบุคคล/เหตุการณ์และความสำคัญ
  2. **Case for the Defense** - นำเสนอหลักฐานด้านบวก
  3. **Case for the Prosecution** - นำเสนอหลักฐานด้านลบ
  4. **The Verdict** - สรุปมุมมองทางประวัติศาสตร์อย่างสมดุล
- **Output**: ไฟล์ `.txt` ใน `court_records/{subject}.txt`
- **Tools**: `write_file`

---

## 🔄 Workflow

```
1. User เข้าสู่ระบบ
   ↓
2. Court Clerk ทักทายและขอชื่อบุคคล/เหตุการณ์
   ↓
3. บันทึกชื่อลง state (PROMPT)
   ↓
4. เริ่ม Trial Process (Loop)
   │
   ├─→ [Iteration 1-4]
   │   │
   │   ├─→ Investigation Team (Parallel)
   │   │   ├─→ Admirer ค้นหาข้อมูลบวก → pos_data
   │   │   └─→ Critic ค้นหาข้อมูลลบ → neg_data
   │   │
   │   └─→ Judge ตรวจสอบข้อมูล
   │       ├─→ ไม่เพียงพอ? → feedback → loop ต่อ
   │       └─→ เพียงพอ? → exit_loop
   │
5. Verdict Writer เขียนรายงานสรุป
   ↓
6. บันทึกไฟล์ใน court_records/
   ↓
7. เสร็จสิ้น
```

---

## 🗂️ State Management

ระบบใช้ **shared state** เพื่อส่งข้อมูลระหว่าง agents:

| State Key | ผู้เขียน | ผู้อ่าน | รูปแบบข้อมูล |
|-----------|---------|---------|--------------|
| `PROMPT` | Court Clerk | ทุก Agent | String - ชื่อบุคคล/เหตุการณ์ |
| `pos_data` | Admirer Agent | Judge, Verdict Writer | List[String] - หลักฐานด้านบวก |
| `neg_data` | Critic Agent | Judge, Verdict Writer | List[String] - หลักฐานด้านลบ |
| `judge_feedback` | Judge Agent | Admirer, Critic | List[String] - คำสั่งเฉพาะจาก Judge |

---

## 🛠️ Tools Used

| Tool | ใช้โดย Agent | ความสามารถ |
|------|-------------|-----------|
| `wikipedia` | Admirer, Critic | ค้นหาข้อมูลจาก Wikipedia |
| `append_to_state` | ทุก Agent | เพิ่มข้อมูลลงใน state |
| `exit_loop` | Judge | สั่งหยุด Loop Agent |
| `write_file` | Verdict Writer | บันทึกรายงานเป็นไฟล์ |

---

## 🎯 Key Design Principles

### 1. **Separation of Concerns**
แต่ละ Agent มีหน้าที่ชัดเจนและไม่ทับซ้อนกัน

### 2. **Parallel Processing**
Admirer และ Critic ทำงานพร้อมกัน เพื่อประหยัดเวลา

### 3. **Iterative Refinement**
Judge Agent สามารถสั่งให้ค้นหาข้อมูลเพิ่มเติมได้หลายรอบ จนกว่าจะสมดุล

### 4. **Neutral Output**
Verdict Writer ไม่มีอคติ - นำเสนอทั้งสองมุมมองอย่างเท่าเทียมกัน

### 5. **Quality Control**
Judge Agent ทำหน้าที่เป็น gatekeeper ที่ตรวจสอบคุณภาพก่อนส่งต่อ

---

## 📊 Example Output Structure

```
court_records/
└── Napoleon_Bonaparte.txt
    ├── Introduction (2-3 paragraphs)
    ├── Case for the Defense (3-4 paragraphs)
    ├── Case for the Prosecution (3-4 paragraphs)
    └── The Verdict (2-3 paragraphs)
```

---

## 💡 Use Cases

- **การศึกษาประวัติศาสตร์**: ให้มุมมองที่สมดุลเกี่ยวกับบุคคลสำคัญ
- **การวิเคราะห์เหตุการณ์**: ประเมินเหตุการณ์ทางประวัติศาสตร์จากหลายมุม
- **การสอนวิจารณญาณ**: แสดงให้เห็นว่าบุคคล/เหตุการณ์มักมีทั้งด้านดีและด้านร้าย
- **Research Assistant**: ช่วยรวบรวมข้อมูลเบื้องต้นสำหรับการเขียนบทความ

---

## 🚀 Advantages of This Architecture

1. **Balanced Perspective**: การแบ่ง Agent เป็น Admirer/Critic รับประกันว่าจะมีข้อมูลทั้งสองด้าน
2. **Quality Assurance**: Judge Agent ป้องกันการสรุปที่เร็วเกินไปหรือไม่สมดุล
3. **Scalability**: สามารถเพิ่ม Agent ใหม่ (เช่น Neutral Observer) ได้ง่าย
4. **Transparency**: การแบ่งบทบาทชัดเจน ตรวจสอบและ debug ได้ง่าย
5. **Efficiency**: Parallel processing ทำให้ประหยัดเวลา

---

## 📋 Requirements

```bash
# Python packages
google-adk
google-genai
google-cloud-logging
python-dotenv
langchain-community
wikipedia
```

---

## ⚙️ Configuration

สร้างไฟล์ `.env` และกำหนดค่า:

```env
MODEL=gemini-2.0-flash-exp
```

---

## 🚀 How to Run

```bash
# Install dependencies
pip install -r requirements.txt

# Run the system
python main.py
```

---

## 📝 Example Interaction

```
Court Clerk: Welcome to The Historical Court! 
             This court examines historical figures by investigating 
             both their achievements and controversies.
             
             Please name a historical figure or event you wish to put on trial.

User: Napoleon Bonaparte

Court Clerk: The court proceedings for Napoleon Bonaparte are now beginning...

[Investigation Phase - Round 1]
Admirer Agent: Searching for achievements...
Critic Agent: Searching for controversies...

Judge Agent: Reviewing evidence... Need more specific information about 
             military campaigns. Requesting additional research.

[Investigation Phase - Round 2]
Admirer Agent: Found details on military innovations...
Critic Agent: Found information on Russian campaign failures...

Judge Agent: Evidence is now sufficient. Proceeding to verdict.

Verdict Writer: Writing comprehensive report...

✓ Report saved to: court_records/Napoleon_Bonaparte.txt
```

---

## 🔍 Technical Details

### Agent Types Used

- **Agent**: Single autonomous agent with specific role
- **SequentialAgent**: Executes sub-agents in sequence
- **ParallelAgent**: Executes sub-agents concurrently
- **LoopAgent**: Repeats sub-agents until exit condition met

### State Flow

```python
{
  "PROMPT": "Napoleon Bonaparte",
  "pos_data": [
    "Military genius who reformed...",
    "Created the Napoleonic Code..."
  ],
  "neg_data": [
    "Failed Russian campaign led to...",
    "Reinstated slavery in colonies..."
  ],
  "judge_feedback": [
    "Need more details on economic policies"
  ]
}
```

---

## 🛡️ Error Handling

- **Max Iterations**: Loop จำกัดที่ 4 รอบเพื่อป้องกันการวนไม่สิ้นสุด
- **State Validation**: ตรวจสอบว่า state เป็น list ก่อนเพิ่มข้อมูล
- **File Writing**: สร้าง directory อัตโนมัติถ้ายังไม่มี

---

## 🔮 Future Enhancements

- [ ] เพิ่ม **Neutral Observer Agent** สำหรับข้อมูลที่เป็นกลาง
- [ ] รองรับการค้นหาจากแหล่งข้อมูลหลายแหล่ง (ไม่เฉพาะ Wikipedia)
- [ ] ส่งออกรายงานในรูปแบบ PDF/HTML
- [ ] เพิ่ม Web UI สำหรับการใช้งาน
- [ ] รองรับภาษาอื่นๆ นอกจากภาษาอังกฤษ

---

**จัดทำโดย**: Kanthi Phrakhienthong
**Version**: 1.0  
**Last Updated**: February 2024  
**License**: MIT
