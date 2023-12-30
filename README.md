# Rapid-Laravel-Apps-With-Filament

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

````php
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

