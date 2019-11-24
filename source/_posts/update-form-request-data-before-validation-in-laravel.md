---
title: Updating Form Request Data Before Validation in Laravel
date: 2019-11-24 22:03:20
tags:
- Validation
- Laravel
- Form Request
- Rules
- Request Data

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1574634761/posts/naples-monument.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1574634761/posts/naples-monument.jpg
thumbnailImagePosition: right
---
### Introduction 
Laravel's Form Requests are a great way of removing validation logic from your controllers. 

There are times were it can be useful to update or change the request data before it is passed to the validator for example formatting postcodes, removing invalid characters or providing default values to data.

The official [documentation](https://laravel.com/docs/master/validation#form-request-validation) shows how we can perform additional logic after the rule sets have been run but not before hand.

Digging through the Form Request [api](https://laravel.com/api/master/Illuminate/Foundation/Http/FormRequest.html#method_prepareForValidation) there is mention of a method called `prepareForValidation` which is an empty method that is called before the actual validation rules are run as can be seen in implementation:

![the prepare for validation method](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1574425624/posts/prepare-for-validation.png)

<div style="text-align: center">[Heres a link to the above code](https://github.com/laravel/framework/blob/master/src/Illuminate/Validation/ValidatesWhenResolvedTrait.php#L37)
</div>

So given that the method is empty how do we use it and go about updating the form request data?

<!-- more -->

### Solution and example of implementation

The `merge` method on the same class is how.

Here's an example of something I was working on recently. 

We had built a Laravel app that is accessed by a React Native application. Our client realised post launch that they needed to only run certain validation against driving licences from Great Britain only. They said that almost all of their clients have GB licences but they still need to be able to accommodate others.

So we amended the validation rules for this particular request to accommodate this new indicator:
```php
// Form request class...
    public function rules()
    {
        return [
            ...
            'gb_licence' => ['required', 'bool'],
        ];
    }
```

However it takes time apps to get reviewed and published into their respective app stores and even then being in the app store doesn't ensure that all clients are using the latest version of that app.

So rolling this code out would break our existing clients since they would fail the `required` rule. 

We could have versioned off the API so that old versions of the app call the old code but newer build call a new route with this new validation, however for such a minor change it felt like overkill. Also it would still mean we'd have to create a default value for older clients otherwise we'd end up with not null errors with our database.

So for the following reasons we opted to create a default value for this `gb_licence` indicator. 

- We could continue to use the existing API (didn't require the API to be versioned)
- We could roll the code immediately without impacting existing clients
- It would work as soon as the apps in either store were released

And this is all it took:
```php
// Form request class...
    protected function prepareForValidation(): void
    {
        $this->merge([
            'gb_licence' => $this->gb_licence ?? true,
        ]);
    }
```

So in this instance if the `gb_licence` field didn't exist it would be created and given a default value of `true`.

Super easy!

I've used the `prepareForValidation` in a number of different FormRequests when needing to pre-format incoming data. A common example being uppercasing and formatting postcodes and vehicle registration numbers:

```php
// Form request class...
    protected function prepareForValidation(): void
    {
        $this->merge([
            'postcode' => mb_strtoupper($this->postcode),
            'reg_no' => mb_strtoupper($this->reg_no),
        ]);
    }
```

*I actually use a number of custom helper functions to actually do this but the above should illustrate how you can amend existing data.*


