# اختبار حدود الصلاحيات باستخدام محاكي السياسات

**المشروع:** مشروع منارة الثامن (Manara AWS Project 8) — FinServ  
**نوع الاختبار:** محاكي سياسات AWS IAM ‏(AWS IAM Policy Simulator)  
**النتيجة:** **نجاح (PASSED) — محاكاة السياسات فقط**

**اللغة:** [العربية](iam-permissions-boundary-test.ar.md) | [English](iam-permissions-boundary-test.md)

> **تنبيه بشأن نطاق التحقق:** تؤكد نتائج هذه الوثيقة سلوك تقييم السياسات داخل محاكي IAM. **لم تُطبَّق حدود صلاحيات على دور IAM فعلي، ولم يُنفَّذ اختبار حيّ للصلاحيات على موارد AWS حقيقية.**

## 1. هدف الاختبار

التحقق من أن **حدود الصلاحيات (Permissions Boundary)** في AWS IAM تستطيع تقييد الأذونات الفعّالة، حتى عندما تسمح **سياسة قائمة على الهوية (Identity-based Policy)** بتنفيذ إجراء معيّن.

## 2. إعداد الاختبار

سُمِح بالإجراءين التاليين في سياسة الهوية المستخدمة في المحاكاة:

- `s3:ListBucket` — عرض قائمة الكائنات داخل حاوية Amazon S3.
- `s3:DeleteBucket` — حذف حاوية Amazon S3.

أما **حدود الصلاحيات** المستخدمة في المحاكاة، فقد سمحت بالإجراء `s3:ListBucket`، ورفضت الإجراء `s3:DeleteBucket` رفضًا صريحًا (**Explicit Deny**).

استُخدمت في الاختبار حاوية S3 **افتراضية وغير موجودة فعليًا** تحمل الاسم `finserv-boundary-demo-example`.

## 3. نتائج محاكاة قرارات السماح والمنع

<table dir="rtl">
<thead>
<tr>
<th align="right" dir="rtl"><strong>الإجراء</strong></th>
<th align="right" dir="rtl"><strong>قرار التقييم النهائي</strong></th>
<th align="right" dir="rtl"><strong>السماح وفق حدود الصلاحيات</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="right" dir="rtl"><code>s3:ListBucket</code></td>
<td align="right" dir="rtl"><code>allowed</code></td>
<td align="right" dir="rtl"><code>true</code></td>
</tr>
<tr>
<td align="right" dir="rtl"><code>s3:DeleteBucket</code></td>
<td align="right" dir="rtl"><code>explicitDeny</code></td>
<td align="right" dir="rtl"><code>false</code></td>
</tr>
</tbody>
</table>

تُظهر النتائج أن محاكي IAM **رفض حذف الحاوية صراحةً** رغم وجود سماح بهذا الإجراء في سياسة الهوية. وبذلك أثبت الاختبار، ضمن نطاق المحاكاة، أن حدود الصلاحيات تستطيع تقييد إجراء تسمح به سياسة الهوية.

## 4. أدلة الاختبار

يمكن الاطلاع على النتائج المسجلة بصيغة JSON، بما فيها قرارات التقييم، في الملف التالي:

[عرض نتائج محاكي سياسات IAM](evidence/iam-permissions-boundary-test.json)

## 5. ما يثبته الاختبار وما لا يثبته

أكد محاكي سياسات AWS IAM أن حدود الصلاحيات منعت الإجراء `s3:DeleteBucket`، على الرغم من وجود عبارة سماح (**Allow**) له في سياسة الهوية المستخدمة في الاختبار.

**ما أُثبت:** طريقة تقييم حدود الصلاحيات بالاشتراك مع سياسة هوية ضمن محاكاة AWS IAM.

**ما لم يُثبت:** فرض حدود الصلاحيات على دور IAM منشور فعليًا، أو نجاح اختبار مباشر على حاوية S3 حقيقية.

**لم يُنشأ أي دور أو مورد IAM، ولم تُنشأ حاوية S3 لهذا الاختبار.**

---

**الوثائق المرتبطة:** [مصفوفة تحقيق متطلبات المشروع — العربية](requirements-compliance.ar.md) · [الضوابط الأمنية المتقدمة — العربية](advanced-security-controls.ar.md)
