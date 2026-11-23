# FinServ — أمثلة سياسات IAM الخاصة بسجلات VPC Flow Logs

**المشروع:** منارة — مشروع FinServ للبنية السحابية الآمنة متعددة الطبقات

**اللغة:** [العربية](README.ar.md) | [English](README.md)

## الغرض من هذا المجلد

يحتوي المجلد [`flowlogs/`](flowlogs/) على مثالين لسياسات **إدارة الهوية والوصول (IAM)** اللازمة لإرسال سجلات حركة مرور الشبكة الافتراضية **VPC Flow Logs** الخاصة بمشروع FinServ إلى **Amazon CloudWatch Logs**:

<ul dir="rtl">
<li dir="rtl" align="right"><a href="flowlogs/finserv-flowlogs-permissions.json"><code>finserv-flowlogs-permissions.json</code></a>: مثال لسياسة الصلاحيات التي يحتاج إليها الدور المستخدَم لنشر السجلات.</li>
<li dir="rtl" align="right"><a href="flowlogs/finserv-flowlogs-trust.json"><code>finserv-flowlogs-trust.json</code></a>: مثال لسياسة الثقة التي تحدد الخدمة المسموح لها بافتراض دور IAM.</li>
</ul>

## معرّفات توضيحية يجب استبدالها

**معرّف حساب AWS‏ `123456789012` الوارد في الأمثلة قيمة افتراضية توضيحية، وليس معرّف الحساب الفعلي الذي نُفّذ فيه المشروع.**

قبل استخدام هذه السياسات في بيئة AWS أخرى، ينبغي مراجعة القيم واستبدال ما يلزم منها وفق البيئة المستهدفة، ولا سيما:

<ul dir="rtl">
<li dir="rtl" align="right"><strong>معرّف حساب AWS:</strong> استخدم معرّف الحساب الصحيح بعد التحقق من هويتك وصلاحياتك.</li>
<li dir="rtl" align="right"><strong>منطقة AWS ‏(Region):</strong> تأكد من مطابقتها للمنطقة التي ستُنشأ فيها موارد تسجيل حركة المرور.</li>
<li dir="rtl" align="right"><strong>مجموعة سجلات CloudWatch Logs:</strong> تأكد من اسم المجموعة ومسارها وصلاحيات الكتابة إليها.</li>
</ul>

## نطاق هذه الملفات

هذه الملفات بصيغة **JSON** توثّق إعدادات السياسات المستخدمة في مختبر FinServ، **وليست قوالب CloudFormation مكتملة يمكن نشرها وحدها لإنشاء البنية التحتية**.

وجود السياسات في المستودع لا يعني أنها مناسبة تلقائيًا لأي حساب أو بيئة أخرى؛ إذ يجب التحقق من الموارد المرجعية وحدود الصلاحيات وعلاقات الثقة قبل استخدامها.

> **حدود التحقق في المشروع:** أثبتت الاختبارات استقبال CloudWatch Logs لسجلات من نوع `NODATA` عبر VPC Flow Logs، لكنها لم تثبت التقاط حركة مرور من نوع `ACCEPT` أو `REJECT`. للمزيد، راجع [تقرير الاختبارات الأمنية](../tests/security-test-results.ar.md).

## المراجع

<ul dir="rtl">
<li dir="rtl" align="right"><a href="../README.ar.md">التوثيق الرئيسي للمشروع باللغة العربية</a></li>
<li dir="rtl" align="right"><a href="../architecture/architecture.ar.md">التصميم المعماري باللغة العربية</a></li>
<li dir="rtl" align="right"><a href="../docs/deployment-guide.ar.md">دليل النشر والتحقق باللغة العربية</a></li>
</ul>
