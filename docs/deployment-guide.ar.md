# FinServ — دليل نشر البنية السحابية باستخدام CloudFormation والتحقق من تنفيذها

**المشروع:** مشروع منارة الثامن — بنية سحابية آمنة متعددة الطبقات باستخدام GuardDuty وKMS وSecurity Hub

**منطقة AWS:** `us-east-1`

**نطاق الوثيقة:** مرجع تعليمي لإجراءات النشر والمراجعة والتحقق. **هذه ليست خطوات مثبتة لإعادة نشر المشروع كاملًا في حساب AWS جديد باستخدام أمر واحد.**

**تاريخ آخر مراجعة للوثيقة الأصلية:** 22 نوفمبر 2026

**اللغة:** [العربية](deployment-guide.ar.md) | [English](deployment-guide.md)

## 1. ملاحظات مهمة بشأن النطاق والسلامة

يحتفظ هذا المستودع بملفات CloudFormation المستخدمة خلال مختبر أمني امتد على عدة أيام. بعض الملفات تراكمي، وبعضها يضبط موارد أنشأتها مجموعات CloudFormation سابقة، وبعضها يمثل إعدادات بديلة أو تجارب مؤقتة. **لا تنشر القوالب كلها معًا، ولا تعِد نشرها في الحساب الأصلي قبل مراجعة مجموعات CloudFormation الموجودة فيه.** فقد تتعارض أسماء الموارد، وقواعد مجموعات الأمان، والإعدادات على مستوى الحساب، ومسجّلات AWS Config، وسياسات AWS Organizations مع موارد موجودة مسبقًا.

يعتمد التطبيق الذي جرى تشغيله والتحقق منه على **CloudFront وموقع ويب محفوظ في حاوية S3 خاصة، وواجهة API Gateway التي توفر المسار `GET /health`، ودالة Lambda مرتبطة بشبكة VPC، وDynamoDB عبر نقطة نهاية من نوع Gateway**. وتحتوي VPC على شبكات فرعية عامة، وأخرى خاصة بالتطبيق، وثالثة معزولة لقواعد البيانات؛ لكن **لا توجد قاعدة بيانات RDS دائمة، ولا مجموعة خوادم EC2 دائمة للويب أو التطبيق، ولا بوابة NAT Gateway**. وتُستخدم مجموعات أمان الويب والتطبيق وقاعدة البيانات أيضًا لتوضيح كيفية حماية أعباء العمل التقليدية ثلاثية الطبقات في المستقبل.

نُفّذت المعالجة التصحيحية في AWS Config **يدويًا**، وعملت القواعد المُدارة في WAF بوضع **Count**، أما سياسة التحكم في الخدمات (SCP) ضمن AWS Organizations فهي **مرجع فقط ولم تُنشر**. ولا تعني نجاحات إنشاء مجموعات منفردة أن المستودع أُعيد نشره كاملًا من الصفر. راجع [مصفوفة تحقيق المتطلبات](requirements-compliance.ar.md) و[نتائج الاختبارات الأمنية الموثقة](../tests/security-test-results.md).

## 2. المتطلبات المسبقة

- حساب AWS توجد موافقة صريحة على إنشاء الموارد فيه وتحمل تكاليفها. ويُفضّل استخدام حساب مستقل للمختبر.
- الإصدار الثاني من AWS CLI وPowerShell على Windows، أو بيئة أوامر أخرى بعد تكييف الأوامر معها.
- بيانات اعتماد AWS مهيأة بصلاحيات مناسبة للقالب المختار. وقد تتطلب بعض قوالب IAM الخيار `CAPABILITY_NAMED_IAM`؛ أما سياسات Organizations فتستلزم سياق إدارة مؤهلًا داخل AWS Organizations.
- صلاحية إنشاء الموارد اللازمة في VPC وS3 وKMS وCloudFormation وLambda وAPI Gateway وDynamoDB وGuardDuty وSecurity Hub وAWS Config وCloudTrail وCloudWatch وSNS وSystems Manager، حسب الحاجة.
- ميزانية محددة وتنبيهات للفوترة؛ إذ قد تترتب رسوم على GuardDuty وSecurity Hub وAWS Config وCloudTrail وCloudWatch وKMS وCloudFront وWAF وموارد RDS التجريبية.
- نسخة محلية من المستودع، وقائمة واضحة بمجموعات CloudFormation والموارد **الموجودة مسبقًا** في الحساب المستهدف.

### فحوص أولية لا تغيّر موارد AWS

نفّذ الأوامر التالية من المجلد الرئيسي للمستودع؛ فهي أوامر **قراءة فقط** لا تُنشئ موارد AWS ولا تحذفها:

```powershell
aws sts get-caller-identity --output json --no-cli-pager
aws configure get region
aws cloudformation list-stacks --region us-east-1 --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE UPDATE_ROLLBACK_COMPLETE --query "StackSummaries[].{Name:StackName,Status:StackStatus}" --output table --no-cli-pager
```

تأكد من الحساب والمنطقة قبل أي نشر. ولا تُدرج بيانات اعتماد فعلية أو أسرارًا خاصة أو معرّفات حسابات غير منقحة داخل المستودع العام.

## 3. ترتيب الاعتماديات المقترح — مرجع معماري

الترتيب الآتي مستنتج من وظائف القوالب والاعتماديات المعروفة بينها. **ولا يعني أنه جرى اختبار نشرها بنجاح في حساب جديد من الصفر.** كذلك يجب عدم نشر بعض القوالب عندما تكون مواردها موجودة بالفعل.

<table dir="rtl">
<thead>
<tr>
<th align="right" dir="rtl"><strong>المرحلة</strong></th>
<th align="right" dir="rtl"><strong>الملفات / مجموعات CloudFormation</strong></th>
<th align="right" dir="rtl"><strong>الاعتماديات والتنبيهات</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="right" dir="rtl">1 — الشبكة VPC</td>
<td align="right" dir="rtl"><code>cloudformation/Recovered-Templates/manara-p8-network.yaml</code></td>
<td align="right" dir="rtl">ينشئ VPC وشبكتين فرعيتين عامتين، وشبكتين خاصتين بالتطبيق، وشبكتين معزولتين لقواعد البيانات. احتفظ بمعرّف VPC ومعرّفات الشبكات الفرعية وجدول التوجيه للمراحل التالية.</td>
</tr>
<tr>
<td align="right" dir="rtl">2 — سياسات الشبكة</td>
<td align="right" dir="rtl"><code>cloudformation/finserv-security-groups.yaml</code></td>
<td align="right" dir="rtl">يحتاج إلى <code>VpcId</code> و<code>DynamoDBPrefixListId</code> الخاص بالمنطقة. يتضمن قاعدة HTTPS/DynamoDB للخروج من طبقة التطبيق.</td>
</tr>
<tr>
<td align="right" dir="rtl">3 — التشفير</td>
<td align="right" dir="rtl"><code>cloudformation/Recovered-Templates/finserv-encryption.yaml</code></td>
<td align="right" dir="rtl">ينشئ مفتاح KMS يديره العميل وحاوية S3 مشفرة بـSSE-KMS. انتبه لتعارض الأسماء والتكاليف.</td>
</tr>
<tr>
<td align="right" dir="rtl">4 — طبقة البيانات</td>
<td align="right" dir="rtl"><code>cloudformation/Day 14/finserv-data-tier.yaml</code></td>
<td align="right" dir="rtl">يحتاج إلى <code>VpcId</code> و<code>AppRouteTableId</code>، وينشئ جدول DynamoDB باسم <code>finserv-transactions</code> ونقطة نهاية Gateway.</td>
</tr>
<tr>
<td align="right" dir="rtl">5 — التطبيق الخاص</td>
<td align="right" dir="rtl"><code>cloudformation/Day 13/finserv-app-tier.yaml</code></td>
<td align="right" dir="rtl">يحتاج إلى مجموعة أمان التطبيق وشبكتين فرعيتين خاصتين به وجدول DynamoDB موجود. ينشئ دالة Lambda المسماة <code>finserv-private-app</code>.</td>
</tr>
<tr>
<td align="right" dir="rtl">6 — واجهة API</td>
<td align="right" dir="rtl"><code>cloudformation/Day 15/finserv-api-tier.yaml</code></td>
<td align="right" dir="rtl">يحتاج إلى دالة <code>finserv-private-app</code>. يتيح <strong>فقط</strong> المسار <code>GET /health</code>؛ ولا يصح وصف <code>/transaction</code> بأنه مسار متاح عبر API.</td>
</tr>
<tr>
<td align="right" dir="rtl">7 — الويب</td>
<td align="right" dir="rtl"><code>cloudformation/Day 12/finserv-web-tier.yaml</code></td>
<td align="right" dir="rtl">ينشئ مصدر S3 خاصًا، وآلية CloudFront OAC، وتوزيع CloudFront. قد يلزم رفع محتوى الموقع بصورة منفصلة.</td>
</tr>
<tr>
<td align="right" dir="rtl">8 — التدقيق والامتثال والكشف</td>
<td align="right" dir="rtl">ملفات الأيام 06–09 الموضحة أدناه</td>
<td align="right" dir="rtl">قد تتعارض المسجّلات ومسارات التدقيق وكواشف التهديدات مع إعدادات قائمة على مستوى الحساب، كما قد يؤدي تفعيل الخدمات إلى تكاليف. افحص الموارد الحالية أولًا.</td>
</tr>
<tr>
<td align="right" dir="rtl">9 — الاستجابة للحوادث والتنبيهات</td>
<td align="right" dir="rtl">ملفات اليومين 10–11</td>
<td align="right" dir="rtl">يجب أن تتطابق أسماء الموارد ومعرّفات ARN في EventBridge وLambda وSNS وسياساتها. اختبار النشر المباشر إلى SNS لا يثبت اكتمال مسار GuardDuty ← Lambda ← SNS.</td>
</tr>
<tr>
<td align="right" dir="rtl">10 — معالجة تصحيحية محدودة النطاق</td>
<td align="right" dir="rtl">ملفات اليوم 16</td>
<td align="right" dir="rtl">تحتاج إلى حاوية اختبار مؤقتة معتمدة، ومستند SSM صالح، ودور IAM محدود الصلاحيات. <strong>المعالجة التلقائية معطلة.</strong></td>
</tr>
<tr>
<td align="right" dir="rtl">11 — WAF والمراقبة</td>
<td align="right" dir="rtl">ملفات اليومين 17 و19</td>
<td align="right" dir="rtl">يتطلب ربط WAF بـCloudFront مراعاة النطاق العام المرتبط بـ<code>us-east-1</code>. تحتاج الإنذارات إلى ARN صحيح لموضوع SNS وأسماء Lambda موجودة. وضع Count في WAF لا يحظر الطلبات.</td>
</tr>
</tbody>
</table>

**احتمال تكرار القواعد:** يحتوي `finserv-security-groups.yaml` بالفعل على قاعدة خروج من التطبيق إلى DynamoDB، كما يُعرّف `Day 14/finserv-dynamodb-egress.yaml` القاعدة نفسها. لا تنشر كليهما قبل فحص صلاحيات الخروج الموجودة في مجموعة الأمان؛ فقد يحدث تعارض.

## 4. جرد قوالب المستودع — 28 ملف YAML

<table dir="rtl">
<thead>
<tr>
<th align="right" dir="rtl"><strong>المجموعة</strong></th>
<th align="right" dir="rtl"><strong>القالب</strong></th>
<th align="right" dir="rtl"><strong>الوظيفة / التصنيف</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="right" dir="rtl">التأسيس</td>
<td align="right" dir="rtl"><code>cloudformation/Recovered-Templates/manara-p8-network.yaml</code></td>
<td align="right" dir="rtl">تأسيس VPC والشبكات الفرعية</td>
</tr>
<tr>
<td align="right" dir="rtl">التأسيس</td>
<td align="right" dir="rtl"><code>cloudformation/finserv-security-groups.yaml</code></td>
<td align="right" dir="rtl">مجموعات أمان الطبقات وسياسات الخروج</td>
</tr>
<tr>
<td align="right" dir="rtl">التأسيس</td>
<td align="right" dir="rtl"><code>cloudformation/Recovered-Templates/finserv-encryption.yaml</code></td>
<td align="right" dir="rtl">مفتاح KMS يديره العميل وحاوية S3 خاصة بتشفير SSE-KMS</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 04</td>
<td align="right" dir="rtl"><code>cloudformation/Day 04/finserv-iam-role.yaml</code></td>
<td align="right" dir="rtl">تمرين ضوابط الوصول في IAM؛ راجع المراجع الداخلية قبل النشر</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 05</td>
<td align="right" dir="rtl"><code>cloudformation/Day 05/finserv-secrets-manager.yaml</code></td>
<td align="right" dir="rtl">تمرين Secrets Manager؛ لا يثبت التدوير التلقائي لبيانات اعتماد RDS بواسطة Lambda</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 06</td>
<td align="right" dir="rtl"><code>cloudformation/Day 06/finserv-cloudtrail.yaml</code></td>
<td align="right" dir="rtl">التدقيق باستخدام CloudTrail</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 07</td>
<td align="right" dir="rtl"><code>cloudformation/Day 07/finserv-aws-config.yaml</code></td>
<td align="right" dir="rtl">حاوية تخزين AWS Config وسياستها؛ لا يحتوي هذا الملف وحده على مسجّل مكتمل</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 07</td>
<td align="right" dir="rtl"><code>cloudformation/Day 07/finserv-config-rules.yaml</code></td>
<td align="right" dir="rtl">قواعد امتثال تحتاج إلى تهيئة خدمة AWS Config مسبقًا</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 08</td>
<td align="right" dir="rtl"><code>cloudformation/Day 08/finserv-guardduty.yaml</code></td>
<td align="right" dir="rtl">إعداد GuardDuty</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 09</td>
<td align="right" dir="rtl"><code>cloudformation/Day 09/finserv-securityhub.yaml</code></td>
<td align="right" dir="rtl">قالب لتجميع نتائج Security Hub؛ فُعّل معيار CIS AWS Foundations Benchmark v5.0 لاحقًا وتحقق تقييمه مباشرة (40 عنصر تحكم ونتائج PASS/FAIL)، وليس بواسطة هذا القالب</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 10</td>
<td align="right" dir="rtl"><code>cloudformation/Day 10/finserv-incident-response.yaml</code></td>
<td align="right" dir="rtl">التصميم الأولي للاستجابة للحوادث</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 11</td>
<td align="right" dir="rtl"><code>cloudformation/Day 11/finserv-incident-response-day11.yaml</code></td>
<td align="right" dir="rtl">نسخة بديلة من الاستجابة للحوادث؛ راجعها قبل إعادة استخدامها</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 11</td>
<td align="right" dir="rtl"><code>cloudformation/Day 11/finserv-sns-alerts.yaml</code></td>
<td align="right" dir="rtl">موضوع إشعارات SNS</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 11</td>
<td align="right" dir="rtl"><code>cloudformation/Day 11/finserv-sns-test.yaml</code></td>
<td align="right" dir="rtl">تجربة اختبار SNS؛ راجعها قبل إعادة الاستخدام</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 12</td>
<td align="right" dir="rtl"><code>cloudformation/Day 12/finserv-web-tier.yaml</code></td>
<td align="right" dir="rtl">طبقة ويب تستخدم S3 خاصًا وCloudFront</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 13</td>
<td align="right" dir="rtl"><code>cloudformation/Day 13/finserv-app-tier.yaml</code></td>
<td align="right" dir="rtl">Lambda مرتبطة بـVPC مع صلاحية الوصول إلى DynamoDB</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 14</td>
<td align="right" dir="rtl"><code>cloudformation/Day 14/finserv-data-tier.yaml</code></td>
<td align="right" dir="rtl">جدول DynamoDB ونقطة نهاية Gateway</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 14</td>
<td align="right" dir="rtl"><code>cloudformation/Day 14/finserv-dynamodb-egress.yaml</code></td>
<td align="right" dir="rtl">قاعدة خروج إضافية؛ يحتمل تكرارها</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 15</td>
<td align="right" dir="rtl"><code>cloudformation/Day 15/finserv-api-tier.yaml</code></td>
<td align="right" dir="rtl">مسار <code>GET /health</code> في API Gateway</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 16</td>
<td align="right" dir="rtl"><code>cloudformation/Day 16/finserv-day16-safe-remediation.yaml</code></td>
<td align="right" dir="rtl">مستند SSM Automation محدود النطاق</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 16</td>
<td align="right" dir="rtl"><code>cloudformation/Day 16/finserv-day16-remediation-lab.yaml</code></td>
<td align="right" dir="rtl">موارد مختبر لاختبار المعالجة التصحيحية بصورة مضبوطة</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 16</td>
<td align="right" dir="rtl"><code>cloudformation/Day 16/finserv-day16-config-remediation.yaml</code></td>
<td align="right" dir="rtl">قاعدة AWS Config وإعدادات المعالجة؛ <code>Automatic: false</code></td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 17</td>
<td align="right" dir="rtl"><code>cloudformation/Day 17/finserv-waf.yaml</code></td>
<td align="right" dir="rtl">مرجع ضوابط WAF للحافة والقواعد المُدارة؛ تحقق من سلوك النسخة المنشورة</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 18</td>
<td align="right" dir="rtl"><code>cloudformation/Day 18/finserv-day18-ebs.yaml</code></td>
<td align="right" dir="rtl">تجربة EBS مشفرة ومؤقتة، حُذفت بعد الاختبار</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 18</td>
<td align="right" dir="rtl"><code>cloudformation/Day 18/finserv-day18-rds-subnet.yaml</code></td>
<td align="right" dir="rtl">تجربة مؤقتة لمجموعة شبكات فرعية خاصة بقاعدة البيانات</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 18</td>
<td align="right" dir="rtl"><code>cloudformation/Day 18/finserv-day18-rds.yaml</code></td>
<td align="right" dir="rtl">تجربة PostgreSQL على RDS بصورة مؤقتة؛ حُذفت بعد الاختبار</td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 18</td>
<td align="right" dir="rtl"><code>cloudformation/Day 18/finserv-day18-scp.yaml</code></td>
<td align="right" dir="rtl">مرجع لسياسة SCP ضمن Organizations؛ <strong>لم يُنشر</strong></td>
</tr>
<tr>
<td align="right" dir="rtl">اليوم 19</td>
<td align="right" dir="rtl"><code>cloudformation/Day 19/finserv-day19-monitoring.yaml</code></td>
<td align="right" dir="rtl">ثلاثة إنذارات CloudWatch؛ تتطلب ARN لموضوع SNS</td>
</tr>
</tbody>
</table>

## 5. المعاملات ومخرجات المجموعات المطلوب الاستعلام عنها

لا تنسخ معرّفات تخص حساب AWS آخر. استعلم بدلًا من ذلك عن موارد الحساب الحالي ومخرجات مجموعات CloudFormation الموجودة فيه.

<table dir="rtl">
<thead>
<tr>
<th align="right" dir="rtl"><strong>المعامل</strong></th>
<th align="right" dir="rtl"><strong>المصدر أو القيمة المتوقعة</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="right" dir="rtl"><code>VpcId</code></td>
<td align="right" dir="rtl">المخرج <code>VpcId</code> من مجموعة الشبكة</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>PrivateSubnet1Id</code> / <code>PrivateSubnet2Id</code></td>
<td align="right" dir="rtl">معرّفان من المخرج <code>PrivateAppSubnetIds</code> الخاص بالشبكة</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>AppRouteTableId</code></td>
<td align="right" dir="rtl">معرّف جدول التوجيه الخاص بالتطبيق، من موارد EC2/VPC أو موارد مجموعة الشبكة؛ <strong>ليس مخرجًا في قالب الشبكة المنشور</strong></td>
</tr>
<tr>
<td align="right" dir="rtl"><code>AppSecurityGroupId</code></td>
<td align="right" dir="rtl">المخرج <code>AppSecurityGroupId</code> من مجموعة الأمان</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>DynamoDBPrefixListId</code></td>
<td align="right" dir="rtl">معرّف قائمة البادئات التي تديرها AWS لخدمة DynamoDB في <code>us-east-1</code>؛ استعلم عنه في الحساب المستهدف</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>DynamoDBTableName</code></td>
<td align="right" dir="rtl"><code>finserv-transactions</code>، ويُنشئه قالب طبقة البيانات</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>SNSTopicArn</code></td>
<td align="right" dir="rtl">المخرج <code>SecurityAlertsTopicArn</code> من مجموعة تنبيهات SNS</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>TestBucketName</code></td>
<td align="right" dir="rtl">اسم حاوية S3 مؤقتة ومُعتمدة للاختبار <strong>فقط</strong>؛ لا تستخدم حاوية إنتاجية</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>RemediationRoleArn</code></td>
<td align="right" dir="rtl">ARN لدور IAM محدود الصلاحيات خاص بتمرين SSM؛ يجب أن يكون موجودًا قبل إعداد المعالجة</td>
</tr>
<tr>
<td align="right" dir="rtl"><code>RemediationDocumentName</code></td>
<td align="right" dir="rtl">مستند SSM Automation المسمى <code>FinServ-SafeS3EncryptionRemediation</code></td>
</tr>
</tbody>
</table>

الأوامر الآتية أمثلة **للاستعلام فقط، دون تعديل الموارد**:

```powershell
aws cloudformation describe-stacks --stack-name manara-p8-network --region us-east-1 --query "Stacks[0].Outputs" --output table --no-cli-pager
aws cloudformation describe-stacks --stack-name finserv-security-groups --region us-east-1 --query "Stacks[0].Outputs" --output table --no-cli-pager
aws ec2 describe-managed-prefix-lists --region us-east-1 --filters Name=prefix-list-name,Values=com.amazonaws.us-east-1.dynamodb --query "PrefixLists[].{Id:PrefixListId,Name:PrefixListName}" --output table --no-cli-pager
```

اسم المجموعة هنا مثال من المختبر الموثق؛ استخدم `list-stacks` للتحقق من الاسم الفعلي في حسابك أولًا.

## 6. فحص القوالب والتحقق منها بأمان

للتحقق من صحة بنية قوالب CloudFormation دون إنشاء موارد، افحص كل ملف على حدة. مثال:

```powershell
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Recovered-Templates/manara-p8-network.yaml" --no-cli-pager
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Day 13/finserv-app-tier.yaml" --no-cli-pager
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Day 14/finserv-data-tier.yaml" --no-cli-pager
```

**لا يختبر** الأمر `validate-template` صلاحيات IAM، أو وجود الموارد المشار إليها، أو سلوك التطبيق أثناء التشغيل، أو التكلفة، أو سلامة الاعتماديات بين المجموعات. راجع أقسام `Parameters` و`Resources` و`Outputs` وأسماء الموارد الثابتة داخل كل قالب قبل نشره.

عند التخطيط لإعادة بناء البنية مستقبلًا، حدّد معاملات كل مجموعة وأنشئ **مجموعة تغييرات (CloudFormation Change Set)**، ثم راجع الإضافات والاستبدالات والحذف المقترحة. احصل على الموافقة قبل تنفيذها، خصوصًا في موارد IAM والتشفير وAWS Config والموارد التي تحتفظ ببيانات.

## 7. التحقق من التنفيذ وتوثيق الأدلة

**تحديث التحقق بتاريخ 22 نوفمبر 2026:** نجحت نتيجة GuardDuty اصطناعية بدرجة خطورة 8 في تشغيل EventBridge وLambda (استدعاء واحد، صفر أخطاء، وتسجيل `ESCALATE`)، بينما ظل إشعار SNS التلقائي دون اختبار. وفُعّل معيار CIS AWS Foundations Benchmark v5.0 مع 40 عنصر تحكم ونتائج فعلية من نوع PASS/FAIL. وتم التحقق عمليًا من خاصية التحقق من سلامة ملفات CloudTrail. كما فُعّل منع الوصول العام إلى S3 على مستوى الحساب والتشفير الافتراضي لـEBS وتحقق تفعيلهُما. أما FinServ VPC Flow Logs فكانت حالتها `ACTIVE` ونجاح التسليم `SUCCESS`؛ واحتوى تدفقان في CloudWatch مرتبطان بواجهات الشبكة ENI على سجلات `NODATA`. **لم يُثبت التقاط سجلات حركة مرور فعلية من نوع ACCEPT/REJECT، ولا تحوّل نتائج Security Hub بعد المعالجة إلى PASSED.**

تُسجّل الأدلة الأقدم من الأيام 21–23 في [تقرير نتائج الاختبارات الأمنية](../tests/security-test-results.md)، ومن أهمها:

- أعاد مسار API Gateway `GET /health` استجابة HTTP `200`، بينما لم يُضبط المسار `/transaction` بوصفه مسارًا في API.
- نجحت معاملة اصطناعية عبر استدعاء Lambda المباشر في الكتابة إلى DynamoDB والقراءة منه ضمن تصميم التطبيق الخاص.
- فُعّل GuardDuty وSecurity Hub، وجرى فحص نتيجة GuardDuty **اصطناعية** لا تدل على اختراق حقيقي.
- أعادت قواعد AWS Config نتائج متوافقة، ونُفذ دليل تشغيل SSM بصورة مضبوطة **يدويًا** وبنجاح.
- أُعدت ثلاثة إنذارات CloudWatch مع إجراءات SNS، ووصل بريد إلكتروني اختباري عبر إرسال **مباشر** إلى SNS. ولم يثبت هذا الاختبار وصول إشعار ناجم عن إطلاق إنذار تلقائيًا.
- شملت الأدلة تجربة S3 مشفرة بمفتاح KMS، وضوابط الوصول إلى خدمات AWS، ومراجعة التكاليف، وحذف موارد التجارب.

أمثلة إضافية لأوامر تحقق **للقراءة فقط**:

```powershell
aws cloudformation describe-stacks --region us-east-1 --stack-name finserv-app-tier --query "Stacks[0].StackStatus" --output text --no-cli-pager
aws lambda get-function-configuration --region us-east-1 --function-name finserv-private-app --query "{State:State,VpcId:VpcConfig.VpcId,Subnets:VpcConfig.SubnetIds}" --output json --no-cli-pager
aws dynamodb describe-table --region us-east-1 --table-name finserv-transactions --query "Table.{Name:TableName,Status:TableStatus}" --output json --no-cli-pager
aws cloudwatch describe-alarms --region us-east-1 --alarm-name-prefix finserv- --query "MetricAlarms[].{Name:AlarmName,State:StateValue,Actions:ActionsEnabled}" --output table --no-cli-pager
```

تفترض الأوامر وجود الموارد المعنية إلى الآن. وإذا لم يُعثر على مورد أو تغيّر اسمه، فهذا لا يعني أن الاختبار الأصلي لم يحدث.

## 8. تنظيف موارد المختبر والتحكم في التكلفة

حُذفت قاعدة بيانات RDS ووحدة تخزين EBS التجريبيتان من اليوم 18 عمدًا بعد التحقق. وقد يستمر المختبر في توليد رسوم استخدام، مثل KMS وCloudTrail وConfig وCloudWatch وSecurity Hub وGuardDuty وWAF. استخدم Cost Explorer ووحدات تحكم الخدمات لمراجعة الرسوم الحالية؛ فالأرقام التاريخية لا تُعد توقعًا للتكاليف القادمة.

قبل حذف **أي** مجموعة CloudFormation، احصر مواردها باستخدام `describe-stack-resources`، وحدّد إن كانت مجموعات أخرى تعتمد عليها، وتأكد من سياسات الاحتفاظ الملائمة لحاويات S3 وكائناتها ومفاتيح KMS والسجلات. فقد يؤدي تفريغ الحاويات أو جدولة حذف مفتاح KMS أو إزالة مسجّل AWS Config أو حذف سجلات CloudTrail إلى فقد الأدلة. **لا يتضمن هذا الدليل عمدًا أمر حذف شاملًا.**

### قائمة التحقق قبل التنظيف

1. تأكد من حساب AWS والمنطقة بدقة، واحصل على إذن بإجراء التنظيف.
2. ميّز بين مجموعات المشروع الحالية ومجموعات التجارب، واحتفظ بنسخة من الأدلة اللازمة للتقييم.
3. راجع الاعتماديات والموارد المحتفظ بها في S3 وKMS، وأنشئ نسخًا احتياطية حيث يلزم.
4. احذف الموارد المؤقتة الموافق على حذفها صراحةً فقط، مع مراعاة الترتيب العكسي للاعتماديات.
5. تحقق من عدم بقاء قواعد RDS أو وحدات EBS أو اللقطات أو الخدمات القابلة للفوترة غير المقصودة.
6. راجع Cost Explorer بعد اكتمال ظهور بيانات الفوترة.

## 9. القيود المعروفة والقرارات الأمنية

- التطبيق المنفذ مختبر صغير قائم على خدمات **دون خوادم دائمة التشغيل (Serverless)**، وليس تطبيقًا ماليًا جاهزًا للإنتاج. طبقة الشبكات الفرعية المعزولة لقواعد البيانات موجودة في التصميم، لكن لا توجد قاعدة RDS مستمرة في التشغيل.
- تعمل القواعد المُدارة في AWS WAF بوضع Count لأغراض المراقبة؛ ولم تُنشر قاعدة الحظر المقترحة.
- بقيت معالجة SSM/AWS Config التصحيحية يدوية عن قصد، وليست تلقائية.
- لم تثبت الأدلة تفعيل سياسة SCP في AWS Organizations أو فرض حدود صلاحيات IAM على أدوار منشورة.
- لا تثبت تمارين Secrets Manager إجراء تدوير تلقائي لبيانات اعتماد RDS باستخدام دالة Lambda مخصصة.
- فُعّل معيار CIS AWS Foundations Benchmark v5.0 مع 40 عنصر تحكم واسترجاع نتائج PASS/FAIL. وبلغت النتيجة 38% (15 ناجحًا و11 فاشلًا) بتاريخ 22 نوفمبر 2026. ولم يُفعّل PCI DSS، ولا يُدّعى تحقيق امتثال كامل.
- لوحظ خطأ في تسليم السجلات لمسار CloudTrail ثانوي في `us-east-2` خلال التدقيق. يجب التحقق من مالكه والغرض منه قبل إجراء تغييرات عليه.
- لم يُختبر المستودع بوصفه وسيلة نشر مؤتمتة كاملة من البداية إلى النهاية في حساب جديد.

**توضيح تكميلي معتمد على وثائق المشروع الأحدث:** نجح اختبار **محاكاة السياسات** لحدود صلاحيات IAM: سمحت المحاكاة بـ`s3:ListBucket` ورفضت `s3:DeleteBucket` صراحةً. **لا يعني ذلك** تثبيت حدود الصلاحيات على دور IAM فعلي أو اختبار فرضها عمليًا. راجع [توثيق المحاكاة](iam-permissions-boundary-test.md). أُضيف هذا التوضيح إلى الترجمة لاستيعاب نتيجة موثقة في المستودع، وليس ادعاءً ورد ضمن النص الأصلي لهذا الدليل.

**للمراجعين:** لمعرفة حالة كل هدف تعليمي، ابدأ بـ[مصفوفة تحقيق المتطلبات](requirements-compliance.ar.md). ولمراجعة مخرجات الأوامر وحدود الاختبارات الفعلية، ارجع إلى [نتائج الاختبارات الأمنية](../tests/security-test-results.md).
