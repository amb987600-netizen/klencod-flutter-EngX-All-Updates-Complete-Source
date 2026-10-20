# بناء APK من مشروع EngX عبر Klencod

> **نطاق هذا الدليل:** EngX تطبيق React/Vite يستخدم Capacitor 7. هذه الحزمة مصدر ومشروع Android، وليست APK جاهزًا. لم يُبنَ APK أو يُختبر Klencod أو جهاز Android ضمن هذا المشروع؛ نفّذ الخطوات وتحقق من الناتج في بيئتك.

## ما هو المؤكد وما ليس مؤكدًا

- المؤكد من ملفات الحزمة: `package.json` يعرّف `@capacitor/android` و`@capacitor/cli` بإصدار 7.x، و`android/` مشروع Gradle، و`capacitor.config.ts` يحدد `webDir: 'dist'` واسم الحزمة `com.engx.app`.
- المؤكد من المشروع: `npm run android:sync` ينفذ `npm run build && cap sync android`، و`npm run android:open` يطلب فتح مشروع Android.
- بحثنا عن Klencod IDE في المصادر المفتوحة المتاحة. القوائم العامة التي ظهرت تصف دعم Flutter/Java/Kotlin/React Native وتذكر بناء APK أو GitHub Actions، لكن لم نجد توثيقًا رسميًا عامًا يثبت استيراد مشروع Capacitor 7 أو تشغيل `npm ci` و`npx cap sync android` أو توافق نسخة Android Gradle Plugin الموجودة هنا. لذلك **لا نعد بأن Klencod سيبني هذا المشروع** ولا نعد بأن كل زر أو مسار في Klencod موجود بنفس الاسم.
- وجود خيار Android/React Native في قائمة Klencod لا يساوي دعم Capacitor. إذا رفض Klencod فتح المشروع أو لم يوفر Node.js/Java/SDK/طرفية، استخدم المسار البديل أدناه على Android Studio أو Gradle، أو شغّل البناء في CI تملكه.

## قبل البدء: متطلبات المشروع الفعلية

من جذر المشروع (`EngX/`) تحتاج إلى:

- Node.js 22 LTS (كما في README) و`npm`.
- اتصال الشبكة في أول تثبيت لتنزيل الحزم وGradle/Android SDK.
- JDK 21 مطلوب لكود Capacitor 7 المستخدم هنا، وهو الاختيار المعتمد في هذا الدليل. Android Gradle Plugin 8.x يحتاج JDK 17 أو أحدث؛ إن ظهرت مشكلة توافق JDK، اختر JDK 21 ولا تستخدم JDK 17 لبناء هذه الحزمة دون تعديل متطلبات Java 21 الخاصة بـCapacitor، ولا تخلط JDKs بين IDE والطرفية.
- Android SDK مع API 35 على الأقل لهذا المشروع (`compileSdkVersion = 35` و`targetSdkVersion = 35`، و`minSdkVersion = 23`).
- Gradle Wrapper الموجود في `android/gradlew` ويطلب Gradle 8.11.1. لا تحتاج إلى تثبيت Gradle عالميًا إذا شغّلت الـ wrapper.

لا يوجد `local.properties` مُلتزم بالمستودع؛ يجب أن يكتشف Android Studio/Gradle SDK أو يضبطه محليًا. لا تُضمّن هذا الملف أو مفاتيح التوقيع في المستودع.

## المسار المقترح داخل Klencod (إن كانت الأدوات متاحة)

1. استورد **جذر المشروع** الذي يحتوي `package.json` و`android/`، وليس مجلد `dist` وحده.
2. افتح طرفية Klencod إن كانت متاحة، وتحقق من الإصدارات:

   ```bash
   node --version
   npm --version
   java -version
   ```

3. ثبّت الاعتماديات من lockfile. **لا تتجاوز `npm ci`**؛ مجلد `node_modules` ليس جزءًا موثوقًا من المصدر وقد يكون غير موجود في نسخة الاستيراد:

   ```bash
   npm ci
   ```

4. أنشئ نسخة الويب ثم انسخها وحدّث اعتماديات Capacitor داخل مشروع Android:

   ```bash
   npm run android:sync
   ```

   هذا الأمر يساوي `npm run build` ثم `cap sync android`. و`cap sync` ينفذ copy ثم update: يضع مخرجات `dist/` في أصول التطبيق ويزامن إضافات Capacitor. لذلك لا يكفي فتح `android/` قبل تثبيت npm وتشغيل sync؛ قد تكون `node_modules` أو `android/app/src/main/assets/public/` ناقصة أو قديمة.

5. إذا كان Klencod يتيح تشغيل Gradle من الجذر، ابنِ نسخة debug موقعة تلقائيًا بمفتاح debug للاختبار فقط:

   ```bash
   cd android
   ./gradlew assembleDebug
   ```

   الناتج المتوقع عادةً هو `android/app/build/outputs/apk/debug/app-debug.apk`. هذا **مسار متوقع من إعداد Gradle** وليس نتيجة اختبار أجري هنا.

6. ثبّت APK على جهاز اختبار أو شغّله من Klencod/ADB فقط بعد مراجعة الصلاحيات واختبار الوظائف. نسخة debug ليست إصدار متجر. لإصدار release يلزم إعداد توقيعك أنت؛ لا توجد مفاتيح توقيع في الحزمة، وهذا الدليل لا ينشئها ولا يضع أسرارًا في الملفات.

إذا عدّلت كود React أو أي إضافة Capacitor، أعد `npm run android:sync` قبل كل إعادة بناء. لا تحذف `android/` لإصلاح خطأ عابر؛ اطبع سجل Gradle الكامل أولًا.

## البديل الموثوق: Android Studio أو Gradle

Capacitor يوثق إدارة مشروع Android عبر Android Studio، ويمكن فتح مجلد `android/` كمشروع. المسار المحافظ هو:

```bash
cd /path/to/EngX
npm ci
npm run android:sync
# ثم افتح android/ في Android Studio، أو:
cd android
./gradlew assembleDebug
```

في Android Studio اختر Gradle JDK مناسبًا (JDK 21 الموصى به هنا)، وثبّت Android SDK API 35، ثم نفّذ **Sync Project with Gradle Files**. يعلن إعداد المشروع `minSdkVersion = 23`، بينما يوثق Capacitor Android دعم API 24+؛ تحقّق من جهازك/محاكيك وفق نسخة Capacitor المستخدمة قبل التوزيع. إذا كان هدفك مجرد APK debug، لا يلزم `cap open android`؛ أما `npm run android:open` فهو اختصار لطلب فتح المشروع في IDE المُسجّل.

## البناء من الهاتف عبر GitHub Actions

الحزمة تتضمن `.github/workflows/android-apk.yml`. ارفع **محتويات مجلد EngX نفسه** إلى جذر مستودعك، بحيث يظهر `package.json` و`.github` و`android` في المستوى الأول. من GitHub افتح **Actions → Build EngX Android APK → Run workflow**. المسار يثبت Node 22 وJDK 21 وAndroid SDK 35، ويشغل الاختبارات وبناء الويب ومزامنة Capacitor ثم `assembleDebug`. بعد نجاحه نزّل `EngX-debug-APK` من **Artifacts** وفك الضغط لتحصل على `app-debug.apk`.

هذا workflow جاهز في المصدر لكنه لم يُشغّل على حساب GitHub ولم يُختبر داخل Klencod هنا. إذا رفع Klencod الملفات دون استبدال workflow المخصص، يمكن متابعة البناء من GitHub؛ لو استبدله بقالب Flutter/React Native فلا تستخدم القالب لبناء EngX، وشغّل workflow المرفق مباشرة. لا تحتاج مفاتيح توقيع شخصية لبناء debug، وهو ليس إصدار نشر. استخدام GitHub Actions خاضع لتوفر الخدمة وحدود حسابك.

## نسخة الويب المضمنة ونسخة المصدر

المستودع يحتوي `dist/`، لكنها مخرجات ويب قابلة لإعادة التوليد وليست بديلًا عن تثبيت الاعتماديات. `capacitor.config.ts` يقول إن `webDir` هو `dist`؛ لذلك ما يظهر داخل APK هو آخر `dist/` نسخته عملية `cap sync`. لتجنب تضمين واجهة قديمة، نفّذ دائمًا:

```bash
npm ci
npm run build
npx cap sync android
```

(`npm run android:sync` يجمع هذه الخطوتين للمشروع.) لا تفتح `dist/index.html` بالنقر المباشر؛ هذا لا يبني APK ولا يضمن التخزين/التشفير، كما هو موضح في README.

## أخطاء شائعة وتشخيص مختصر

- **`cap: command not found` أو حزم Capacitor مفقودة:** لم تُنفّذ `npm ci` من جذر المشروع، أو لم تُستورد `package-lock.json`.
- **Gradle لا يجد SDK:** ثبّت API 35 واضبط SDK في Android Studio أو `local.properties` محليًا؛ لا تلتزم به.
- **خطأ Java/Gradle:** افحص `java -version` وGradle JDK في IDE. AGP 8.x لا يعمل على JDK قديم.
- **APK يفتح واجهة قديمة/بيضاء:** أعد `npm run build` ثم `npx cap sync android`، وتحقق من نجاح الأمر قبل Gradle.
- **إضافات Capacitor أو LiteRT تفشل في البناء:** لا تحذف dependency ولا تغيّر نسخ Android عشوائيًا؛ احفظ سجل الخطأ وارجع لإصدارات `package.json` وملفات Gradle الحالية.

## مصادر مفتوحة راجعتها

- [Capacitor Android documentation](https://capacitorjs.com/docs/android): Android project عبر Android Studio، إضافة/فتح المنصة ومتطلبات API 24+ في توثيق Capacitor الحالي.
- [Capacitor CLI: `cap sync`](https://capacitorjs.com/docs/cli/commands/sync): يوضح أن sync يشغّل copy ثم update.
- [Android build documentation](https://developer.android.com/build): بنية Gradle وGradle Wrapper وملفات SDK المحلية.
- [Android JDK guidance](https://developer.android.com/build/jdks): يوضح اختيار JDK واحتياج AGP 8.x إلى JDK 17 أو أحدث.
- لا يوجد في هذه المصادر توثيق رسمي عام يثبت توافق Klencod مع Capacitor؛ لذلك بقيت ادعاءات Klencod في هذا الدليل محدودة بما يمكن التحقق منه من القوائم العامة، لا باعتبارها ضمانًا.

- [قائمة Klencod العامة على APKPure](https://apkpure.com/klencod-ide/com.flutteride.codweva): تصف القوالب الأربعة وبناء APK عبر GitHub Actions ولا تثبت دعم Capacitor.
