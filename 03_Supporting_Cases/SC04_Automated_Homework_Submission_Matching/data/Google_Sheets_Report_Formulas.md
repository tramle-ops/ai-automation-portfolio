# Google Sheets Report Formulas

The formulas below are native Google Sheets formulas. Use them after uploading the sample workbook to Google Sheets; do not rely on the Excel compatibility wrappers in the `.xlsx` export.

The formulas assume these columns in `Submissions`:

| Column | Field |
|---|---|
| B | `assignment_id` |
| H | `normalized_class_code` |
| I | `score` |
| J | `match_status` |
| M | `matched_student_id` |

They also assume `Student_Master` uses A–E for `student_id`, `official_name`, `normalized_name`, `class_code`, and `active`.

## 1. Student_Report

1. Clear the current contents of `Student_Report`.
2. Put `official_name` in `F1`. The formula below creates the headers in A–E itself.
3. Paste this formula into `A1`. It spills across columns A–E and includes only resolved submissions.

```gs
=IFERROR(
  QUERY(
    FILTER(
      {Submissions!B2:B,Submissions!M2:M,Submissions!H2:H,Submissions!I2:I},
      (Submissions!J2:J="AUTO_MATCHED")+(Submissions!J2:J="MANUALLY_MATCHED"),
      Submissions!M2:M<>""
    ),
    "select Col1, Col2, Col3, count(Col4), max(Col4)
     group by Col1, Col2, Col3
     label Col1 'assignment_id',
           Col2 'student_id',
           Col3 'class_code',
           count(Col4) 'attempt_count',
           max(Col4) 'highest_score'",
    0
  ),
  ""
)
```

4. Paste this formula into `F2`. It automatically fills the official name for every reported student.

```gs
=ARRAYFORMULA(IF(B2:B="",,IFNA(VLOOKUP(B2:B,Student_Master!A:B,2,FALSE))))
```

## 2. Class_Report

1. Clear the current contents of `Class_Report`.
2. Put `active_student_count` in `F1` and `completion_rate` in `G1`. The formula below creates the headers in A–E itself.
3. Paste this formula into `A1`. It spills across columns A–E.

```gs
=IFERROR(
  QUERY(
    Student_Report!A2:E,
    "select A, C, count(B), avg(E), max(E)
     where A is not null
     group by A, C
     label A 'assignment_id',
           C 'class_code',
           count(B) 'submitted_student_count',
           avg(E) 'average_score',
           max(E) 'highest_score'",
    0
  ),
  ""
)
```

4. Paste this formula into `F2`. It counts active students in each class and spills down automatically.

```gs
=MAP(B2:B,LAMBDA(class_code,IF(class_code="",,COUNTIFS(Student_Master!D:D,class_code,Student_Master!E:E,TRUE))))
```

5. Paste this formula into `G2`. It calculates the completion rate for each reported class.

```gs
=ARRAYFORMULA(IF(B2:B="",,IFERROR(C2:C/F2:F,)))
```

## 3. Verify the reports

- `Student_Report` must exclude `PENDING_REVIEW`, `RESUBMISSION_REQUIRED`, and `REJECTED` rows.
- `attempt_count` and `highest_score` must aggregate by assignment, matched student, and class.
- `Class_Report.submitted_student_count` must equal the number of resolved students in `Student_Report`.
- `completion_rate` must equal `submitted_student_count / active_student_count`.
