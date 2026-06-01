# Frontend Rendering Specification - UniversalProviderDisplayPayload

מקור אמת: `agent_universal_payload_simulator.html`, `provider_search_agent_openapi.yaml`.

מסמך זה מחליף את אפיון אכלוס המסכים הישן. הוא מתאר איך קליינט מציג את ה-response שה-Agent מחזיר, בלי להניח שהקליינט מכיר את ה-JSON המקורי ובלי להעביר לוגיקה עסקית לקליינט.

## מבנה Response

ה-response הרשמי הוא:

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

כללים:

1. `metadata` קיים פעם אחת בלבד ברמת ה-response.
2. אין `metadata` בתוך כל result.
3. כל אובייקט בתוך `results[]` הוא רשומת תוצאה אחת.
4. כל result מכיל `entity`, `presentation`, `components`.
5. מבני ה-components הם מבני החוזה של OpenAPI והסימולטור.

## אחריות Agent מול Client

| נושא | אחריות Agent | אחריות Client |
|---|---|---|
| נתונים להצגה | כן | לא מחשב ממקור |
| חוקים עסקיים | כן | לא משחזר |
| טקסטים וכותרות ב-v1 | כן | מציג |
| contentKeys עתידיים | כן | לא פונה לאומברקו |
| סדר תוצאות חיפוש | כן, דרך `presentation.searchResult.componentIds` | מציג לפי הסדר |
| פרטי רופא layout | לא | ממפה רכיבי מידע לקומפוננטות UI ואזורים |
| פעולת אמצעי קשר | כן, דרך `preferredAction` | מפעיל את הפעולה |

## metadata

`metadata` משמש לזיהוי response, לקוח, גרסה, שפה, מקור תוכן וחוקים עסקיים.

הקליינט יכול להשתמש בו ל-debug, telemetry, הצגת כיוון שפה או בדיקת גרסה. הוא לא צריך להשתמש בו כדי להרכיב את רכיבי המסך.

שדות חשובים:

| Path | שימוש בפרונט |
|---|---|
| `metadata.locale.direction` | כיוון תצוגה, ב-v1 `rtl`. |
| `metadata.client` | debug/telemetry; הקליינט עצמו כבר יודע מי הוא. |
| `metadata.content` | מידע על מקור תוכן; לא מילון תצוגה. |
| `metadata.businessRules` | audit/debug. |
| `metadata.sourceData` | audit/debug. |

אין להשתמש ב-`metadata.content.keys`; השדה לא חלק מהחוזה.

## Results

מסך תוצאות חיפוש מציג את `response.results`.

אם אין תוצאות:

```html
<div class="provider-empty">אין תוצאות להצגה</div>
```

אם יש תוצאות, כל result מוצג ככרטיס תוצאה.

## תוצאות חיפוש

בתוצאות חיפוש ל-Agent יש סדר תצוגה מחייב:

```json
"presentation": {
  "searchResult": {
    "title": "תוצאות חיפוש",
    "contentKeys": { "title": "provider.searchResult.title" },
    "componentIds": [
      "entity",
      "specializations",
      "professionalizations",
      "languages",
      "remarks",
      "icons",
      "address",
      "absences"
    ]
  }
}
```

כללי רינדור:

1. הקליינט עובר על `presentation.searchResult.componentIds` לפי הסדר.
2. עבור כל componentId, הקליינט קורא את `components[componentId]`, למעט `entity` שנמצא ב-`result.entity`.
3. אם לרכיב יש `isVisible=false`, הקליינט מדלג עליו.
4. הקליינט לא מוסיף לוגיקת מיון משלו ברמת רכיבי תוצאת החיפוש.

### entity בכרטיס תוצאה

מקור:

```json
result.entity
```

שדות מרכזיים:

| Path | שימוש |
|---|---|
| `entity.displayName` | שם הרופא/ישות לתצוגה. |
| `entity.name` | שם השירות, לא בהכרח השם לתצוגת header. |
| `entity.subtitle` | טקסט משני אם מוצג במוקאפ. |
| `entity.profileUrl` | יעד ניווט אם הקליינט תומך. |
| `entity.image` | מידע לאווטאר; ב-v1 אין URL תמונה. |

### מידע מקצועי בכרטיס תוצאה

אין להשתמש ב-`summaryAttributes`.

מקורות:

| מידע | Path |
|---|---|
| מומחיות | `components.specializations.values[]` |
| התמקצעות | `components.professionalizations.values[]` |
| שפות | `components.languages.values[]` |

כל ערך הוא:

```json
{ "value": "...", "sortOrder": 1 }
```

הקליינט מציג לפי `sortOrder` בתוך `values[]`.

### הודעות וסימונים בתוצאות

| רכיב | Path | שימוש |
|---|---|---|
| הודעות | `components.remarks.items[]` | מציג רק פריטים גלויים ורלוונטיים לכרטיס. |
| אייקונים | `components.icons.items[]` | מציג אייקונים/תגיות כמו נגישות, שב"ן, לא מקבל חברים חדשים. |
| כתובת | `components.address` | מציג עיר/כתובת לפי המוקאפ. |
| היעדרות | `components.absences.items[]` | מציג היעדרות גלויה אם קיימת. |

## מסך פרטי רופא

במסך פרטי רופא ה-Agent אינו מנהל layout.

החוזה:

```json
"presentation": {
  "details": {
    "title": "איתור שירות",
    "contentKeys": { "title": "provider.details.screenTitle" },
    "layoutPolicy": "clientManaged",
    "availableComponentIds": [
      "remarks",
      "entity",
      "specializations",
      "professionalizations",
      "languages",
      "icons",
      "absences",
      "schedule",
      "address",
      "facil",
      "services",
      "appointments",
      "contacts",
      "resume"
    ]
  }
}
```

כללים מחייבים:

1. `availableComponentIds` היא רשימת רכיבי מידע זמינים בלבד.
2. אין לה משמעות של סדר תצוגה.
3. אין לה משמעות של מיקום במסך.
4. אין לה משמעות של שיוך לאזור layout.
5. הקליינט אחראי למפות רכיבי מידע לקומפוננטות UI ולאזורי התצוגה במסך.
6. הקליינט מדלג על רכיבים שבהם `isVisible=false`.
7. `sortOrder` נשאר רק למיון פנימי בתוך רשימות, לא למיקום רכיבי מסך.

## רכיבי מידע

### TextListComponent

רכיבים:

- `specializations`
- `professionalizations`
- `languages`

מבנה:

```json
{
  "id": "professionalizations",
  "title": "התמקצעות",
  "isVisible": true,
  "contentKeys": { "title": "provider.professionalizations.title" },
  "values": [
    { "value": "משפחה", "sortOrder": 1 }
  ]
}
```

הקליינט מציג את `title` ואת הערכים לפי `sortOrder` אם המוקאפ דורש כותרת. בכרטיס תוצאה אפשר להציג חלק מהערכים בלי כותרת, לפי החלטת UI.

### remarks

אין כותרת כללית לרכיב `remarks`.

מבנה:

```json
{
  "id": "remarks",
  "isVisible": true,
  "items": []
}
```

כל item כולל בין היתר:

| שדה | שימוש |
|---|---|
| `isVisible` | האם ההודעה מוצגת. |
| `type` | סוג הודעה. |
| `sourceTypeCode` | קוד מקור. |
| `targetArea` | אזור יעד סמנטי, לא layout פיזי מחייב. |
| `text` | טקסט להצגה. |
| `displayStyle` | סגנון הצגה. |
| `severity` | חומרה/צבע אם קיים. |
| `sortOrder` | מיון פנימי בתוך remarks. |

### icons

אין כותרת כללית לרכיב `icons`.

מבנה item:

| שדה | שימוש |
|---|---|
| `id` | מזהה אייקון. |
| `type` | accessibility, limitedMembership, shabanConsultation, shabanTreatments וכו'. |
| `text` | טקסט נלווה. |
| `icon` | מזהה אייקון סמנטי. |
| `targetArea` | יעד סמנטי. |
| `severity` | משמעות תצוגתית. |
| `sortOrder` | מיון פנימי. |

### address

מבנה מרכזי:

```json
{
  "id": "address",
  "title": "כתובת",
  "isVisible": true,
  "contentKeys": { "title": "provider.address.title" },
  "displayAddress": "...",
  "navigationUrl": null,
  "accessibility": {},
  "messages": []
}
```

הקליינט מציג את `displayAddress`. אם `navigationUrl=null`, אין לנווט בפועל אלא אם הקליינט יודע לבנות ניווט בעצמו לפי מדיניות שלו.

### absences

מבנה:

```json
{
  "id": "absences",
  "title": "היעדרויות וממלאי מקום",
  "isVisible": true,
  "contentKeys": { "title": "provider.absences.title" },
  "items": []
}
```

כללי תצוגה:

1. להציג רק אם `isVisible=true`.
2. בתוך `items[]`, פריט עם `isDisplayed=true` הוא הפריט שמיועד לתצוגה ראשית.
3. `displayText` כבר מחושב על ידי ה-Agent, כולל מגדר וממלא מקום אם קיים.
4. הקליינט לא מחשב את טקסט ההיעדרות.

### schedule

מבנה:

```json
{
  "id": "schedule",
  "title": "שעות פעילות",
  "isVisible": true,
  "contentKeys": { "title": "provider.schedule.title" },
  "groups": []
}
```

כותרות קבוצות לפי `schedule_type`:

| schedule_type | type | כותרת קבוצה |
|---|---|---|
| 1 | `reception` | שעות פעילות הרופא |
| 2 | `appointments` | זימון תורים |
| 5 | `additional` | נוסף |
| 14 | `phone` | שעות פעילות טלפונית |
| 15 | `teamReception` | קבלת קהל צוות |

הקליינט מציג שעות מתוך `displayDays` אם קיים. `displayDays` כבר מכיל איחוד ימים רצופים עם אותן שעות ואותן הערות בדיוק.

מבנה `displayDays`:

```json
{
  "dayCodes": [1, 2],
  "dayLabel": "א׳-ב׳",
  "isRange": true,
  "rows": [
    {
      "startTime": "08:00",
      "endTime": "12:00",
      "hoursText": "08:00 עד 12:00",
      "notes": null,
      "isClosed": false,
      "sortOrder": 1
    }
  ],
  "sortOrder": 1
}
```

אין להשתמש במבנה שטוח של `dayOfWeek/startTime/endTime` ברמת היום.

### services

רכיב ניתוחים וטיפולים:

```json
{
  "id": "services",
  "title": "ניתוחים וטיפולים",
  "isVisible": true,
  "contentKeys": { "title": "provider.services.title" },
  "groups": []
}
```

כללי תצוגה:

1. כל group מוצג כאקורדיון/רשימה לפי UI.
2. `display.defaultOpen`, `display.previewRows`, `display.hasOverflow` מגיעים מה-Agent.
3. הקליינט לא מחשב אם לפתוח או לסגור קבוצה.
4. `coverageText` מוצג כפי שהגיע.

### appointments

`appointments` הוא רכיב מסך של זימון תורים, לא אמצעי קשר.

```json
{
  "id": "appointments",
  "title": "זימון תורים",
  "isVisible": true,
  "contentKeys": { "title": "provider.appointments.title" },
  "messages": []
}
```

אין לערבב אותו עם `contacts.items[type="appointments"]`.

### contacts

`contacts` הוא רכיב אמצעי תקשורת.

```json
{
  "id": "contacts",
  "title": "אמצעי תקשורת",
  "isVisible": true,
  "contentKeys": { "title": "provider.contacts.title" },
  "items": [],
  "messages": []
}
```

כל item מייצג סוג אמצעי קשר:

```json
{
  "type": "appointments",
  "sourceCode": "02",
  "label": "זימון תורים",
  "sortOrder": 2,
  "values": []
}
```

כל value מוצג בנפרד. אסור לחבר מספרים למחרוזת אחת.

פעולות:

| preferredAction | התנהגות פרונט |
|---|---|
| `{ "type": "tel", "uri": "tel:*3555" }` | פתיחת חיוג/קישור טלפון. |
| `{ "type": "mailto", "uri": "mailto:a@b.com" }` | פתיחת מייל. |
| `{ "type": "copy", "value": "*3555" }` | העתקה ללוח. |
| `null` | טקסט בלבד, ללא פעולה. |

הקליינט לא בונה `tel:` או `mailto:` לבד; הוא משתמש ב-`uri` שקיבל.

### resume

`resume` מחליף את `details` הישן.

```json
{
  "id": "resume",
  "title": "פרטים נוספים",
  "isVisible": true,
  "contentKeys": { "title": "provider.resume.sectionTitle" },
  "sections": []
}
```

כללי תצוגה:

1. להציג sections לפי `sortOrder`.
2. להציג items בתוך section לפי `sortOrder`.
3. להשתמש ב-`label` ו-`value` כפי שהגיעו.
4. לא להשתמש ב-`payload.details` הישן.

### facil

רכיב עתידי למתקנים:

```json
{
  "id": "facil",
  "title": "מתקנים",
  "isVisible": false,
  "contentKeys": { "title": "provider.facil.title" },
  "items": []
}
```

ב-v1 בדרך כלל לא יוצג.

## Payload Tab בסימולטור

טאב `Payload` בסימולטור מציג את ה-response המלא:

```json
{
  "metadata": {},
  "results": []
}
```

כפתור `העתק Response מלא` מעתיק את כל ה-response.

כפתור `העתק תוצאה נבחרת` מעתיק רק את ה-result שנבחר, כלומר ללא `metadata`.

## כללי תאימות לפרונט

1. לא להשתמש ב-`summaryAttributes`.
2. לא להשתמש ב-`availability`, `locations`, `details`, `entity.badges`, `entity.accessibility` מהמבנה הישן.
3. להשתמש רק ב-`result.entity`, `result.presentation`, `result.components`.
4. בתוצאות חיפוש לכבד את `presentation.searchResult.componentIds`.
5. בפרטי רופא לכבד את `layoutPolicy=clientManaged`: הקליינט מנהל layout בעצמו.
6. בכל component לבדוק `isVisible` לפני הצגה.
7. בתוך רשימות להשתמש ב-`sortOrder` מקומי בלבד.
8. לא לפרש `availableComponentIds` כסדר תצוגה.
9. לא לבנות לוגיקה עסקית בצד הקליינט.
10. לא לפנות לאומברקו מהקליינט; הטקסטים מגיעים כבר ב-response.

## בדיקות קבלה לפרונט

| בדיקה | ציפייה |
|---|---|
| response ללא results | מוצגת הודעת אין תוצאות. |
| result ללא component גלוי | הרכיב לא מוצג. |
| details.availableComponentIds | לא מכתיב סדר או מיקום. |
| schedule.displayDays | מוצג במקום לחשב איחוד ימים בפרונט. |
| שני מספרי זימון תורים | מוצגים כשני ערכים נפרדים. |
| preferredAction=null | מוצג טקסט ללא פעולה. |
| preferredAction.copy | מוצגת פעולה העתקה. |
| preferredAction.tel | משתמשים ב-`uri`. |
| resume | משתמשים ב-`components.resume`, לא ב-`details`. |
| שפות/מומחיות/התמקצעות | מגיעות מ-components נפרדים, לא מ-summaryAttributes. |
