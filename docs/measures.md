# Measure dictionary

> Trích từ định nghĩa TMDL của model (không công bố trong repo này).

Tổng số measure: **138** (134 trong bảng `A - Measure`, 4 trong các bảng tham số). Dưới đây chỉ trích 7 measure để minh họa kỹ thuật; phần còn lại không công bố.

## Measures per display folder

| Display folder | Measures |
|---|---|
| 5. Lớp BU | 20 |
| 4. Lượt học\4.2. Vào lớp | 16 |
| 6. Đặt lịch GS\6.1. Chiều lớp | 16 |
| 1. Xếp lớp | 14 |
| 3. Khai giảng\3.1. Hủy | 14 |
| 4. Lượt học\4.1. Hủy | 12 |
| 3. Khai giảng | 8 |
| 2. ACS | 7 |
| 8. Mở lớp\8.1. Mở lớp theo KPI | 5 |
| 8. Mở lớp\8.2. Mở lớp theo vận hành | 5 |
| 8. Mở lớp\8.3. Switch | 5 |
| (bảng what-if parameter) | 4 |
| 6. Đặt lịch GS\6.2. Chiều GS | 4 |
| 4. Lượt học | 3 |
| 7. Đổi CN | 3 |
| 0. Other | 1 |
| 8. Mở lớp | 1 |

## Ngưỡng tách khỏi DAX

> Lưu ý: bảng `dim_policy` chỉ có trong bản ẩn danh dùng để chụp ảnh và trích tài liệu. Ở model gốc, ngưỡng được viết trực tiếp trong DAX; giá trị ngưỡng ở đây chỉ là giá trị minh họa.

### số GS regular đạt điều kiện

Gia sư regular đạt đủ ba điều kiện về số ca; ngưỡng đọc từ `dim_policy` bằng LOOKUPVALUE và giữ trong VAR trước khi vào CALCULATE.

Format: `#,0`

```dax
VAR _min_total = CALCULATE(LOOKUPVALUE(dim_policy[policy_value], dim_policy[policy_key], "regular_min_total_slots"), REMOVEFILTERS(dim_policy))
VAR _min_evening = CALCULATE(LOOKUPVALUE(dim_policy[policy_value], dim_policy[policy_key], "regular_min_evening_slots"), REMOVEFILTERS(dim_policy))
VAR _min_required = CALCULATE(LOOKUPVALUE(dim_policy[policy_value], dim_policy[policy_key], "regular_min_required_slots"), REMOVEFILTERS(dim_policy))
RETURN
CALCULATE(
    [số GS regular active],
    FILTER(
        VALUES(fact_schedule_class[teacher_id]),
        CALCULATE(COUNT(fact_schedule_class[id]), fact_schedule_class[IS_SCHEDULE_REQUIRED] = 1) >= _min_required
    ),
    FILTER(
        VALUES(fact_schedule_class[teacher_id]),
        CALCULATE(COUNT(fact_schedule_class[id]), fact_schedule_class[evening_shift] = 1) >= _min_evening
    ),
    FILTER(
        VALUES(fact_schedule_class[teacher_id]),
        CALCULATE(COUNT(fact_schedule_class[id])) >= _min_total
    )
)
```

## Tham số what-if

### số lớp bị backup x+ lần

Tham số what-if cho ngưỡng số lần backup; SUMMARIZECOLUMNS theo tháng x lớp rồi lọc theo ngưỡng.

Format: `#,0`

```dax
VAR _slt = SELECTEDVALUE(para_backup[para_backup], 3)
RETURN
COUNTROWS(
    FILTER(
        SUMMARIZECOLUMNS(
            fact_regular_session[learn_month],
            fact_regular_session[class_code],

            "total_session",
                CALCULATE(
                    DISTINCTCOUNT(fact_regular_session[class_session_code]),
                    fact_regular_session[session_status] = "COMPLETED",
                    fact_regular_session[student_status] IN {
                        "ATTENDED",
                        "MISSED_BY_STUDENT"
                    }
                ),

            "total_backup_session",
                CALCULATE(
                    DISTINCTCOUNT(fact_regular_session[class_session_code]),
                    fact_regular_session[is_backup] = 1,
                    fact_regular_session[session_status] = "COMPLETED",
                    fact_regular_session[student_status] IN {
                        "ATTENDED",
                        "MISSED_BY_STUDENT"
                    }
                )
        ),
        [total_backup_session] >= _slt
            && NOT ISBLANK(fact_regular_session[class_code])
    )
)
```

## Bảng ảo và iterator

### số HS nghỉ nhiều buổi trong tháng

Bảng ảo bằng SUMMARIZE + FILTER, đếm số HS vượt ngưỡng nghỉ (ngưỡng từ `dim_policy`).

Format: `#,0`

```dax
VAR _max_absence = CALCULATE(LOOKUPVALUE(dim_policy[policy_value], dim_policy[policy_key], "student_absence_alert_over"), REMOVEFILTERS(dim_policy))
VAR _student_cancel =
    SUMMARIZE(
        FILTER(
            fact_regular_session,
            fact_regular_session[student_status] <> "ATTENDED"
            && CONTAINSSTRING(fact_regular_session[session_status], "COMPLETED")
        ),
        fact_regular_session[student_id],
        "cancel_cnt", COUNTROWS(fact_regular_session)
    )
RETURN
COUNTROWS(
    FILTER(
        _student_cancel,
        [cancel_cnt] > _max_absence
    )
)
```

## Time intelligence

### V_ACS vs Last Month

Nhãn text ▲/▼ kèm % thay đổi so với tháng trước, dùng cho thẻ KPI.

```dax
VAR lastMonth =
    CALCULATE(
        [ACS],
        DATEADD('dim_date'[report_date], -1, MONTH)
    )
VAR _dif =
    DIVIDE([ACS] - lastMonth, lastMonth)
VAR _format =
    FORMAT(ABS(_dif), "0.00%")
RETURN
SWITCH(
    TRUE(),
    ISBLANK(lastMonth), BLANK(),
    _dif >= 0, "▲ " & _format & " vs Last Month",
    _dif < 0, "▼ " & _format & " vs Last Month",
    BLANK()
)
```

## Field switch và dynamic format string

### switch_ACS

Hand-built field switch: slicer trên bảng `switch` chọn chỉ số; formatStringDefinition đổi định dạng theo chỉ số được chọn.

```dax
SWITCH(
    SELECTEDVALUE('switch'[switch])
    , "Buổi học", [số buổi học thực tế]
    , "Lượt học", [số lượt học thực tế]
    , "ACS", [ACS]
)
```

Dynamic format string:

```dax
SWITCH(
    SELECTEDVALUE('switch'[switch]),
    "Buổi học", "#,0",
    "Lượt học", "#,0",
    "ACS", "#,0.00",
    "#,0"
)
```

## KPI so với mục tiêu (REMOVEFILTERS)

### % lớp thực đạt

Tỷ lệ lớp thực tế trên tổng toàn mô hình; REMOVEFILTERS(dim_level_teacher) cho mẫu số.

Format: `0.00%;-0.00%;0.00%`

```dax
VAR _act =
    [số lớp đang vận hành]
VAR _total_model =
    CALCULATE(
        [số lớp đang vận hành],
        REMOVEFILTERS(dim_level_teacher)
    )
RETURN
DIVIDE(_act, _total_model, BLANK())
```

## Tỷ lệ cơ bản

### ACS

Số lượt học bình quân trên mỗi buổi học (DIVIDE an toàn).

Format: `0.00`

```dax
DIVIDE([số lượt học thực tế], [số buổi học thực tế], BLANK())
```
