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
Alterar a validação do banco de dados para o LDAP, alterar o método validateCredentials de ./vendor/laravel/framework/src/Illuminate/Auth/EloquentUserProvider.php
