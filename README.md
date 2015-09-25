

<img src="http://owen.com.br/imagens_para_web/auditing.png" style="width: 100%" alt="laravel-auditing" />

[![Latest Stable Version](https://poser.pugx.org/owen-it/laravel-auditing/version)](https://packagist.org/packages/owen-it/laravel-auditing)
[![Total Downloads](https://poser.pugx.org/owen-it/laravel-auditing/downloads)](https://packagist.org/packages/owen-it/laravel-auditing)
[![Latest Unstable Version](https://poser.pugx.org/owen-it/laravel-auditing/v/unstable)](//packagist.org/packages/owen-it/laravel-auditing)
[![License](https://poser.pugx.org/owen-it/laravel-auditing/license.svg)](https://packagist.org/packages/owen-it/laravel-auditing)

It is always important to have change history records in the system. The Auditing does just that simple and practical way, you simply extends it in the model you would like to register the change log. 

> Auditing is based on the package [revisionable](https://packagist.org/packages/VentureCraft/revisionable)

## Installation

Auditing is installable via [composer](http://getcomposer.org/doc/00-intro.md), the details are [here](https://packagist.org/packages/owen-it/laravel-auditing).

Run the following command to get the latest version package

```
composer require owen-it/laravel-auditing
```
Open ```config/app.php``` and register the required service provider.

```php
'providers' => [
    // ...
    OwenIt\Auditing\AuditingServiceProvider::class,
],
```

> Note: This provider is important for the publication of configuration files.

Use the following command to publish settings:

```
php artisan vendor:publish
```
Now you need execute the mitration to create the table ```logs``` in your database, this table is used for save logs of altering.

```
php artisan migrate
```


## Docs
* [Dreams (Example)](#example)
* [Implementation](#implementation)
* [Configuration](#configuration)
* [Getting the Logs](#getting)
* [Featuring Log](#featuring)
* [Contributing](#contributing)
* [Having problems?](#faq)
* [license](#license)


<a name="example"></a>
## Dreams (Examle)
Dreams is a developed api to serve as an example or direction for developers using laravel-auditing. You can access the application [here](https://dreams-.herokuapp.com). The back-end (api) was developed in laravel 5.1 and the front-end (app) in angularjs, the detail are these:

* [Link for application](https://dreams-.herokuapp.com) 
* [Source code api-dreams](https://github.com/owen-it/api-dreams)
* [Source code app-dreams](https://github.com/owen-it/app-dreams)

<a name="implementation"></a>
## Implementation

### Implementation using ```Trait```

To register the change log, use the trait `OwnerIt\Auditing\AuditingTrait` in the model you want to audit

```php
// app/Models/People.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use OwenIt\Auditing\AuditingTrait;

class People extends Model 
{
    use AuditingTrait;
    //...
}

```

> Note: Traits require PHP >= 5.4

### Base implementation Legacy Class

To register the chage log with Legacy class, extend the class `OwnerIt\Auditing\Auditing` in the model you want to audit. Example:

```php
// app/Models/People.php

namespace App\Models;

use OwenIt\Auditing\Auditing;

class People extends Auditing 
{
    //...    
}
```

<a name="configuration"></a>
### Configuration

The Auditing behavior settings are carried out with the declaration of attributes in the model. See the examples below:

* Turn off logging after a number "X": `$historyLimit = 500`
* Disable / enable logging (Audit): `$auditEnabled = false`
* Turn off logging for specific fields: `$dontKeep = ['campo1', 'campo2']`

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class People extends Model 
{
    use OwenIt\Auditing\AuditingTrait;

    protected $auditEnabled  = false;      // Desativa o registro de log nesta model.
    protected $historyLimit = 500;         // Desativa o registro de log após 500 registros.
    protected $dontKeep = ['cpf', 'nome']; // Informe os campos que NÃO deseja registrar no log.
    protected $auditableTypes = ['created', 'saved', 'deleted']; // Informe quais ações deseja auditar
}
```

<a name="getting"></a>
## Getting the Logs

```php
namespace App\Http\Controllers;

use App\Models\People;

class MyAppController extends BaseController 
{

    public function index()
    {
        $people = People::find(1); // Get people
        $people->logs; // Get all logs
        $people->logs->first(); // Get first log
        $people->logs->last();  // Get last log
        $people->logs->find(2); // Selects log
    }

    ...
}
```
Getting logs with user responsible for the change.
```php
use OwenIt\Auditing\Log;

$logs = Log::with(['user'])->get();

```
or
```php
use App\Models\People;

$logs = People::logs->with(['user'])->get();

```

> Note: Remember to properly define the user model in the file ``` config/auth.php ```
>```php
> ...
> 'model' => App\User::class,
> ... 
>```

<a name="featuring"></a>
## Featuring Log

You it can set custom messages for presentation of logs. These messages can be set for both the model as for specific fields.The dynamic part of the message can be done by targeted fields per dot segmented as`{objeto.field} or {objeto.objeto.field}`. 

Set messages to the model
```php
namespace App;

use OwenIt\Auditing\Auditing;

class People extends Auditing 
{
    ...

	public static $logCustomMessage = '{user.nome} been updated by {old.nome}';
	public static $logCustomFields = [
	    'nome' => 'Before {old.nome} and after {new.nome}',
	    'cpf'  => 'Before {old.cpf}  and after {new.cpf}' 
	];
	
	...
}
```
Getting change logs 
```php
    
    // app\Http\Controllers\MyAppController.php 
    ...
    public function auditing()
    {
    	$pessoa = Pessoa::find(1); // Get people
    	return View::make('auditing', ['logs' => $pessoa->logs]); // Get logs
    }
    ...
    
```
Apresentando registros de log:
```php
    // resources/views/my-app/auditing.blade.php
    ...
    <ol>
        @forelse($log as $logs)
            <li>
                {{ $log->customMessage }}
                <ul>
                    @forelse($custom as $log->customFields)
                        <li>$custom</li>
                    @endforelse
                </ul>
            </li>
        @empty
            <p>No logs</p>
        @endforelse
    </ol>
    ...
    
```
Answer:
<ol>
  <li>Jhon Doe atualizou os dados de Rafael      
    <ul>
      <li>Antes Rafael | Depois Rafael França</li>
      <li>Antes 00000000000 | Depois 11122233396 </li>
    </ul>
  </li>                
  <li>...</li>
</ol>

<a name="contributing"></a>
## Contributing

Contribuições são bem-vindas; para manter as coisas organizadas, todos os bugs e solicitações devem ser abertas na aba issues do github para o projeto principal, no [owen-it/laravel-auditing/issues](https://github.com/owen-it/laravel-auditing/issues)

Todos os pedidos de pull devem ser feitas para o branch develop, para que possam ser testados antes de serem incorporados pela branch master.

<a name="faq"></a>
## Having problems?

Se você está tendo problemas com o uso deste pacote, existe probabilidade de alguém já ter enfrentado o mesmo problema. Você pode procurar respostas comuns para os seus problemas em:

* [Github Issues](https://github.com/owen-it/laravel-auditing/issues?page=1&state=closed)

<a name="license"></a>
### License

O pacote laravel-auditoria é software open-source licenciado sob a [licença MIT](http://opensource.org/licenses/MIT)

