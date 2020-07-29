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