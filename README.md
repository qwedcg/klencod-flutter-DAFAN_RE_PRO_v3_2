# DAFAN RE PRO v3.2 — Lua engine integration

تمت إضافة محرك Lua المقتبس من الملف الذي زوّدني به المستخدم `nadeem.py` إلى مجلد `tools/`.

## ما الذي يفعله؟
- يفك تنسيق Bytecode مخصصاً يبدأ بترويسة `\x1bLua` وإصدار `0x53` وفق بنية ومفتاح XOR الموجودين في المصدر.
- يحوّل البنية إلى Lua 5.3 قياسي ثم يستدعي `luadec` لإنتاج المصدر.
- يعيد تجميع المصدر بواسطة `luac5.3` ثم يعيد تحويل Opcode والثوابت إلى التنسيق المخصص.
- يحفظ الناتج في مسار منفصل ولا يسمح بكتابة الناتج فوق ملف Bytecode الأصلي.
- تم تعديل تنظيف النص ليحافظ على UTF-8 والنص العربي بدلاً من حذف المحارف غير ASCII.

## مهم جداً
هذا محرك مرافق يعمل عبر Termux، وليس منفذاً داخل واجهة APK Java الحالية. يحتاج Termux وPython ومكتبة `rich`، ويحتاج ملف `luadec` متوافقاً مع Lua 5.3 داخل `~/DAFAN_RE_PRO/EXES/luadec`. يمكن تثبيت `luac5.3` عبر حزمة Termux `lua53` إذا كانت متاحة في مستودع جهازك.

المحلّل الأصلي يفترض تنسيقاً مخصصاً محدداً: ترويسة بطول 34 بايت، Lua 5.3، ترتيب بيانات/Opcode محدد، ومفتاح XOR ثابت. لا يدعم تلقائياً Lua 5.1/5.2/5.4 أو كل تعديلات الألعاب. كما أن `luadec` و`luac5.3` يجب أن يكونا متوافقين مع بنية الملف. لم يتم اختبار فك/تجميع ملف لعبة حقيقي في هذه الحزمة.

## التثبيت على Termux
1. فك ضغط المشروع.
2. انسخ مجلد `tools` إلى مساحة Termux أو إلى مسار ظاهر في Termux.
3. نفّذ: `bash tools/setup_termux.sh`
4. ضع برنامج `luadec` المتوافق في `~/DAFAN_RE_PRO/EXES/luadec` ثم نفّذ `chmod +x ~/DAFAN_RE_PRO/EXES/luadec`.
5. ضع ملف Bytecode الأصلي في مجلد `ORIGINAL LUA`.

## فك Lua
```bash
python tools/DAFAN_LUA_ENGINE.py decompile \
  "$HOME/DAFAN_RE_PRO/LUA_TOOL/ORIGINAL LUA/example.lua" \
  -o "$HOME/DAFAN_RE_PRO/LUA_TOOL/SOURCE LUA/example.lua"
```

افتح ملف المصدر في `SOURCE LUA` وعدّله، ثم احفظه في `EDITED LUA` أو استخدم مساره مباشرة. ملف `.lua.orig` الذي ينشأ بجانب المصدر يحفظ نسخة المقارنة الأصلية.

## تجميع Lua
```bash
python tools/DAFAN_LUA_ENGINE.py compile \
  "$HOME/DAFAN_RE_PRO/LUA_TOOL/EDITED LUA/example.lua" \
  "$HOME/DAFAN_RE_PRO/LUA_TOOL/ORIGINAL LUA/example.lua" \
  "$HOME/DAFAN_RE_PRO/LUA_TOOL/COMPILED LUA/example.lua"
```

## حالة APK
تطبيق Android الموجود في `app/` ما زال يعرض أدوات التحليل والتحرير النصي السابقة. المحرك الجديد يضاف كمكوّن Termux مرافق؛ لا تدّعي هذه النسخة أن APK يستدعيه داخلياً.
