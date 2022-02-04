# Slot Machine

_[English](README.md) | **Русский**_

[![Slot Machine](https://img.youtube.com/vi/Y3zFFcCXAkc/0.jpg)](https://youtu.be/Y3zFFcCXAkc)

AMX Mod X плагин для Counter-Strike.

Плагин позволяет размещать на карте слот машины (также известные как однорукие бандиты) и играть в них. Игрок делает ставку и ожидает выпадения выигрышной комбинации символов. Победа засчитывается при заполнении целого ряда или главной диагонали одинаковыми символами. Присутвует API для добавления собственных призов и вспомогательный инструмент для кастомизации модели игрового автомата.

## Команды
* `slot_machine` — меню размещения игровых автоматов на карте.

## Настройки
Конфигурация плагина хранится в директории *addons/amxmodx/configs/slot_machine*. В файле *_pattern.json* содержится матрица 3 на 8 с разметкой символов для барабанов игрового автомата. Символ определяется числом от 0 до 7. Индекс символа совпадает с индексом награды. На данный момент отсутствует возможность изменения размерности матрицы. Пример содержимого файла конфигурации:

```json
[
	[0, 1, 2, 0, 3, 2, 4, 5],
	[1, 0, 2, 4, 0, 2, 3, 5],
	[2, 0, 3, 4, 2, 0, 5, 1]
]
```

### Добавление собственных призов и ставок
Файл исходного кода *next21_slot_machine.sma* содержит только базовую функциональность плагина без выдачи наград. Призы и ставки должны быть реализованы в отдельном плагине при помощи выделенного для этой цели API:

```pawn
/**
 * Вызывается, когда клиент выигрывает в игровом автомате
 *
 * @param 	iPlayer			- Индекс клиента
 * @param	iPrize			- Индекс награды
 */
forward client_slot_machine_win(const iPlayer, const iPrize)

/**
 * Вызывается перед активацией игрового автомата клиентом
 *
 * @param 	iPlayer			- Индекс клиента
 * @return					- Использовать PLUGIN_HANDLED при необходимости прервать активацию
 */
forward client_slot_machine_spin(const iPlayer)
```

В файле *addons/amxmodx/scripting/next21_slot_machine_money.sma* присутствует пример реализации денежной системы для слот машины:

```pawn
#include <slotmachine>

// Денежная ставка
#define BET 100

// Денежная награда
new const GAME_PRIZES[] =
{
	200,
	300,
	500,
	800,
	1000,
	10000
}

public client_slot_machine_win(const iPlayer, const iPrize)
{
    new iAddMoney = GAME_PRIZES[iPrize]
    rg_add_account(iPlayer, iAddMoney)
}

public client_slot_machine_spin(const iPlayer)
{
    if (get_member(iPlayer, m_iAccount) < BET)
		return PLUGIN_HANDLED

    rg_add_account(iPlayer, -BET)
    return PLUGIN_CONTINUE
}
```

### Изменение модели игрового автомата
Скрипт *slot_machine_texgen.py* позволяет относительно быстро сгенерировать текстуру и UV-разметку барабанов по заданным параметрам. Для этого необходимо:

1) Разместить файл *addons/amxmodx/configs/slot_machine/_pattern.json* в директорию *cfg*.
2) В файле *cfg/text.json* указать названия ставок и призов, цвета надписей и тени.
3) Разместить в директорию *symbols* изображения символов барабанов. Названия файлов должны совпадать с номерами символов (от 0 до 7).
4) Запустить скрипт при помощи интерпретатора Python 3. В системе должен быть установлен модуль [Pillow](https://pillow.readthedocs.io/en/stable/).
5) Скомпилировать с помощью studiomdl или любым другим компилятором модель игрового автомата в директори *dist*.

Качество сгенерированных текстур может быть недостаточно высоким. Для решения данной проблемы предлагается пересохранить полученные изображения с расширением png в любом другом растровом редакторе (например GIMP).

## Требования
- [Reapi](https://github.com/s1lentq/reapi)

## Авторы
- [Psycrow](https://github.com/Psycrow101)