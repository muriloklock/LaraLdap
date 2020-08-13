# LaraLdap
Simulação da implementaçaõ simples de validação de credenciais em LDAP/active Directory.
Aplica alteração no método de checar se a senha é válida

# Etapa 1 - Intalando Laravel
```bash=
$ composer create-project --prefer-dist laravel/laravel teste
```

# Etapa 2 - Adequando permissões da estrutura de diretórios do projeto
Mais em https://laravel.com/docs/7.x/installation#configuration.
Alterar permissões da árvore de diretórios storage e bootstrap/cache
```bash=
$ touch storage/logs/laravel.log
$ chmod -R 0777 storage bootstrap/cache
```
# Etapa 3 - Gerar UI
```bash=
$ composer require laravel/ui
$ php artisan ui bootstrap --auth
$ npm install && npm run dev
```
# Etapa 4 - Ajustar dados ndo banco de dados no arquivo .env
Ajustar no arquivo .env os dados de conexão ao banco de dados

# Etapa 5 - Adequar tabela de usuários
Ajusta o arquivo migrate da tabela de usuarios.
```bash=
php artisan migrate
```

# Etapa 6 - Adequar o controller de login 
Alterar o arquivo app/Http/Controllers/Auth/LoginController.php para usar o campo username.
```php=
    public function username()
    {
        return 'username';
    }
```

# Etapa 7 - Adequar a página de login
Alterar o arquivo /var/www/teste/resources/views/auth/login.blade.php para usar o *username* no lugar de *email*.
```php=
    <div class="form-group row">
        <label for="username" class="col-md-4 col-form-label text-md-right">{{ __('Username') }}</label>

        <div class="col-md-6">
            <input id="username" type="username" class="form-control @error('username') is-invalid @enderror" name="username" value="{{ old('username') }}" required autocomplete="username" autofocus>

            @error('username')
            <span class="invalid-feedback" role="alert">
                <strong>{{ $message }}</strong>
            </span>
            @enderror
        </div>
    </div>
```

# Etapa 8 - Validar credenciais no LDAP
Alterar a validação de usuários (username e passowrd) do banco de dados para o LDAP.
Para isso vamos extender o método validateCredentials padrão do Laravel(EloquentUser).
Crie o arquivo OiUserProvider.php em app/Providers/ e dentro dele coloque o método com sua modificação. No meu caso, fiz a validação via LDAP.
```php=
<?php
namespace App\Providers;

use Illuminate\Auth\EloquentUserProvider as UserProvider;
use Illuminate\Contracts\Auth\Authenticatable as UserContract;


class OiUserProvider extends UserProvider {

    /**
     * Overrides the framework defaults validate credentials method 
     *
     * @param UserContract $user
     * @param array $credentials
     * @return bool
     */
    public function validateCredentials(UserContract $user, array $credentials) {
        $plain = $credentials['password'];

        if ($ldapconn = @ldap_connect(env('LDAP_SERVER', false)) ) {
            @ldap_set_option($ldapconn, LDAP_OPT_REFERRALS, 0);
            @ldap_set_option($ldapconn, LDAP_OPT_DEREF, 3);
            if (@ldap_start_tls($ldapconn)) {
                $ldaprdn = "OI\\" . $credentials['username'];
                if ($ldapbind = @ldap_bind($ldapconn, $ldaprdn, $plain) ) {
                    $filter = "cn=" . $credentials['username'];
                    if ($res = @ldap_search($ldapconn, env('LDAP_BASEDN', false), $filter)) {
                        $entries = @ldap_get_entries($ldapconn, $res);
                        if ($entries['count']>0) {
                            return true;
                        }
                    }
                    @ldap_unbind($ldapconn);
                }
            }
        }
        return false;
    }

}
```

Depois de feita a modificação acima, o Laravel tem que saber como chamar o novo provider.
Vamos fazr isto extendento do método boot() no arquivo app/Providers/AuthServiceProvider.php.
```php=
public function boot()
    {
        $this->registerPolicies();

        \Illuminate\Support\Facades\Auth::provider('oiuserprovider', function($app, array $config) {
		    return new OiUserProvider($app['hash'], $config['model']);
	    });
    }
```

E finalmente vamos alterar as configurações do Laravel para usar o novo user provider.
Edite o arquivo config/auth.php em 'providers' -> 'users'.
```php=
    'providers' => [
        'users' => [
            //'driver' => 'eloquent',
            'driver' => 'oiuserprovider',
            'model' => App\User::class,
        ],
```

# Etapa 9 - Adequar formulário de Registro.
Alterar o formulário de registro de usuário.
Alterar os arquivos:
./resources/views/auth/register.blade.php
Adicionar:
```php=
                        <div class="form-group row">
                            <label for="username" class="col-md-4 col-form-label text-md-right">{{ __('Username') }}</label>

                            <div class="col-md-6">
                                <input id="username" type="text" class="form-control @error('username') is-invalid @enderror" name="username" value="{{ old('username') }}" required autocomplete="username" autofocus>

                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{{ $message }}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
```

./app/User.php
Adicionar aos campos fillable a coluna username.
```php=
    protected $fillable = [
        'username', 'name', 'email', 'password',
    ];
```

./app/Http/Controllers/Auth/RegisterController.php
Alterar função validator para:
```php=
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'username' => ['required', 'string', 'max:8'],
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ]);
    }
```

Alterar função create para:
```php=
    protected function create(array $data)
    {
        return User::create([
            'username' => $data['username'],
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);
    }
```