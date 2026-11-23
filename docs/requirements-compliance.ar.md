# مشروع منارة الثامن: مصفوفة التحقق من استيفاء المتطلبات

**المشروع:** FinServ — بنية سحابية آمنة متعددة الطبقات على AWS  
**البرنامج:** مسار منارة التعليمي لهندسة الحلول على AWS  
**منطقة AWS:** `us-east-1`  
**تاريخ المراجعة الأصلية:** 22 نوفمبر 2026  
**أساس الأدلة:** قوالب المشروع الموجودة في المستودع والاختبارات المسجلة للأيام 21–24. وتعكس الحالات المبينة أدناه الأدلة الموثقة، ولا تعني إجراء تحقق مباشر جديد من موارد AWS.

**اللغة:** [العربية](requirements-compliance.ar.md) | [English](requirements-compliance.md)

## فهرس المحتويات

<ol dir="rtl" align="right">
<li dir="rtl" align="right"><a href="#تعريف-حالات-التحقق">تعريف حالات التحقق</a></li>
<li dir="rtl" align="right"><a href="#المخرجات-المطلوبة">المخرجات المطلوبة</a></li>
<li dir="rtl" align="right"><a href="#الأهداف-المعمارية-والأمنية">الأهداف المعمارية والأمنية</a></li>
<li dir="rtl" align="right"><a href="#حدود-التنفيذ-المهمة">حدود التنفيذ المهمة</a></li>
<li dir="rtl" align="right"><a href="#اختبارات-التحقق-المقترحة-لاحقًا">اختبارات التحقق المقترحة لاحقًا</a></li>
<li dir="rtl" align="right"><a href="#مراجع-المستودع">مراجع المستودع</a></li>
</ol>

## تعريف حالات التحقق

<ul dir="rtl" align="right">
<li dir="rtl" align="right"><strong>تم التحقق:</strong> توجد في المشروع أدلة على تنفيذ ناجح واختبار ذي صلة.</li>
<li dir="rtl" align="right"><strong>نُفّذ دون اختبار شامل:</strong> توجد إعدادات أو شيفرة، لكن لم يُثبت عمل المسار كاملًا من بدايته إلى نهايته.</li>
<li dir="rtl" align="right"><strong>جزئي / تجربة عملية:</strong> نُفّذ جانب من المتطلب، أو أُنشئ مورد مؤقت للتجربة ثم حُذف.</li>
<li dir="rtl" align="right"><strong>لم يُثبت بالتجربة:</strong> لا تتوافر أدلة كافية على عمل القدرة المطلوبة كاملة.</li>
</ul>

## المخرجات المطلوبة

<table dir="rtl">
<thead>
<tr><th align="right" dir="rtl"><strong>المتطلب</strong></th><th align="right" dir="rtl"><strong>الدليل</strong></th><th align="right" dir="rtl"><strong>الحالة</strong></th></tr>
</thead>
<tbody>
<tr><td align="right" dir="rtl">المخطط المعماري</td><td align="right" dir="rtl"><a href="../architecture/architecture.md">عرض التصميم المعماري</a>، مع ملف <code>.drawio</code> قابل للتحرير وصورة PNG للمعاينة عند توافرها</td><td align="right" dir="rtl">وجود الملفات بالمستودع مؤكد</td></tr>
<tr><td align="right" dir="rtl">مستودع GitHub عام وملف README</td><td align="right" dir="rtl"><a href="../README.ar.md">الدليل العربي</a> و<a href="../README.md">الدليل الإنجليزي</a> وقوالب CloudFormation وتقرير الاختبارات</td><td align="right" dir="rtl">نُشر المستودع؛ يُنصح بالتحقق من إمكانية فتحه دون تسجيل دخول قبل التسليم</td></tr>
<tr><td align="right" dir="rtl">عرض مباشر أو مقطع فيديو</td><td align="right" dir="rtl">استُخدمت نقاط وصول CloudFront وAPI Gateway أثناء الاختبارات</td><td align="right" dir="rtl">اختياري وفق الوثيقة الأصلية؛ لم يُوثَّق تسجيل فيديو أو تقديم عرض مباشر بوصفه ملفًا للتسليم</td></tr>
</tbody>
</table>

## الأهداف المعمارية والأمنية

<table dir="rtl">
<thead>
<tr><th align="right" dir="rtl"><strong>متطلب منارة</strong></th><th align="right" dir="rtl"><strong>أدلة المشروع وحدود التحقق</strong></th><th align="right" dir="rtl"><strong>الحالة</strong></th></tr>
</thead>
<tbody>
<tr><td align="right" dir="rtl">بنية ثلاثية الطبقات تعتمد الدفاع متعدد الطبقات</td><td align="right" dir="rtl">تصميم شبكات فرعية عامة وخاصة للتطبيق ومعزولة لقواعد البيانات عبر منطقتَي توافر؛ التطبيق العامل فعليًا يستخدم Lambda وDynamoDB، لا قاعدة RDS دائمة</td><td align="right" dir="rtl">تم التحقق من التصميم المعماري؛ وتنفيذ التطبيق دون خوادم دائمة (Serverless)</td></tr>
<tr><td align="right" dir="rtl">تشفير البيانات المخزنة باستخدام KMS</td><td align="right" dir="rtl">تأكد تشفير S3 بنمط SSE-KMS ومفتاح يديره العميل؛ وأُجريت تجارب مؤقتة على تشفير EBS وRDS</td><td align="right" dir="rtl">تم التحقق في S3؛ والتحقق جزئي عبر جميع الخدمات المذكورة</td></tr>
<tr><td align="right" dir="rtl">تشفير البيانات أثناء النقل</td><td align="right" dir="rtl">توثيق مسارات HTTPS عبر API Gateway وCloudFront، والوصول إلى DynamoDB عبر HTTPS</td><td align="right" dir="rtl">تم التحقق ضمن المسارات المختبرة</td></tr>
<tr><td align="right" dir="rtl">سياسات KMS والوصول باستخدام المنح (Grants)</td><td align="right" dir="rtl">فُحص المفتاح الذي يديره العميل واستخدامه؛ ولم يثبت اختبار منفصل لسير عمل يعتمد على منح KMS</td><td align="right" dir="rtl">جزئي</td></tr>
<tr><td align="right" dir="rtl">تخزين SecureString في SSM Parameter Store بمفتاح KMS يديره العميل</td><td align="right" dir="rtl">لم تُثبت تجربة مكتملة للنشر والاختبار لهذا التكوين</td><td align="right" dir="rtl">لم يُثبت بالتجربة</td></tr>
<tr><td align="right" dir="rtl">التخزين المركزي للأسرار باستخدام Secrets Manager</td><td align="right" dir="rtl">وُثقت تجارب تخزين الأسرار وبيانات الاعتماد التي تديرها RDS</td><td align="right" dir="rtl">جزئي / تجربة عملية</td></tr>
<tr><td align="right" dir="rtl">التدوير التلقائي لبيانات اعتماد RDS باستخدام Lambda</td><td align="right" dir="rtl">اكتملت تجربة RDS المؤقتة ثم حُذف موردها؛ ولم يُثبت سير عمل دورة تدوير مجدولة باستخدام Lambda</td><td align="right" dir="rtl">لم يُثبت من البداية إلى النهاية</td></tr>
<tr><td align="right" dir="rtl">كشف التهديدات باستخدام GuardDuty</td><td align="right" dir="rtl">فُعّل الكاشف وفُحصت نتائج اختبار اصطناعية من فئتَي Tor وBackdoor EC2/C&amp;C؛ وشغّلت نتيجة Backdoor بخطورة 8 كلًا من EventBridge وLambda بنجاح. ليست حادثة فعلية</td><td align="right" dir="rtl">تم التحقق من مسار الاختبار الاصطناعي</td></tr>
<tr><td align="right" dir="rtl">فئات نتائج GuardDuty: Recon وBackdoor وCryptoCurrency</td><td align="right" dir="rtl">فُحصت نتائج Tor وBackdoor C&amp;C الاصطناعية؛ ولم تُعرض تجارب لفئتَي Recon وCryptoCurrency</td><td align="right" dir="rtl">جزئي</td></tr>
<tr><td align="right" dir="rtl">تجميع النتائج في Security Hub</td><td align="right" dir="rtl">رُوجعت النتائج إلى جانب ضوابط أمنية أخرى في AWS</td><td align="right" dir="rtl">تم التحقق</td></tr>
<tr><td align="right" dir="rtl">أطر الامتثال CIS وPCI DSS ودرجات التقييم</td><td align="right" dir="rtl">فُعّل CIS AWS Foundations Benchmark v5.0 مع 40 عنصر تحكم مفعّلًا، واستُرجعت نتائج النجاح والإخفاق. بلغت نتيجة CIS نسبة <strong>38%</strong> (15 ناجحًا و11 مخفقًا) بتاريخ <strong>22 نوفمبر 2026</strong>؛ ولم يُفعّل PCI DSS</td><td align="right" dir="rtl">تقييم CIS يعمل؛ لا يُدّعى تحقق الامتثال الكامل</td></tr>
<tr><td align="right" dir="rtl">قواعد الامتثال في AWS Config</td><td align="right" dir="rtl">أعادت ثلاث قواعد حالة <code>COMPLIANT</code></td><td align="right" dir="rtl">تم التحقق</td></tr>
<tr><td align="right" dir="rtl">المعالجة التصحيحية التلقائية بواسطة AWS Config وSSM</td><td align="right" dir="rtl">نُفذ دليل تشغيل SSM Automation يدويًا بنجاح؛ وظلت المعالجة التلقائية معطلة</td><td align="right" dir="rtl">تم التحقق من المعالجة اليدوية؛ ولم تُثبت المعالجة التلقائية</td></tr>
<tr><td align="right" dir="rtl">تدقيق استدعاءات API عبر CloudTrail في مناطق متعددة</td><td align="right" dir="rtl">تأكد تشغيل التسجيل في مناطق متعددة، وتشفير KMS، وتسجيل أحداث الإدارة للقراءة والكتابة، والتحقق من سلامة ملفات السجلات. لم تُفعّل أحداث بيانات كائنات S3</td><td align="right" dir="rtl">تم التحقق من تدقيق أحداث الإدارة</td></tr>
<tr><td align="right" dir="rtl">حماية WAF وShield Standard</td><td align="right" dir="rtl">نُشر CloudFront مع مجموعات قواعد WAF مُدارة في وضع <strong>Count</strong>؛ ولم يُثبت نجاح حظر طلب فعلي</td><td align="right" dir="rtl">جزئي؛ مراقبة دون إثبات الحظر النشط</td></tr>
<tr><td align="right" dir="rtl">مبدأ الحد الأدنى من الامتيازات في IAM</td><td align="right" dir="rtl">رُوجعت صلاحيات Lambda المحددة لـ DynamoDB ومجموعات الأمان المقيّدة</td><td align="right" dir="rtl">تم التحقق من السياسات التي شملتها المراجعة؛ وليس تدقيقًا شاملًا لـ IAM</td></tr>
<tr><td align="right" dir="rtl">حدود الصلاحيات (IAM Permissions Boundaries)</td><td align="right" dir="rtl">أثبتت <a href="iam-permissions-boundary-test.md">محاكاة سياسات IAM</a> السماح بـ <code>ListBucket</code> والرفض الصريح لـ <code>DeleteBucket</code>؛ دون إرفاق حد صلاحيات بدور IAM فعلي</td><td align="right" dir="rtl">تم التحقق بالمحاكاة؛ ولم يُثبت تطبيقها الفعلي</td></tr>
<tr><td align="right" dir="rtl">سياسة SCP لمنع استخدام حساب الجذر</td><td align="right" dir="rtl">مرجع لسياسة SCP مع تحقق من القالب فقط؛ لم يُطبّق على AWS Organizations</td><td align="right" dir="rtl">مرجع فقط</td></tr>
<tr><td align="right" dir="rtl">الاستجابة للحوادث باستخدام EventBridge وLambda</td><td align="right" dir="rtl">في 22 نوفمبر، طابقت نتيجة GuardDuty اصطناعية عالية الخطورة قاعدة EventBridge، وشغّلت Lambda مرة واحدة دون أخطاء، وأنتجت القرار <code>ESCALATE</code>. كانت إشعارات SNS معطلة</td><td align="right" dir="rtl">تم التحقق من معالجة الحادث؛ ولم يُختبر إشعار SNS</td></tr>
<tr><td align="right" dir="rtl">الإنذارات باستخدام CloudWatch وSNS</td><td align="right" dir="rtl">أُعدّت ثلاثة إنذارات، وتأكد وصول رسالة SNS تجريبية مباشرة عبر البريد الإلكتروني</td><td align="right" dir="rtl">تم التحقق من إرسال SNS المباشر؛ ولم يُختبر الإشعار الناتج عن انتقال حالة إنذار</td></tr>
<tr><td align="right" dir="rtl">أدلة الامتثال والحوادث</td><td align="right" dir="rtl"><a href="../tests/security-test-results.md">تقرير الاختبارات الأمنية</a> وقوالب CloudFormation بصيغة YAML</td><td align="right" dir="rtl">تم التحقق من وجود التوثيق</td></tr>
</tbody>
</table>

## حدود التنفيذ المهمة

<ol dir="rtl" align="right">
<li dir="rtl" align="right">يستخدم التطبيق المختبر فعليًا مسار معاملات يعتمد على Lambda وDynamoDB دون خوادم دائمة التشغيل. أما شبكات قواعد البيانات المعزولة وتجربة RDS المؤقتة فتوضح مفهوم طبقة البيانات، <strong>ولا تعني</strong> أن التطبيق الحالي يستعمل RDS.</li>
<li dir="rtl" align="right">وصول رسالة SNS تجريبية مباشرة <strong>لا يثبت</strong> أن إنذار CloudWatch أو حدث GuardDuty/EventBridge أرسل إشعارًا تلقائيًا.</li>
<li dir="rtl" align="right">تسجّل قواعد WAF في وضع <strong>Count</strong> الطلبات المطابقة، لكنها لا تحظرها من تلقاء نفسها. وتختلف الحماية الأساسية التي توفرها Shield Standard عن اختبار موثّق لحظر طلبات طبقة التطبيق.</li>
<li dir="rtl" align="right">نجاح تنفيذ دليل SSM Automation يدويًا <strong>لا يثبت</strong> تفعيل المعالجة التصحيحية التلقائية في AWS Config.</li>
<li dir="rtl" align="right">وجود قالب CloudFormation دليل على توثيق التنفيذ، <strong>وليس برهانًا</strong> على إمكان نشر جميع الموارد دفعة واحدة في بيئة جديدة دون إعدادات إضافية.</li>
<li dir="rtl" align="right">حُذفت موارد RDS وEBS المؤقتة بعد التجارب؛ ولا يصح وصفها بأنها مكونات دائمة في التطبيق العامل.</li>
<li dir="rtl" align="right">لا يجوز اعتبار تفعيل Security Hub وحده شهادة امتثال لمعيار CIS أو PCI DSS.</li>
</ol>

## اختبارات التحقق المقترحة لاحقًا

> **تنبيه:** البنود التالية أعمال لاحقة اختيارية، ولا تُعد نتائج اختبارات مكتملة ما لم يُذكر خلاف ذلك صراحةً.

<ul dir="rtl" align="right">
<li dir="rtl" align="right"><strong>تم التحقق:</strong> تأكد التحقق من سلامة ملفات CloudTrail وتشغيل التسجيل متعدد المناطق باستخدام AWS CLI. وبقيت أحداث بيانات كائنات S3 خارج التكوين المثبت.</li>
<li dir="rtl" align="right"><strong>تم التحقق:</strong> شغّلت نتيجة GuardDuty اصطناعية EventBridge وLambda، وأنتجت القرار <code>ESCALATE</code> (استدعاء واحد دون أخطاء). <strong>معلّق:</strong> التحقق من إرسال إشعار SNS تلقائيًا عبر Lambda، وكذلك إشعارات إنذارات CloudWatch.</li>
<li dir="rtl" align="right"><strong>تم التحقق:</strong> فُعّل معيار CIS AWS Foundations Benchmark v5.0 وقُيّم بتاريخ 22 نوفمبر 2026، بنتيجة <strong>38%</strong> (15 ناجحًا و11 مخفقًا من أصل 40 عنصر تحكم مفعّلًا). <strong>متابعة:</strong> مراجعة الضوابط المخفقة وتأكيد إعادة التقييم بعد المعالجة للضوابط <code>S3.1</code> و<code>EC2.7</code> و<code>FinServ EC2.6</code>. ولم تُعرض العناصر الأربعة عشر الأخرى في الملخص الموثق بوصفها ناجحة أو مخفقة.</li>
<li dir="rtl" align="right">اختبار المعالجة التلقائية عبر AWS Config على مورد اختباري مؤقت وغير إنتاجي فقط، بعد مراجعة الصلاحيات والآثار المحتملة.</li>
<li dir="rtl" align="right">إضافة تجربة إثبات مفهوم لتدوير أسرار Secrets Manager وفق جدول زمني وفي نطاق مستقل، إذا سمح الوقت والتكلفة.</li>
</ul>

## مراجع المستودع

<ul dir="rtl" align="right">
<li dir="rtl" align="right"><a href="../README.ar.md">التوثيق العربي للمشروع</a></li>
<li dir="rtl" align="right"><a href="../README.md">التوثيق الإنجليزي للمشروع</a></li>
<li dir="rtl" align="right"><a href="../tests/security-test-results.md">تقرير الاختبارات والتحقق الأمني</a></li>
<li dir="rtl" align="right"><a href="../architecture/architecture.md">العرض المعماري</a></li>
<li dir="rtl" align="right"><a href="../cloudformation/">قوالب CloudFormation</a></li>
<li dir="rtl" align="right"><a href="cis-remediation-plan.md">خطة معالجة نتائج CIS v5.0</a></li>
<li dir="rtl" align="right"><a href="requirements-compliance.md">النسخة الإنجليزية من هذه المصفوفة</a></li>
</ul>

**التقييم العام:** يضم المشروع أدلة على تنفيذ جانب كبير من البنية المعمارية الأساسية والتشفير والكشف والتدقيق والاختبارات. أما بعض قدرات الأتمتة المتقدمة والحوكمة والتدوير ونتائج تقييم الأطر الأمنية، فما زالت جزئية أو غير مثبتة. **ويعود القرار النهائي بشأن اجتياز المشروع إلى مراجعي منارة.**
