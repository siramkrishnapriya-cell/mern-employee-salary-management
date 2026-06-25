# Bug Reports — MERN Employee Salary Management

These bugs were found through manual exploratory testing of the live application
(forked from bluedone/mern-employee-salary-management), logged in as an admin user
seeded directly into the `data_pegawai` table.

---

## BUG-001: Total Gaji calculation incorrect for employee Gilbert Hutapea

**Severity:** High
**Module:** Transaksi → Data Gaji (Payroll)
**Status:** Open

### Steps to Reproduce
1. Log in as admin
2. Navigate to **Transaksi → Data Gaji**
3. Filter: Bulan = April, Tahun = 2023
4. Click "Tampilkan Data"
5. Observe the row for employee **Gilbert Hutapea** (NIK 1218052110020002)

### Expected Result
Total Gaji = Gaji Pokok + Tunjangan Transport + Uang Makan − Potongan

```
4,000,000 + 600,000 + 400,000 − 100,000 = 4,900,000
```

### Actual Result
Total Gaji displays as **Rp. 5,900,000** — exactly **Rp. 1,000,000 higher**
than the correct value.

### Verification Against Other Rows
The same formula was checked against the other 3 visible rows in the same
table view, and all three calculate correctly:

| Employee        | Gaji Pokok | Transport | Makan   | Potongan | Expected  | Actual    | Match? |
|------------------|-----------:|----------:|--------:|---------:|----------:|----------:|:------:|
| Gilbert Hutapea  | 4,000,000  | 600,000   | 400,000 | 100,000  | 4,900,000 | 5,900,000 | ❌ NO  |
| Layla Siregar    | 2,500,000  | 300,000   | 200,000 | 0        | 3,000,000 | 3,000,000 | ✅     |
| Zilong Sibarani  | 2,200,000  | 300,000   | 200,000 | 100,000  | 2,600,000 | 2,600,000 | ✅     |
| Nana Silaban     | 2,500,000  | 300,000   | 200,000 | 0        | 3,000,000 | 3,000,000 | ✅     |

This rules out a universal formula bug — the error is specific to this one
record, which is more concerning than a systemic bug because it cannot be
caught by simply reviewing the calculation logic in code; it suggests a
data-level or update-path issue (e.g., a field was edited once and the total
was not recalculated, or a duplicate/stale value is being summed in).

### Impact
This directly mirrors the core risk the business scenario describes: a wrong
salary calculation means an employee is paid an incorrect amount with no
visible flag in the UI. Because this only affects one record, it is the kind
of error that is most likely to go unnoticed in a real payroll run — a
uniform bug affecting everyone would be caught immediately by complaints;
a single silent overpayment of this kind may not be caught until a financial
audit or reconciliation.

### Suggested Test Coverage
- An automated test that, for every row returned by the Data Gaji API,
  asserts `total_gaji === gaji_pokok + tunjangan_transport + uang_makan - potongan`.
  This is exactly the kind of test that would have caught this bug
  immediately and should run in CI on every change to payroll-related code.

---

## BUG-002: Edit (pencil) button on Data Gaji table is non-functional

**Severity:** Medium
**Module:** Transaksi → Data Gaji (Payroll)
**Status:** Open

### Steps to Reproduce
1. Log in as admin
2. Navigate to **Transaksi → Data Gaji**
3. Filter and display any month's data
4. Click the pencil/edit icon in the **Actions** column for any row
   (tested on both a correctly-calculated row and the row affected by BUG-001)

### Expected Result
Clicking the edit icon should open a form or modal allowing the admin to
view and correct that employee's salary record.

### Actual Result
Nothing happens. No modal opens, no navigation occurs, and no visible error
or loading state appears. The click registers (cursor behavior is normal)
but produces no observable effect.

### Impact
Without a working edit function, there is no way for an admin to fix a
known-bad record through the UI — including the exact error documented in
BUG-001. The only way to correct it would be direct database access, which
bypasses any validation or audit trail the application might otherwise
provide. A broken "fix it" path makes every other data-integrity bug in
this system effectively permanent from the UI's perspective.

### Notes
- The delete (trash can) icon was not tested for the same issue; this should
  be checked separately, since if delete also fails silently, it points to a
  broader pattern of dead action handlers rather than an edit-specific bug.
- Browser console showed no JavaScript errors at the time of testing,
  suggesting either the onClick handler is not wired to any function, or
  it silently catches/swallows an error without surfacing it — both are
  worth flagging to a developer as the next debugging step.

### Suggested Test Coverage
- An E2E test that clicks the edit action and asserts that a modal/form
  becomes visible. This is a simple test to write and would have caught
  this regression immediately — exactly the kind of low-effort, high-value
  test the assignment brief asks for ("a test suite of 10 tests that each
  protect a specific person from a specific failure").
