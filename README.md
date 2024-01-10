# Rapid-Laravel-Apps-With-Filament

## 2. Filament Forms

### 2-1 Introduction to Filament

#### Install Filament

```sh
composer require filament/filament:"^3.1" -W
 
php artisan filament:install --panels
```
Note both livewire and Alpine are also installed.

#### Change colors

Inside `AdminPanelProvider.php`, you can easily change the colors of the app.

````php
return $panel
  ->colors(colors:[
    'primary' => Color::Indigo,
    'gray' => Color::Slate,
  ])
````
#### Create Filament resource

```sh
php artisan make:filament-resource Conference --generate
```
Note that if you use `--generate` then the table and form will be automatically filled.

### 2-2 Basic form inputs

#### Set up hot reloading

First go to `vite.config.js`, import `refreshPaths` and change `refresh` as below.

```php
import laravel, {refreshPaths } from 'laravel-vite-plugin';
...
 refresh:[
  ... refreshPaths,
  'app/Livewire/**',
  'app/Filament/**',
 ],
``` 
Second, add `register` method in `AdminPanelProvider.php`.

```php
public function register(): void
{
  parent::register();
  FilamentView::registerRenderHook(name: 'panels::body.end', hook: fn(): string => Blade::render("@vite('resources/js/app.js')"));
} 
```

```sh
npm install
npm run dev
```

#### Form fields

Add `->required()` or `->requiredIf()` to make the field required. Remove required field's star by adding `->markAsRequired(condition: false)`.

`->maxLength(60)` is the same as `->rules(rules: ['max:60',])`.

One can add helper text or hint: `->helpeText(text: 'The name of the conference.')`, `->hint(hint:'name of the conference')`, `->hintIcon(icon: 'heroicon-****')`, `->hintAction('learn more', fn() => 'https://www.google.com')`.

Add prefix and/or suffix: `->url()->prefix(label: 'https://')` ('https://' will be displayed inside the input field), `->prefixIcon(iconName:'heroicon-o-globe-alt')`, `->suffix(label:'.com')`.

Add default value: `->default(state: 'My Conference')` (only applies to create page, not edit page).

RichText/MarkdownEditor toolbar customization: `->disableToolbarButtons( buttonsToDisable: ['italic'] )`, `->toolbarButtons(buttons: ['h2', 'bold'])`.

Disable rowser native `DatePicker` and `DateTimePicker`: `->native(false)`.

Use `foreignIdFor`: `$table->foreignIdFor(model:Venue::class)->nullable();`

### 2-3 Select input

#### Create an Enum 

```php
namespace App\Enums; // create a Enums directory in app directory

enum Region: string
{
 case US = 'US';
 case EU = 'EU';
 case Online = 'Online';
}
```
Add casts to Venue model:

```php
protected $casts = [
 'id' => 'integer',
 'region' => Region::class,
];
```

Use the enum in `Forms\Components\Select`:

```php
Forms\Components\Select::make(name:'region')
 ->enum(enum:Region::class) // validate and only accept those values
 ->options(options:Region::class), // give options in enum
```

#### Install ray for debug

```sh
composer require spatie/ray
```
#### Move `form` and `table` definition to Model

```php
 return $form
  ->schema(componets: Venue::getForm());
```


#### Limit Venue options by Region

All form components are livewire models. By default, live upate is disabled. Add `->live()` to region Select to enable live update which will update the server every time a different select is made, and  cause rerender of the whole livewire component.

```php
Forms\Components\Select::make(name:'region')
 ->live() // enable live update for this model. 
 ->enum(enum:Region::class) // validate and only accept those values
 ->options(options:Region::class), // give options in enum
Forms\Components\Select::make('venue_id') // got rerendered every time a different region is selected.
 ->relationship('venue', 'name', modifyQueryUsing: function(Builder $query, Forms\Get $get) {
  ray(...args: $get(path: 'region')); // displays every time
  return $query->where('region', $get(path: 'region'));
})
 ->searchable()
 ->preload() // if you have a small number of data
 ->editOptionForm(schema: Venue::getForm()) // an edit button will appear on the right! a modal will pop up for editing.
 ->createOptionForm(schema: Venue::getForm()) // popup modal for create
```

### 2-4 Checkbox List

Add a json column to the `speakers` table. And add casts to `Speaker` model.

```php
$table->json(column: 'qualifications');

protected $casts = [
 'id' => 'integer',
 'qualifications' => 'array',
];
```

Add checkbox list to the form.

```php
CheckboxList::make('qualifications')
 ->columnSpanFull()
 ->searchable()
 ->bulkToggleable()
 ->options([
  'business-leader' => 'Business Leader',
  'first-time' => 'First Time Speaker',
 ])
 ->descriptions([
  'business-leader' => 'Who is a business leader',
  'first-time' => 'First time to speak',
 ])
 ->columns(3),
```

### 2-5 Layouts

#### Organize the form into Sections, Fieldset

```php
Section::make(heading: 'Conference Details')
  ->aside()
  ->collapsible()
  ->description('blah blah....')
  ->icon('heroicon-o-...')
  ->columns(2) // section has two columns
  ->schema(components: [
    TextInput::make('name')
      ->columnSpanFull(), // or columnSpan(2)
    DateTimePicker::make('start_date') // by default will take up one colum span
      ->native(false),
    DateTimePicker::make('end_date'),  // one column span

    Fieldset::make('Status')
      ->columns(2)
      ->schema([
        Toggle::make('is_published')
          ->default(true),
        Select::make('status')
          ->options([
            'draft' => 'Draft',
            'published' => 'Published',
            'archived' => 'Archived',
          ])
          ->required(),
      ]),
  ])

#### Organize using tabs


```php
Tabs::make()
->columnSpanFull()
->tabs([
  Tabs\Tab::make(label:'Conference Details')
  ->schema([
    
  ]),
  Tabs\Tab::make(label:'Location')
  ->schema([
    
  ]),
])
```

There is also a Wizard that can fill up form step by step.










## 4. Other Filament Packages

### 4-1. Infolist

#### `slideOver()` instead of Edit Page

```php
protected function getHeaderActions(): Array
{
  return [
    Actions\EditAction::make()
      ->slideOver()
      ->form(Speaker::getForm())
  ];
}
```

#### Display something not in the database

```php
TextEntry::make('has_spoken')
  ->getStateUsing(function($record) {
      return $record->talks()->where('status', TalkStatus::APPROVED)->count() > 0 ? 'Previous Speaker' : 'Has Not Spoken';
    })
  ->badge()
  ->color(function($state) {
    if($state === 'Previous Speaker') {
      return 'success';
    }
    return 'primary';
  }),

```

#### Display `RichText` in `TextEntry`

```php
TextEntry::make('bio')
  ->extraAttributes(['class' => 'prose dark:prose-invert'])
  ->html(),
```

