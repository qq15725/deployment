基于 laravel5.5 , 记录如何构建部署自用常用包.

## 使用的包

- [dingo/api](https://github.com/dingo/api) 一个 RESTful API 包
- [tymon/jwt-auth](https://github.com/tymondesigns/jwt-auth) JSON Web Token 身份验证包
- [zizaco/entrust](https://github.com/Zizaco/entrust) 角色权限包

自造
    
- [wxm/ddoc](https://github.com/qq15725/ddoc) 提供接口文档, 数据库字典文档, 网页呈现.

## 安装

1. 截止更新日期, 修复 bug 及兼容 laravel5.5, 添加自定义 `repositories` 地址.

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
    ],
    ```

2. `composer.json` 添加 `require` , 执行 `composer update` .
    
    ```php
    "require": {
        "wxm/ddoc": "1.0.*@dev",
        "zizaco/entrust": "v0.5.0beta",
        "dingo/api": "v1.0.0-beta10",
        "tymon/jwt-auth": "1.0.*@dev"
    },
    ```
    
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
    
## 明天再写