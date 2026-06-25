# Bug Reports — MERN Employee Salary Management

These bugs were found through manual exploratory testing of the live application
(forked from bluedone/mern-employee-salary-management), logged in as an admin user
seeded directly into the data_pegawai table.

---

## BUG-001: Total Gaji calculation incorrect for employee Gilbert Hutapea

**Severity:** High
**Module:** Transaksi -> Data Gaji (Payroll)
**Status:** Open

### Steps to Reproduce
1. Log in as admin
2. Navigate to Transaksi -> Data Gaji
3. Filter: Bulan = April, Tahun = 2023
4. Click "Tampilkan Data"
5. Observe the row for employee Gilbert Hutapea (NIK 1218052110020002)

### Expected Result
Total Gaji = Gaji Pokok + Tunjangan Transport + Uang Makan - Potongan
4,000,000 + 600,000 + 400,000 - 100,000 = 4,900,000

### Actual Result
Total Gaji displays as Rp. 5,900,000 - exactly Rp. 1,000,000 higher than the correct value.

### Verification Against Other Rows
| Employee        | Gaji Pokok | Transport | Makan   | Potongan | Expected  | Actual    | Match? |
|------------------|-----------:|----------:|--------:|---------:|----------:|----------:|:------:|
| Gilbert Hutapea  | 4,000,000  | 600,000   | 400,000 | 100,000  | 4,900,000 | 5,900,000 | NO  |
| Layla Siregar    | 2,500,000  | 300,000   | 200,000 | 0        | 3,000,000 | 3,000,000 | YES |
| Zilong Sibarani  | 2,200,000  | 300,000   | 200,000 | 100,000  | 2,600,000 | 2,600,000 | YES |
| Nana Silaban     | 2,500,000  | 300,000   | 200,000 | 0        | 3,000,000 | 3,000,000 | YES |

This rules out a universal formula bug. The error is specific to this one record,
which is more concerning than a systemic bug because it cannot be caught by
reviewing the calculation logic in code alone; it suggests a data-level or
update-path issue.

### Impact
This directly mirrors the core risk in the business scenario: a wrong salary
calculation means an employee is paid an incorrect amount with no visible flag
in the UI. Because this only affects one record, it is the kind of error most
likely to go unnoticed in a real payroll run.

### Suggested Test Coverage
An automated test that, for every row returned by the Data Gaji API, asserts
total_gaji === gaji_pokok + tunjangan_transport + uang_makan - potongan.
This is exactly the kind of test that would catch this bug immediately and
should run in CI on every change to payroll-related code.

---

## BUG-002: Edit (pencil) button on Data Gaji table is non-functional

**Severity:** Medium
**Module:** Transaksi -> Data Gaji (Payroll)
**Status:** Open

### Steps to Reproduce
1. Log in as admin
2. Navigate to Transaksi -> Data Gaji
3. Filter and display any month's data
4. Click the pencil/edit icon in the Actions column for any row

### Expected Result
Clicking the edit icon should open a form or modal allowing the admin to
view and correct that employee's salary record.

### Actual Result
Nothing happens. No modal opens, no navigation occurs, no visible error or
loading state appears.

### Impact
Without a working edit function, there is no way for an admin to fix a
known-bad record (such as BUG-001) through the UI. The only way to correct
it would be direct database access, bypassing any validation the app provides.

### Notes
- Delete icon was not tested for the same issue; should be checked separately.
- Browser console showed no JavaScript errors at the time of testing.

### Suggested Test Coverage
An E2E test that clicks the edit action and asserts that a modal/form becomes
visible. This is a simple, high-value test per the assignment's stated
philosophy of protecting specific people from specific failures.