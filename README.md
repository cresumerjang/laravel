# laravel
라라벨 학습을 정리합니다.

## 마이그레이션
https://laravel.kr/docs/migrations
#### 마이그레이션 생성
php artisan make:migration create_posts_table --create=posts
생성후 app\database\migrations\ 에 마이그레이션.php 파일이 생성되며
적절한 스키마를 작성해준다.
Scheme::create // 테이블생성
Scheme::drop // 테이블제거
Scheme::table // 테이블 생성, 제거를 제외한 나머지 스키마작업
마이그레이션.php파일은 db스키마이며 artisan migrate 명령으로 실행시 DB 테이블을 생성 및 변경한다.
```
절절한 스키마
up => 마이그레이션 실행시
down => ?? 롤백시?
up, down은 짝이 맞아야한다.
class MemebrInformationTable extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title');
            $table->text('body');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('posts');
    }
}
```

#### 마이그레이션 열 추가
php artisan make:migration add_name_to_authors_table --table=authors
위명령을 입력하면 새로운 마이그레이션 파일이 생성되며 해당 파일에 변경할 스키마(--table=authors)에 적용할 작업을
Scheme::create // 테이블생성
Scheme::drop // 테이블제거
Scheme::table // 테이블 생성, 제거를 제외한 나머지 스키마작업
적절히 작성해주면 된다.

#### 마이그레이션 롤백
php artisan migrate:rollback
롤백시 ClassNotFound 예외 발생시 composer dump-autoload --optimize 명령으로 해결한다.
컴포저가 아티즌 콘솔의 변경 사항을 알지 못한 이슈로 컴포저가 만들어 놓은 오토로드 레지스트리를 다시 만들어준다.
crud는 model에서 처리한다.?

#### 마이그레이션 초기화
php artisan migrate:reset
는 모든 마이그레이션을 롤백하고 데이터베이스 초기화만 수행하고
php artisan migrate:refresh
명령어는 모든 마이그레이션을 롤백하고 데이터베이스를 초기화한 후 마이그레이션을 다시실행한다.

## 모델
모델을 통해 table 데이터 삽입, 업데이트 방법
인스턴스를 통한 방법
  $author = new App\Author;
  $author->email = 'foo@bar.com';
  $author->password = 'password';
  $author->save();

파사드패턴을 통한방법
  App\Author::create([
    'email' => 'bar@baz.com',
    'password' => bcrypt('password'),
  ]);

위 경우 두가지 에러가 발생한다.
  1. 타임스템프 컬럼 이슈
    created_at, updated_at 칼럼 없어서 에러가 발생한다.
    (created_at, updated_at 칼럼은 테이블 갱신시 자동으로 처리된다)
    따라서 사용되는 모델에 $timestamps에 false라고 명시해주거나
    created_at, updated_at을 직접 칼럼에 추가해줘야한다.
  2. 대량할당 이슈
    대량할당이 불가능하기 때문에 사용되는 모델내에 대량할당 가능한 필드를 $fillable에 배열로 명시해줘야 한다.

// app/Author.php
class Author extends Model
{
    public $timestamps = false;
    protected $fillable = ['email', 'password'];
}

## 컨트롤러
#### 일반컨트롤러
```
php artisan make:controller WelcomeController
```
Route::get('/', 'WelcomeController@index');
#### REST API 컨트롤러
```
$ php artisan make:controller ArticlesController --resource
```
Route::resource('articles', 'ArticlesController');
$ php artisan route:list

###### HTTP 메서드 오버라이드(포스트맨)
_method=put,delete

## CSRF 토큰처리
app\http\middleware\VerifyCsrfToken.php에
protected $except = [] 토큰처리하지 않을 api리스트추가
미들웨어는 전역 미들웨어와 라우트 미들웨어로 전역 미들웨어는 무조건 거쳐야하고 라우트미들웨어는 특정라우트에만 적용할 수 있다.
```
// app/Http/Middleware/VerifyCsrfToken.php
class VerifyCsrfToken extends BaseVerifier
{
    protected $except = [
        'articles',
        'articles/*'
    ];
}
```

## 미들웨어
app\http\Kernel.php에
$middlewareGroups에 개별 미들웨어나 그룹 미들웨어로 등록하여 필요한 기능을 손쉽게 사용할 수 있다.

## 라라벨 내장 사용자 인증
https://appkr.github.io/l5book-snippets/draft/1009-authentication.html

## ORM
#### 1:1 관계
일대일 관계 정의는 모델 클래스에 관련된 테이블명 모델명과 동일한 메소드를 만들고 안에 __hasOne()__ 메소드로 대상 모델을 지정해 주면 됩니다.
```
class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}
```
대상이 되는 profiles 테이블은 __belongsTo()__  메소드로 속하는 테이블을 기술해 줍니다.
```
class Profile extends Model
{
    /**
     * Get the user that owns the profile.
     */
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

```
만약 profiles 테이블의 users 테이블을 참조하는 참조키 컬럼명이 user_id 가 아닐 경우 belongsTo 의 두 번째 파라미터에 참조키 컬럼명을 기술하면 됩니다.
참조키의 이름이 fk_user_id 일 경우 아래처럼 작성해 줍니다.
```
public function user()
{
    return $this->belongsTo(User:class, 'fk_user_id');
}
```

1:1 관계를 맺어주고 나면 아래처럼 사용이 가능합니다.
```
$profile = User::find(1)->profile;
```

#### 1:N 관계
부모가 되는 모델에 자식의 테이블명과 일치하는 메소드를 만들고 hasMany($related, $foreignKey = null, $localKey = null) 를 사용하여 자식 모델을 기술해 주면 되며 필수인 첫 번째 파라미터는 관련된 테이블 모델명입니다.
```
class Project extends Model
{
    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
}
```
<!-- 만약 참조 키 이름은 다음과 같은 로직에 의해서 결정되며 정확한 참조 키 이름을 알고 싶으면 다음 코드를 artisan tinker 에서 돌려서 알아 낼 수 있습니다.
>>> snake_case(class_basename('App\ArticleCategory')).'_id';            
=> "article_category_id"
 참조 키 이름이  관례와 다를 경우 두 번째 파라미터로 명시적인 참조키 이름을 지정해 주면 됩니다.
  -->
Tasks 테이블은 belongsTo($related, $foreignKey = null, $otherKey = null, $relation = null) 메소드를 사용하여 첫 번째 파라미터로 부모 모델 클래스를 지정해 줍니다.
```
class Task extends Model
{
    public function project()
    {
        return $this->belongsTo(Project::class);
    }
}
```
Eloquent ORM 은 관계가 지정된 컬럼명을 관례에 따라 snake case 명명법으로 '참조모델명_id' 로 찾으므로 Task 모델의 프로젝트 테이블에 대한 참조키는 project_id 가 되며 만약 레거시 테이블이라 관례에 따라 작명되지 않았다면 hasMany() 나 belongsTo()의 두번째 파라미터에 해당 컬럼명을 지정해 주면 됩니다.
```
class Task extends Model
{
    public function project()
    {
        return $this->belongsTo(Project::class, 'project_fk_name');
    }
}
```
// 동적프로퍼티를 리턴(콜렉션)
$tasks = App\Project::findOrFail(1)->__tasks__
```
get_class($tasks)

$tasks 변수의 타입은
"Illuminate\Database\Eloquent\Collection" 이며
project_id 가 1 인 App\Task 객체들을 담고 있습니다.
```

동적 프로퍼티는 사용이 쉽지만 다양한 조건으로 검색할 수 없으므로 만약 모델을 가져올 때 추가 검색 조건이 필요할 경우는 동적 프로퍼티가 아닌 메소드 호출을 하고 조건문을 추가 해야 합니다.

// hasMany 객체 리턴
$tasks = App\Project::findOrFail(1)->__tasks()__
```
get_class($tasks)

$tasks 변수는 콜렉션이 아니라
Illuminate\Database\Eloquent\Relations 을 상속받은 HasMany 객체입니다.

// hasMany객체를 통하여 조건처리
$tasks = Project::findOrFail($id)->tasks()->orderBy('name')->get();
```

Task모델의 부모 모델을 찾으려면 아래처럼 사용
```
App\Task::find(1)->project
```

#### N:N 관계
다대다 관계는 pivot 테이블이 필요하기 때문에 관계를 맺은 테이블들을 가지고 pivot테이블 마이그레이션을 생성해줍니다. 생성 규칙은 a-z순의 단수형으로 '_'로 조합하여 사용됩니다.
```
php artisan make:migration create_role_user_table --create=role_user
```
마이그레이션 파일을 아래와 같이 작성하여 pivot이 참조할 각 테이블의 참초키를 생성하고 걸어줍니다.
```
public function up()
{
    Schema::create('role_user', function (Blueprint $table) {
        $table->increments('id');
        $table->timestamps();

        // users 테이블에 대한 참조키
        $table->integer('user_id')->unsigned();
        $table->foreign('user_id')->references('id')->on('users');

        // roles 테이블에 대한 참조키
        $table->integer('role_id')->unsigned();
        $table->foreign('role_id')->references('id')->on('roles');

        // user_id 와 role_id 컬럼은 유일해야 함.
        $table->unique(['user_id', 'role_id']);
    });
}
```
생성된 마이그레이션 파일은 관례상 role_users테이블을 참조하지만 pivot테이블 명은 단수여야 하므로 아래와 같이 테이블 명을 명시적으로 선언해 줍니다.
```
class RoleUser extends Model
{
    protected $table = 'role_user';
}
```

모델에 관계를 맺을 테이블명과 동일한 메소드를 만들고 belongsToMany($related, $table = null, $foreignKey = null, $otherKey = null, $relation = null) 메소드를 호출해 주면 됩니다.
```
class User
{
   /**
     * The roles that belong to the user.
     */
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}
```
```
class Role extends Model
{
    public function users()
    {
        return $this->belongsToMany(User::class);
    }
}
```


___
__작성중__

라라벨에서 기본적으로 모델명은 단수, 해당 모델이 참조하는 테이블은 복수로 사용되며 관계를 맺어줄때는 관계맺을 모델이 참조하는 테이블명으로(복수형 메소드) 메소드 이름을 사용해야한다.
그리고 외래키는 기본적으로 모델명_id로(모델이 참조하는 테이블x 단수 형태의 모델명) 참조되며 이름이 다를경우 관계를 맺어주는 메소드에 파라미터로 전달해줘야한다.(belongsTo, belongsToMany...)

라라벨의 생명주기 동안의 전역환경 설정 파일은 config/ 아래에 있습니다.
config/아래 설정 파일들은 env() 메소드를 사용하여 라라벨 프로젝트 루트의 .env파일을 참조하는데
.env파일이 없다면 cp .env.example .env 로 복사하여 생상하고 .env파일에 설정값을 작성해줍니다.
config/아래 설정 파일에서 직접 설정값을 작성하지 않고 .env파일로 따로 관리하는 이유는
1. 웹서비스 환경에 따라 다른 설정값을 적용해야할 때
2. 비밀번호처럼 민감한 정보라 버전관리 시스템에 등록되면 안되는 경우
* config/설정파일.php에 .env참조 대신 직접 작성해도 동일하게 동작합니다.
* 라라벨 내장서버 구동중 .env나 config/설정파일.php의 내용이 변경되면 내장서버를 다시 구동해줘야 합니다.


table(복수)과 모델(단수) 이름이 관례와 다를경우 엘로퀀트에게 아래와같이 알려준다.
class Author extends Model
{
  portected $table = 'users';
}

엘로퀀트는 모든 테이블에 updated_at, created_at이 있다고 가정하고
새로운 인스턴스를 테이블에 저장할때 현재의 타임스템프 값을 할당하는데
실제로 해당 컬럼이 존재하지 않아 에러가 발생한다.
엘로퀀트의 타임스템프 자동입력을
public $timestamps = false;로 모델에 넣어 끄던지 해당열을 만들어줘야한다.
protected $fillable = ['email', 'password']; // 대량할당 허용
protected $guarded = []; // 대량할당 비허용 목록
비밀번호는 아래 메소드 사용하여 해싱처리하여 삽입
bcrypt(패스워드값)
