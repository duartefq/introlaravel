## Intro ao Laravel 5.*

### Arquitetura

- Laravel: MVC (Model-View-Controller)
- Django: MTV (Model-Template-View)

### Instalação

- [Vagrant + Homestead](https://laravel.com/docs/master/homestead) (Recomendado para Windows / Mac)
	* Ambiente homogêneo entre equipes

- [Valet](https://laravel.com/docs/master/valet) (Mac)

- [Manual](https://laravel.com/docs/master/installation) (Recomendado para Linux / Mac)

### Workflow Laravel

* Todo comando executado com prefixo `php artisan [comando]`

#### Modelo

- `make:model [Model] -m` (cria model + migrations)

Dir: `app/`

Exemplo:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Test extends Model
{
    //
}
```

#### Migrations

- `make:migration [nome migration]`, convenção `create_`, `add_`, etc. (expressando ação)

Dir: `database/migrations/`
Exemplo (`2016_10_05_015152_create_tests_table`):

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateTestsTable extends Migration
{
    public function up()
    {
        Schema::create('tests', function (Blueprint $table) {
            $table->increments('id');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('tests');
    }
}
```

#### Controllers

- `make:controller [NomeController (controller criado com metodos CRUD)]`

Dir: `app/Http/Controllers/`

Estrutura de um Controller:

```php
namespace App\Http\Controllers\Test;
use Illuminate\Http\Request;

use App\Http\Requests;
use App\Http\Controllers\Controller;

class TestController extends Controller
{
    public function index() {}
    public function create() {}
    public function store(Request $request) {}
    public function show($id) {}
    public function edit($id) {}
    public function update(Request $request, $id) {}
    public function destroy($id) {}	
}
```

Exemplo de método:

```php
	public function showProfile($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
```


#### Rotas

Dir:

* [5.2] `app/Http/routes.php`

* [5.3] `routes/`

Métodos disponíveis:

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Exemplo:

```php
	Route::get('user/{id}', function ($id) {
	    return 'User '.$id;
	});

	// opcional
	Route::get('user/{name?}', function ($name = null) {
	    return $name;
	});

	// regex
	Route::get('user/{id}', function ($id) {
	    //
	})->where('id', '[0-9]+');

	// Named routes
	Route::get('user/profile', 'UserController@showProfile')->name('profile');

	
	/* chamando a "rota nomeada" */

	// Gerando URL com base na rota nomeada...
	$url = route('profile');

	// Gerando um redirect com base na rota nomeada...
	// utilizado em controllers
	return redirect()->route('profile');

	/* route groups */

	// Sintaxe: 

	Route::group([], function() { /* rotas */ })

	// Exemplos:

	Route::group(['namespace' => 'Admin'], function()
	{
	    // Controllers dentro do namespace "App\Http\Controllers\Admin"

	    Route::group(['namespace' => 'User'], function() {
	        // Controllers dentro do namespace "App\Http\Controllers\Admin\User"
	    });
	});

	// Autorizando ou restringindo acesso de usuários
	Route::group(['middleware' => 'auth'], function () {
	    Route::get('user/list', function ()    {
	        // Usa Auth Middleware
	    });

	    Route::get('user/profile', function () {
	        // Usa Auth Middleware
	    });
	});

	Route::group(['middleware' => 'guest'], function () {
	    Route::get('/', function () {
	        // Usa Auth Middleware
	    });
	});

	// Múltiplos groups
	
	Route::group(['middleware' => 'auth'], function () {

		Route::group(['prefix' => 'user'], function() {
			Route::get('list', function ()    {
		        // Usa Auth Middleware e prefixo de rota "user"
		    });

		    Route::get('profile', function () {
		        // Usa Auth Middleware e prefixo de rota "user"
		    });
		})
	});
```

#### Blade Templates

Dir: `resources/views/`

O template base:

Dir: `resources/views/layouts/master.blade.php`

```blade
<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Template filhos:

Dir: `resources/views/child.blade.php`

```blade
@extends('layouts.master')

@section('title', 'Titulo da pagina')

@section('sidebar')
    @parent

    <p>Isto será adicionado à section sidebar do master.</p>
@endsection

@section('content')
    <p>Conteúdo (content) do body.</p>
@endsection
```

Manipulando dados nos templates (exemplo)

Rota:

```php
Route::get('hello', 'Test/TestController@hello');
```

Controller:

```php

namespace App\Http\Controllers\Test;

public function hello()
{
	return view('welcome', ['nome' => 'Sorriso']);
}
```

Template:

```blade
@extends('layouts.master')
@section('title', 'Welcome')
@section('content')
    <p>Olá, {{ $nome }}.</p>
@endsection
```

##### Templates (Resumo)

**Herança**

* `@yield('area')`: define uma `area` onde child pode acrescentar conteúdo

* `@section('area')`: acrescenta conteúdo a uma `area` do master/parent

* `@extends('layouts.master')`: relação child <-- master

* `@include('partials.subview', ['optional' => 'data'])`: inclui sub-views (parciais), como forms. Opção de passar variaveis.

**Exibido dados**

* `{{ $var }}`, exibe valor de `$var`

* `@{{ $var }}`, ao utilizar frameworks javascripts, evitando assim conflito de sintaxe (ex: Vue.js, Angular),

* `{{ $var or 'Default' }}`, conteúdo de `$var` caso exista, 'Default' caso contrário
	
	* `{{ isset($var) ? $var : 'Default' }}`, similar

**Estrutura de controle**

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
    You are not signed in.
@endunless
```
```blade
<title>
    @hasSection ('title')
        @yield('title') - App Name
    @else
        App Name
    @endif
</title>
```

**Loops**

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```
