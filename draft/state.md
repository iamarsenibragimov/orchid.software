## Состояние экрана

В PHP когда мы хотим сохранить промежуточный результат между запросами, обычно используется хранилища, например cессия, базы данных или быстрые хранилища типа Redis.

Для более простых случаях можно передать состояние прямо в url адресе, скорее всего вы видели это, например, `operation?result=success`

Каждый из этих способов уместен и вполне применим.

## Публичные свойства

Но В Laravel Orchid есть удобное решение для хранения небольшого количества информации (Например, модель Eloquent). 
Это сделано благодаря тому, что каждый запрос передаёт с собой состояние всех публичный свойств экрана.
Давайте попробуем это на практике, создадим новый экран под названием `StateScreen` командой artisan:

```bash
php artisan orchid:screen StateScreen
```

Затем зарегистрируем его в файле маршрута:

```php
use App\Orchid\Screens\StateScreen;

Route::screen('state', StateScreen::class)->name('state');
```

Добавим следующий код в только что созданный экран, чтобы установить и отобразить простую строку "Привет мир":

```php
<?php

namespace App\Orchid\Screens;

use Illuminate\Http\Request;
use Orchid\Screen\Screen;
use Orchid\Support\Color;
use Orchid\Screen\Fields\Label;
use Orchid\Screen\Actions\Button;
use Orchid\Support\Facades\Layout;

class StateScreen extends Screen
{
    /**
     * Fetch data to be displayed on the screen.
     *
     * @return array
     */
    public function query(): array
    {
        return [
            'message' => 'Hello Word!',
        ];
    }

    /**
     * The name of the screen displayed in the header.
     *
     * @return string|null
     */
    public function name(): ?string
    {
        return 'State';
    }

    /**
     * The screen's layout elements.
     *
     * @return \Orchid\Screen\Layout[]|string[]
     */
    public function layout(): array
    {
        return [
            Layout::rows([
                Label::make('message')
                    ->title('Your message:'),
            ]),
        ];
    }
}
```

Перейдем в браузере на нашу страницу и посмотрим как выглядит экран. На нём должна быть выведена надпись "Hello Word!". 
Добавим в наш экран простой метод в котором посмотрим на содержимое запроса:

```php
/**
 * The screen's layout elements.
 *
 * @return \Orchid\Screen\Layout[]|string[]
 */
public function layout(): array
{
    return [
        Layout::rows([
            Label::make('message')
                ->title('Your message:'),

            Button::make('Submit')
                ->type(Color::DARK)
                ->method('submit'),
        ]),
    ];
}

/**
 * @param \Illuminate\Http\Request $request
 *
 * @return void
 */
public function submit(Request $request)
{
    dd($request->all());
}
```

После нажатия на кнопку "Отправить" мы видем, что значение `message` нету и мы потеряли состояние `Hello Word!`. 
Классическое решение обычно заключается в добавление в форму `input` с типом `hidden`, что бы он был скрыт для пользователя.
Но Orchid может автоматически решать такие задачи, для этого нужно определить публичное свойство с тем же именем ключа что и в `query`

```php
class StateScreen extends Screen
{
    public $message;

    //...


    public function submit()
    {
        dd($this->message);
    }
}

```

Теперь если мы отправим форму, то значение свойства сохраниться таким же каким мы его задали в процессе.