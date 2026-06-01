# מסמך 2: אכלוס מסכי תוצאות חיפוש ופרטי רופא מה-Payload

מקור אמת: `source_of_truth_simulator_with_provider_design.html`.

המסמך מתאר איך ה-payload שנוצר במסמך 1 מאכלס שני מסכים:

1. מסך תוצאות חיפוש.
2. מסך פרטי רופא.

## פערים והנחות לפני יישום

1. הרנדר הנוכחי הוא HTML בסימולטור, לא קומפוננטות מערכת אמיתית. המסמך מתאר את חוזה הנתונים וההתנהגות, לא CSS מחייב.
2. האווטאר במסכים מוצג כטקסט קבוע `"דר"` ולא משתמש ב-`entity.image.url`.
3. במסך תוצאות, לחיצה על שם רופא רק מחליפה tab בסימולטור למסך פרטים; במערכת אמיתית צריך להחליף זאת לניווט.
4. קישור כתובת משתמש ב-`navigationUrl || "#"`, וב-payload הנוכחי `navigationUrl` הוא תמיד `null`.
5. כפתור דוא"ל: בדסקטופ מנסה להעתיק ללוח, במובייל פותח `mailto`. זה תלוי בהרשאות דפדפן.
6. אקורדיונים נוצרים עם מזהה אקראי בזמן render; אין deep-link או state יציב לפתיחה/סגירה.

## עקרונות כלליים

### בריחה מ-XSS

כל ערך שמוזרק ל-HTML עובר `escapeHtml`.

ערכים שנכנסים לתוך handler של JavaScript עוברים גם `escapeJsString`.

### שליפת attributes

כדי להציג ערכי `summaryAttributes`, משתמשים ב:

```js
getAttributeValues(payload, type)
```

הפונקציה:

1. מחפשת ב-`payload.entity.summaryAttributes` פריט לפי `type`.
2. מחזירה `attr.values[].value`.
3. מסננת ערכים ריקים.
4. אם אין attribute, מחזירה `[]`.

### שורת metadata

```js
renderMetaLine(label, values)
```

אם יש ערכים מלאים, מוצג:

```html
<b>{label}:</b> value1, value2<br>
```

אם אין ערכים, לא מוצג דבר.

## מסך תוצאות חיפוש

פונקציה מרכזית: `renderSearchResults(payloads)`.

אם אין payloads:

```html
<div class="provider-empty">אין תוצאות להצגה</div>
```

אחרת נוצר container:

```html
<div class="search-results-screen">
  {SearchResultCard[]}
</div>
```

כל payload מוצג באמצעות `renderSearchResult(payload, index)`.

## כרטיס תוצאת חיפוש

מבנה הכרטיס:

```html
<article class="search-result-card">
  <div class="search-result-hero">
    <div>
      {שם}
      {metadata}
      {היעדרות ראשונה}
    </div>
    {avatar}
  </div>
  <div class="search-result-footer">
    {נגישות}
    {תגיות}
    {עיר}
    {הערות חירום}
  </div>
</article>
```

### שם רופא

מקור:

```js
payload.entity.displayName
```

התנהגות:

1. מוצג ככפתור טקסט.
2. לחיצה מפעילה `openEntityDetails(index)`.
3. בסימולטור הפעולה בוחרת את ה-payload ומעבירה ל-tab של מסך פרטים.

אם השם ריק, מוצגת מחרוזת ריקה.

### אווטאר

מוצג קבוע:

```html
<div class="search-result-avatar">דר</div>
```

לא נעשה שימוש ב-`payload.entity.image`.

### metadata

מקורות:

| רכיב | payload path | אופן תצוגה |
|---|---|---|
| מומחיות | `entity.summaryAttributes[type="specialization"]` | ערכים מופרדים בפסיק, ללא label |
| התמקצעות | `entity.summaryAttributes[type="professionalization"]` | `התמקצעות: ...` |
| שפות | `entity.summaryAttributes[type="language"]` | `שפות: ...` |

סדר תצוגה:

1. מומחיות.
2. התמקצעות.
3. שפות.

שדות ריקים לא מוצגים.

### היעדרות ראשונה

מקור:

```js
payload.availability.absences
```

תנאי הצגה:

1. `payload.availability.absences` קיים.
2. `payload.availability.absences.isVisible === true`.
3. נמצא פריט ראשון שבו `item.isDisplayed === true`.

תוכן:

```text
{firstAbsence.displayText}{, firstAbsence.reason}
```

אם `reason` ריק, לא מוסיפים אותו.

### footer

#### נגישות

מקור:

```js
payload.entity.accessibility
```

תנאי:

```js
accessibility.isVisible === true
```

תצוגה:

```text
♿ {accessibility.text || "נגיש"}
```

#### תגיות

מקור:

```js
payload.entity.badges
```

כל תגית מוצגת כ-chip עם:

```js
badge.text || ""
```

במסך תוצאות אין הבחנה ויזואלית בין סוגי תגיות, חוץ מנגישות.

#### עיר

העיר נלקחת לפי קדימות:

1. `payload.locations.items.find(loc => nonEmpty(loc.city)).city`
2. הערך הראשון מתוך `summaryAttributes[type="city"]`
3. מחרוזת ריקה

אם נמצאה עיר:

```text
⌖ {city}
```

#### הערות חירום

מקור:

```js
payload.remarks.items
```

תנאי לכל remark:

```js
remark.isVisible === true &&
remark.type === "emergency" &&
remark.targetArea === "pageTop" &&
nonEmpty(remark.text)
```

כל הערה מוצגת:

```text
✓ {remark.text}
```

## מסך פרטי רופא

פונקציה מרכזית: `renderScreen(payload)`.

אם אין payload:

```html
<div class="provider-empty">אין payload להצגה</div>
```

אחרת נבנה מסך:

```html
<div class="provider-screen service-screen">
  {topParts}
  <div class="service-layout {single אם אין sideParts}">
    <div class="service-main-column">{mainParts}</div>
    <div class="service-side-column">{sideParts}</div>
  </div>
</div>
```

## סדר הרכבת מסך פרטים

### topParts

1. כותרת קבועה:

```html
<h2 class="service-screen-title">איתור שירות</h2>
```

2. הערות עליונות, רק אם `payload.remarks.isVisible`.
3. כרטיס header של הרופא.

### mainParts

נבנים לפי הסדר הבא:

1. `העדרויות וממלאי מקום` אם `payload.availability.absences.isVisible`.
2. `שעות פעילות` אם `payload.availability.scheduleGroups.length > 0`.
3. `כתובת` אם `payload.locations.isVisible`.
4. `ניתוחים וטיפולים` אם `payload.services.isVisible`.
5. `פרטים נוספים` אם `payload.details.isVisible`.

### sideParts

נבנים לפי הסדר הבא:

1. `זימון תורים` אם קיימת תגית `limitedMembership`.
2. `אמצעי תקשורת` אם `payload.contacts.isVisible`.

אם `sideParts` ריק, layout מקבל class נוסף `single`.

## Header רופא

פונקציה: `renderHeader(payload)`.

מקורות:

| רכיב | payload path |
|---|---|
| שם | `entity.displayName` |
| מומחיות | `summaryAttributes[type="specialization"]` |
| התמקצעות | `summaryAttributes[type="professionalization"]` |
| שפות | `summaryAttributes[type="language"]` |
| subtitle | `entity.subtitle` |
| עיר | `summaryAttributes[type="city"]` |
| נגישות | `entity.accessibility` |
| לא מקבל חברים | `entity.badges[type="limitedMembership"]` |
| שב"ן | כל badge שאינו `limitedMembership` |

מבנה:

1. hero עם אווטאר קבוע `"דר"`.
2. שם רופא.
3. metadata:
   - `מומחיות: ...`
   - `התמקצעות: ...`
   - `שפות: ...`
   - `entity.subtitle`
   - ערי `city`
4. chips:
   - נגישות בירוק.
   - `limitedMembership` באדום.
   - תגיות שב"ן בכחול עם prefix `שב"ן`.

שדות ריקים לא מוצגים.

## הערות עליונות

פונקציה: `renderRemarks(payload)`.

נלקחות רק הערות שעונות על:

```js
r.isVisible && r.targetArea !== "locations"
```

כל הערה מוצגת כהודעת notice:

```html
<div class="provider-notice">
  <div class="provider-icon-dot">i</div>
  <div>{r.text}</div>
</div>
```

בפועל, לפי מסמך 1, רק הערות חירום הן visible.

## היעדרויות וממלאי מקום

פונקציה: `renderAbsences(payload)`.

תנאי:

1. קיים `payload.availability.absences`.
2. `absences.isVisible === true`.
3. יש לפחות פריט אחד עם `isDisplayed === true`.

כל פריט מוצג:

```text
! {item.displayText}{, item.reason}
```

אם `reason` ריק, לא מוסיפים אותו.

רק פריטים עם `isDisplayed=true` מוצגים, גם אם קיימות היעדרויות נוספות ב-payload.

## שעות פעילות

פונקציה: `renderScheduleOnly(payload)`.

תנאי:

```js
payload.availability.scheduleGroups.length > 0
```

לכל קבוצה ב-`scheduleGroups` נוצר אקורדיון.

כותרת אקורדיון:

```js
group.title
```

מצב פתיחה:

```js
group.display.defaultOpen !== false
```

כלומר:

1. `reception` פתוח כברירת מחדל.
2. שאר הסוגים סגורים כברירת מחדל.

מבנה טבלה:

| עמודה | מקור |
|---|---|
| יום | `day.dayLabel`, רק כאשר `row.showDayLabel=true` |
| שעה | `row.hoursText` |
| הערה | `row.notes || ""` |

ימים מוצגים אם:

```js
day.isVisible !== false
```

בפועל, ימים ללא שורות מקבלים `isVisible=false` ולכן לא מוצגים.

## כתובת

פונקציה: `renderLocations(payload)`.

לכל location מחושב:

```js
displayAddress = fullAddress || [city, street, houseNumber].filter(Boolean).join(" ")
```

רק locations עם `displayAddress` מלא מוצגים.

תצוגה:

1. לינק כתובת:

```html
<a href="{navigationUrl || '#'}">{displayAddress}</a>
```

2. הודעת נגישות אם `loc.accessibility.isVisible`.
3. הודעות מתוך `loc.messages[]`.

## ניתוחים וטיפולים

פונקציה: `renderServices(payload)`.

תנאי:

```js
payload.services.groups.length > 0
```

לכל group נוצר אקורדיון.

כותרת:

```js
group.title || ""
```

שורת טיפול:

| רכיב | payload path |
|---|---|
| שם טיפול | `item.name` |
| תג כיסוי | `item.coverageText` |
| class לכיסוי | `item.coverageType || "unknown"` |

התנהגות preview:

1. אם `group.display.hasOverflow=true`, ה-preview מציג `items.slice(0, previewRows)`.
2. אם אין overflow, אין preview נפרד.
3. `defaultOpen` מגיע מ-`group.display`.

לפי יצירת payload:

1. עד 2 טיפולים: האקורדיון פתוח.
2. 3 ומעלה: האקורדיון סגור ומציג preview של 2 טיפולים.

## פרטים נוספים

פונקציה: `renderDetails(payload)`.

תנאי:

```js
payload.details.sections.length > 0
```

לכל section:

1. מוצגת כותרת `section.title`.
2. לכל item:
   - אם יש `item.label`, מוצג `{label}:`.
   - מוצג `item.value`.

מבנה:

```html
<div class="kv">
  <b>{section.title}</b>
  <div><span class="muted">{item.label}:</span> {item.value}</div>
</div>
```

## זימון תורים

פונקציה: `renderAppointments(payload)`.

המסך אינו מציג בפועל טלפון זימון תורים כאן. הוא מציג הודעה רק במקרה של תגית מוגבלת:

```js
payload.entity.badges.find(badge => badge.type === "limitedMembership")
```

אם קיימת תגית:

```html
<div class="provider-notice">
  <div class="provider-icon-dot">i</div>
  <div>{limited.text}</div>
</div>
```

אם לא קיימת, section זימון תורים לא נוצר.

## אמצעי תקשורת

פונקציה: `renderContacts(payload)`.

תנאי:

```js
payload.contacts.items.length > 0
```

לכל קבוצת contact:

| רכיב | payload path |
|---|---|
| תווית | `contact.label` |
| ערכים | `contact.values[].value` |

עבור `contact.type === "email"`:

1. הערך מוצג ככפתור.
2. לחיצה מפעילה `handleEmailClick(email)`.
3. במובייל: `window.location.href = mailto`.
4. בדסקטופ: ניסיון להעתיק ל-clipboard ולהציג alert.
5. אם ההעתקה נכשלת: fallback ל-`mailto`.

עבור כל contact אחר:

הערך מוצג כטקסט בלבד. למרות שה-payload כולל `tel:` actions, הסימולטור לא הופך אותם ללינקים.

אחרי כל אמצעי הקשר, מוצגות הודעות:

```js
payload.contacts.messages[]
```

כל הודעה מוצגת כ-notice.

## אקורדיון

פונקציה: `renderAccordion(title, fullContent, display, previewContent)`.

כל אקורדיון כולל:

1. כפתור כותרת.
2. preview, אם נשלח `previewContent` והוא שונה מ-`fullContent`.
3. full content.

מצב פתיחה ראשוני:

```js
const isOpen = display.defaultOpen !== false;
```

אם פתוח:

1. `section-content` מוצג.
2. `section-preview` מוסתר.
3. chevron הוא `⌃`.

אם סגור:

1. `section-content` מוסתר.
2. `section-preview` מוצג, אם קיים.
3. chevron הוא `⌄`.

לחיצה הופכת את המצב.

## ניהול state בסימולטור

```js
state = {
  payloads: [],
  selectedIndex: 0,
  activeTab: "screen"
}
```

בעת שינוי JSON:

1. `JSON.parse`.
2. `extractHits(parsed)`.
3. `sources.map(transformSourceToPayload)`.
4. `selectedIndex=0`.
5. רנדר מחדש של כל המסכים.

בעת פתיחת פרטים מתוצאת חיפוש:

```js
openEntityDetails(index)
```

הפעולה:

1. מעדכנת `state.selectedIndex`.
2. מעדכנת `state.activeTab = "screen"`.
3. מסמנת את tab פרטי רופא כפעיל.
4. מרנדרת מחדש.

## טבלת אכלוס מקוצרת

| מסך | אזור | payload path | תנאי הצגה |
|---|---|---|---|
| תוצאות | שם | `entity.displayName` | תמיד, גם אם ריק |
| תוצאות | מומחיות | `summaryAttributes.specialization` | אם יש ערכים |
| תוצאות | התמקצעות | `summaryAttributes.professionalization` | אם יש ערכים |
| תוצאות | שפות | `summaryAttributes.language` | אם יש ערכים |
| תוצאות | היעדרות | `availability.absences.items[isDisplayed]` | אם `absences.isVisible` |
| תוצאות | נגישות | `entity.accessibility` | אם `isVisible` |
| תוצאות | תגיות | `entity.badges` | אם קיימות |
| תוצאות | עיר | `locations.items[].city` או `summaryAttributes.city` | אם יש עיר |
| תוצאות | הערות חירום | `remarks.items[type=emergency]` | אם visible ו-pageTop |
| פרטים | header | `entity` + `summaryAttributes` + `badges` | תמיד |
| פרטים | הערות | `remarks.items` | אם `remarks.isVisible` |
| פרטים | היעדרויות | `availability.absences` | אם `isVisible` |
| פרטים | שעות | `availability.scheduleGroups` | אם יש קבוצות |
| פרטים | כתובת | `locations.items` | אם `locations.isVisible` ויש כתובת |
| פרטים | טיפולים | `services.groups` | אם `services.isVisible` |
| פרטים | פרטים נוספים | `details.sections` | אם `details.isVisible` |
| פרטים | זימון תורים | `entity.badges.limitedMembership` | אם קיימת תגית |
| פרטים | אמצעי קשר | `contacts.items` | אם `contacts.isVisible` |

## כללי empty state

1. אין payload למסך פרטים: `אין payload להצגה`.
2. אין תוצאות חיפוש: `אין תוצאות להצגה`.
3. שדה ריק בתוך אזור קיים בדרך כלל לא מוצג.
4. אזור שלם לא מוצג אם תנאי ה-`isVisible` או אורך המערך אינם מתקיימים.
5. `locations.isVisible` תמיד true ב-payload, אבל בפועל לא יוצג location אם אין `displayAddress`.
