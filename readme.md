基于 laravel5.5 , 记录如何构建部署自用常用包.

## 创建 laravel5.5 项目

```shell
composer create-project --prefer-dist laravel/laravel laravel5_5 "5.5.*"
```

## 使用的包

- [dingo/api](https://github.com/dingo/api) `一个 RESTful API 包.`
- [tymon/jwt-auth](https://github.com/tymondesigns/jwt-auth) `JSON Web Token 身份验证包.`
- [zizaco/entrust](https://github.com/Zizaco/entrust) `角色权限包.`

自造
    
- [wxm/ddoc](https://github.com/qq15725/ddoc) `提供接口文档, 数据库字典文档, 网页呈现.`

## 安装及部署

1. 截止更新日期, 修复 bug 及兼容 laravel5.5, `composer.json` 需要添加自定义 `repositories` 地址.

    ```php
     "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/qq15725/api"
        },
        {
            "type": "vcs",
            "url": "https://github.com/qq15725/blueprint"
        },
        {
            "type": "vcs",
            "url": "https://github.com/qq15725/entrust"
        }
    ]
    ```

2. `composer.json` 添加 `require` , 执行 `composer update` .
    
    ```php
    "require": {
        "wxm/ddoc": "1.0.*@dev",
        "zizaco/entrust": "v0.5.0beta",
        "dingo/api": "v1.0.0-beta10",
        "tymon/jwt-auth": "1.0.*@dev"
    }
    ```
    
    > composer update 途中可能需要提供 github 个人私钥.
    
3. 发布资源

    发布 `Zizaco\Entrust` 配置
    ```shell
    php artisan vendor:publish --provider="Zizaco\Entrust\EntrustServiceProvider"
    ```

    发布 `Dingo\Api` 配置
    ```shell
    php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
    ```

    发布 `Wxm\DDoc` 配置
    ```shell
    php artisan vendor:publish --provider="Wxm\DDoc\DDocServiceProvider"
    ```
    
4. 执行数据库迁移

    执行 `Zizaco\Entrust` 数据库迁移
    ```shell
    php artisan entrust:migration
    ```
    
## 创建 Models

1. Role

    ```php
    <?php namespace App;
    
    use Zizaco\Entrust\EntrustRole;
    
    class Role extends EntrustRole
    {
     
    }
    ```
    
2. Permission
    
    ```php
    <?php namespace App;
    
    use Zizaco\Entrust\EntrustPermission;
    
    class Permission extends EntrustPermission
    {
     
    }
    ```
    
3. User

    结合 jwt 验证和角色权限的用户模型例子:  

    ```php
    <?php
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Tymon\JWTAuth\Contracts\JWTSubject;
    use Zizaco\Entrust\Traits\EntrustUserTrait;
    
    class User extends Authenticatable implements JWTSubject
    {
        use Notifiable;
        use EntrustUserTrait; // add this trait to your user model
    
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = [
            'name', 'email', 'password',
        ];
    
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = [
            'password', 'remember_token',
        ];
    
        /**
         * Get the identifier that will be stored in the subject claim of the JWT.
         *
         * @return mixed
         */
        public function getJWTIdentifier()
        {
            return $this->getKey();
        }
    
        /**
         * Return a key value array, containing any custom claims to be added to the JWT.
         *
         * @return array
         */
        public function getJWTCustomClaims()
        {
            return [];
        }
    }
    ```