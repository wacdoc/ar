أدخل مستودع التكوين ops.soft ، وقم بتشغيل `./ssl.sh` ، وسيتم إنشاء مجلد `conf` في **الدليل العلوي** .

## الديباجة

يمكن لـ SMTP شراء الخدمات مباشرة من موردي السحابة ، مثل:

* [أمازون SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [دفع البريد الإلكتروني السحابي علي](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

يمكنك أيضًا إنشاء خادم البريد الخاص بك - إرسال غير محدود ، وتكلفة إجمالية منخفضة.

أدناه ، نوضح خطوة بخطوة كيفية إنشاء خادم البريد الخاص بنا.

## اختيار الخادم

يتطلب خادم SMTP الذاتي الاستضافة عنوان IP عام مع فتح المنافذ 25 و 456 و 587.

لقد حجبت السحابة العامة شائعة الاستخدام هذه المنافذ افتراضيًا ، وقد يكون من الممكن فتحها عن طريق إصدار أمر عمل ، لكنها مزعجة للغاية بعد كل شيء.

أوصي بالشراء من مضيف به هذه المنافذ مفتوحة ويدعم إعداد أسماء نطاقات عكسية.

هنا ، أوصي بـ [Contabo](https://contabo.com) .

Contabo هي شركة استضافة مقرها في ميونيخ ، ألمانيا ، تأسست عام 2003 بأسعار تنافسية للغاية.

إذا اخترت اليورو كعملة للشراء ، فسيكون السعر أرخص (الخادم بذاكرة 8 جيجابايت و 4 وحدات معالجة مركزية يكلف حوالي 529 يوانًا سنويًا ، ورسوم التثبيت الأولية مجانية لمدة عام واحد).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

عند تقديم طلب ، `prefer AMD` ، وسيحظى الخادم المزود بوحدة معالجة مركزية AMD بأداء أفضل.

في ما يلي ، سوف آخذ VPS الخاص بشركة Contabo كمثال لتوضيح كيفية إنشاء خادم البريد الخاص بك.

## تكوين نظام أوبونتو

نظام التشغيل هنا هو Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

إذا كان الخادم على ssh يعرض `Welcome to TinyCore 13!` (كما هو موضح في الشكل أدناه) ، فهذا يعني أن النظام لم يتم تثبيته بعد. الرجاء قطع اتصال ssh وانتظر بضع دقائق لتسجيل الدخول مرة أخرى.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

عندما يظهر `Welcome to Ubuntu 22.04.1 LTS` ، تكون عملية التهيئة قد اكتملت ، ويمكنك متابعة الخطوات التالية.

### [اختياري] تهيئة بيئة التطوير

هذه الخطوة اختيارية.

للراحة ، أضع التثبيت وتكوين النظام لبرنامج ubuntu في [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

قم بتشغيل الأمر التالي للتثبيت بنقرة واحدة.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

المستخدمون الصينيون ، يرجى استخدام الأمر التالي بدلاً من ذلك ، وسيتم تعيين اللغة والمنطقة الزمنية وما إلى ذلك تلقائيًا.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### يقوم Contabo بتمكين IPV6

قم بتمكين IPV6 حتى يتمكن SMTP أيضًا من إرسال رسائل بريد إلكتروني بعناوين IPV6.

تحرير `/etc/sysctl.conf`

قم بتعديل أو إضافة الأسطر التالية

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

تابع [البرنامج التعليمي contabo: إضافة اتصال IPv6 إلى الخادم الخاص بك](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

قم بتحرير `/etc/netplan/01-netcfg.yaml` ، أضف بضعة أسطر كما هو موضح في الشكل أدناه (ملف التكوين الافتراضي Contabo VPS يحتوي بالفعل على هذه الأسطر ، فقط قم بإلغاء التعليق عليها).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

ثم `netplan apply` لجعل التكوين المعدل ساري المفعول.

بعد نجاح التهيئة ، يمكنك استخدام `curl 6.ipw.cn` لعرض عنوان IPv6 لشبكتك الخارجية.

## استنساخ عمليات مستودع التكوين

```
git clone https://github.com/wactax/ops.soft.git
```

## قم بإنشاء شهادة SSL مجانية لاسم المجال الخاص بك

يتطلب إرسال البريد شهادة SSL للتشفير والتوقيع.

نستخدم [acme.sh](https://github.com/acmesh-official/acme.sh) لإنشاء الشهادات.

acme.sh هي أداة توقيع شهادات آلية مفتوحة المصدر ،

أدخل مستودع التكوين ops.soft ، وقم بتشغيل `./ssl.sh` ، وسيتم إنشاء مجلد `conf` في **الدليل العلوي** .

ابحث عن مزود DNS الخاص بك من [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) ، قم بتحرير `conf/conf.sh`

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

ثم قم بتشغيل `./ssl.sh 123.com` لإنشاء شهادات `123.com` و `*.123.com` لاسم المجال الخاص بك.

سيقوم التشغيل الأول بتثبيت [acme.sh](https://github.com/acmesh-official/acme.sh) تلقائيًا وإضافة مهمة مجدولة للتجديد التلقائي. يمكنك أن ترى `crontab -l` ، هناك مثل هذا الخط على النحو التالي.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

مسار الشهادة التي تم إنشاؤها هو شيء مثل `/mnt/www/.acme.sh/123.com_ecc。`

سيقوم تجديد الشهادة باستدعاء البرنامج النصي `conf/reload/123.com.sh` ، وتعديل هذا البرنامج النصي ، ويمكنك إضافة أوامر مثل `nginx -s reload` لتحديث ذاكرة التخزين المؤقت للشهادة للتطبيقات ذات الصلة.

## بناء خادم SMTP مع chasquid

[chasquid](https://github.com/albertito/chasquid) هو خادم SMTP مفتوح المصدر مكتوب بلغة Go.

كبديل لبرامج خادم البريد القديمة مثل Postfix و Sendmail ، يعد chasquid أبسط وأسهل في الاستخدام ، كما أنه أسهل في التطوير الثانوي.

Run `./chasquid/init.sh 123.com` تلقائيًا بنقرة واحدة (استبدل 123.com باسم مجال الإرسال الخاص بك).

## تكوين توقيع البريد الإلكتروني DKIM

يتم استخدام DKIM لإرسال توقيعات البريد الإلكتروني لمنع الرسائل من التعامل معها كرسائل غير مرغوب فيها.

بعد تشغيل الأمر بنجاح ، ستتم مطالبتك بتعيين سجل DKIM (كما هو موضح أدناه).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

ما عليك سوى إضافة سجل TXT إلى DNS الخاص بك (كما هو موضح أدناه).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## عرض حالة الخدمة والسجلات

 `systemctl status chasquid` عرض حالة الخدمة.

حالة التشغيل العادي كما هو موضح في الشكل أدناه

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 يمكن `grep chasquid /var/log/syslog` أو `journalctl -xeu chasquid` عرض سجل الأخطاء.

## تكوين اسم المجال العكسي

اسم المجال العكسي هو السماح بعنوان IP ليتم حلها إلى اسم المجال المقابل.

يمكن أن يؤدي تعيين اسم مجال عكسي إلى منع التعرف على رسائل البريد الإلكتروني على أنها بريد عشوائي.

عند استلام البريد ، سيقوم الخادم المستلم بإجراء تحليل عكسي لاسم المجال على عنوان IP الخاص بخادم الإرسال لتأكيد ما إذا كان لدى الخادم المرسل اسم مجال عكسي صالح.

إذا لم يكن لخادم الإرسال اسم مجال عكسي أو إذا كان اسم المجال العكسي لا يتطابق مع عنوان IP الخاص بخادم الإرسال ، فقد يتعرف الخادم المستلم على البريد الإلكتروني على أنه بريد عشوائي أو يرفضه.

قم بزيارة [https://my.contabo.com/rdns](https://my.contabo.com/rdns) وقم بالتهيئة كما هو موضح أدناه

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

بعد تعيين اسم المجال العكسي ، تذكر أن تقوم بتكوين دقة إعادة توجيه اسم المجال ipv4 و ipv6 إلى الخادم.

## قم بتحرير اسم المضيف الخاص بـ chasquid.conf

قم بتعديل `conf/chasquid/chasquid.conf` إلى قيمة اسم المجال العكسي.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

ثم قم بتشغيل `systemctl restart chasquid` لإعادة تشغيل الخدمة.

## أسيوط النسخ الاحتياطي إلى مستودع git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

على سبيل المثال ، أقوم بعمل نسخة احتياطية من مجلد conf إلى عملية github الخاصة بي على النحو التالي

قم بإنشاء مستودع خاص أولاً

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

أدخل دليل conf وأرسله إلى المستودع

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## أضف المرسل

يجري

```
chasquid-util user-add i@wac.tax
```

يمكن إضافة المرسل

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### تحقق من تعيين كلمة المرور بشكل صحيح

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

بعد إضافة المستخدم ، سيتم تحديث `chasquid/domains/wac.tax/users` ، تذكر إرساله إلى المستودع.

## يضيف DNS سجل SPF

SPF (إطار سياسة المرسل) هي تقنية للتحقق من البريد الإلكتروني تُستخدم لمنع الاحتيال عبر البريد الإلكتروني.

يتحقق من هوية مرسل البريد عن طريق التحقق من أن عنوان IP الخاص بالمرسل يطابق سجلات DNS لاسم المجال الذي يدعي أنه كذلك ، مما يمنع المحتالين من إرسال رسائل بريد إلكتروني مزيفة.

يمكن أن تؤدي إضافة سجلات نظام التعرف على هوية المرسل (SPF) إلى منع التعرف على رسائل البريد الإلكتروني كرسائل غير مرغوب فيها قدر الإمكان.

إذا كان خادم اسم المجال الخاص بك لا يدعم نوع نظام التعرف على هوية المرسل (SPF) ، فما عليك سوى إضافة سجل نوع TXT.

على سبيل المثال ، يكون SPF الخاص بـ `wac.tax` على النحو التالي

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF لـ `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

لاحظ أنني قمت `include:_spf.google.com` هنا ، وذلك لأنني سأقوم بتهيئة `i@wac.tax` كعنوان الإرسال في صندوق بريد Google لاحقًا.

## تهيئة DNS DMARC

DMARC هو اختصار لـ (مصادقة الرسائل المستندة إلى المجال والإبلاغ والمطابقة).

يتم استخدامه لالتقاط ارتداد نظام التعرف على هوية المرسل (SPF) (ربما يكون ذلك بسبب أخطاء في التكوين أو أن شخصًا آخر يتظاهر بأنه أنت لإرسال بريد عشوائي).

إضافة سجل TXT `_dmarc` ،

المحتوى على النحو التالي

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

معنى كل معلمة على النحو التالي

### ع (السياسة)

يشير إلى كيفية التعامل مع رسائل البريد الإلكتروني التي تفشل في التحقق من نظام التعرف على هوية المرسل (SPF) أو DKIM (البريد المعرف بمفاتيح المجال). يمكن تعيين المعلمة p على واحدة من ثلاث قيم:

* لا شيء: لم يتم اتخاذ أي إجراء ، فقط نتيجة التحقق تتم إعادتها إلى المرسل من خلال آلية الإبلاغ بالبريد الإلكتروني.
* عزل: ضع البريد الذي لم يجتاز التحقق في مجلد البريد العشوائي ، لكنه لن يرفض البريد مباشرة.
* الرفض: الرفض المباشر لرسائل البريد الإلكتروني التي تفشل في التحقق.

### fo (خيارات الفشل)

يحدد مقدار المعلومات التي يتم إرجاعها بواسطة آلية إعداد التقارير. يمكن ضبطه على إحدى القيم التالية:

* 0: تقرير نتائج التحقق لجميع الرسائل
* 1: الإبلاغ عن الرسائل التي تفشل في التحقق فقط
* د: قم فقط بالإبلاغ عن حالات فشل التحقق من اسم النطاق
* s: الإبلاغ عن حالات فشل التحقق من نظام التعرف على هوية المرسل (SPF) فقط
* ل: الإبلاغ فقط عن إخفاقات التحقق من DKIM

### روا وروف

* rua (عنوان URL للإبلاغ عن التقارير الإجمالية): عنوان البريد الإلكتروني لتلقي التقارير المجمعة
* ruf (الإبلاغ عن URI لتقارير الطب الشرعي): عنوان البريد الإلكتروني لتلقي التقارير التفصيلية

## أضف سجلات MX لإعادة توجيه رسائل البريد الإلكتروني إلى Google Mail

نظرًا لأنني لم أتمكن من العثور على صندوق بريد مجاني للشركة يدعم العناوين العالمية (Catch-All ، يمكنه تلقي أي رسائل بريد إلكتروني يتم إرسالها إلى اسم المجال هذا ، دون قيود على البادئات) ، فقد استخدمت chasquid لإعادة توجيه جميع رسائل البريد الإلكتروني إلى صندوق بريد Gmail الخاص بي.

**إذا كان لديك صندوق بريد أعمال مدفوع خاص بك ، فالرجاء عدم تعديل MX وتخط هذه الخطوة.**

تحرير `conf/chasquid/domains/wac.tax/aliases` ، قم بتعيين إعادة توجيه صندوق البريد

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` يشير إلى جميع رسائل البريد الإلكتروني ، `i` هو بادئة عنوان البريد الإلكتروني للمستخدم المرسل التي تم إنشاؤها أعلاه. لإعادة توجيه البريد ، يحتاج كل مستخدم إلى إضافة سطر.

ثم أضف سجل MX (أشير مباشرة إلى عنوان اسم المجال العكسي هنا ، كما هو موضح في السطر الأول في الشكل أدناه).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

بعد اكتمال التهيئة ، يمكنك استخدام عناوين بريد إلكتروني أخرى لإرسال رسائل بريد إلكتروني إلى `i@wac.tax` و `any123@wac.tax` لمعرفة ما إذا كان بإمكانك تلقي رسائل بريد إلكتروني في Gmail.

إذا لم يكن كذلك ، فتحقق من سجل chasquid ( `grep chasquid /var/log/syslog` ).

## أرسل بريدًا إلكترونيًا إلى i@wac.tax باستخدام Google Mail

بعد تلقي Google Mail للبريد ، كنت أتمنى بطبيعة الحال الرد بـ `i@wac.tax` بدلاً من i.wac.tax@gmail.com.

قم بزيارة [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) وانقر فوق "إضافة عنوان بريد إلكتروني آخر".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

بعد ذلك ، أدخل رمز التحقق الذي تلقيته عبر البريد الإلكتروني الذي تمت إعادة توجيهه إليه.

أخيرًا ، يمكن تعيينه كعنوان المرسل الافتراضي (جنبًا إلى جنب مع خيار الرد بنفس العنوان).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

بهذه الطريقة ، نكون قد أكملنا إنشاء خادم بريد SMTP وفي نفس الوقت استخدمنا Google Mail لإرسال واستقبال رسائل البريد الإلكتروني.

## أرسل بريدًا إلكترونيًا تجريبيًا للتحقق مما إذا كان التكوين ناجحًا أم لا

أدخل `ops/chasquid`

`direnv allow` بتثبيت التبعيات (تم تثبيت direnv في عملية التهيئة السابقة ذات المفتاح الواحد وإضافة خطاف إلى الصدفة)

ثم اركض

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

معنى المعلمات على النحو التالي

* المستخدم: اسم مستخدم SMTP
* تمرير: كلمة مرور SMTP
* إلى: المستلم

يمكنك إرسال بريد إلكتروني تجريبي.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

يوصى باستخدام Gmail لتلقي رسائل بريد إلكتروني تجريبية للتحقق من نجاح التكوينات.

### تشفير TLS القياسي

كما هو موضح في الشكل أدناه ، يوجد هذا القفل الصغير ، مما يعني أنه تم تمكين شهادة SSL بنجاح.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

ثم انقر فوق "إظهار البريد الإلكتروني الأصلي"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

كما هو موضح في الشكل أدناه ، تعرض صفحة بريد Gmail الأصلي DKIM ، مما يعني أن تهيئة DKIM ناجحة.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

تحقق من تم الاستلام في رأس البريد الإلكتروني الأصلي ، ويمكنك أن ترى أن عنوان المرسل هو IPV6 ، مما يعني أنه تم أيضًا تكوين IPV6 بنجاح.
