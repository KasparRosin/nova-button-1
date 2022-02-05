## Nova Button

[![Latest Version on Github](https://img.shields.io/github/release/dillingham/nova-button.svg?style=flat-square)](https://packagist.org/packages/dillingham/nova-button)
[![Total Downloads](https://img.shields.io/packagist/dt/dillingham/nova-button.svg?style=flat-square)](https://packagist.org/packages/dillingham/nova-button) [![Twitter Follow](https://img.shields.io/twitter/follow/im_brian_d?color=%231da1f1&label=Twitter&logo=%231da1f1&logoColor=%231da1f1&style=flat-square)](https://twitter.com/im_brian_d)

Nova package for rendering buttons on index, detail and lens views.

Use buttons to trigger backend events, navigate nova routes or visit links.

![example-users](https://user-images.githubusercontent.com/57711725/152637226-e7047831-b726-4940-95c9-617db4d42de4.png)

> ℹ️ This package is a continuation of [dillingham/nova-button](https://github.com/dillingham/nova-button) and has been created
> since it seemed to be abandoned.

### Requirements

This package requires the following minimum versions:

| What    | Minimum |
|---------|---------|
| PHP     | 7.4     |
| Laravel | 7.0     |
| Nova    | 3.0     |

### Installation

You can install this package by running the following command:

```bash
composer require dnwjn/nova-button
```

### Usage

```php
use NovaButton\Button;
```
```php
public function fields(Request $request)
{
    return [
        ID::make('ID', 'id')->sortable(),
        Text::make('Name', 'name'),
        Button::make('Notify'),
    ];
}
```

Quick links: [Button Styles](https://github.com/dnwjn/nova-button#button-styles) | [Event text / style](https://github.com/dnwjn/nova-button#button-state) | [Navigation](https://github.com/dnwjn/nova-button#button-navigation) | [CSS classes](https://github.com/dnwjn/nova-button#button-classes) | [Lens example](https://github.com/dnwjn/nova-button#example)

---

### Backend events

By default, clicking the button will trigger a backend event via ajax.

Default event: `NovaButton\Events\ButtonClick`

The event will receive the resource model it was triggered from & the key

- `$event->resource` = `model`
- `$event->key` = `"notify"`

Adding a custom key

```php
Button::make('Notify', 'notify-some-user')
```
Adding a custom event
```php
Button::make('Notify')->event('App\Events\NotifyRequested')
```

You register listeners in your EventServiceProvider

### Nova Routes

You can also choose to navigate any of the Nova routes

```php
Button::make('Text')->route('vuejs-route-name', ['id' => 1])
Button::make('Text')->index('App\Nova\User')
Button::make('Text')->detail('App\Nova\User', $this->user_id)
Button::make('Text')->create('App\Nova\User')
Button::make('Text')->edit('App\Nova\User', $this->user_id)
Button::make('Text')->lens('App\Nova\User', 'users-without-confirmation')
```
You can also enable a resource's filters 
```php
Button::make('Text')->index('App\Nova\Order')->withFilters([
    'App\Nova\Filters\UserOrders' => $this->user_id,
    'App\Nova\Filters\OrderStatus' => 'active',
])
```

### Links
```php
Button::make('Text')->link('https://nova.laravel.com')
Button::make('Text')->link('https://nova.laravel.com', '_self')
```

### Visiblity

You will likely want to show or hide buttons depending on model values
```php
Button::make('Activate')->visible($this->is_active == false),
Button::make('Deactivate')->visible($this->is_active == true),
```

Also [field authorization](https://nova.laravel.com/docs/1.0/resources/authorization.html#fields) via canSee() & [showing / hiding fields](https://nova.laravel.com/docs/1.0/resources/fields.html#showing-hiding-fields) hideFromIndex(), etc

### Reload
After events are triggered, reload the page. 

```php
Button::make('Notify')->reload()
```
If you click many buttons, reloading will wait for all buttons to finish.

If an error occurs, it will not reload the page.


### Confirm
You can require a confirmation for descructive actions

```php
Button::make('Cancel Account')->confirm('Are you sure?'),
Button::make('Cancel Account')->confirm('title', 'content'),
```

### Button state
When using events, you want visual feedback for the end user.

This is especially useful for long running listeners.

```php
Button::make('Remind User')->loadingText('Sending..')->successText('Sent!')
```

| Event | Text | Style |
| -- | -- | -- |
| loading | `loadingText('Loading..')` | `loadingStyle('grey-outline')` |
| success | `successText('Done!')` | `successStyle('success')` |
| error | `errorText('Failed')` | `errorStyle('danger')` |

Defaults defined in the `nova-button` config. Add methods when you want to change for specific resources


### Button styles

This package makes use of [tailwind-css](https://tailwindcss.com) classes / default: `link`

```php
Button::make('Confirm')->style('primary')
```

| Fill  | Outline | Link |
|---|---|---|
| primary | primary-outline | primary-link |
| success | success-outline | success-link |
| danger | danger-outline | danger-link |
| warning | warning-outline | warning-link |
| info | info-outline | info-link |
| grey | grey-outline | grey-link |

Each key adds classes from the `nova-button` config
```php
'primary' => 'btn btn-default btn-primary'
```

### Style config
Publish the nova-button config to add / edit [available styles & defaults](https://github.com/dnwjn/nova-button/blob/master/config/nova-button.php) 
```
php artisan vendor:publish --tag=nova-button -- force
```

### Button classes

You can also add classes manually

```php
Button::make('Refund')->classes('some-class')
```
Also able to style the following css classes

```css
.nova-button
.nova-button-{resource-name}
.nova-button-success
.nova-button-error
.nova-button-loading
```

---

# Example

Use [lenses](https://nova.laravel.com/docs/1.0/lenses/defining-lenses.html) with buttons for a very focused user experience 

![example-lenses](https://user-images.githubusercontent.com/57711725/152637243-ebd753c2-5eda-4749-b8ba-c98ceb162e5b.png)

```php
<?php

namespace App\Nova\Lenses;

class UsersWithoutConfirmation extends Lens
{
    public static function query(LensRequest $request, $query)
    {
        return $query
            ->select(['users.id', 'users.name'])
            ->whereNull('email_verified_at');
    }

    public function fields(Request $request)
    {
        return [
            ID::make('ID', 'id'),
            Text::make('Name', 'name'),
            Button::make('Mark As Confirmed'),
        ];
    }
}
```
Register a listener for `\NovaButton\Events\ButtonClick` in your [EventServiceProvider](https://laravel.com/docs/5.7/events)
```php
<?php

namespace App\Listeners;

class ConfirmUser
{
    public function handle($event)
    {
        if ($event->key == 'mark-as-confirmed') {
            $event->resource->email_verified_at = now();
            $event->resource->save();
        }
    }
}
```
No `key` check required when you register an event for this listener

```php
Button::make('Confirm')->event('App\Events\ConfirmClick')
```

# Telescope inspection

![example-telescope](https://user-images.githubusercontent.com/57711725/152637248-4bf65fa8-a270-48b9-aff3-08e6193eab6c.png)
