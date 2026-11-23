# تقرير الاختبارات الأمنية والتحقق من تشغيل مكونات المشروع

**المشروع:** مشروع منارة الثامن (Manara AWS Project 8)

**منطقة AWS:** `us-east-1`

**البيئة:** مختبر تطبيقي لعرض ضوابط الأمن السحابي على AWS

**تواريخ التحقق:** 21–22 نوفمبر 2026

**الحالة العامة:** اكتملت فحوص الأيام 21–23، ونجح في اليوم 24 اختبار التكامل بين GuardDuty وEventBridge وLambda. ويوثّق القسم العاشر نتائج التحقق من معيار CIS v5.0، وإعدادات منع الوصول العام إلى S3 على مستوى الحساب، والتشفير الافتراضي لـEBS، ووصول سجلات VPC Flow Logs إلى CloudWatch. ولا يزال إرسال إشعارات SNS تلقائيًا، وتحقيق الامتثال الكامل لمعيار CIS، والتقاط سجلات الحركة الفعلية من النوعين `ACCEPT` و`REJECT` أمورًا لم يُتحقق منها.

**اللغة:** [العربية](security-test-results.ar.md) | [English](security-test-results.md)

> **نطاق التقرير:** جميع النتائج أدناه مستمدة من الاختبارات الموثقة في المشروع، ولا تعني إعادة فحص الموارد الحية وقت قراءة هذه الوثيقة. وتشير كلمة **«ناجح»** إلى نجاح الفحص المحدد فقط، ولا تعني اعتماد البيئة للاستخدام الإنتاجي.

## 1. الغرض من التقرير ونطاقه

يسجّل هذا التقرير الضوابط الأمنية وسلوك التطبيق اللذين جرى التحقق منهما عمليًا خلال مشروع FinServ. ويعني التصنيف **PASS (ناجح)** أن الفحص المذكور قد نجح ضمن نطاقه؛ ولا يُعدّ شهادة بجاهزية بيئة إنتاجية، أو ضمانًا للأمن الشامل، أو دليلًا على نجاح نشر جميع قوالب CloudFormation.

يستخدم التطبيق بيانات تجريبية اصطناعية. وتتيح واجهة API Gateway حاليًا المسار `GET /health` فقط، أما سير عمل المعاملات فقد اختُبر باستدعاء دالة Lambda مباشرة، وليس من خلال مسار معاملات في API Gateway.

## 2. ملخص نتائج التحقق

<table dir="rtl">
<thead><tr><th align="right"><strong>المجال</strong></th><th align="right"><strong>الفحص المنفّذ</strong></th><th align="right"><strong>النتيجة</strong></th></tr></thead>
<tbody>
<tr><td align="right">تصميم VPC</td><td align="right">مراجعة توزيع الشبكات الفرعية وجداول التوجيه وقواعد مجموعات الأمان.</td><td align="right"><strong>PASS</strong> — تمت مراجعة الإعدادات.</td></tr>
<tr><td align="right">عزل الشبكات</td><td align="right">السماح بالاتصال الوارد إلى مجموعة أمان التطبيق عبر TCP 8080 من مجموعة أمان الويب، والاتصال الصادر عبر TCP 5432 إلى مجموعة أمان قواعد البيانات، واتصال HTTPS إلى قائمة بادئات DynamoDB.</td><td align="right"><strong>PASS</strong> — تمت مراجعة الإعدادات.</td></tr>
<tr><td align="right">نقطة نهاية DynamoDB من نوع Gateway</td><td align="right">حالتها <code>available</code> ومرتبطة بجدول توجيه التطبيق.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">IAM</td><td align="right">مراجعة سياسة مضمنة لدور التطبيق تقصر الصلاحيات على <code>PutItem</code> و<code>GetItem</code> و<code>DescribeTable</code> لجدول المعاملات.</td><td align="right"><strong>PASS</strong> — مراجعة سياسة محددة النطاق.</td></tr>
<tr><td align="right">تشفير S3</td><td align="right">التشفير الافتراضي باستخدام SSE-KMS بمفتاح يديره العميل، مع تفعيل S3 Bucket Key وحظر SSE-C.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">منع الوصول العام إلى S3</td><td align="right">التحقق من إعدادات الحماية الأربع ضمن S3 Block Public Access في مرحلة سابقة من المشروع.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">KMS</td><td align="right">التحقق من تفعيل مفتاح يديره العميل، ورفع كائن اختباري مشفّر إلى S3 واسترجاعه.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">فحص صحة API Gateway</td><td align="right">أعاد المسار <code>GET /health</code> الرمز HTTP 200 والاستجابة <code>status: healthy</code>.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">Lambda وDynamoDB</td><td align="right">نجح الاستدعاء المباشر لـLambda في تنفيذ عمليتَي <code>PutItem</code> و<code>GetItem</code> على بيانات معاملات اصطناعية.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">CloudWatch Logs</td><td align="right">تأكيد وجود سجلات تنفيذ فحص الصحة والمعاملات.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">CloudFront</td><td align="right">التحقق سابقًا من اكتمال نشر التوزيع والحصول على استجابات HTTP 200.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">تشفير كائن موقع CloudFront</td><td align="right">ظهر الترويس <code>x-amz-server-side-encryption: AES256</code> عند استرجاع كائن الموقع، ما يثبت استخدام SSE-S3 لذلك الكائن فقط. ولا يثبت الترويس وحده إعداد التشفير الافتراضي للحاوية.</td><td align="right"><strong>PASS</strong> — للكائن المسترجَع.</td></tr>
<tr><td align="right">AWS WAF</td><td align="right">فحص مقاييس مجموعات القواعد المُدارة والطلبات المأخوذة عيناتها في وضع <strong>Count</strong>.</td><td align="right"><strong>PASS</strong> — وضع المراقبة فقط.</td></tr>
<tr><td align="right">AWS Config</td><td align="right">أعادت ثلاث قواعد خاصة بالمشروع الحالة <code>COMPLIANT</code>.</td><td align="right"><strong>PASS</strong> — حالة القواعد المبلّغ عنها.</td></tr>
<tr><td align="right">المعالجة التصحيحية باستخدام SSM</td><td align="right">اكتملت سابقًا معالجة يدوية محددة النطاق لإعدادات تشفير S3.</td><td align="right"><strong>PASS</strong> — تنفيذ يدوي.</td></tr>
<tr><td align="right">GuardDuty</td><td align="right">الكاشف في حالة <code>ENABLED</code>؛ وتم فحص نتائج اصطناعية مرتبطة بـTor ونتيجة Backdoor بدرجة خطورة 8.</td><td align="right"><strong>PASS</strong> — نتائج اصطناعية فقط.</td></tr>
<tr><td align="right">GuardDuty ← EventBridge ← Lambda</td><td align="right">طابقت نتيجة بدرجة خطورة 8 قاعدة EventBridge، وعالجتها Lambda بقرار <code>ESCALATE</code> دون أخطاء.</td><td align="right"><strong>PASS</strong> — تكامل معالجة الحادث، مع تخطي SNS.</td></tr>
<tr><td align="right">Security Hub</td><td align="right">استرجاع نتائج الفحص؛ وكانت نتائج الامتثال الخمس المعروضة تحمل الحالتين <code>PASSED</code> و<code>RESOLVED</code>.</td><td align="right"><strong>PASS</strong> — الاسترجاع والمراجعة.</td></tr>
<tr><td align="right">CloudTrail الخاص بـFinServ</td><td align="right">التحقق من عمل التسجيل في <code>finserv-security-trail</code> دون الإبلاغ عن أخطاء في التسليم.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">إنذارات CloudWatch</td><td align="right">ثلاثة إنذارات للمشروع بحالة <code>OK</code> وإجراءات الإنذار مفعّلة.</td><td align="right"><strong>PASS</strong> — الإعداد والحالة فقط.</td></tr>
<tr><td align="right">اشتراك SNS</td><td align="right">تأكيد اشتراك البريد الإلكتروني.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">التسليم المباشر عبر SNS</td><td align="right">نشر رسالة اختبار واستلامها عبر Gmail.</td><td align="right"><strong>PASS</strong> — إرسال مباشر.</td></tr>
<tr><td align="right">تشفير EBS</td><td align="right">التحقق من وحدة تخزين EBS مؤقتة مشفّرة ثم حذفها.</td><td align="right"><strong>PASS</strong> — تجربة مؤقتة.</td></tr>
<tr><td align="right">أمن RDS</td><td align="right">اختبار مثيل PostgreSQL خاص ومشفّر مع بيانات اعتماد مُدارة، ثم حذفه.</td><td align="right"><strong>PASS</strong> — تجربة مؤقتة.</td></tr>
<tr><td align="right">إعداد أسرار RDS</td><td align="right">ملاحظة إعداد تدوير بيانات اعتماد المستخدم الرئيسي المُدارة بواسطة RDS وإصدار محدّث للسر أثناء التجربة؛ وهو تدوير تديره AWS وليس دالة Lambda مخصّصة.</td><td align="right"><strong>PASS</strong> — إعداد وإصدار سر، وليس اختبار دورة تدوير مجدولة مكتملة.</td></tr>
<tr><td align="right">تنظيف الموارد</td><td align="right">لم تُظهر عملية الجرد في <code>us-east-1</code> مثيلات RDS أو وحدات تخزين EBS أو لقطات RDS يدوية.</td><td align="right"><strong>PASS</strong> — جرد محدد النطاق.</td></tr>
<tr><td align="right">مراجعة التكلفة</td><td align="right">التكلفة الإجمالية غير الممزوجة على مستوى الحساب خلال 1–21 نوفمبر: قرابة <strong>1.65 دولار أمريكي</strong>.</td><td align="right"><strong>PASS</strong> — ضمن الميزانية المرجعية البالغة 20 دولارًا؛ وقد تتأخر بيانات الفوترة.</td></tr>
</tbody>
</table>

## 3. اليوم 21 — اختبار تكامل التطبيق

**التاريخ:** 21 نوفمبر 2026

**النتيجة:** **PASS** لمسارات التطبيق التي نُفّذت بالفعل.

- **API Gateway:** أعاد المسار `GET /health` الرمز HTTP `200` واستجابة الصحة المتوقعة.
- **التوجيه (Routing):** أعاد المسار `GET /transaction` الرمز `404` لأن **هذا المسار غير مُهيأ أصلًا**. وهذا قيد في التوجيه، وليس فشلًا في إجراء معاملة DynamoDB.
- **Lambda:** كانت الدالة `finserv-private-app` بحالة `Active`، وتعمل باستخدام Python 3.12 داخل VPC المتوقعة وشبكتين فرعيتين خاصتين للتطبيق.
- **DynamoDB:** كان جدول `finserv-transactions` بحالة `ACTIVE`.
- **اختبار المعاملة المباشر:** أعادت Lambda القيم `statusCode: 200` و`verified: true` و`operation: PutItem + GetItem`.
- **معرّف المعاملة الاصطناعية:** `c91949aa-52ba-4b69-a922-a58a918af4d6`.
- **الأداء:** زمن التنفيذ 377.98 مللي ثانية؛ وأقصى استخدام ملحوظ للذاكرة 90 ميغابايت.
- **السجلات:** سجّل CloudWatch حفظ المعاملة واسترجاعها.

**حدود الاختبار:** اختُبر فحص الصحة اختبارًا متكاملًا من طرف إلى طرف عبر API Gateway، أما الكتابة إلى DynamoDB والقراءة منه فقد اختُبرتا باستدعاء Lambda مباشرة؛ **ولم** تُختبرا عبر نقطة معاملات في API Gateway.

## 4. اليوم 22 — المراقبة الأمنية واكتشاف التهديدات

**التاريخ المدوّن للاختبارات:** 21 نوفمبر 2026

**النتيجة:** **PASS** لإعداد الخدمات واسترجاع نتائج الكشف والتسليم المباشر عبر SNS.

- **GuardDuty:** الكاشف مفعّل، مع تفعيل مصادر بيانات CloudTrail وDNS وVPC Flow Logs. وبقيت ميزات حماية اختيارية أخرى معطّلة.
- **نتيجة GuardDuty:** كانت نتيجة عقدة خروج Tor عيّنة مُنشأة صراحة (`sample: true`)، **وليست دليلًا على وقوع هجوم مؤكد**.
- **Security Hub:** تم استرجاع النتائج بنجاح، وكانت نتائج الامتثال المعروضة تحمل الحالتين `PASSED` و`RESOLVED`.
- **CloudTrail:** كان مسار التدقيق الرئيسي الخاص بـFinServ يسجّل الأحداث دون الإبلاغ عن أخطاء تسليم.
- **AWS Config:** أعادت قواعد المشروع الثلاث حالة `COMPLIANT`.
- **CloudWatch:** كانت إنذارات المشروع الثلاثة بحالة `OK`، مع تفعيل إجراءات الإشعار.
- **SNS:** تأكد اشتراك البريد الإلكتروني، ونجح تسليم رسالة **مباشرة** إلى Gmail.
- **معرّف رسالة SNS الاختبارية:** `44207327-55e0-546d-b5f7-105ce2dbec62`.

**ما لم يُتحقق منه بعد:** التسليم التلقائي الناتج عن انتقال إنذار CloudWatch فعليًا إلى حالة إنذار، أو التسليم عبر تكامل GuardDuty إلى SNS. **ولا يُثبت** نجاح رسالة SNS المباشرة صحة هذين المسارين.

## 5. اليوم الثالث والعشرون: تدقيق الصلاحيات والتشفير والشبكات والتكلفة

**التاريخ:** 21 نوفمبر 2026

**النتيجة:** **PASS** للضوابط التي جرت مراجعتها، مع ملاحظة متابعة واحدة تخص CloudTrail.

- **IAM:** تسمح سياسة دور Lambda الخاص بالتطبيق بالإجراءات `dynamodb:PutItem` و`GetItem` و`DescribeTable` على جدول DynamoDB الخاص بالمشروع فقط. ولا تشمل هذه المراجعة جميع الصلاحيات الفعلية المحتملة الناتجة عن سياسات أخرى مرفقة أو ضوابط على مستوى المؤسسة.
- **S3:** التشفير الافتراضي باستخدام SSE-KMS بمفتاح يديره العميل مفعّل، وكذلك Bucket Key، مع حظر SSE-C.
- **نقطة نهاية VPC:** نقطة نهاية DynamoDB من نوع Gateway بحالة `available` ومرتبطة بجدول توجيه التطبيق.
- **مجموعة الأمان:** تأكد تقييد قواعد الدخول والخروج، ولم تُرصد قاعدة خروج غير مقيّدة إلى `0.0.0.0/0` في مجموعة أمان التطبيق.
- **الموارد المؤقتة:** لم تظهر مثيلات RDS ولا وحدات تخزين EBS ولا لقطات RDS يدوية في المنطقة المفحوصة.
- **التكلفة المسجّلة:** بلغت التكلفة غير الممزوجة على مستوى الحساب `USD 1.6473` للفترة 1–21 نوفمبر 2026، وأسهمت KMS بنحو `USD 1.13`. وقد تتأخر بيانات Cost Explorer ولا تكون خاصة بالمشروع وحده.
- **CloudTrail الرئيسي:** كان المسار `finserv-security-trail` يعمل ويسجّل بنجاح.
- **ملاحظة CloudTrail قديم:** المسار `security-trail` في المنطقة `us-east-2` يعود إلى مختبر AWS سابق ولا يرتبط بمشروع FinServ. التسجيل فيه معطّل، ووجهته السابقة في S3 لم تعد موجودة (HTTP `404`). وتُرك المسار دون تغيير.

## 6. اليوم الرابع والعشرون: اختبار التكامل بين اكتشاف التهديدات والاستجابة للحوادث

**تاريخ الاختبار:** 22 نوفمبر 2026، نحو الساعة 02:47 بتوقيت اليمن (`UTC+03:00`)، الموافق 21 نوفمبر 2026 الساعة 23:47 بالتوقيت العالمي UTC في سجلات CloudWatch.

**المنطقة:** `us-east-1`

**النتيجة:** **PASSED — ناجح**

**الهدف:** التأكد من أن نتيجة كشف من GuardDuty تُفعّل قاعدة EventBridge، ثم تستدعي دالة Lambda الخاصة بالاستجابة للحوادث.

**نوع نتيجة الاختبار:** `Backdoor:EC2/C&CActivity.B`

**درجة الخطورة:** `8.0` — عالية (High)

**مصدر النتيجة:** نتيجة اصطناعية أُنشئت لأغراض الاختبار في GuardDuty.

**النتائج التي جرى التحقق منها:**

- `EventBridge MatchedEvents`: **1**.
- `EventBridge Invocations`: **1**.
- `Lambda Invocations`: **1**.
- `Lambda Errors`: **0**.
- قرار Lambda: `ESCALATE`.
- معرّف نتيجة الاختبار: `5f165576d34240a8a225223a355882f2`.
- أظهر سجل المعالجة القيمة `sample: false`، مع أن النتيجة أُنشئت باستخدام واجهة GuardDuty الخاصة بتوليد النتائج الاصطناعية.
- اكتمل تنفيذ Lambda بنجاح.
- تتوفر أدلة التنفيذ في سجلات CloudWatch Logs.

**قيد الاختبار:** كانت إشعارات SNS معطلة باستخدام `ENABLE_NOTIFICATIONS=false`، وسجّلت Lambda القيمة `notificationStatus=SKIPPED`. لذلك لم يُختبر تسليم رسائل البريد الإلكتروني تلقائيًا عبر الاستجابة للحوادث.

**الاستنتاج:** **نجحت** معالجة نتيجة الكشف الاصطناعية عبر المسار GuardDuty ← EventBridge ← Lambda. أما تكامل إشعارات SNS فلم يُتحقق منه. ولا يجوز اعتبار النتيجة حادثًا حقيقيًا لمجرد أن مخرجات Lambda أظهرت `sample: false`؛ فقد أُنشئت النتيجة عبر واجهة توليد العينات الاصطناعية.

## 7. الفحوص المتبقية وحدود التحقق

<table dir="rtl">
<thead><tr><th align="right"><strong>البند</strong></th><th align="right"><strong>الحالة</strong></th><th align="right"><strong>التوضيح</strong></th></tr></thead>
<tbody>
<tr><td align="right">التسليم من إنذار CloudWatch إلى SNS</td><td align="right"><strong>لم يُختبر</strong> — NOT TESTED</td><td align="right">تم التحقق من إعداد الإنذار، لكن لم يُحفَّز انتقاله إلى حالة الإنذار عمدًا.</td></tr>
<tr><td align="right">معالجة GuardDuty ← EventBridge ← Lambda</td><td align="right"><strong>ناجح</strong> — PASS</td><td align="right">طابقت النتيجة الاصطناعية بدرجة خطورة 8 القاعدة؛ واستُدعيت Lambda مرة واحدة وسجّلت <code>ESCALATE</code> دون أخطاء في 22 نوفمبر. وبقيت إشعارات SNS معطّلة.</td></tr>
<tr><td align="right">مسار API الخاص بالمعاملات</td><td align="right"><strong>لم يُنفذ</strong> — NOT IMPLEMENTED</td><td align="right">لا تتيح API Gateway سوى <code>GET /health</code>؛ واختُبرت معاملة DynamoDB باستدعاء Lambda مباشرة.</td></tr>
<tr><td align="right">إعادة النشر الكاملة باستخدام IaC</td><td align="right"><strong>لم تُختبر</strong> — NOT TESTED</td><td align="right">اجتازت قوالب مختارة ومعدّلة اختبار CloudFormation للتحقق من صحة القالب، ولا يضمن ذلك نجاح نشر جميع المكونات.</td></tr>
<tr><td align="right">الحظر باستخدام AWS WAF</td><td align="right"><strong>لم يُنفذ</strong> — NOT IMPLEMENTED</td><td align="right">كانت مجموعات القواعد المُدارة في وضع <code>Count</code>، ولم ينجح نشر قاعدة الحظر المخصصة التي جرت محاولة إضافتها.</td></tr>
<tr><td align="right">سياسة AWS Organizations SCP</td><td align="right"><strong>لم تُنشر</strong> — NOT DEPLOYED</td><td align="right">اقتصر الأمر على التحقق من قالب مرجعي.</td></tr>
<tr><td align="right">موارد EBS وRDS في اليوم 18</td><td align="right"><strong>أُزيلت</strong> — REMOVED</td><td align="right">حُذفت موارد العروض الأمنية المؤقتة بعد التحقق.</td></tr>
<tr><td align="right">CloudTrail القديم في <code>us-east-2</code></td><td align="right"><strong>خارج نطاق المشروع</strong> — OUTSIDE PROJECT SCOPE</td><td align="right">يعود إلى مختبر سابق؛ التسجيل معطّل ووجهة S3 مفقودة، ولم تُجرَ تغييرات عليه.</td></tr>
<tr><td align="right">وثائق GitHub العامة</td><td align="right"><strong>متاحة؛ ويُنصح بمراجعة المحتوى النهائي</strong></td><td align="right">تأكد الوصول العام إلى GitHub ونُشرت ملفات التصميم؛ وتبقى مراجعة التنسيق والدقة والمعلومات الحساسة ممارسة ضرورية.</td></tr>
</tbody>
</table>

## 8. أدلة التحقق ومبادئ الاختبار

تشمل الأدلة نتائج أوامر AWS CLI، وسجلات تنفيذ Lambda واستجاباتها، وتأكيد تسليم رسالة SNS، وقوالب CloudFormation في المستودع، والمخططات المعمارية. ولا تُنشر مخرجات الطرفية الخام تلقائيًا في هذا المستودع؛ ويجب عدم إدراج معرّفات حساسة أو بيانات اعتماد ضمن الأدلة العامة.

استخدمت الاختبارات بيانات اصطناعية. وكانت معظم الفحوص الأمنية للقراءة فقط، كما حُذفت الموارد المؤقتة القابلة للفوترة بعد استكمال تجاربها. وهذا المشروع **مختبر تعليمي**، وليس منصة مالية إنتاجية أو بيئة خضعت لتدقيق أمني رسمي.

## 9. الجاهزية للتسليم

**التقييم العام:** تم التحقق من ضوابط الأيام 21–23، وتكامل معالجة الحوادث في اليوم 24 عبر GuardDuty ← EventBridge ← Lambda، وكذلك فحوص CIS وVPC Flow Logs في القسم العاشر، كلٌّ ضمن نطاقه الموثق. ويحتوي المستودع العام على وثائق المشروع. وينبغي عند التسليم الإفصاح عن القيود المتبقية، ومنها انتظار إعادة التقييم في Security Hub، ووجود سجلات `NODATA` فقط ضمن VPC Flow Logs، وعدم اختبار أتمتة الإشعارات.

يُبقي التقرير مسارات التكامل الناقصة أو غير المختبرة ظاهرة بوضوح، بدلًا من تصنيف جميع الفحوص بأنها ناجحة.

## 10. التحقق النهائي من تقييم الامتثال وسجلات حركة الشبكة

**التاريخ:** 22 نوفمبر 2026  
**المنطقة:** `us-east-1`

<table dir="rtl">
<thead><tr><th align="right"><strong>الاختبار</strong></th><th align="right"><strong>الدليل</strong></th><th align="right"><strong>النتيجة</strong></th></tr></thead>
<tbody>
<tr><td align="right">منع الوصول العام إلى S3</td><td align="right">جميع إعدادات الحماية الأربع مفعّلة في الحاويات الثماني وعلى مستوى الحساب.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">CloudFront بعد تعديل S3</td><td align="right">استجابة HTTPS بالرمز HTTP 200 OK.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">التشفير الافتراضي لـEBS</td><td align="right">مفعّل في <code>us-east-1</code> باستخدام <code>alias/aws/ebs</code>.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">CIS AWS Foundations v5.0</td><td align="right">تفعيل 40 عنصر تحكم؛ النتيجة 38% (نجح 15 وفشل 11).</td><td align="right"><strong>تشغيلي</strong> — OPERATIONAL</td></tr>
<tr><td align="right">FinServ VPC Flow Logs</td><td align="right"><code>fl-063e45cabc8c366bb</code> بحالة <code>ACTIVE</code>؛ والتسليم <code>SUCCESS</code>.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">الاحتفاظ بسجلات CloudWatch</td><td align="right">ضبط مجموعة السجلات <code>/aws/vpc/finserv-flow-logs</code> للاحتفاظ لمدة 7 أيام.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">وصول سجلات التدفق</td><td align="right">مسارا سجلات لواجهتَي شبكة Lambda؛ تم استرجاع خمس سجلات <code>NODATA</code> من كل منهما.</td><td align="right"><strong>PASS</strong></td></tr>
<tr><td align="right">التقاط حركة المرور الفعلية</td><td align="right">لم يتم إثبات وجود سجلات <code>ACCEPT</code> أو <code>REJECT</code>.</td><td align="right"><strong>لم يُتحقق منه</strong> — NOT VERIFIED</td></tr>
<tr><td align="right">إعادة التقييم في Security Hub</td><td align="right">ما زالت عناصر <code>S3.1</code> و<code>EC2.7</code> و<code>FinServ EC2.6</code> تعرض طوابع زمنية لحالة <code>FAILED</code> سابقة للمعالجة.</td><td align="right"><strong>معلّق</strong> — PENDING</td></tr>
<tr><td align="right">VPC Flow Logs للشبكة الافتراضية الافتراضية</td><td align="right">لم تُهيَّأ.</td><td align="right"><strong>مفتوح</strong> — OPEN</td></tr>
</tbody>
</table>

**واجهتا الشبكة اللتان تم التحقق منهما:** `eni-09ead21e43fe21d0a` (`10.20.11.46`) و`eni-0fa20d4944a6137af` (`10.20.12.48`).

**حدود التحقق:** تثبت سجلات `NODATA` وصول بيانات إلى CloudWatch، لكنها لا تثبت تسجيل حركة مرور شبكية فعلية. كما بقيت معالجة IAM على مستوى الحساب، والمصادقة متعددة العوامل باستخدام جهاز مادي لحساب الجذر (root)، وبيانات التواصل الأمنية، وتدقيق أحداث بيانات كائنات S3، وسجلات Flow Logs للشبكة الافتراضية الافتراضية خارج نطاق المعالجات التي جرى التحقق منها. **ولا يُدّعى تحقيق امتثال شامل لمعيار CIS.**

---

**مراجع ذات صلة:** [ملخص المشروع بالعربية](../README.ar.md) · [مصفوفة تحقيق المتطلبات بالعربية](../docs/requirements-compliance.ar.md) · [دليل النشر والتحقق بالعربية](../docs/deployment-guide.ar.md) · [الضوابط الأمنية المتقدمة بالعربية](../docs/advanced-security-controls.ar.md)
