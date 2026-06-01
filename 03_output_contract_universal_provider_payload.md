# Output Contract - UniversalProviderDisplayPayload

מסמך זה מגדיר את חוזה הפלט שה-Agent מחזיר לקליינט אחרי המרת ה-JSON המקורי למבנה תצוגה אחיד. החוזה מבוסס על הסימולטור המעודכן `agent_universal_payload_simulator.html`.

## מטרת החוזה

הפלט צריך לאפשר לכל קליינט - אפליקציה, אונליין, מל"ה, מדריך השירותים או קליינט עתידי - להציג את אותו מידע עם מינימום לוגיקה בצד הקליינט.

הקליינט מקבל מה-Agent:

- נתוני ישות להצגה.
- רשימת רכיבי מסך.
- סדר רכיבים למסך תוצאות חיפוש בלבד, ורשימת רכיבי מידע זמינים למסך פרטי רופא.
- טקסטים וכותרות לתצוגה בגרסה 1.
- מפתחות תוכן עתידיים לאומברקו.
- החלטות עסקיות שכבר חושבו באורקסטרטור/Agent.
- פעולות מומלצות לאמצעי קשר לפי סוג endpoint.

## שם וגרסה

| פריט | ערך |
|---|---|
| Schema name | `UniversalProviderDisplayPayload` |
| Schema version | `1.0` |
| Payload version | `1.0` |
| Ruleset version | `provider-search-v1` |
| Entity support in v1 | `doctor` בלבד |
| Future entity support | מרפאות, מכונים, סדנאות, בתי מרקחת ועוד |

## מבנה על

```json
{
  "metadata": {},
  "results": [
    {
      "entity": {},
      "presentation": {},
      "components": {}
    }
  ]
}
```

## עקרונות חוזה

1. הפלט מאוחד: response אחד כולל `metadata` ברמת התגובה ו-`results[]` עם רשומות תוצאה.
2. הקליינט לא מרכיב ידע עסקי. בתוצאות חיפוש הוא קורא את `presentation.searchResult.componentIds` ומציג את הרכיבים לפי הסדר. במסך פרטי רופא הקליינט אחראי למפות את רכיבי המידע לקומפוננטות UI ולאזורי תצוגה בצד שלו.
3. אין `summaryAttributes` בפלט החדש. מומחיות, התמקצעות ושפות הם רכיבים נפרדים.
4. `remarks` נשאר רכיב עצמאי, ללא כותרת כללית, כי ההודעות מפוזרות באזורים שונים במסך.
5. `icons` נשאר רכיב עצמאי, ללא כותרת כללית, כהכנה לאייקונים נוספים בעתיד.
6. `contacts` הוא רכיב מסך של אמצעי תקשורת.
7. `appointments` הוא רכיב מסך של זימון תורים, ואינו זהה ל-contact מסוג `appointments`.
8. `resume` מחליף את `details`.
9. `facil` קיים כרכיב עתידי למתקנים, גם אם בגרסה 1 הוא לא מוצג.
10. בגרסה 1 תוכן וכותרות hardcoded, אבל כל רכיב תומך ב-`contentKeys` כדי לאפשר טעינה עתידית מאומברקו.

## metadata

| שדה | סוג | חובה | תיאור |
|---|---|---:|---|
| `requestId` | string | כן | מזהה בקשה. בסימולטור ערך קבוע `simulator-request`; במימוש אמיתי יגיע מהבקשה או ייווצר בצד שרת. |
| `payloadVersion` | string | כן | גרסת הפיילואד. בגרסה זו `1.0`. |
| `schemaName` | string | כן | שם הסכמה: `UniversalProviderDisplayPayload`. |
| `schemaVersion` | string | כן | גרסת הסכמה. |
| `supportedEntityTypes` | array<string> | כן | ב-v1 מכיל `doctor` בלבד. |
| `entityTypeVersions` | object | כן | מיפוי סוג ישות לגרסה, למשל `{ "doctor": "1.0" }`. |
| `generatedAt` | string datetime | כן | זמן יצירת הפלט בפורמט ISO. |
| `locale` | object | כן | מוחזר בפלט בלבד: `{ "language": "he", "country": "IL", "direction": "rtl" }`. |
| `client` | object | כן | client metadata כפי שנגזר מה-endpoint. |
| `content` | object | כן | מקור תוכן וכותרות. ב-v1 hardcoded; בעתיד Umbraco. |
| `businessRules` | object | כן | גרסת החוקים שהופעלה ומקורם. |
| `sourceData` | object | כן | מידע על מקור הנתונים והטרנספורמציה. |

### metadata.client לפי endpoint

| Endpoint | `clientType` | `platform` | `channel` |
|---|---|---|---|
| אפליקציה | `mobile` | `app` | `memberApp` |
| אונליין | `web` | `web` | `online` |
| מל"ה | `internalSystem` | `web` | `mlh` |
| מדריך השירותים | `web` | `web` | `serviceGuide` |

### metadata.content

```json
{
  "mode": "hardcoded",
  "provider": "agent",
  "futureProvider": "umbraco",
  "contentVersion": "hc-v1"
}
```

משמעות: בגרסה 1 הטקסטים והכותרות מגיעים מה-Agent/Orchestrator כ-hardcoded. בעתיד אותם שדות יכולים להתאכלס מאומברקו ללא שינוי חוזה מול הקליינט.

## results[]

כל אובייקט בתוך `results[]` מכיל את מבנה הרשומה הפנימי:

```json
{
  "entity": {},
  "presentation": {},
  "components": {}
}
```

אין לשכפל `metadata` בתוך כל רשומה.

## entity

| שדה | סוג | תיאור |
|---|---|---|
| `isVisible` | boolean | תמיד `true` עבור ישות תקינה. |
| `entityType` | string | ב-v1 צפוי `doctor`. |
| `id` | string | מזהה ישות, לפי זמינות מקור: `entity_id`, `service_provider_id`, `object_id`. |
| `sourceType` | string | קוד סוג ישות מהמקור, למשל `chapter_code`. |
| `sourceCode` | string | קוד מקור, למשל `object_id`. |
| `name` | string | שם השירות מהמקור: `service_name`. |
| `displayName` | string | שם לתצוגה: `service_provider_name`, ואם חסר אז `service_name`. |
| `subtitle` | string | תיאור משני, כרגע `treat_area_string`. |
| `profileUrl` | string | כתובת פרופיל אם קיימת במקור. |
| `image` | object | מידע לאווטאר/תמונה. |

### entity.image

| שדה | סוג | תיאור |
|---|---|---|
| `type` | string | ב-v1 `avatar`. |
| `gender` | string | מגדר כפי שהגיע מהמקור. |
| `avatarType` | string | `female`, `male`, או `default`. |
| `url` | string/null | כרגע `null`. |
| `alt` | string | טקסט חלופי, לפי `service_provider_name` או `service_name`. |

## presentation

`presentation` מגדיר את אופן צריכת רכיבי המידע במסכים. בתוצאות חיפוש הוא כולל סדר תצוגה מחייב. במסך פרטי רופא הוא כולל מדיניות `clientManaged` ורשימת רכיבים זמינים בלבד, ללא סדר, מיקום או שיוך לאזורי layout.

```json
{
  "entityType": "doctor",
  "searchResult": {
    "title": "תוצאות חיפוש",
    "contentKeys": { "title": "provider.searchResult.title" },
    "componentIds": ["entity", "specializations", "professionalizations", "languages", "remarks", "icons", "address", "absences"]
  },
  "details": {
    "title": "איתור שירות",
    "contentKeys": { "title": "provider.details.screenTitle" },
    "layoutPolicy": "clientManaged",
    "availableComponentIds": ["remarks", "entity", "specializations", "professionalizations", "languages", "icons", "absences", "schedule", "address", "facil", "services", "appointments", "contacts", "resume"]
  }
}
```

בתוצאות חיפוש הקליינט מציג רכיבים לפי `componentIds`. אם רכיב מופיע ברשימה אבל `isVisible=false`, הקליינט לא מציג אותו.

במסך פרטי רופא ה-Agent אינו מחזיר סדר תצוגה, מיקום במסך או שיוך לאזורי layout. `availableComponentIds` היא רשימת רכיבי מידע זמינים בלבד, ואינה הוראת תצוגה. הקליינט אחראי להגדיר בצד שלו את מיפוי רכיבי המידע לקומפוננטות UI ולאזורי התצוגה במסך.

## components

רשימת הרכיבים ב-v1:

```json
{
  "specializations": {},
  "professionalizations": {},
  "languages": {},
  "address": {},
  "remarks": {},
  "icons": {},
  "facil": {},
  "absences": {},
  "schedule": {},
  "services": {},
  "appointments": {},
  "contacts": {},
  "resume": {}
}
```

## רכיבי רשימת טקסט

משמש עבור `specializations`, `professionalizations`, `languages`.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | מזהה הרכיב. |
| `title` | string | כותרת לתצוגה. |
| `isVisible` | boolean | האם יש ערכים להצגה. |
| `contentKeys.title` | string | מפתח תוכן עתידי מאומברקו. |
| `values` | array<object> | ערכים להצגה. |

מבנה ערך:

| שדה | סוג | תיאור |
|---|---|---|
| `value` | string | הטקסט להצגה. |
| `sortOrder` | number | סדר הצגה בתוך הרשימה. |

## address

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `address`. |
| `title` | string | `כתובת`. |
| `isVisible` | boolean | true אם קיימת כתובת להצגה. |
| `contentKeys.title` | string | `provider.address.title`. |
| `city` | string/null | עיר. |
| `street` | string/null | רחוב. |
| `streetCode` | string/null | קוד רחוב. |
| `houseNumber` | string/null | מספר בית. |
| `fullAddress` | string/null | כתובת מלאה מהמקור. |
| `displayAddress` | string/null | כתובת לתצוגה. |
| `navigationUrl` | string/null | שמור לעתיד. |
| `accessibility` | object | מידע נגישות ברמת הכתובת. |
| `messages` | array | הודעות הקשורות לכתובת. |

## remarks

אין כותרת לרכיב זה.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `remarks`. |
| `isVisible` | boolean | true אם קיימת לפחות הודעה גלויה. |
| `items` | array<object> | הודעות. |

מבנה הודעה:

| שדה | סוג | תיאור |
|---|---|---|
| `isVisible` | boolean | האם ההודעה מוצגת. |
| `type` | string | סוג ההודעה לאחר מיפוי. |
| `sourceTypeCode` | string | קוד מקור. |
| `targetArea` | string | האזור במסך שבו ההודעה מיועדת להשתלב. |
| `text` | string | טקסט ההודעה. |
| `displayStyle` | string | סגנון תצוגה: inline/highlight/regular. |
| `severity` | string/null | חומרה/צבע סמנטי. |
| `sortOrder` | number | סדר פנימי מחושב. |

## icons

אין כותרת לרכיב זה.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `icons`. |
| `isVisible` | boolean | true אם קיימים אייקונים/סימונים להצגה. |
| `items` | array<object> | אייקונים. |

מבנה אייקון:

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | מזהה אייקון. |
| `type` | string | סוג סימון: נגישות, שב"ן, קבלת חברים חדשים וכו'. |
| `text` | string | טקסט נלווה. |
| `icon` | string | מזהה אייקון סמנטי. |
| `targetArea` | string | אזור במסך, כרגע `entityHeader`. |
| `severity` | string | משמעות תצוגתית. |
| `source` | string | מקור הנתון. |
| `sortOrder` | number | סדר תצוגה. |

## facil

רכיב עתידי למתקנים.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `facil`. |
| `title` | string | `מתקנים`. |
| `isVisible` | boolean | ב-v1 תמיד `false`. |
| `contentKeys.title` | string | `provider.facil.title`. |
| `items` | array | ב-v1 ריק. |

## absences

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `absences`. |
| `title` | string | `היעדרויות וממלאי מקום`. |
| `isVisible` | boolean | true אם יש היעדרות שמוצגת. |
| `contentKeys.title` | string | `provider.absences.title`. |
| `items` | array<object> | היעדרויות. |

מבנה היעדרות:

| שדה | סוג | תיאור |
|---|---|---|
| `startDate` | string date | תאריך התחלה בפורמט ISO. |
| `endDate` | string date | תאריך סיום בפורמט ISO. |
| `displayDate` | string | תאריך לתצוגה. |
| `dateType` | string | `single` או `range`. |
| `displayText` | string | טקסט מלא מחושב לתצוגה. |
| `isDisplayed` | boolean | האם ההיעדרות מוצגת כברירת מחדל. |
| `isToday` | boolean | שמור להמשך. |
| `reason` | string/null | סיבת היעדרות אם מותר להציג. |
| `replacementEntity` | object | ממלא מקום אם נמצא. |
| `sortOrder` | number | סדר תצוגה. |

## schedule

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `schedule`. |
| `title` | string | `שעות פעילות`. |
| `isVisible` | boolean | true אם קיימות קבוצות שעות. |
| `contentKeys.title` | string | `provider.schedule.title`. |
| `groups` | array<object> | קבוצות שעות לפי סוג. |

מבנה קבוצה:

| שדה | סוג | תיאור |
|---|---|---|
| `type` | string | סוג שעות: reception/appointments/additional/phone/teamReception/other. |
| `title` | string | כותרת הקבוצה. |
| `sortOrder` | number | סדר הקבוצה. |
| `days` | array<object> | ימים מקוריים לפי 1-7. |
| `displayDays` | array<object> | ימים אחרי איחוד טווחים להצגה. |
| `display` | object | הגדרות פתיחה/תצוגה. |

`displayDays` מאחד ימים רצופים אם שעות הפעילות וההערות שלהם זהות בדיוק.

## services

רכיב ניתוחים וטיפולים.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `services`. |
| `title` | string | `ניתוחים וטיפולים`. |
| `isVisible` | boolean | true אם קיימים שירותים/טיפולים. |
| `contentKeys.title` | string | `provider.services.title`. |
| `groups` | array<object> | קבוצות שירותים. |

מבנה קבוצה:

| שדה | סוג | תיאור |
|---|---|---|
| `groupCode` | string | קוד קבוצה מהמקור או `00000`. |
| `title` | string | שם קבוצה או `ללא קטגוריה`. |
| `sortOrder` | number | סדר לפי הופעה ראשונה במקור. |
| `contentSource` | string | `originalJson` או `hardcodedFallback`. |
| `display` | object | הגדרות פתיחה/סגירה. |
| `items` | array<object> | פריטי טיפול/שירות. |

מבנה פריט:

| שדה | סוג | תיאור |
|---|---|---|
| `code` | string/null | קוד טיפול. |
| `name` | string | שם הטיפול להצגה. |
| `coverageType` | string | `basket` או `supplementary`. |
| `coverageText` | string | `בסל` או `משלים`. |
| `locationName` | string/null | שמור לעתיד. |
| `selfParticipationText` | string/null | שמור לעתיד. |
| `selfParticipationUrl` | string/null | שמור לעתיד. |
| `sortOrder` | number | סדר הפריט לפי הופעה במקור. |
| `contentKeys.coverageText` | string | מפתח תוכן עתידי לטקסט הכיסוי. |

## appointments

רכיב מסך של זימון תורים. אינו אמצעי קשר.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `appointments`. |
| `title` | string | `זימון תורים`. |
| `isVisible` | boolean | ב-v1 true רק אם קיימת הודעת קבלת חברים חדשים. |
| `contentKeys.title` | string | `provider.appointments.title`. |
| `messages` | array<object> | הודעות שקשורות לזימון תורים. |

## contacts

רכיב מסך של אמצעי תקשורת.

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `contacts`. |
| `title` | string | `אמצעי תקשורת`. |
| `isVisible` | boolean | true אם קיימים אמצעי קשר. |
| `contentKeys.title` | string | `provider.contacts.title`. |
| `items` | array<object> | קבוצות אמצעי קשר. |
| `messages` | array<object> | הודעות נלוות, למשל הודעת דוא"ל. |

מבנה item:

| שדה | סוג | תיאור |
|---|---|---|
| `type` | string | `phone`, `appointments`, `fax`, `mobile`, `email`, `other`. |
| `sourceCode` | string | קוד אמצעי קשר מהמקור. |
| `label` | string | שם אמצעי הקשר. |
| `sortOrder` | number | סדר סוג אמצעי הקשר: `phone=1`, `appointments=2`, `fax=3`, `mobile=4`, `email=5`, `other=999`. |
| `values` | array<object> | ערכים. כל מספר/כתובת מוצג כערך נפרד. |

מבנה value:

| שדה | סוג | תיאור |
|---|---|---|
| `value` | string | הטלפון/מייל/פקס להצגה. |
| `preferredAction` | object/null | הפעולה המחייבת לקליינט לפי endpoint. |
| `sortOrder` | number | סדר ערך בתוך קבוצת הקשר. |

## resume

| שדה | סוג | תיאור |
|---|---|---|
| `id` | string | `resume`. |
| `title` | string | `פרטים נוספים`. |
| `isVisible` | boolean | true אם קיימים פרטי רישיון או resume. |
| `contentKeys.title` | string | `provider.resume.sectionTitle`. |
| `sections` | array<object> | סקשנים. |

מבנה section:

| שדה | סוג | תיאור |
|---|---|---|
| `type` | string | `license` או `resume`. |
| `title` | string | כותרת הסקשן. |
| `contentKeys.title` | string | מפתח תוכן אם קיים. |
| `sortOrder` | number | סדר סקשן. |
| `items` | array<object> | שדות להצגה. |

מבנה item:

| שדה | סוג | תיאור |
|---|---|---|
| `label` | string | שם השדה לתצוגה. |
| `value` | string | ערך השדה. |
| `sortOrder` | number | סדר שדה. |
| `contentKeys.label` | string | מפתח תוכן עתידי אם קיים. |
