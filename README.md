# Rapid-Laravel-Apps-With-Filament

## 2. Filament Forms

### Install Filament

```sh
composer require filament/filament:"^3.1" -W
 
php artisan filament:install --panels
```
Note both livewire and Alpine are also installed.

### Change colors

Inside `AdminPanelProvider.php`, you can easily change the colors of the app.

````php
return $panel
  ->colors(colors:[
    'primary' => Color::Indigo,
    'gray' => Color::Slate,
  ])
````
### Create Filament resource

```sh
php artisan make:filament-resource Conference --generate
```
Note that if you use `--generate` then the table and form will be automatically filled.


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

