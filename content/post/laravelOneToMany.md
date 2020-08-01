---
title: "Laravel one-to-many Relationship"
summary: "Laravel 一對多關聯的學習紀錄。"
featured_image: "/images/laravel.jpeg"
tags: [學習筆記,PHP]
date: 2020-07-26T18:34:02+08:00
toc: true
disable_share: true
---

當一筆資料同時關聯到另一張表的許多資料時，我們通常會用一對多來操作，比如說一個用戶 (user) 會有很多則貼文 (posts) ，我會盡可能的從創建到使用都記錄下來，希望可以幫到同樣使用這個框架的 ~~未來的自己~~ 你。

## Environment
- PHP 7.3.18
- Laravel 6.18.20

## Migration

{{< inlineCode >}}php artisan make:migration create_users_table{{< /inlineCode >}}

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('name');
    $table->timestamps();
}
```

{{< inlineCode >}}php artisan make:migration create_posts_table{{< /inlineCode >}}

```php
Schema::create('posts', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->bigInteger('user_id')->unsigned();
    $table->text->content();
    $table->timestamps();
    
    $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
}
```

## Models

{{< inlineCode >}}php artisan make:model Models/User{{< /inlineCode >}}

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'users';

    protected $guarded = [];
    
    public function posts()
    {
        return $this->hasMany('App\Models\Post');
    }    
}
```

{{< inlineCode >}}php artisan make:model Models/Post{{< /inlineCode >}}

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $table = 'posts';

    protected $guarded = [];

    public function user()
    {
        return $this->belongsTo('App\Models\User');
    }
}
```

## How to use
我通常會在 Controller 和 Model 之間多一層 Repository 來操作和整理資料，讓 Model 盡可能的只有對 DB 的設定 (例如可填充或關聯等)，讓 Controller 盡可能的只做流程控制。

所以我會在 Repository 裡面注入 Model：

### Get
{{< inlineCode >}}UserRepository.php{{< /inlineCode >}}
```php
namespace App\Repositories;

use App\Models\User;
use App\Models\Post;

class UserRepository
{
    protected $_user;

    public function __construct(User $user)
    {
        $this->_user = $user;
    }
    
    public function getPosts($id)
    {
        $user = $this->_user::find($id);
        
        return $user->posts;
    }
}
```

{{< inlineCode >}}PostRepository.php{{< /inlineCode >}}
```php
namespace App\Repositories;

use App\Models\Post;

class PostRepository
{
    protected $_post;
    
    public function __construct(Post $post)
    {
        $this->_post = $post;
    }
    
    public function getUser($id)
    {
        $post = $this->_post::find($id);
        
        return $post->user;
    }
}
```

### Associate
```php
function associate($user_id, $post_id)
{
    $user = $_user::find($user_id);

    // 如果有注入 Post Model 的話
    $post = $_post::find($post_id);
    
    $user->posts()->associate($post)->save;
    // or
    $post->user()->associate($user)->save;
}
```

### Dissociate
```php
function dissociate($user_id, $post_id)
{
    $user = $_user::find($user_id);

    // 如果有注入 Post Model 的話
    $post = $_post::find($post_id);

    $user->posts()->dissociate();
}
```

### Create
```php
function createPostWithCreate($user_id)
{
    $user = $_user::find($user_id);

    $user->posts()->create([
        'content' => 'blablabla...'
    ]);
    
    // ...
}
```

```php
function createPostWithCreateMany($user_id, $post_id)
{
    $user = $_user::find($user_id);
    
    $user->posts()->createMany([
        ['content' => 'blablabla...'],
        ['content' => 'Another blablabla...'],
    ]);

    // ...
}
```

```php
function createPostWithSave($user_id)
{
    $post = new App\Models\Post(['content' => 'blablabla...']);

    $user = $_user::find($user_id);

    $user->posts()->save($post);
}
```

```php
function createPostWithSaveMany($user_id, $post_id)
{
    $user = $_user::find($user_id);
    
    $user->posts()->saveMany([
        new App\Models\Post(['content' => 'blablabla...']),
        new App\Models\Post(['content' => 'Another blablabla...']),
    ]);

    // ...
}
```

{{< inlineCode >}}create{{< /inlineCode >}} 和 {{< inlineCode >}}save{{< /inlineCode >}} 的差別在於 {{< inlineCode >}}create{{< /inlineCode >}} 接收的參數是 PHP 的陣列，而 {{< inlineCode >}}save{{< /inlineCode >}} 接收的是一個 Model 物件。

從 Laravel 的 source code 可以看到：
{{< inlineCode >}}vendor/laravel/framework/src/Illuminate/Database/Eloquent/Builder.php{{< /inlineCode >}}
```php
/**
 * Save a new model and return the instance.
 *
 * @param  array  $attributes
 * @return \Illuminate\Database\Eloquent\Model|$this
 */
public function create(array $attributes = [])
{
    return tap($this->newModelInstance($attributes), function ($instance) {
        $instance->save();
    });
}
```

{{< inlineCode >}}tap{{< /inlineCode >}} 是 Laravel 的 Helper Function：
{{< inlineCode >}}vendor/laravel/framework/src/Illuminate/Support/helpers.php{{< /inlineCode >}}
```php
/**
 * Call the given Closure with the given value then return the value.
 *
 * @param  mixed  $value
 * @param  callable|null  $callback
 * @return mixed
 */
function tap($value, $callback = null)
{
    if (is_null($callback)) {
        return new HigherOrderTapProxy($value);
    }

    $callback($value);

    return $value;
}
```

接收兩個參數 {{< inlineCode >}}$value{{< /inlineCode >}} 和 {{< inlineCode >}}$callback{{< /inlineCode >}}，並將 {{< inlineCode >}}$value{{< /inlineCode >}} 放進 {{< inlineCode >}}$callback{{< /inlineCode >}} 後返回。

所以我們回到 create 就可以看出，它其實只是在裡面幫你把 new 物件這件事做掉而已。

### Delete

這會把 user 所有的 posts 都刪掉
```php
public function delete($user_id)
{
    $user = $_user::find($user_id);

    $user->posts()->delete();

    // ...
}
```

## 參考資料
1. [Laravel 官方文件](https://laravel.com/docs/6.x/eloquent-relationships)