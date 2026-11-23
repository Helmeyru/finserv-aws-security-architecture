# خطة معالجة الملاحظات الأمنية وفق معيار أساسيات أمن سحابة أمازون

**المشروع:** مشروع منارة الثامن (Manara Project 8) — بنية FinServ السحابية الآمنة متعددة الطبقات  
**منطقة AWS:** `us-east-1`  
**تاريخ الأدلة:** 22 نوفمبر 2026 (تُعرض الطوابع الزمنية في نتائج Security Hub بتوقيت UTC)  
**نطاق الوثيقة:** مختبر تعليمي لأمن AWS، والغرض من الوثيقة التخطيط للمعالجة التصحيحية؛ ولا تمثل شهادة امتثال أو ادعاءً بإصلاح المشكلات المذكورة.

**اللغة:** [العربية](cis-remediation-plan.ar.md) | [English](cis-remediation-plan.md)

## 1. نتائج التقييم الأمني والأدلة المسجلة

أظهرت لوحة معلومات **AWS Security Hub** المسجلة نتيجة قدرها **38%**، مع **15 ضابطًا ناجحًا و11 ضابطًا فاشلًا** من أصل **40 ضابطًا مفعّلًا**. وفي استعلام لاحق للقراءة فقط باستخدام AWS CLI، ظهرت **26 نتيجة اكتشاف نشطة وفاشلة (ACTIVE/FAILED)** موزعة على **12 معرّفًا مختلفًا لضوابط Security Hub**. وأظهر الحقل `Compliance.AssociatedStandards[].StandardsId` القيمة **`standards/cis-aws-foundations-benchmark/v/5.0.0`** لكل نتيجة أعادها الاستعلام.

هذان **رصدان مختلفان**: عدد الضوابط في لوحة المعلومات ليس بالضرورة مساويًا لعدد نتائج الاكتشاف المتعلقة بالموارد، بسبب توقيت التحديث أو التجميع أو تغير الحالات. لذلك **لا يجوز استبدال النسبة التاريخية البالغة 38% بنسبة جديدة غير مستندة إلى قياس فعلي**. ويجب إعادة الاستعلام عن لوحة المعيار ونتائج الاكتشاف الحالية قبل الادعاء بأي تحسن عقب المعالجة. كما أن تفصيل الضوابط الأربعين السابق لا يحدد حالات الضوابط الأربعة عشر الأخرى.

> **الحالة لجميع البنود أدناه: مفتوحة / لم يتم التحقق من المعالجة (OPEN / REMEDIATION NOT VERIFIED).** يجب البدء بفحص آمن يقتصر على القراءة، والحصول على الموافقة قبل تعديل أي مورد. **لا تطبّق تلقائيًا تعديلات جماعية** على سياسات S3 أو ميزة MFA Delete أو IAM أو CloudTrail أو AWS Config.

## 2. الملاحظات الأمنية وخطوات المعالجة المقترحة

<table dir="rtl">
<thead>
<tr>
<th align="right" dir="rtl"><strong>الضابط</strong></th>
<th align="right" dir="rtl"><strong>عدد النتائج النشطة</strong></th>
<th align="right" dir="rtl"><strong>الخطر أو نتيجة الفحص</strong></th>
<th align="right" dir="rtl"><strong>الأولوية</strong></th>
<th align="right" dir="rtl"><strong>المعالجة المقترحة</strong></th>
<th align="right" dir="rtl"><strong>أدلة التحقق المطلوبة</strong></th>
<th align="right" dir="rtl"><strong>الحالة</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="right" dir="rtl"><strong>IAM.6</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">الحساب الجذري (Root) لا يحقق متطلبات المصادقة متعددة العوامل باستخدام جهاز مادي (Hardware MFA).</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">مراجعة وسائل استعادة الوصول إلى الحساب الجذري، وتسجيل جهاز مصادقة مادي مقاوم للتصيّد ومعتمد وفق سياسة الحساب، مع توفير وسيلة استعادة إضافية حيثما كانت مدعومة.</td>
<td align="right" dir="rtl">مراجعة إعدادات MFA للحساب الجذري وانتظار إعادة التقييم في Security Hub.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>IAM.15</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">الحد الأدنى لطول كلمات مرور IAM أقل من 14 حرفًا.</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">ضبط سياسة كلمات المرور في حساب IAM بحيث يكون الحد الأدنى <strong>14 حرفًا أو أكثر</strong>، بعد مراجعة أثر التغيير على مستخدمي IAM.</td>
<td align="right" dir="rtl"><code>aws iam get-account-password-policy</code> ثم التحقق من إعادة تقييم الضابط.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>IAM.16</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">عدم فرض قيود تمنع إعادة استخدام كلمات المرور السابقة.</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">تحديد سجل مناسب لمنع إعادة استخدام كلمات المرور، مثل آخر 24 كلمة مرور إذا سمحت سياسة الحساب بذلك.</td>
<td align="right" dir="rtl"><code>aws iam get-account-password-policy</code> ثم التحقق من إعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>Account.1</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">غياب جهة الاتصال الأمنية البديلة على مستوى حساب AWS.</td>
<td align="right" dir="rtl">متوسطة</td>
<td align="right" dir="rtl">إضافة جهة اتصال أمنية بديلة تُراقَب باستمرار، والتحقق من صحة بياناتها.</td>
<td align="right" dir="rtl"><code>aws account get-alternate-contact --alternate-contact-type SECURITY</code> عند توافر الصلاحية، ثم إعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>IAM.18</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">عدم وجود دور IAM مخصص للوصول إلى AWS Support أثناء الحوادث.</td>
<td align="right" dir="rtl">متوسطة</td>
<td align="right" dir="rtl">مراجعة نص الضابط وتصميم صلاحيات الوصول إلى AWS Support، ثم إنشاء دور مناسب ومحدد الصلاحيات وبسياسة ثقة معتمدة عند الحاجة.</td>
<td align="right" dir="rtl">مراجعة الدور وسياساته ثم إعادة التقييم في Security Hub.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>CloudTrail.7</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">حاوية S3 المستخدمة لحفظ سجلات CloudTrail لا تفعّل تسجيل طلبات الوصول إلى الحاوية (S3 Server Access Logging).</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">تحديد حاوية CloudTrail المقصودة بدقة، وتوجيه سجلات الوصول إلى حاوية أخرى مستقلة ومحمية؛ مع منع التسجيل الذاتي المتكرر والتحقق من وصول السجلات.</td>
<td align="right" dir="rtl">إعدادات تسجيل الوصول إلى حاوية S3، وأدلة تسليم السجلات، وإعادة تقييم الضابط.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>Config.1</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">إعداد مسجّل AWS Config أو الدور المرتبط بالخدمة لا يطابق متطلبات الضابط.</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">فحص مسجّل التكوين الحالي ودور IAM المستخدم، والتخطيط للانتقال إلى دور مرتبط بالخدمة فقط إذا كانت هناك حاجة مبررة، دون تعطيل التسجيل.</td>
<td align="right" dir="rtl"><code>aws configservice describe-configuration-recorders</code>، وحالة المسجّل، وإعادة تقييم النتيجة.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>S3.22</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">عدم تدقيق أحداث الكتابة على مستوى كائنات S3.</td>
<td align="right" dir="rtl">متوسطة</td>
<td align="right" dir="rtl">تحديد نطاق أحداث بيانات CloudTrail الخاصة <strong>بالكتابة</strong> للحاويات أو البادئات المعتمدة، وتقدير تكلفة أحداث البيانات أولًا.</td>
<td align="right" dir="rtl">محددات أحداث CloudTrail، وحدث كتابة اختباري، وإعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>S3.23</strong></td>
<td align="right" dir="rtl">1</td>
<td align="right" dir="rtl">عدم تدقيق أحداث القراءة على مستوى كائنات S3.</td>
<td align="right" dir="rtl">متوسطة</td>
<td align="right" dir="rtl">تحديد نطاق أحداث بيانات CloudTrail الخاصة <strong>بالقراءة</strong> للحاويات أو البادئات المعتمدة، وتقدير تكلفة أحداث البيانات أولًا.</td>
<td align="right" dir="rtl">محددات أحداث CloudTrail، وحدث قراءة اختباري، وإعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>S3.5</strong></td>
<td align="right" dir="rtl"><strong>7</strong></td>
<td align="right" dir="rtl">عدة حاويات S3 تفتقر إلى سياسة صريحة تفرض الاتصال المشفر باستخدام TLS فقط.</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">تحديد الموارد السبعة المتأثرة جميعها، وإضافة قاعدة <code>Deny</code> آمنة عندما تكون <code>aws:SecureTransport=false</code>، مع دمجها في السياسات الحالية التي يحتاج إليها CloudFront OAC وCloudTrail وConfig، وعدم استبدال تلك السياسات بالكامل.</td>
<td align="right" dir="rtl">فحص <code>get-bucket-policy</code> لكل حاوية، وإجراء اختبار HTTPS غير ضار، ومراجعة السياسة، ثم إعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>S3.20</strong></td>
<td align="right" dir="rtl"><strong>8</strong></td>
<td align="right" dir="rtl">ميزة <code>MFA Delete</code> غير مفعّلة في الحاويات المتأثرة.</td>
<td align="right" dir="rtl">متوسطة / حساسة تشغيليًا</td>
<td align="right" dir="rtl">تحديد كل حاوية متأثرة وحالة تفعيل الإصدار فيها (Versioning)، وتقييم ملاءمة MFA Delete للمختبر. تتطلب هذه الميزة إجراءات خاصة باستخدام الحساب الجذري وواجهة API وقد تعقّد عمليات CloudFormation؛ <strong>لا تفعّلها جماعيًا</strong>.</td>
<td align="right" dir="rtl">فحص <code>get-bucket-versioning</code> لكل حاوية، وتوثيق قرار المخاطر، وإعادة التقييم أو تسجيل استثناء مقبول للمختبر.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
<tr>
<td align="right" dir="rtl"><strong>EC2.2</strong></td>
<td align="right" dir="rtl"><strong>2</strong></td>
<td align="right" dir="rtl">مجموعات الأمان الافتراضية لبعض شبكات VPC تسمح بحركة مرور.</td>
<td align="right" dir="rtl">مرتفعة</td>
<td align="right" dir="rtl">تحديد شبكات VPC ومجموعات الأمان الافتراضية المعنية، وجرد واجهات الشبكة (ENIs) التي تستخدمها، ثم إزالة قواعد الدخول والخروج غير الضرورية فقط بعد التحقق من التبعيات.</td>
<td align="right" dir="rtl"><code>describe-security-groups</code>، وجرد الاستخدام، ثم إعادة التقييم.</td>
<td align="right" dir="rtl">مفتوحة</td>
</tr>
</tbody>
</table>

### الأعداد المسجلة

- مجموع النتائج: **26 نتيجة اكتشاف** = **8 × S3.20 + 7 × S3.5 + 2 × EC2.2 + 9 نتائج منفردة لضوابط أخرى**.
- **12 معرّفًا مختلفًا للضوابط**؛ جميع النتائج المعادة في الاستعلام كانت `ACTIVE` و`FAILED`، وحالة سير العمل `NEW`.
- هذه الأعداد **ليست نسبة امتثال جديدة** ولا تثبت أن الضوابط الاثني عشر كلها محسوبة حاليًا بوصفها فاشلة في أحدث نتيجة للوحة المعلومات.

## 3. خطوات الفحص الآمن دون تعديل الموارد

1. حفظ نسخة من نتائج اكتشاف Security Hub الحالية وتسجيل وقت الرصد؛ ثم تجميع النتائج بحسب `Compliance.SecurityControlId` و`Resources[].Id`، حتى لا تُحدد أهداف المعالجة على سبيل التخمين.
2. تحديد حاويات S3 السبع المتأثرة بضابط `S3.5`، والحاويات الثماني المتأثرة بضابط `S3.20`، ومجموعتي الأمان الافتراضيتين المتأثرتين بضابط `EC2.2`، مع تجنب تعديل الصلاحيات على نطاق واسع.
3. فحص سياسة كلمات مرور IAM في الحساب، وجهة الاتصال الأمنية البديلة، وخيارات مصادقة الحساب الجذري، ودور الدعم الحالي.
4. فحص مسجّل AWS Config ودور التسجيل، وحاوية الوجهة الفعلية لسجلات CloudTrail، قبل تعديل أي إعداد.
5. تقدير التكلفة الإضافية لأحداث بيانات CloudTrail وتخزين سجلات الوصول إلى S3، ومقارنتها بميزانية المختبر.
6. الحصول على موافقة صريحة لكل تغيير مخطط له، وتنفيذ معالجة ضابط واحد في كل مرة على موارد غير إنتاجية، ثم اختبار استمرار عمل الموارد وتوثيق الأدلة.
7. إعادة فحص Security Hub بعد تحديث تقييم الضابط، وتسجيل النتائج **قبل المعالجة وبعدها لكل ضابط ومورد**، لا الاكتفاء بنسبة الامتثال الإجمالية.

## 4. حدود التقييم وخطوات المتابعة

- سبق التحقق من تسجيل **أحداث الإدارة (Management Events)** في CloudTrail الرئيسي لمشروع FinServ عبر مناطق متعددة، وكذلك سلامة ملفات السجل. أما **أحداث البيانات على مستوى كائنات S3** فهي متطلبات منفصلة لم يُثبت تفعيلها.
- أرسلت VPC Flow Logs الخاصة بالمشروع سجلات `NODATA` إلى CloudWatch؛ ولم يُثبت بعد التقاط بيانات حركة المرور من نوع `ACCEPT` أو `REJECT`. كما بقيت سجلات Flow Logs لشبكة VPC الافتراضية خارج نطاق التحقق الموثق.
- نجح اختبار المعالجة التصحيحية **اليدوية** باستخدام AWS Config وSSM Automation؛ وظلت المعالجة التلقائية معطلة عمدًا. ولا يجوز تغيير ذلك ضمن هذا التدقيق دون خطة اختبار محددة النطاق.
- فشل ضابط `S3.5` لا يعني أن الوصول عبر HTTPS كان مستحيلًا، أو أن CloudFront OAC لم يعمل. فالضابط يختبر وجود سياسة صريحة على حاوية S3 لفرض الاتصال المشفر.
- تقييم CIS المذكور **ليس شهادة امتثال رسمية ولا إقرارًا بجاهزية البيئة للإنتاج**.

## 5. أوامر جمع الأدلة (للقراءة فقط)

الأوامر التالية مأخوذة من الوثيقة الإنجليزية دون تغيير في معانيها أو معاملاتها:

```powershell
aws securityhub get-findings --region us-east-1 --query "Findings[?Compliance.Status=='FAILED' && RecordState=='ACTIVE'].[Compliance.SecurityControlId,Resources[].Id]" --output json --no-cli-pager
aws iam get-account-password-policy --output json --no-cli-pager
aws configservice describe-configuration-recorders --region us-east-1 --output json --no-cli-pager
aws cloudtrail describe-trails --region us-east-1 --query "trailList[].{Name:Name,S3BucketName:S3BucketName,IsMultiRegionTrail:IsMultiRegionTrail}" --output table --no-cli-pager
```

إذا فشل أحد الأوامر بسبب عدم كفاية صلاحيات الحساب، **فوثّق هذا القيد**؛ ولا توسّع الصلاحيات لمجرد إظهار نتيجة تدقيق ناجحة.

## 6. مراجع المستودع

- [الصفحة الرئيسية للمشروع — العربية](../README.ar.md) | [الإنجليزية](../README.md)
- [مصفوفة تحقيق المتطلبات — العربية](requirements-compliance.ar.md) | [الإنجليزية](requirements-compliance.md)
- [نتائج الاختبارات الأمنية — العربية](../tests/security-test-results.ar.md) | [الإنجليزية](../tests/security-test-results.md)
- [دليل نشر CloudFormation والتحقق — العربية](deployment-guide.ar.md) | [الإنجليزية](deployment-guide.md)

---

**ملاحظة توثيقية:** تعرض هذه الوثيقة النتائج التاريخية وخطوات المعالجة المقترحة كما وردت في المرجع الإنجليزي بتاريخ 22 نوفمبر 2026. لم تُنفّذ أي إجراءات معالجة على AWS عند إعداد هذه الترجمة، ولم يُجرَ فحص مباشر جديد للحساب.
