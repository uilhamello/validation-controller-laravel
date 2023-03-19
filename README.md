In this tutorial we are going talk a good way about request validation.

This tutorial is based in the tutorial "crud-api-by-laravel" which has a basic CRUD API Structure https://github.com/uilhamello/crud-api-by-laravel


Let's get started! :star_struck:


<h2>Validation</h2>

contextualizing: It is common to see datas from request beein validated at controller file. It is a pratical way to validate datas before input them into database. 
But there is a cleaner way to do that, abstrating the validation from controller in a specific class, and leaves controller clean just with its bussiness. So that is what we are going to create, a separete sublayer just for validate request datas. A great Single Responsibility Principle

<h3>1° Step - Creating a StoreRequest file</h3>


```php
php arisan make:request ProductStoreRequest
```

The above command has create a file app/Http/Requests/ProductStoreRequest.php:

Great! We now have our validate file. Let's try to use it with our ProductController.

At the Store method, replace de parameter type Request to ProductStoreRequest.

```php
    public function store(ProductStoreRequest $request)
    {
        return Product::create($request->post());
    }
```
If you try to store, it is going to show you the error message '403 THIS ACTION IS UNAUTHORIZED.'. Great! It is working haha 

Now last open the app/Http/Requests/ProductStoreFile.php

The autorize() method is determing if the user can makes the request. It is returning false, and that is why we are now getting the message '404 UNAUTHORIZED'

Let's change it to return 'true' like the below:

```php
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }
```

End now we have the route product@store working as usual.

<h4>Validating request datas </h4>

To route store we are specting name and quantity data from request.
We are going to put that both is required and de name can not be menor then 3 caracters and the quantity has to be a integer data. It is configure into the rules() method:

```php
    public function rules()
    {
        return [
            'name' => 'required|min:4',
            'quantity' => 'required|integer|max:2000'
        ];
    }

```

In ProductController let's change Request type-hints on store method parameter for our ProductStoreRequest

```php
    public function store(ProductStoreRequest $request)
    {
        return Product::create($request->post());
    }
```

So if you try to request product@store with a data request that doesn't match the validations rules you gonna be automaticly redirect. Other hand it is going to work as usual.

As we are building on API project we do not need a redirect, so let's change the action to just show a json validate error message.

On ProductStoreRequest include to use befor FormRequest, like this:

```php
+  use Illuminate\Contracts\Validation\Validator;
+  use Illuminate\Http\Exceptions\HttpResponseException;
   use Illuminate\Foundation\Http\FormRequest;
```

Then insert this method on ProductStoreRequest class:


```php
    protected function failedValidation(Validator $validator) {
        throw new HttpResponseException(
                response()->json($validator->errors(), 422)
            );
    }
```
This is going to handle any validate fail and throw a validate message.


Now lets configure our message validation. Let's create a method on ProductStoreRequest called messages(). Need to be that name.

```php
    protected function messages() {
        return array[
            'name.required' => ' The name is required',
            'quantity.required' => 'The quantity is required',
        ];
    }
```

We have a personal error validation messages.


That is Done! :star_struck:


