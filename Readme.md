# ارزیابی امنیتی Firmware های نهفته با استفاده از ابزارهای Fuzzing

## Table of Contents
1. [About The Project](#about-the-project)
2. [Tools](#tools)
3. [Getting Started](#getting-started)
   - [Implementation Details](#implementation-details)
   - [How to Run](#how-to-run)
4. [Results](#results)
5. [Related Links](#related-links)
6. [Authors](#authors)

## About The Project

این پروژه به ارزیابی امنیتی Firmware های نهفته با استفاده از ابزارهای Fuzzing مخصوص سامانه‌های نهفته می‌پردازد.

در این پروژه ابتدا مقاله **Fuzzware** (Scharnówski et al., USENIX Security 2022) به‌عنوان یکی از رویکردهای پیشرفته برای مدل‌سازی MMIO بررسی شد. سپس ابزار **P2IM** روی Firmware یک Gateway مبتنی بر **STM32** و پروتکل **Firmata** اجرا شد.

هدف اصلی، بررسی روش‌های مختلف مدل‌سازی تعامل Firmware با سخت‌افزار، کاهش Input Overhead و شناسایی آسیب‌پذیری‌های امنیتی در Firmware های واقعی است.

## Tools

در این پروژه از ابزارها و فناوری‌های زیر استفاده شده است:

- **Fuzzware**: چارچوب Fuzzing برای Firmware های نهفته با مدل‌سازی دقیق MMIO و استفاده از Dynamic Symbolic Execution.
- **P2IM**: رویکرد Pattern-based برای مدل‌سازی رفتار Peripheral ها در اجرای Firmware.
- **QEMU**: محیط شبیه‌سازی مورد استفاده برای اجرای Firmware و Replay کردن Crash ها.
- **Unicorn Engine**: ISA Emulator مورد استفاده در معماری Fuzzware.
- **AFL / AFL++**: موتورهای Coverage-guided Fuzzing.
- **angr**: موتور Dynamic Symbolic Execution مورد استفاده در Fuzzware.
- **STM32 / ARM Cortex-M**: پلتفرم Firmware مورد بررسی.
- **Firmata**: پروتکل ارتباطی مورد استفاده در Gateway.
- **objdump و addr2line**: ابزارهای مورد استفاده برای نگاشت آدرس Crash به کد منبع.

## Getting Started

### Implementation Details

فرآیند پروژه در چند مرحله انجام شد:

1. مطالعه و بررسی مقاله Fuzzware و مفهوم **Input Overhead**.
2. بررسی مدل‌های مختلف دسترسی به MMIO شامل:
   - Constant
   - Passthrough
   - Bitextract
   - Set
   - Identity
3. بررسی نحوه استفاده Fuzzware از DSE محلی برای استخراج مدل دسترسی MMIO.
4. بررسی محدودیت‌های روش‌های قبلی مانند High-level Emulation، Pattern-based Modeling و Symbolic Execution Guided.
5. آماده‌سازی Firmware Gateway مبتنی بر STM32 و پروتکل Firmata برای اجرای P2IM.
6. اجرای Firmware در QEMU و انجام کمپین Fuzzing.
7. جمع‌آوری و Triage کردن Crash ها با استفاده از Replay، استخراج PC و Backtrace و نگاشت آدرس‌ها به کد منبع.
8. تحلیل فنی یک Crash و بررسی علت ریشه‌ای آن.
9. مقایسه یافته به‌دست‌آمده با آسیب‌پذیری گزارش‌شده در مقاله P2IM.

### How to Run

برای اجرای بخش بررسی‌شده در این پروژه، Firmware Gateway در محیط QEMU و با استفاده از P2IM اجرا شد.

فرآیند کلی اجرا:

1. آماده‌سازی Firmware Gateway مبتنی بر STM32.
2. آماده‌سازی محیط P2IM.
3. اجرای Firmware در QEMU.
4. اجرای کمپین Fuzzing و جمع‌آوری Crash ها.
5. Replay کردن Crash ها در QEMU.
6. استخراج PC و Backtrace.
7. نگاشت آدرس Crash به کد منبع با `arm-none-eabi-addr2line` و `objdump`.
8. بررسی تابع و مسیر اجرایی منتهی به Crash.

## Results

### بررسی Fuzzware

Fuzzware روی ۷۷ نمونه Firmware از ۱۹ پلتفرم سخت‌افزاری ارزیابی شده است.

نتایج گزارش‌شده در مقاله شامل موارد زیر است:

- کاهش Input Space بین **۳۹.۴٪ تا ۴۹.۸۳٪** در پیاده‌سازی سطح بایت و تا **۹۵.۵٪** در حالت فرضی سطح بیت.
- تولید میانگین ۶۲ مدل در هر آزمایش ۲۴ ساعته؛ هر مدل حدود ۶ ثانیه زمان برده و کمتر از ۲٪ زمان کل آزمایش را مصرف کرده است.
- پاس شدن هر ۶۶ تست واحد P2IM.
- به‌طور میانگین حدود **۴۴٪ پوشش کد بیشتر از P2IM** و **۶۱٪ بیشتر از µEmu** در مقایسه مستقیم روی ۲۱ نمونه Firmware واقعی.
- کشف ۳ باگ گزارش‌نشده در نمونه‌های مورد استفاده کارهای رقیب.
- کشف مجموعاً ۱۵ آسیب‌پذیری ناشناخته در Stack های شبیه Zephyr و NG-Contiki و ثبت ۱۲ CVE.
- از میان ۶۱ Crash یکتا، ۴۲ مورد آسیب‌پذیری امنیتی واقعی، ۱۶ مورد ناشی از مقداردهی اولیه نامناسب و ۳ مورد False Positive گزارش شده‌اند.

### اجرای P2IM روی Firmware Gateway

در اجرای P2IM روی Firmware Gateway:

- تعداد **۸۴ Crash** با Signal `SIGSEGV` جمع‌آوری شد.
- Crash ها با Replay در QEMU و استخراج PC و Backtrace بررسی شدند.
- یکی از Crash ها در تابع `HAL_UART_TxCpltCallback` در آدرس `0x08008774` بررسی شد.
- در این مسیر، مقدار Function Pointer خوانده‌شده از جدول Callback برابر `NULL` بود.
- کد بدون بررسی NULL بودن Pointer، دستور `blx r3` را اجرا کرد و در نتیجه اجرای برنامه به آدرس `0x00000000` منتقل شد.
- خطای حاصل `UNMAPPED_FETCH_ERR_UC` و سپس `SIGSEGV` بود.

### Root Cause

علت ریشه‌ای Crash، نبودن بررسی NULL روی Function Pointer قبل از فراخوانی غیرمستقیم بود.

CWE های مرتبط:

- **CWE-476 — NULL Pointer Dereference**
- **CWE-252 — Unchecked Return Value**

این یافته از نظر کلاس آسیب‌پذیری با مورد گزارش‌شده در مقاله P2IM، یعنی ترکیب **CWE-129 + CWE-787**، هم‌خانواده است؛ هرچند CWE دقیق و سازوکار Crash متفاوت هستند.

### Positive False Consideration

با توجه به اینکه P2IM می‌تواند Interrupt را با Timing دلخواه مدل کند، احتمال Positive False وجود دارد. برای اعتبارسنجی Crash باید Initialization Code مربوط به UART بررسی شود تا مشخص شود آیا در سخت‌افزار واقعی ممکن است UART فعال باشد ولی Callback متناظر ثبت نشده باشد یا خیر.

## Related Links

- [Fuzzware Paper — USENIX Security 2022](https://www.usenix.org/conference/usenixsecurity22/presentation/scharnowski)
- [Fuzzware Repository](https://github.com/fuzzware-fuzzer/fuzzware)
- [QEMU](https://www.qemu.org/)
- [Unicorn Engine](https://www.unicorn-engine.org/)
- [angr](https://angr.io/)
- [AFL++](https://aflplus.plus/)
- [EDK II](https://github.com/tianocore/edk2)

## Authors

- حسین پایمرد
- علیرضا سرباز

**Course:** آزمایشگاه اینترنت اشیا  
**Semester:** نیم‌سال تابستان ۱۴۰۴–۱۴۰۵
