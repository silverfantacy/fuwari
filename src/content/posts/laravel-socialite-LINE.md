---
title: 整合 Laravel Socialite 的 LINE 登入功能
published: 2024-04-20
description: 紀錄如何使用 Laravel Socialite 整合 LINE 登入功能。步驟包括安裝 Laravel Socialite、安裝 Socialite Providers 套件、設定 LINE 整合資料、修改資料庫結構以及建立處理 LINE 登入和回呼的控制器。
image: 'https://media.virtualxnews.com/2024/04/01cf3fcb99cbcdf924b6746aa8524589.png'
tags: [Laravel, Socialite, LINE]
category: 後端
draft: false
---

紀錄如何使用 Laravel Socialite 整合 LINE 登入功能。步驟包括安裝 Laravel Socialite、安裝 Socialite Providers 套件、設定 LINE 整合資料、修改資料庫結構以及建立處理 LINE 登入和回呼的控制器。

## 安裝 Laravel Socialite

首先，使用 Composer 安裝 Laravel Socialite 套件：

```bash
composer require laravel/socialite
```

安裝的版本可能會根據你的 Laravel 版本而有所不同。以下是一個範例的 `composer.json` 檔案，包含了 Laravel Socialite 和其他相依套件的版本資訊：

```json
"require": {
    "php": "^8.2",
    "ext-pdo_mysql": "*",
    "facebook/php-business-sdk": "^13.0",
    "firebase/php-jwt": "^6.9",
    "guzzlehttp/guzzle": "^7.2",
    "laravel/framework": "^10.10",
    "laravel/sanctum": "^3.2",
    "laravel/socialite": "^5.5",
    "laravel/tinker": "^2.8",
    "livewire/livewire": "^3.3",
    "nesbot/carbon": "^2.66",
    "revolution/laravel-google-sheets": "^6.0",
    "socialiteproviders/line": "^4.1"
},
```

接著，打開 `config/app.php` 檔案，並在 `providers` 部分加入以下程式碼：

```php
Laravel\Socialite\SocialiteServiceProvider::class,
```

然後，在 `aliases` 部分加入以下程式碼：

```php
'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

## 安裝 Socialite Providers 套件

由於 Laravel Socialite 沒有提供 LINE driver，需要使用 Socialite Providers 套件來整合 LINE 登入功能。首先，根據要整合的社群帳號選擇安裝相應的套件。以 LINE 為例，執行以下指令：

```bash
composer require socialiteproviders/line
```

接著，打開 `config/app.php` 檔案，將 `Laravel\Socialite\SocialiteServiceProvider::class` 從 `providers` 陣列中移除或註解掉，並加入 `SocialiteProviders\Manager\ServiceProvider::class`：

```php
// 'providers' 部分
// Laravel\Socialite\SocialiteServiceProvider::class,

// Socialite Providers
SocialiteProviders\Manager\ServiceProvider::class,
```

然後，打開 `app/Providers/EventServiceProvider.php` 檔案，在 `listen` 陣列中加入以下程式碼：

```php
// 'listen' 部分
\SocialiteProviders\Manager\SocialiteWasCalled::class => [
    // LINE 帳號登入
    'SocialiteProviders\\Line\\LineExtendSocialite@handle',
],
```

這樣就完成了 Socialite Providers 套件的安裝和配置。

## 設定 LINE 整合資料

在進行 LINE 登入整合之前，我們需要在 `.env` 檔案中加入 LINE 相關的設定資料。請在 `.env` 檔案中加入以下程式碼：

```plaintext
LINE_CLIENT_ID=YOUR_LINE_CLIENT_ID
LINE_CLIENT_SECRET=YOUR_LINE_CLIENT_SECRET
LINE_REDIRECT_URI=https://your-domain.com/auth/line/callback
```

請將 `YOUR_LINE_CLIENT_ID` 和 `YOUR_LINE_CLIENT_SECRET` 替換為你的 LINE 應用程式的 Client ID 和 Client Secret。同時，將 `https://your-domain.com/auth/line/callback` 替換為你應用程式的回呼 URL。

註：LINE 應用程式的 Client ID 和 Client Secret 可以在 [LINE Developers](https://developers.line.biz/en/) 中註冊並取得。
![LINE login 註冊](https://media.virtualxnews.com/2024/04/d53c62a008b7214608f1df81ab100ac5.png)

接著，打開 `config/services.php` 檔案，並在 `providers` 部分加入以下程式碼：

```php
'line' => [
    'client_id' => env('LINE_CLIENT_ID'),
    'client_secret' => env('LINE_CLIENT_SECRET'),
    'redirect' => env('LINE_REDIRECT_URI')
],
```

這樣就完成了 LINE 整合資料的設定。

## 更新配置快取

在設定完成後，執行以下指令更新配置快取：

```bash
php artisan config:cache
```

## 修改資料庫結構

為了儲存 LINE 登入相關的資料，我們需要修改資料庫結構。首先，建立一個 migration 檔案，並在使用者資料表中加入 LINE 帳號的定義欄位。請執行以下指令：

```bash
php artisan make:migration add_line_id_to_users_table --table=users
```

這將建立一個名為 `add_line_id_to_users_table` 的 migration 檔案。請打開這個檔案，並將下面的程式碼加入到 `up` 方法中：

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddLineIdToUserTable extends Migration
{
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            //加入 LINE ID 欄位到 google_account欄位之後
            $table->string('line_id', 64)->nullable()->after('google_account');
            //建立索引
            $table->index(['line_id'], 'user_l_idx');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            //移除欄位
            $table->dropColumn('line_id');
        });
    }
}
```

這個 migration 檔案會在 `users` 資料表中加入 `line_id` 欄位，並且建立一個索引。

接著，執行 migration 指令來更新資料庫結構：

```bash
php artisan migrate
```

這樣就完成了資料庫結構的修改。

## 建立控制器

最後，我們需要建立一個控制器來處理 LINE 登入相關的操作。如果還沒有建立控制器，請先使用以下指令建立一個控制器：

```bash
php artisan make:controller LineAuthController
```

接著，在 `LineAuthController` 控制器中，建立兩個方法：`lineLogin` 和 `lineLoginCallback`。請在 `lineLogin` 方法中將使用者重新導向至 LINE 帳戶登入頁面，並在 `lineLoginCallback` 方法中處理 LINE 回傳的資料。以下是一個簡單的範例：

```php
use Socialite;

class LineAuthController extends Controller
{
    public function lineLogin()
    {
        return Socialite::driver('line')->redirect();
    }

    public function lineLoginCallback()
    {
        $user = Socialite::driver('line')->user();

        dd($user);
    }
}
```

如果要直接取得 LINE 登入的 URL，可以使用以下程式碼：

```php
public function getLineLoginUrl()
{
    return Socialite::driver('line')->stateless()->redirect()->getTargetUrl();
}
```

上面的程式碼只是一個簡單的範例，需要根據自己的需求進行修改。

這樣就完成了 LINE 登入功能的整合。你可以根據自己的需求進一步擴充這個功能，例如將 LINE 帳號與使用者資料進行綁定，或是儲存 LINE 登入相關的資料到資料庫中。

> 參考資料：
> 
> [Laravel Socialite](https://laravel.com/docs/11.x/socialite)
> 
> [用 Laravel Socialite 導入社群帳號登入](https://abo.tw/articles/laravel/user-laravel-socialite-for-social-login)
> 
> [LINE 文件](https://developers.line.biz/en/docs/line-login/overview/)
