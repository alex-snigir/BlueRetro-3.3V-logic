*Read this in other languages: [English](README_EN.md)*

# BlueRetro + Murmulator: Bluetooth-джойстики для ретро-компьютера на перфборде

Подключение Bluetooth-геймпадов (Xbox One, PS/Switch и других HID-устройств) к ретро-компьютеру [Murmulator](https://murmulator.ru/howto) (RP2040) в качестве двух NES/Dendy-совместимых джойстиков через самодельный адаптер [BlueRetro](https://github.com/darthcloud/BlueRetro) (ESP32), выступающий мостом.

BlueRetro эмулирует на выходе стандартный протокол сдвигового регистра NES/Dendy (CLOCK/LATCH/DATA) и подключается напрямую к джойстик-портам DE-9 Мурмулятора — проводные NES-контроллеры в проекте не используются.

## Железо

- **Хост:** Murmulator (RP2040, сборка на перфборде) — VGA, PS/2-клавиатура, аудио, джойстик-порты.
- **Адаптер:** самодельный BlueRetro на ESP32 D1 mini (MH-ET LIVE, CH9102), прошивка по спецификации HW1, собственная KiCad-схема с глобальным и портовыми светодиодами статуса, входной защитой питания и вынесенной кнопкой BOOT/IO0.
- **Кабели:** JST XH 4-pin (BlueRetro) → DE-9 (Murmulator), распиновка и разводка описаны в project summary.
- **Схемы:** KiCad-проект платы BlueRetro — [`schematics/BlueRetro3V3/`](schematics/BlueRetro3V3/) (готовый PDF: [`BlueRetro3V3.pdf`](schematics/BlueRetro3V3/BlueRetro3V3.pdf)).

## Документация

| Файл | Описание |
|---|---|
| [`doc/blueretro-murmulator-project-summary.md`](doc/blueretro-murmulator-project-summary.md) / [`_EN`](doc/blueretro-murmulator-project-summary_EN.md) | Полное техническое описание: выбор платы, распиновка, питание, LED-индикация, прошивка, проверка осциллографом |
| [`doc/blueretro-murmulator-user-guide.md`](doc/blueretro-murmulator-user-guide.md) / [`_EN`](doc/blueretro-murmulator-user-guide_EN.md) | Повседневное использование: пейринг и отключение Bluetooth-геймпадов |
| [`murmulator-vga-project-summary.md`](https://github.com/alex-snigir/Murmulator-RP2040-Black-Perfboard-VGA/blob/master/doc/murmulator-vga-project-summary.md) / [`_EN`](https://github.com/alex-snigir/Murmulator-RP2040-Black-Perfboard-VGA/blob/master/doc/murmulator-vga-project-summary_EN.md) | Базовая сборка Мурмулятора на перфборде (хост-платформа) — отдельный репозиторий |

## Статус

Железо собрано и проверено: пейринг, переключение между двумя портами и протокол CLOCK/LATCH/DATA подтверждены осциллографом на реальных нажатиях кнопок Xbox One.

## Благодарности

Проект основан на [BlueRetro](https://github.com/darthcloud/BlueRetro) авторства darthcloud (CERN-OHL-P-2.0 / Apache-2.0) и платформе [Murmulator](https://murmulator.ru/howto).
