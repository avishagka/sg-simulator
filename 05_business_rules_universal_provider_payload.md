# Business Rules - UniversalProviderDisplayPayload

מסמך זה מרכז את החוקים העסקיים שעל המפתח לממש ביצירת פלט ה-Agent. החוקים מבוססים על הסימולטור המעודכן `agent_universal_payload_simulator.html`.

## עיקרון ארכיטקטוני

כל ה"מוח" נמצא ב-Orchestrator/Agent:

- איסוף חוקים עסקיים.
- איסוף תוכן לתצוגה.
- מיפוי JSON מקורי לפלט אחיד.
- מיון.
- החלטה מה גלוי.
- החלטה איפה רכיב מוצג.
- החלטה איזו פעולה זמינה לכל אמצעי קשר.
- הכנה עתידית לאומברקו.

הקליינט הוא שכבת רינדור. הוא לא אמור לשחזר חוקים עסקיים.

## תחולת v1

| נושא | החלטה |
|---|---|
| סוג ישות נתמך | רופאים בלבד (`doctor`). |
| סוגי ישות עתידיים | מרפאות, מכונים, סדנאות, בתי מרקחת ועוד. |
| מבנה פלט | אחיד לכל הקליינטים. |
| תוכן | hardcoded ב-Agent/Orchestrator. |
| Umbraco | לא פעיל ב-v1, אבל הפלט צריך לתמוך בו באמצעות `contentKeys`. |
| בקשת קליינט | מינימלית. endpoint קובע את סוג הקליינט. |
| locale | לא נשלח בבקשה; מוחזר ב-response metadata. |

## חוקי endpoint

1. לכל קליינט יש endpoint נפרד.
2. הבקשה לא חייבת לכלול `clientContext`.
3. הבקשה לא חייבת לכלול `locale`.
4. הבקשה לא חייבת לכלול `responseOptions`.
5. ה-Agent מחזיר ב-`metadata.client` את הקליינט שזוהה לפי endpoint לצורכי debug, תצוגה ובקרה.

טבלת endpoint:

| Endpoint | clientType | platform | channel |
|---|---|---|---|
| אפליקציה | `mobile` | `app` | `memberApp` |
| אונליין | `web` | `web` | `online` |
| מל"ה | `internalSystem` | `web` | `mlh` |
| מדריך השירותים | `web` | `web` | `serviceGuide` |

## חוקי חוזה פלט

1. הפלט נקרא `UniversalProviderDisplayPayload`.
2. הפלט כולל ארבעה חלקים ראשיים: `metadata`, `entity`, `presentation`, `components`.
3. אותו payload משמש גם לתוצאות חיפוש וגם לפרטי רופא.
4. מסך תוצאות חיפוש משתמש ב-`presentation.searchResult.componentIds` כסדר תצוגה מחייב.
5. מסך פרטי רופא משתמש ב-`presentation.details.layoutPolicy=clientManaged` וב-`presentation.details.availableComponentIds` כרשימת רכיבי מידע זמינים בלבד.
6. רכיב שאינו רלוונטי נשאר בפלט עם `isVisible=false` כשיש בכך ערך חוזי/עתידי.
7. הקליינט לא צריך להכיר את ה-JSON המקורי.
8. הקליינט לא צריך לדעת איזה שדה מקור מזין איזה רכיב.

## חוקי הצגת רכיבים

1. בתוצאות חיפוש הקליינט מציג רכיבים לפי הסדר ב-`presentation.searchResult.componentIds`.
2. במסך פרטי רופא הקליינט אחראי להגדיר בצד שלו את מיפוי רכיבי המידע לקומפוננטות UI ולאזורי התצוגה במסך.
3. במסך פרטי רופא ה-Agent מחזיר את רכיבי המידע הזמינים ואת התוכן שלהם בלבד.
4. במסך פרטי רופא ה-Agent אינו מחזיר סדר תצוגה, מיקום במסך או שיוך לאזורי layout.
5. `presentation.details.availableComponentIds` היא רשימת רכיבים זמינים בלבד, לא הוראת סדר או מיקום.
6. אם רכיב קיים אבל `isVisible=false`, הקליינט מדלג עליו.
7. כותרות רכיבים מגיעות בפלט דרך `title`.
8. מפתחות תוכן עתידיים מגיעים דרך `contentKeys`.
9. אין להוסיף `displayLabel` גנרי לכל שדה. משתמשים בשדות קיימים לפי הקשר:
   - `title` לכותרת רכיב/סקשן.
   - `label` לשם שדה בתוך פריט.
   - `text` להודעה.
   - `value` לערך.
   - `displayName` לשם ישות.
   - `displayAddress` לכתובת לתצוגה.
   - `displayText` לטקסט מחושב מלא.

## חוקי entity

1. `entity.name` חייב להיות `service_name`.
2. `entity.displayName` הוא `service_provider_name`, ואם חסר אז `service_name`.
3. `entity.entityType` מחושב לפי `chapter_code`.
4. ב-v1 מצופה שהישות תהיה `doctor`; סוגים אחרים נשמרים כהכנה עתידית בלבד.
5. תמונה אמיתית אינה קיימת ב-v1; משתמשים ב-avatar סמנטי לפי מגדר.

## חוקי מידע מקצועי

1. מומחיות מגיעה מ-`specialization_list`.
2. אם אין `specialization_list`, מותר fallback ל-`service_name_doc`.
3. התמקצעות מגיעה מ-`treat_area_list[].treat_area`.
4. שפות מגיעות מ-`languages[].language`.
5. שפות ממוינות לפי `seqnr_languages`.
6. אם אין `languages`, מותר fallback ל-`language_list` מפוצל בפסיקים.
7. `summaryAttributes` לא יוצא בפלט החדש. במקום זאת יש רכיבים נפרדים:
   - `specializations`
   - `professionalizations`
   - `languages`

## חוקי remarks

1. `remarks` נשאר רכיב עצמאי.
2. אין כותרת כללית לרכיב `remarks`, כי ההודעות מפוזרות במסך לפי `targetArea`.
3. רק הודעות עם `remark_text` לא ריק נכנסות לפלט.
4. רק הודעות חירום מסוג `10`, `0010`, `00010` גלויות כברירת מחדל.
5. קוד `0004` ממופה לסוג `name`.
6. לכל remark יש:
   - `targetArea`
   - `displayStyle`
   - `severity`
   - `sortOrder`
7. מיון remarks מחושב כך:

```text
sortOrder = sortBase * 1000 + remark_seqnr * 10 + remark_line_number
```

8. לכן ערך כמו `2011` אינו "מספר רץ פשוט"; הוא מספר מורכב שמקודד:
   - `2` = קבוצת מיון עסקית של סוג הודעה.
   - `01` = סדר ההודעה.
   - `1` = מספר שורה.

## חוקי icons

1. `icons` הוא רכיב הכנה לסימונים ואייקונים.
2. אין לו כותרת כללית.
3. אייקון נגישות נוצר אם `accessibility_bool=true`.
4. סימון "לא מקבל חברים חדשים" נוצר אם אחד מהשדות הבאים true:
   - `noMember`
   - `no_member`
   - `no_member_icon`
   - `limited_memb_icon`
5. תגיות שב"ן נוצרות לפי `shaban_type`, גם אם `shaban_ind=false`.
6. `shaban_type=1` יוצר ייעוץ בתשלום נוסף.
7. `shaban_type=2` יוצר ניתוחים וטיפולים בתשלום נוסף.
8. `shaban_type=3` יוצר את שתי תגיות שב"ן.

## חוקי address

1. אם `full_address` קיים, הוא הערך המועדף ל-`displayAddress`.
2. אם `full_address` חסר, יש להרכיב כתובת מ-`city_name`, `street_name`, `house_number`.
3. כתובת מוצגת רק אם יש `displayAddress`.
4. נגישות יכולה להופיע גם בכתובת וגם ב-icons.
5. `navigationUrl` שמור לעתיד וב-v1 הוא `null`.

## חוקי facil

1. `facil` הוא רכיב עתידי למתקנים.
2. ב-v1 הוא קיים בפלט אבל `isVisible=false`.
3. הרשימה `items` ריקה ב-v1.

## חוקי היעדרויות וממלאי מקום

1. היעדרויות נבנות מ-`absence[]`.
2. תאריכים מומרים מפורמט SAP `dd.mm.yyyy` לפורמט ISO.
3. היעדרות עם תאריך התחלה ותאריך סיום זהים היא `single`.
4. היעדרות עם תאריכים שונים היא `range`.
5. ממלא מקום מגיע מ-`subtitute[]` או `substitute[]`.
6. ממלא מקום משויך רק אם תאריכי התחלה וסיום זהים לתאריכי ההיעדרות.
7. טקסט ההיעדרות מחושב לפי מגדר הרופא/רופאה.
8. רק ההיעדרות הראשונה מוצגת כברירת מחדל.
9. סיבת היעדרות מוצגת רק אם `absence_reason_dis=true`.

## חוקי שעות פעילות

1. שעות פעילות נבנות מ-`schedule[]`.
2. רשומה ללא `schedule_type` לא נכנסת לפלט.
3. קבוצות שעות ממוינות לפי קוד עסקי:
   - שעות פעילות הרופא
   - זימון תורים
   - נוסף
   - שעות פעילות טלפונית
   - קבלת קהל צוות
   - אחר
4. בתוך יום, שורות שעות ממוינות לפי `availability_start_time`.
5. `schedule_type=1` פתוח כברירת מחדל.
6. `schedule_type=2` סגור כברירת מחדל.
7. לכל יום נשמרים הנתונים המקוריים ב-`days`.
8. להצגה יש להשתמש ב-`displayDays`.
9. ימים רצופים עם אותן שעות ואותן הערות בדיוק מאוחדים לטווח.
10. אין לאחד ימים לא רצופים גם אם השעות זהות.
11. חתימת איחוד יום כוללת:
    - `startTime`
    - `endTime`
    - `hoursText`
    - `notes`
    - `isClosed`

## חוקי contacts

1. `contacts` הוא רכיב מסך של אמצעי תקשורת.
2. contact מסוג `appointments` הוא אמצעי קשר לזימון תורים, ולא רכיב המסך `appointments`.
3. אין לחבר כמה מספרים לערך אחד.
4. כל מספר טלפון/זימון/נייד חייב להיות item נפרד בתוך `values[]`.
5. אמצעי קשר ממוינים לפי קוד עסקי:
   - טלפון
   - זימון תורים
   - פקס
   - נייד
   - דוא"ל
   - אחר
6. ערכים כפולים מוסרים לפי `{type}:{value}`.
7. דוא"ל יוצר הודעת disclaimer ב-`contacts.messages`.
8. ב-`contacts.items[].values[]` משתמשים רק ב-`preferredAction`; אין להחזיר `action`, `desktopAction` או `mobileAction`.

### פעולות לפי endpoint

| Endpoint | טלפון/נייד/זימון תורים | דוא"ל | פקס | אחר |
|---|---|---|---|---|
| אפליקציה | `tel:` | `mailto:` | ללא פעולה | ללא פעולה |
| אונליין | `copy` | `mailto:` | ללא פעולה | ללא פעולה |
| מל"ה | `copy` | `mailto:` | ללא פעולה | ללא פעולה |
| מדריך השירותים | `copy` | `mailto:` | ללא פעולה | ללא פעולה |

דוגמה חשובה:

אם קיימים שני מספרי זימון תורים:

```text
*3555
1700505353
```

הפלט חייב לשמור אותם כשני ערכים נפרדים. באפליקציה:

```json
[
  { "value": "*3555", "preferredAction": { "type": "tel", "uri": "tel:*3555" } },
  { "value": "1700505353", "preferredAction": { "type": "tel", "uri": "tel:1700505353" } }
]
```

אסור ליצור:

```text
*35551700505353
```

## חוקי services - ניתוחים וטיפולים

1. `services` הוא רכיב ניתוחים וטיפולים.
2. מקור הנתונים הוא `serv_treats[]`.
3. רשומה נכנסת אם יש `cpt_code` או `sg_treat_name`.
4. קיבוץ נעשה לפי `cpt_pubt`.
5. אם אין `cpt_pubt`, משתמשים ב-`00000`.
6. שם קבוצה מגיע מ-`cpt_pubt_name`, ואם חסר אז `ללא קטגוריה`.
7. קבוצות ממוינות לפי הופעה ראשונה במקור.
8. פריטים ממוינים לפי סדר הופעה במקור.
9. שם פריט טיפול מגיע לפי סדר עדיפות:
   - `sg_treat_name`
   - `cell_treat_name`
   - `hebrew_text`
   - `hebrew_service_name`
10. `treat_is_personal=true` מייצר:
    - `coverageType=supplementary`
    - `coverageText=משלים`
11. אחרת:
    - `coverageType=basket`
    - `coverageText=בסל`
12. אם בקבוצה יש עד 2 פריטים, הקבוצה פתוחה ואין overflow.
13. אם בקבוצה יש 3 פריטים ומעלה, הקבוצה סגורה עם `previewRows=2`.

## חוקי appointments

1. `appointments` הוא רכיב מסך נפרד.
2. אין לערבב אותו עם `contacts.items[type=appointments]`.
3. ב-v1 הרכיב מציג הודעות בלבד.
4. אם קיים badge מסוג `limitedMembership`, נוצרת הודעה באזור זימון תורים.
5. אם אין הודעות, `appointments.isVisible=false`.

## חוקי resume

1. `resume` מחליף את `details`.
2. `resume.title` הוא `פרטים נוספים`.
3. אם יש `license_number`, נוצר סקשן `רישיון`.
4. label של מספר רישיון הוא `מספר רישיון`.
5. `resume[]` מייצר סקשנים נוספים.
6. מכל רשומת resume לוקחים רק זוגות label/value תקינים:
   - `resume_title1` + `resume_title1_contents`
   - `resume_title2` + `resume_title2_contents`
   - `resume_title3` + `resume_title3_contents`
7. אם `resume_topic` חסר, כותרת fallback היא `פרטי השכלה ומומחיות`.
8. סקשן רישיון תמיד לפני סקשני resume.

## חוקי Umbraco עתידי

1. ב-v1 אין גישה לאומברקו.
2. כל הטקסטים העסקיים hardcoded ב-Agent/Orchestrator.
3. הפלט חייב לכלול `contentKeys` כדי שבעתיד האורקסטרטור יוכל לאכלס תוכן מאומברקו.
4. הקליינט לא צריך לדעת אם הטקסט הגיע מ-hardcoded או מאומברקו.
5. מקור התוכן מתועד ב-`metadata.content`.
6. שינוי מקור תוכן מאוחר יותר לא אמור לשבור חוזה קליינט.

## חוקי בדיקה למפתח

המפתח צריך לוודא לפחות את הבדיקות הבאות:

| בדיקה | ציפייה |
|---|---|
| `chapter_code=001` | ממופה ל-`doctor`. |
| מומחיות | מגיעה מ-`specialization_list`. |
| התמקצעות | מגיעה מ-`treat_area_list[].treat_area`. |
| שפות | ממוינות לפי `seqnr_languages`. |
| `shaban_type=2` | יוצר תגית ניתוחים וטיפולים גם אם `shaban_ind=false`. |
| `shaban_type=3` | יוצר שתי תגיות שב"ן. |
| remarks קוד `0004` | ממופה ל-type `name`. |
| contact values מרובים | לא מתחברים למחרוזת אחת. |
| אפליקציה + זימון תורים | כל מספר מקבל `preferredAction.type=tel`. |
| אונליין/מל"ה/מדריך + זימון תורים | כל מספר מקבל `preferredAction.type=copy`. |
| דוא"ל | מקבל `mailto` ומייצר הודעת disclaimer. |
| פקס/אחר | ללא פעולה. |
| schedule rows | ממוינים לפי שעת התחלה. |
| ימים זהים ורצופים | מאוחדים ב-`displayDays`. |
| ימים זהים ולא רצופים | לא מאוחדים. |
| services עם 2 פריטים | פתוח וללא overflow. |
| services עם 3 פריטים | סגור עם `previewRows=2`. |
