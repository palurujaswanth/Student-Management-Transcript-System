# Enterprise Academic Management & Transcript Generation System (`ZSTUDENT_APP`)

[![SAP ABAP](https://img.shields.io/badge/SAP-ABAP%207.40%2B-blue.svg)](https://www.sap.com/)
[![UI Technology](https://img.shields.io/badge/UI-Dynpro%20%7C%20OO--ALV-orange.svg)]()
[![Forms](https://img.shields.io/badge/Forms-SAP%20SmartForms-brightgreen.svg)]()
[![Architecture](https://img.shields.io/badge/Architecture-Modular%20Type%20M%20Dialog-lightgrey.svg)]()

An enterprise-grade Student Information and Academic Performance Tracking application developed in **SAP ABAP**. The system integrates **Dialog Module Pool Programming (Dynpro)**, **Interactive Object-Oriented ALV Grid (`CL_GUI_ALV_GRID`)**, and automated **SAP SmartForms** document processing to provide complete lifecycle academic tracking and transcript generation.

---

## 📌 Table of Contents
- [Business Overview](#-business-overview)
- [System Architecture](#-system-architecture)
- [Key Features](#-key-features)
- [Database Schema (DDIC)](#-database-schema-ddic)
- [Program Structure & Modularization](#-program-structure--modularization)
- [Interactive UI Workflow](#-interactive-ui-workflow)
- [SmartForm Integration](#-smartform-integration)
- [Technical Challenges & Architectural Solutions](#-technical-challenges--architectural-solutions)
- [Installation & Deployment](#-installation--deployment)
- [Author & Acknowledgments](#-author--acknowledgments)

---

## 🏢 Business Overview

University and college administration offices frequently struggle with fragmented student records split across disparate legacy tables, manual spreadsheet reporting, and slow transcript dispatch workflows.

**`ZSTUDENT_APP`** eliminates these operational bottlenecks by providing:
1. **Dynamic Student Inquiries:** Rapid lookup via parameter-driven selection screens.
2. **Interactive Academic Performance Cockpits:** Rich grid-based reporting with real-time responsive column fitting and row drill-down capabilities.
3. **Automated Grade & Attendance Analytics:** Drill-down popups into multi-semester subject marks, total scores, and attendance rates.
4. **Instant Transcript Publishing:** Automated print preview, PDF export, and physical spool dispatch of certified academic transcripts via SAP SmartForms.

---

## 🏛 System Architecture

The project follows standard SAP clean-code and modularization principles, separating data definition, flow logic, UI events, and reporting routines into discrete layers:

```
                          ┌───────────────────────────┐
                          │   Transaction Code:       │
                          │      ZSTUDENT_APP         │
                          └─────────────┬─────────────┘
                                        │
                                        ▼
                          ┌───────────────────────────┐
                          │   Screen 100 (Dynpro)     │
                          │   Search & Filter Cockpit │
                          └─────────────┬─────────────┘
                                        │
                      ┌─────────────────┴─────────────────┐
                      │ [Search Data]                     │ [Print Transcript]
                      ▼                                   ▼
        ┌───────────────────────────┐       ┌───────────────────────────┐
        │   Screen 200 (Dynpro)     │       │   SmartForm Driver:       │
        │   Custom Container        │       │   SSF_FUNCTION_MODULE_NAME│
        │   (MY_CONTAINER)          │       └─────────────┬─────────────┘
        └─────────────┬─────────────┘                     │
                      │                                   ▼
        ┌─────────────┴─────────────┐       ┌───────────────────────────┐
        │   CL_GUI_ALV_GRID         │       │ Z_STUDENT_TRANSCRIPT_FORM │
        │   Master Records Summary  │       │ Multi-semester transcript │
        └─────────────┬─────────────┘       └───────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │ [Hotspot / Results Action]│
        ▼                           ▼
┌───────────────────────────────────────────────────┐
│ Dynamic Performance Popup (REUSE_ALV_GRID_DISPLAY)│
│ Semester, Marks, Attendance %, Status             │
└───────────────────────────────────────────────────┘
```

---

## ✨ Key Features

- **Decoupled Screen Navigation:** Screen 100 handles search filters; Screen 200 hosts an active container ALV grid with sticky navigation (`BACK`, `EXIT`, `CANCEL`).
- **Object-Oriented ALV Grid (`CL_GUI_ALV_GRID`):**
  - Full-width container scaling enabled via dynamic coordinate resizing.
  - Custom column configuration through `LVC_T_FCAT` (with explicit `outputlen` preventing text truncation).
  - Clickable hotspot links on primary keys (`ROLLNO`).
- **Context-Aware Event Handling:**
  - Dual-event listener registering `handle_double_click` and `handle_hotspot_click`.
  - Coordinate resolution combining `GET_CURRENT_CELL` and `GET_SELECTED_ROWS` to ensure drill-down popups always reflect the active cursor focus.
- **Dynamic Popup Drilldown:** Instant display of multi-semester data (`ZSTUDENTS_MARKS`) using `REUSE_ALV_GRID_DISPLAY` embedded in modal window coordinates.
- **Official Transcript Generation (SmartForms):**
  - Text node segregation isolating student personal info (`IS_STUDENT_HEADER`) from repeated course results.
  - Multi-column line type (`L_DATA`) aligning column headers with looped rows (`IT_STUDENT_MARKS INTO WA_MARKS`).

---

## 🗄 Database Schema (DDIC)

The application relies on two custom Data Dictionary tables created in transaction `SE11`:

### 1. `ZSTUDENT_MASTER` (Header Table)
| Field Name | Key | Data Element | Domain | Description |
| :--- | :---: | :--- | :--- | :--- |
| `MANDT` | **Key** | `MANDT` | `CLNT` | Client |
| `ROLLNO` | **Key** | `NUMC10` | `NUMC10` | Student Roll Number |
| `FIRSTNAME` | | `CHAR30` | `CHAR30` | First Name |
| `LASTNAME` | | `CHAR30` | `CHAR30` | Last Name |
| `COURSE` | | `CHAR20` | `CHAR20` | Academic Stream (e.g., B.TECH) |
| `GENDER` | | `CHAR1` | `CHAR1` | Gender (M/F) |
| `EMAIL` | | `CHAR50` | `CHAR50` | Student Email Address |

### 2. `ZSTUDENTS_MARKS` (Item / Relational Performance Table)
| Field Name | Key | Data Element | Domain | Description |
| :--- | :---: | :--- | :--- | :--- |
| `MANDT` | **Key** | `MANDT` | `CLNT` | Client |
| `ROLLNO` | **Key** | `NUMC10` | `NUMC10` | Student Roll Number |
| `SEMESTER` | **Key** | `NUMC2` | `NUMC2` | Semester Number (01, 02, ...) |
| `TOT_MARKS` | | `INT4` | `INT4` | Marks Obtained |
| `MAX_MARKS` | | `INT4` | `INT4` | Maximum Marks Possible |
| `ATT_PCT` | | `DEC5_2` | `DEC5_2` | Attendance Percentage |
| `RESULTS` | | `CHAR10` | `CHAR10` | Semester Result (`PASS` / `FAIL`) |

---

## 📂 Program Structure & Modularization

The module pool program **`Z_STUDENT_MAJOR_PROG`** (Type M) is structured across dedicated includes:

```
Z_STUDENT_MAJOR_PROG
 │
 ├── Z_STUDENT_TOP   (Global Data, Type definitions, ALV Objects, Event Handler definition)
 ├── Z_STUDENT_SEL   (PBO & PAI Modules for Screen 100 and Screen 200)
 └── Z_STUDENT_F01   (Subroutines: Data fetch, Container creation, Field catalog, Popup, SmartForm)
```

### File Manifest:
* **`Z_STUDENT_TOP`**: Defines data structures (`ty_stu_master`), work areas, container references (`go_custom_container`, `go_alv_grid`), and local event handler classes (`lcl_event_handler`).
* **`Z_STUDENT_SEL`**:
  * `STATUS_0100` / `USER_COMMAND_0100`: Captures input criteria, invokes search routines, and triggers conditional navigation (`CALL SCREEN 200`).
  * `STATUS_0200` / `USER_COMMAND_0200`: Coordinates menu actions, screen pushbuttons (`BTN_RESULTS`), cursor resolution, and return operations.
* **`Z_STUDENT_F01`**:
  * `build_lvc_fieldcatalog`: Builds column properties using an optimized macro with explicit output lengths.
  * `display_alv_grid`: Instantiates custom container and binds event listeners.
  * `display_marks_popup`: Renders modal drill-down for relational semester data.
  * `trigger_smartform`: Dynamically fetches the generated function module name and invokes SmartForm printing.

---

## 🖥 Interactive UI Workflow

### Screen 100: Input & Parameter Selection
* Input fields for **Roll Number** and **Course**.
* Direct action pushbuttons for:
  * **Search Data:** Retrieves master data and navigates to summary view.
  * **Print Transcript:** Bypasses grid navigation to directly spool the student transcript.
  * **Exit:** Terminates program session.

### Screen 200: Master Record Summary Cockpit
* Embedded Custom Container (`MY_CONTAINER`) rendering `CL_GUI_ALV_GRID`.
* Zebra striping, column auto-adjustment, and primary key hotspot link formatting.
* On-screen action button **`Results`** complementing top application menu commands.

### Semester Results Modal Popup
* Clicking any roll number hotspot or selecting a row and clicking **Results** launches a responsive modal dialog (`REUSE_ALV_GRID_DISPLAY`) isolated to lines 6–18 and columns 15–95.

---

## 🖨 SmartForm Integration (`Z_STUDENT_TRANSCRIPT_FORM`)

The official university transcript is produced via SAP SmartForms:

* **Dynamic FM Resolution:** The driver uses `SSF_FUNCTION_MODULE_NAME` to determine the runtime function module `/1BCDWB/SF*` generated for the form.
* **Typed Interface Contracts:**
  * `IS_STUDENT_HEADER`: Typed directly against DDIC structure `ZSTUDENT_MASTER`.
  * `IT_STUDENT_MARKS`: Typed directly against DDIC table `ZSTUDENTS_MARKS`.
* **Form Layout Structure:**
  * **Header Node:** Static text segment populating student identity metrics (`ROLLNO`, `FIRSTNAME`, `LASTNAME`, `COURSE`, `EMAIL`).
  * **Table Node (`L_DATA`):** 5-column line type defining uniform dimensions for headers and line items:
    * `Semester` (2.5 cm)
    * `Marks Obtained` (3.5 cm)
    * `Max Marks` (3.0 cm)
    * `Attendance %` (3.5 cm)
    * `Result` (3.5 cm)

---

## 🛠 Technical Challenges & Architectural Solutions

| Challenge Encountered | Root Cause | Solution Implemented |
| :--- | :--- | :--- |
| **`CALL_FUNCTION_CONFLICT_TYPE` Runtime Dump** | Program work area `ls_header` was locally defined rather than matching the DDIC type assigned to `IS_STUDENT_HEADER`. | Re-typed `ls_header TYPE zstudent_master` in subroutine `TRIGGER_SMARTFORM`. |
| **Input Value Persistence Failure** | Dynpro was executing PAI commands before transporting updated input field values. | Placed explicit `FIELD <variable>` statements in Screen 100 flow logic prior to module calls. |
| **Active Row Drift on Popup Click** | `get_selected_rows` returned initial indices when users clicked table cells without ticking the row header. | Implemented `get_current_cell` as the primary coordinate locator, falling back to selected rows. |
| **ALV Column Text Truncation** | Default `cwidth_opt` collapsed short text fields to their minimum character length. | Set explicit `outputlen` parameters across all catalog columns and removed restrictive width constraints. |
| **SmartForm Header / Marks Misalignment** | Semester performance items were referenced inside static text blocks outside the Table loop. | Restructured form layout by isolating student header text and rendering academic items inside Table node loops with line type `L_DATA`. |

---

## 🚀 Installation & Deployment

### Prerequisites
* SAP NetWeaver AS ABAP 7.40 or higher / SAP S/4HANA.
* Authorization for transactions: `SE11`, `SE38`, `SE51`, `SE41`, `SE93`, `SMARTFORMS`.
* (Optional) **abapGit** client installed for automated transport.

### Manual Setup via SAP GUI:
1. **Create Database Tables (`SE11`):**
   * Create `ZSTUDENT_MASTER` and `ZSTUDENTS_MARKS` matching the schema specifications above.
   * Maintain test data using `SE16N`.
2. **Import SmartForm (`SMARTFORMS`):**
   * Create form `Z_STUDENT_TRANSCRIPT_FORM`.
   * Configure Form Interface and Table line type `L_DATA`.
   * Activate the form.
3. **Create Module Pool Program (`SE38` / `SE80`):**
   * Create program `Z_STUDENT_MAJOR_PROG` (Type M).
   * Create includes `Z_STUDENT_TOP`, `Z_STUDENT_SEL`, and `Z_STUDENT_F01`.
   * Paste respective source codes from this repository.
4. **Design Screens (`SE51`):**
   * **Screen 0100:** Add Roll No, Course inputs, and pushbuttons (`SEARCH`, `PRINT`, `EXIT`).
   * **Screen 0200:** Add Custom Container `MY_CONTAINER` (enable vertical/horizontal resizing) and button `BTN_RESULTS`.
5. **Create GUI Status (`SE41`):**
   * Create `STATUS_100` and `STATUS_200` with standard function keys (`BACK`, `EXIT`, `CANCEL`) and custom button codes (`RESULTS`, `PRINT_SF`).
6. **Assign Transaction Code (`SE93`):**
   * Create transaction `ZSTUDENT_APP` linked to Program `Z_STUDENT_MAJOR_PROG` and starting screen `0100`.
   * Check GUI attributes (SAP GUI for Windows/HTML).

---

## 👨‍💻 Author & Contributions
* **Developer:** Paluru Jaswanth Pranay
* **Specialization:** SAP ABAP Development, Dynpro UI Engineering, S/4HANA Migrations
