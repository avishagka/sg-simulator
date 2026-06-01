# Mapping Specification - UniversalProviderDisplayPayload

מסמך זה מגדיר איך מייצרים את פלט ה-Agent מתוך ה-JSON המקורי. המסמך מבוסס על הסימולטור המעודכן `agent_universal_payload_simulator.html`.

## כללי המרה כלליים

| כלל | הגדרה |
|---|---|
| מקור נתונים | `originalJson` שמתקבל משאילתת הנתונים של Agent/Orchestrator. |
| יעד | `UniversalProviderDisplayPayload`. |
| גרסה | `1.0`. |
| ישויות נתמכות ב-v1 | רופאים בלבד (`doctor`). |
| אחריות לוגיקה | Agent/Orchestrator. |
| אחריות קליינט | רינדור רכיבים מתוך `results[].presentation` ו-`results[].components`. |
| תוכן ב-v1 | hardcoded בתוך Agent/Orchestrator. |
| תוכן עתידי | Umbraco באמצעות `contentKeys`. |

## מיפוי metadata

`metadata` מופיע פעם אחת בלבד ברמת ה-response. אין לשכפל `metadata` בתוך כל רשומה ב-`results[]`.

| Payload path | סוג | מקור | לוגיקה / החלטה עסקית |
|---|---|---|---|
| `metadata.requestId` | string | בקשה נכנסת או יצירת שרת | במימוש אמיתי יש להעביר/לייצר מזהה בקשה. בסימולטור קבוע `simulator-request`. |
| `metadata.payloadVersion` | string | קבוע | `1.0`. |
| `metadata.schemaName` | string | קבוע | `UniversalProviderDisplayPayload`. |
| `metadata.schemaVersion` | string | קבוע | `1.0`. |
| `metadata.supportedEntityTypes` | array | החלטה עסקית | ב-v1 רק `["doctor"]`. |
| `metadata.entityTypeVersions.doctor` | string | החלטה עסקית | `1.0`. |
| `metadata.generatedAt` | datetime | Agent | זמן יצירת הפלט בפורמט ISO. |
| `metadata.locale` | object | החלטה עסקית/endpoint | מוחזר בפלט בלבד: עברית ישראל, RTL. הקליינט לא שולח locale ב-v1. |
| `metadata.client` | object | endpoint/orchestrator | נגזר מסוג ה-endpoint, לא מהבקשה עצמה. |
| `metadata.content` | object | החלטה עסקית | ב-v1 `hardcoded`; עתידי `umbraco`. |
| `metadata.businessRules` | object | החלטה עסקית | מציין שהוחלו חוקים ב-Agent: `provider-search-v1`. |
| `metadata.sourceData` | object | החלטה עסקית | מקור הנתונים: `serviceGuide`, סוג מקור: `originalJson`, גרסת טרנספורמציה. |

## מיפוי endpoint ל-client

| Endpoint | client metadata | משמעות |
|---|---|---|
| אפליקציה | `{ "clientType": "mobile", "platform": "app", "channel": "memberApp" }` | קליינט אפליקטיבי. |
| אונליין | `{ "clientType": "web", "platform": "web", "channel": "online" }` | אתר אונליין. |
| מל"ה | `{ "clientType": "internalSystem", "platform": "web", "channel": "mlh" }` | מערכת פנימית. |
| מדריך השירותים | `{ "clientType": "web", "platform": "web", "channel": "serviceGuide" }` | מדריך השירותים. |

## מיפוי results[]

| Payload path | סוג | מקור | לוגיקה / החלטה עסקית |
|---|---|---|---|
| `results[]` | array<object> | רשומות מקור לאחר `extractHits` | כל רשומת מקור תקינה הופכת ל-result אחד. |
| `results[].entity` | object | רשומת מקור | מבנה entity לפי המיפוי להלן. |
| `results[].presentation` | object | החלטות תצוגה | מבנה presentation לפי המיפוי להלן. |
| `results[].components` | object | רשומת מקור + חוקים עסקיים | רכיבי המידע. אין להוסיף `metadata` ברמת result. |

## מיפוי entity

כל הנתיבים בפרק זה נמצאים תחת `results[].entity`.

| Payload path | סוג | מקור ב-JSON המקורי | לוגיקה / החלטה עסקית |
|---|---|---|---|
| `entity.isVisible` | boolean | קבוע | `true` עבור רשומת מקור תקינה. |
| `entity.entityType` | string | `chapter_code` | `001` או `01` => `doctor`; `003` או `03` => `therapist`; אחרת `other`. ב-v1 מצופה `doctor`. |
| `entity.id` | string | `entity_id`, `service_provider_id`, `object_id` | נלקח לפי זמינות בסדר הזה. |
| `entity.sourceType` | string | `chapter_code` | קוד סוג ישות מהמקור. |
| `entity.sourceCode` | string | `object_id` | קוד מקור/אובייקט. |
| `entity.name` | string | `service_name` | שם השירות. זה השדה שתוקן לפי הדרישה: `name` צריך להיות `service_name`. |
| `entity.displayName` | string | `service_provider_name`, fallback `service_name` | שם הישות לתצוגה. |
| `entity.subtitle` | string | `treat_area_string` | תיאור משני אם קיים. |
| `entity.profileUrl` | string/null | `url` | כתובת פרופיל אם קיימת. |
| `entity.image.type` | string | קבוע | `avatar`. |
| `entity.image.gender` | string/null | `service_provider_gender` | ערך מקור. |
| `entity.image.avatarType` | string | `service_provider_gender` | אישה/F/female => `female`; גבר/M/male => `male`; אחרת `default`. |
| `entity.image.url` | null | החלטה עסקית | ב-v1 אין תמונה אמיתית. |
| `entity.image.alt` | string | `service_provider_name`, fallback `service_name` | טקסט חלופי. |

## מיפוי רכיבי מידע מקצועי

| Component | Payload path | מקור ב-JSON המקורי | לוגיקה |
|---|---|---|---|
| מומחיות | `components.specializations.values[]` | `specialization_list` | כל ערך לא ריק הופך ל-`{ value, sortOrder }`. אם אין `specialization_list`, fallback ל-`service_name_doc`. |
| התמקצעות | `components.professionalizations.values[]` | `treat_area_list[].treat_area` | כל `treat_area` לא ריק הופך ל-`{ value, sortOrder }`. זו ההגדרה המחייבת: התמקצעות ← `treat_area_list`. |
| שפות | `components.languages.values[]` | `languages[].language` | כל שפה לא ריקה הופכת ל-`{ value, sortOrder }`. מיון לפי `seqnr_languages`. אם אין `languages`, fallback לפיצול `language_list` לפי פסיקים. |

כותרות הרכיבים:

| Component | Title | content key |
|---|---|---|
| `specializations` | `מומחיות` | `provider.specializations.title` |
| `professionalizations` | `התמקצעות` | `provider.professionalizations.title` |
| `languages` | `שפות` | `provider.languages.title` |

## מיפוי address

| Payload path | מקור | לוגיקה |
|---|---|---|
| `components.address.title` | hardcoded | `כתובת`; בעתיד `provider.address.title`. |
| `components.address.isVisible` | מחושב | true אם קיימת כתובת לתצוגה. |
| `components.address.city` | `city_name` | ערך מקור. |
| `components.address.street` | `street_name` | ערך מקור. |
| `components.address.streetCode` | `street_code` | ערך מקור. |
| `components.address.houseNumber` | `house_number` | ערך מקור. |
| `components.address.fullAddress` | `full_address` | ערך מקור. |
| `components.address.displayAddress` | `full_address` או חיבור `city_name street_name house_number` | אם `full_address` קיים משתמשים בו; אחרת מרכיבים כתובת מחלקים קיימים. |
| `components.address.navigationUrl` | החלטה עסקית | `null` ב-v1. |
| `components.address.accessibility.isVisible` | `accessibility_bool` | true אם יש נגישות. |
| `components.address.accessibility.text` | `accessibility`, fallback `נגיש` | מוצג רק אם `accessibility_bool=true`. |
| `components.address.messages` | החלטה עסקית | ריק ב-v1. |

## מיפוי remarks

מקור: `remarks[]`. רק רשומות עם `remark_text` לא ריק נכנסות לפלט.

| קוד מקור | type | targetArea | displayStyle | sort base | החלטה עסקית |
|---|---|---|---|---:|---|
| `0010`, `00010`, `10` | `emergency` | `pageTop` | `highlight` | 1 | גלוי כברירת מחדל, severity `green`. |
| `0004` | `name` | `entity` | `inline` | 2 | הודעת שם, לא גלויה כברירת מחדל. |
| `0006` | `contact` | `contacts` | `inline` | 3 | הודעת אמצעי קשר. |
| `0002` | `address` | `locations` | `inline` | 4 | הודעת כתובת. |
| `0003` | `schedule` | `availability` | `inline` | 5 | הודעת שעות. |
| `0005` | `occupation` | `entity` | `regular` | 6 | הודעת עיסוק/מקצוע. |
| `0008` | `clinicServices` | `services` | `regular` | 7 | הודעת שירותים. |
| `0020`, `0001` | `additionalInfo` | `details` | `regular` | 8 | מידע נוסף. |
| אחר | `other` | `details` | `regular` | 999 | fallback. |

חישוב `sortOrder`:

```text
sortOrder = sortBase * 1000 + remark_seqnr * 10 + remark_line_number
```

אם `remark_seqnr` חסר משתמשים באינדקס הרשומה + 1. אם `remark_line_number` חסר משתמשים ב-1.

## מיפוי icons

| Payload item type | מקור | לוגיקה / החלטה עסקית |
|---|---|---|
| `accessibility` | `accessibility_bool`, `accessibility` | אם `accessibility_bool=true`, מוסיפים אייקון נגישות. טקסט: `accessibility` או fallback `נגיש`. |
| `limitedMembership` | `noMember`, `no_member`, `no_member_icon`, `limited_memb_icon` | אם אחד מהם true, מוסיפים סימון `לא מקבל חברים חדשים`. |
| `shabanConsultation` | `shaban_type` | אם `shaban_type` הוא 1 או 3, מוסיפים `ייעוץ בתשלום נוסף`. |
| `shabanTreatments` | `shaban_type` | אם `shaban_type` הוא 2 או 3, מוסיפים `ניתוחים וטיפולים בתשלום נוסף`. |

הערה: יצירת תגיות שב"ן מתבססת על `shaban_type`, גם אם `shaban_ind=false`.

## מיפוי facil

| Payload path | מקור | לוגיקה |
|---|---|---|
| `components.facil.id` | קבוע | `facil`. |
| `components.facil.title` | hardcoded | `מתקנים`; בעתיד `provider.facil.title`. |
| `components.facil.isVisible` | החלטה עסקית | `false` ב-v1. |
| `components.facil.items` | החלטה עסקית | מערך ריק ב-v1. |

## מיפוי absences

מקורות:

- `absence[]`
- `subtitute[]` או `substitute[]`
- `service_provider_gender`

| Payload path | מקור | לוגיקה |
|---|---|---|
| `startDate` | `absence_bdate` | המרה מתאריך SAP `dd.mm.yyyy` לפורמט ISO `yyyy-mm-dd`. |
| `endDate` | `absence_edate` | המרה מתאריך SAP לפורמט ISO. |
| `dateType` | מחושב | אם תאריך התחלה = תאריך סיום אז `single`, אחרת `range`. |
| `displayDate` | מחושב | תאריך בודד `dd/mm/yyyy`; טווח `dd/mm/yyyy - dd/mm/yyyy`. |
| `reason` | `absence_reason_dis`, `absence_reason` | מוצג רק אם `absence_reason_dis=true`. |
| `replacementEntity` | `subtitute[]` / `substitute[]` | ממלא מקום מותאם לפי אותם תאריכי התחלה וסיום. |
| `displayText` | מחושב | טקסט היעדרות מלא, מותאם מגדרית וכולל ממלא מקום אם קיים. |
| `isDisplayed` | מחושב | רק ההיעדרות הראשונה מוצגת כברירת מחדל. |
| `sortOrder` | מחושב | לפי מיון תאריך התחלה. |

התאמת ממלא מקום:

```text
absence.absence_bdate == substitute.sub_bdate
AND
absence.absence_edate == substitute.sub_edate
```

## מיפוי schedule

מקור: `schedule[]`.

רשומות ללא `schedule_type` לא נכנסות לפלט.

### מיפוי סוגי schedule

| `schedule_type` | type | title | sortOrder | defaultOpen |
|---:|---|---|---:|---|
| 1 | `reception` | `שעות פעילות הרופא` | 1 | true |
| 2 | `appointments` | `זימון תורים` | 2 | false |
| 5 | `additional` | `נוסף` | 3 | false |
| 14 | `phone` | `שעות פעילות טלפונית` | 4 | false |
| 15 | `teamReception` | `קבלת קהל צוות` | 5 | false |
| אחר | `other` | `schedule_desc` או `אחר` | 999 | false |

### מיפוי rows

| Payload path | מקור | לוגיקה |
|---|---|---|
| `days[].dayCode` | `availability_week_day` | ימים 1-7. |
| `days[].dayLabel` | קבוע | 1=א׳, 2=ב׳, 3=ג׳, 4=ד׳, 5=ה׳, 6=ו׳, 7=שבת. |
| `rows[].startTime` | `availability_start_time` | ערך מקור. |
| `rows[].endTime` | `availability_end_time` | ערך מקור. |
| `rows[].hoursText` | מחושב | `{startTime} עד {endTime}`. |
| `rows[].notes` | `frequency_desc` | הערות/תדירות. |
| `rows[].availableMorning` | `available_morning` | boolean. |
| `rows[].availableAfternoon` | `available_afternoon` | boolean. |
| `rows[].availableEvening` | `available_evening` | boolean. |
| `rows[].sortOrder` | מחושב | לפי מיון שעת התחלה בתוך אותו יום. |

### איחוד ימים להצגה

בנוסף ל-`days`, הפלט כולל `displayDays`.

ימים רצופים מאוחדים לטווח אם יש להם בדיוק אותה חתימה:

- אותן שורות שעות.
- אותו `startTime`.
- אותו `endTime`.
- אותו `hoursText`.
- אותן `notes`.
- אותו `isClosed`.

דוגמה: אם ימים א׳ וב׳ כוללים בדיוק `08:00-12:00` ואותה הערה, `displayDays` יחזיר שורה עם `dayLabel: "א׳-ב׳"`.

## מיפוי contacts

מקור: `contact[]`. רק רשומות עם `contact_detail` לא ריק נכנסות לפלט.

### מיפוי סוגי contact

| `contact_code` | type | label | sortOrder |
|---|---|---|---:|
| `01` | `phone` | `טלפון` | 1 |
| `02` | `appointments` | `זימון תורים` | 2 |
| `03` | `fax` | `פקס` | 3 |
| `04` | `mobile` | `נייד` | 4 |
| `05` | `email` | `דוא"ל` | 5 |
| אחר | `other` | `contact_desc` או `אחר` | 999 |

### כללי קיבוץ ומניעת כפילויות

| כלל | לוגיקה |
|---|---|
| נרמול קוד | `contact_code` מרופד לשתי ספרות באמצעות `padStart(2, "0")`. |
| קיבוץ | הערכים מקובצים לפי `type`. |
| מניעת כפילויות | ערך כפול מוסר לפי מפתח `{type}:{value}`. |
| מיון קבוצות | לפי קוד עסקי: טלפון, זימון תורים, פקס, נייד, דוא"ל, אחר. |
| מיון ערכים | לפי סדר הופעה בתוך אותה קבוצה. |
| הפרדת ערכים | כל מספר/מייל/פקס נשאר item נפרד בתוך `values[]`; אין לחבר מספרים למחרוזת אחת. |
| פעולות | בתוך `values[]` מחזירים רק `preferredAction`; אין להחזיר `action`, `desktopAction` או `mobileAction`. |

### preferredAction לפי endpoint

| Endpoint | phone/mobile/appointments | email | fax/other |
|---|---|---|---|
| אפליקציה | `tel:` | `mailto:` | ללא פעולה |
| אונליין | `copy` | `mailto:` | ללא פעולה |
| מל"ה | `copy` | `mailto:` | ללא פעולה |
| מדריך השירותים | `copy` | `mailto:` | ללא פעולה |

דוגמאות:

```json
{
  "type": "appointments",
  "values": [
    {
      "value": "*3555",
      "preferredAction": { "type": "tel", "uri": "tel:*3555" }
    },
    {
      "value": "1700505353",
      "preferredAction": { "type": "tel", "uri": "tel:1700505353" }
    }
  ]
}
```

באונליין/מל"ה/מדריך השירותים אותם ערכים יקבלו:

```json
{ "type": "copy", "value": "*3555" }
```

## מיפוי services - ניתוחים וטיפולים

מקור: `serv_treats[]`.

רשומה נכנסת לפלט אם יש `cpt_code` או `sg_treat_name`.

| Payload path | מקור | לוגיקה |
|---|---|---|
| `components.services.title` | hardcoded | `ניתוחים וטיפולים`; בעתיד `provider.services.title`. |
| `groups[].groupCode` | `cpt_pubt` | אם חסר, `00000`. |
| `groups[].title` | `cpt_pubt_name` | אם חסר, `ללא קטגוריה`. |
| `groups[].sortOrder` | מחושב | לפי הופעה ראשונה של הקבוצה במקור. |
| `groups[].contentSource` | מחושב | אם `groupCode=00000` אז `hardcodedFallback`, אחרת `originalJson`. |
| `items[].code` | `cpt_code` | קוד טיפול. |
| `items[].name` | `sg_treat_name`, fallback `cell_treat_name`, `hebrew_text`, `hebrew_service_name` | שם הטיפול להצגה. |
| `items[].coverageType` | `treat_is_personal` | true => `supplementary`; אחרת `basket`. |
| `items[].coverageText` | `treat_is_personal` | true => `משלים`; אחרת `בסל`. |
| `items[].sortOrder` | מחושב | לפי אינדקס הרשומה במקור + 1. |
| `items[].contentKeys.coverageText` | מחושב | `provider.services.coverage.supplementary` או `provider.services.coverage.basket`. |

### display של services

| תנאי | תוצאה |
|---|---|
| עד 2 פריטים בקבוצה | `defaultOpen=true`, `hasOverflow=false`, `previewRows=2`. |
| יותר מ-2 פריטים בקבוצה | `defaultOpen=false`, `hasOverflow=true`, `previewRows=2`. |

## מיפוי appointments

רכיב `appointments` הוא אזור מסך, לא contact.

| Payload path | מקור | לוגיקה |
|---|---|---|
| `components.appointments.title` | hardcoded | `זימון תורים`; בעתיד `provider.appointments.title`. |
| `components.appointments.messages[]` | badge מסוג `limitedMembership` | אם יש סימון `לא מקבל חברים חדשים`, נוצרת הודעה באזור זימון תורים. |
| `components.appointments.isVisible` | מחושב | true אם יש לפחות הודעה אחת. |

## מיפוי resume

מקורות:

- `license_number`
- `resume[]`

| Payload path | מקור | לוגיקה |
|---|---|---|
| `components.resume.title` | hardcoded | `פרטים נוספים`; בעתיד `provider.resume.sectionTitle`. |
| `sections[type=license]` | `license_number` | אם קיים, נוצר סקשן `רישיון` בסדר 1. |
| `license.items[].label` | hardcoded | `מספר רישיון`; בעתיד `provider.resume.license.numberLabel`. |
| `license.items[].value` | `license_number` | מספר הרישיון. |
| `sections[type=resume]` | `resume[]` | לכל רשומה נבנה סקשן אם יש לפחות זוג label/value תקין. |
| `sections[type=resume].title` | `resume_topic` | אם חסר: `פרטי השכלה ומומחיות`. |
| `items[].label` | `resume_title1/2/3` | רק אם label וגם value קיימים. |
| `items[].value` | `resume_title1/2/3_contents` | רק אם label וגם value קיימים. |
| `sections[].sortOrder` | מחושב | רישיון = 1; resume = `10 + resume_code` או `10 + index + 1`. |

## מיפוי presentation

| Payload path | מקור | לוגיקה |
|---|---|---|
| `presentation.entityType` | `chapter_code` | זהה ל-`entity.entityType`. |
| `presentation.searchResult.title` | hardcoded | `תוצאות חיפוש`; בעתיד `provider.searchResult.title`. |
| `presentation.searchResult.componentIds` | החלטה עסקית | סדר תצוגה מחייב לכרטיס תוצאות חיפוש: `entity`, `specializations`, `professionalizations`, `languages`, `remarks`, `icons`, `address`, `absences`. |
| `presentation.details.title` | hardcoded | `איתור שירות`; בעתיד `provider.details.screenTitle`. |
| `presentation.details.layoutPolicy` | החלטה עסקית | `clientManaged`. במסך פרטי רופא הקליינט אחראי למיפוי רכיבי המידע לקומפוננטות UI ולאזורי התצוגה. |
| `presentation.details.availableComponentIds` | החלטה עסקית | רשימת רכיבי מידע זמינים בלבד: `remarks`, `entity`, `specializations`, `professionalizations`, `languages`, `icons`, `absences`, `schedule`, `address`, `facil`, `services`, `appointments`, `contacts`, `resume`. אין משמעות של סדר, מיקום או שיוך לאזור layout. |

## contentKeys

ב-v1 הערכים hardcoded אבל `contentKeys` חייבים להישאר בפלט כדי לתמוך באומברקו בעתיד.

| תוכן | content key |
|---|---|
| תוצאות חיפוש | `provider.searchResult.title` |
| כותרת מסך פרטים | `provider.details.screenTitle` |
| מומחיות | `provider.specializations.title` |
| התמקצעות | `provider.professionalizations.title` |
| שפות | `provider.languages.title` |
| כתובת | `provider.address.title` |
| מתקנים | `provider.facil.title` |
| היעדרויות וממלאי מקום | `provider.absences.title` |
| שעות פעילות | `provider.schedule.title` |
| ניתוחים וטיפולים | `provider.services.title` |
| זימון תורים | `provider.appointments.title` |
| אמצעי תקשורת | `provider.contacts.title` |
| פרטים נוספים | `provider.resume.sectionTitle` |
| רישיון | `provider.resume.license.title` |
| מספר רישיון | `provider.resume.license.numberLabel` |
| כיסוי בסל | `provider.services.coverage.basket` |
| כיסוי משלים | `provider.services.coverage.supplementary` |
| הודעת דוא"ל | `provider.contacts.emailDisclaimer` |
