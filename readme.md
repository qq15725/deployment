基于 Laravel5.5 部署一套 RESTful API 高效开发模板  

## 创建 laravel5.5 项目

```shell
composer create-project --prefer-dist laravel/laravel laravel5_5 "5.5.*"
```

## 推荐使用的包

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

    获得 `Zizaco\Entrust` 需要的迁移文件
    ```shell
    php artisan entrust:migration
    ```
    
    执行数据库迁移
    ```shell
    php artisan migration
    ```
    
5. 生成 JWT 所需要的密钥
    
    ```shell
    php artisan jwt:secret
    ```
    
6. 为 dingo/api 添加 `.env` 配置

    例子:
     
    ```
    API_STANDARDS_TREE=vnd
    API_SUBTYPE=emall
    API_PREFIX=api
    API_VERSION=v1
    ```
    
## 创建 Models

1. Role

    ```php
    <?php 
    namespace App;
    
    use Zizaco\Entrust\EntrustRole;
    
    class Role extends EntrustRole
    {
     
    }
    ```
    
2. Permission
    
    ```php
    <?php 
    namespace App;
    
    use Zizaco\Entrust\EntrustPermission;
    
    class Permission extends EntrustPermission
    {
     
    }
    ```
    
3. User

    结合 JWT 验证和角色权限的用户模型例子:  

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

## 使用参考

- dingo/api

    `routes/api.php` 下注册 `dingo/api` 路由, 例如:
    
    ```php
    $api = app('Dingo\Api\Routing\Router'); 
    
    $api->version('v1', function ($api) {
        $api->get('/', function () {
            return 'a Dingo\Api examples';
        });
    });
    ```
    
    具体使用详情查看 [https://github.com/dingo/api/wiki](https://github.com/dingo/api/wiki)
    
- zizaco/entrust

    创建角色
    
    ```php
    $owner = new Role();
    $owner->name         = 'owner';
    $owner->display_name = 'Project Owner'; // optional
    $owner->description  = 'User is the owner of a given project'; // optional
    $owner->save();
    
    $admin = new Role();
    $admin->name         = 'admin';
    $admin->display_name = 'User Administrator'; // optional
    $admin->description  = 'User is allowed to manage and edit other users'; // optional
    $admin->save();
    ```
    
    分配角色给用户
    
    ```php
    $user = User::where('username', '=', 'michele')->first();
    
    // role attach alias
    $user->attachRole($admin); // parameter can be an Role object, array, or id
    
    // or eloquent's original technique
    $user->roles()->attach($admin->id); // id only
    ```
    
    创建权限
    
    ```php
    $createPost = new Permission();
    $createPost->name         = 'create-post';
    $createPost->display_name = 'Create Posts'; // optional
    // Allow a user to...
    $createPost->description  = 'create new blog posts'; // optional
    $createPost->save();
    
    $editUser = new Permission();
    $editUser->name         = 'edit-user';
    $editUser->display_name = 'Edit Users'; // optional
    // Allow a user to...
    $editUser->description  = 'edit existing users'; // optional
    $editUser->save();
    
    $admin->attachPermission($createPost);
    // equivalent to $admin->perms()->sync(array($createPost->id));
    
    $owner->attachPermissions(array($createPost, $editUser));
    // equivalent to $owner->perms()->sync(array($createPost->id, $editUser->id));
    ```
    
    检查角色及权限
    
    ```php
    $user->hasRole('owner');   // false
    $user->hasRole('admin');   // true
    $user->can('edit-user');   // false
    $user->can('create-post'); // true
    ```
    
    具体使用详情查看  [https://github.com/Zizaco/entrust](https://github.com/Zizaco/entrust)
    
    
 - JWT 验证登录注册例子:  
 
    Api/Auth/LoginController.php
    
     ```php
     <?php
     
     namespace App\Http\Api\Auth;
     
     use App\User;
     use Illuminate\Foundation\Auth\AuthenticatesUsers;
     use Illuminate\Http\Request;
     use App\Http\Controllers\Controller;
     use Illuminate\Support\Facades\Hash;
     use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
     use Tymon\JWTAuth\Facades\JWTAuth;
     
     class LoginController extends Controller
     {
         use AuthenticatesUsers;
     
         public function login(Request $request)
         {
             $user = User::where('email', $request->email)->orWhere('name', $request->email)->first();
             if($user && Hash::check($request->get('password'), $user->password)) {
                 $token = JWTAuth::fromUser($user);
                 return $this->sendLoginResponse($request, $token);
             }
             return $this->sendFailedLoginResponse($request);
         }
     
         public function sendLoginResponse(Request $request, $token)
         {
             $this->clearLoginAttempts($request);
             return $this->authenticated($token);
         }
     
         public function authenticated($token)
         {
             return $this->response->array([
                 'token' => $token,
                 'status_code' => 200,
                 'message' => 'User Authenticated'
             ]);
         }
     
         public function sendFailedLoginResponse()
         {
             throw new UnauthorizedHttpException("Bad Credentials");
         }
     
         public function logout()
         {
             $this->guard()->logout();
         }
     }
     ```
     
     Api/Auth/RegisterController.php
     
     ```php
     <?php
     namespace App\Http\Api\Auth;
     
     use App\Http\Controllers\Controller;
     use App\User;
     use Dingo\Api\Exception\StoreResourceFailedException;
     use Illuminate\Foundation\Auth\RegistersUsers;
     use Illuminate\Http\Request;
     use Illuminate\Support\Facades\Validator;
     use Tymon\JWTAuth\Facades\JWTAuth;
     
     class RegisterController extends Controller
     {
         use RegistersUsers;
     
         public function register(Request $request)
         {
             $validator = $this->validator($request->all());
             if($validator->fails()) {
                 throw new StoreResourceFailedException("Validation Error", $validator->errors());
             }
             $user = $this->create($request->all());
             if($user) {
                 $token = JWTAuth::fromUser($user);
                 return $this->response->array([
                     "token" => $token,
                     "message" => "User created",
                     "status_code" => 201
                 ]);
             } else {
                 return $this->response->error("User Not Found...", 404);
             }
         }
     
         protected function validator(array $data)
         {
             return Validator::make($data, [
                 'name' => 'required|unique:users',
                 'email' => 'required|email|max:255|unique:users',
                 'password' => 'required|min:6',
             ]);
         }
     
         protected function create(array $data)
         {
             return User::create([
                 'name' => $data['name'],
                 'email' => $data['email'],
                 'password' => bcrypt($data['password']),
             ]);
         }
     }
     ```

