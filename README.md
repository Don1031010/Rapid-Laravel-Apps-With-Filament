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

