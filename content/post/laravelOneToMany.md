---
title: "Laravel one-to-many Relationship"
summary: "Laravel 一對多關聯。"
featured_image: "/images/laravel.jpeg"
tags: [學習筆記,PHP]
date: 2020-07-26T18:34:02+08:00
toc: true
disable_share: true
---

當一筆資料同時關聯到另一張表的許多資料時，我們通常會用一對多來操作，比如說一個用戶 (user) 會有很多則貼文 (post) ，我會盡可能的從創建到使用都記錄下來，希望可以幫到同樣使用這個框架的 ~~未來的自己~~ 你。

## Environment
- PHP 7.3.18
- Laravel 6.18.20

## Migration

{{< inlineCode >}}php artisan make:migration create_users_table{{< /inlineCode >}}

```php=
Schema::create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('name');
    $table->timestamps();
}
```

{{< inlineCode >}}php artisan make:migration create_posts_table{{< /inlineCode >}}

```php=
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

```php=
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

```php=
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
UserRepository.php
```php=
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

PostRepository.php
```php=
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

### Create
```php=
function createPost($user)
{
    $user->posts()->create([
        'content' => 'blablabla...'
    ]);
    
    return;
}
```

### Associate
```php=
function associate($user_id, $post_id)
{
    $user = $_user::find($user_id);

    // 如果有注入 Post Model 的話
    $post = $_post::find($post_id);
    
    $post->user()->associate($user)->save;
}
```