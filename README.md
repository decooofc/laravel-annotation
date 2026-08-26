# Anotações Laravel

## Migration

### O que é
Versionamento do banco de dados, como se fosse um github, só que para o DB

### Vantagens

#### Versionamento
O laravel sempre versiona, sempre permite com que a gente consiga olhar o que está acontecendo com o banco de dados.

Por padrão o `laravel` cria por padrão um versionamento, dentro da pasta `@/database/migrations`, segunido a estrutura:

1. Data
2. O que aconteceu

Exemplo:

![Database folder on migration](./imagens-anotação/database-migration.png)

Com os nomes dos arquivos acima, já conseguimos entender um pouco sobre o que aconteceu e quando foi feito.

#### Reversível

Com o `migration`, conseguimos "voltar atrás" de algum versionamento, como o github, por exemplo:

- Criamos uma tabela, que já foi para o ar, existem usuários utilizando-a, mas aconteceu um erro, conseguimos "voltar atrás", revertendo essa ação.

**Como funciona**

Em qualquer `migration` nós temos uma função/método **`up()`**, que envia para o ar essa mudança no banco de dados.

E também temos a função **`down()`** para gente fazer um rollback caso seja necessário, por isso as migrations são reversíveis.

#### Compartilhável

Imagina que estamos a um projeto legacy de 5 anos atrás e não sabemos como configurar o banco de dados, sendo assim, o laravel resolve esse problema, com o comando `php artisan migrate` no terminal:

```bash
php artisan migrate
```

Desse jeito o `laravel` vai fazer tudo necessário e já vai rodar todas as tabelas, colunas, relacionamentos etc.

Por isso é tão fácil trabalhar com um sistema de compartilhamento de banco de dados com migrations.

#### Agnóstico

As migrations são agnósticas, o que signifca que, independente do banco de dados que os desenvolvedores estão utilizando (`sqlite | mysql | mongodb | etc.`) é função do `laravel` conversar com o banco de dados independente da linguagem de banco de dados utilizada.

Então, caso começamos o banco de dados com `SQLite` e queremos mudar para `mySQL`, com o `migrations` podemos realizar essa mudança de um modo muito simples.

### Criando migration

Para criar uma migration, podemos rodar o comando no terminal:

```bash
php artisan make:migration
```

Logo após isso, ele vai perguntar o nome da migration, é importante que sigamos um padrão para nomear a migration, exemplo: `create_habits_table`

![Created migration](./imagens-anotação/created-migration.png)
<small>*Migration criada*</small>

Agora que criamos a migration, o arquivo já foi criado dentro de `@/database/migrations`, mas ela não subiu ao ar ainda, ou seja, no nosso banco de dados, não vai existir essa tabela, podemos verificar o status das migrations com o comando:

#### Status migration

```bash
php artisan migrate:status
```

![Status migration](./imagens-anotação/migrate-status.png)
<small>*Status do migrate*</small>

E caso terminamos de configurar nossa migration, e queremos subir ela ao banco de dados, podemos rodar:

#### Upload migration

```bash
php artisan migrate
```
![Upload migration to DB](./imagens-anotação/up-migration.png)
<small>*Migration subindo para o DB*</small>

### Deletando migrations

Existem alguns jeitos de removermos uma tabela do banco de dados via migrations, um desses jeitos é remover a partir de um rollback:

```bash
php artisan migrate:rollback
```

Desse modo, estamos dando um `rollback` para o último lote de migration executada, podemos observar os lotes com o [status](#status-migration) da migrate:

![Migration status before rollback](./imagens-anotação/status-before-rollback.png)

No nosso caso, criamos a migration `create_habits_table` no *`lote [2]`*, então se rodarmos o rollback direto, vamos voltar para as migrations no *`lote [1]`*.

![Migration status after rollback](./imagens-anotação/migrate-status.png)
<small>*Migration status depois do rollback*</small>

E agora, como o status está `pending`, podemos **deletar** o arquivo da migration com o status pendente.

#### Flags no rollback

- `--step=`: Indicamos quantas migrations queremos dar rollback a partir da última, por exemplo: <br/> 
  `php artisan migrate:rollback --step=2` -> rollback nas duas últimas migrations.

- `--batch=`: Indicamos o lote em qual queremos dar rollback, por exemplo: <br/>
  `php artisan migrate:rollback --batch=1` -> rollback em todas as migrations no lote 1.

### Fresh

Existe um método nas migrations chamado `fresh`, aonde ele irá fazer um refresh da migration:

1. Deleta todas as tabelas;
2. Prepara o banco de dados;
3. Cria as migrations;
4. Roda as migrations;

![Migration fresh](./imagens-anotação/migration-fresh.png)
<small>*Fresh migration*</small>

Ele pode ser utilizado para atualizar uma tabela se fizermos uma alteração, por exemplo, excluírmos uma coluna ou alteramos um campo.

> O `fresh` nunca deve ser usado em produção, com o banco de dados já pronto;

### Alterando algum dado na tabela

Se formos alterar a tabela ou alguma coluna dentro da tabela, podemos fazer isso de duas maneiras:

- `migrate:fresh`
- `Criar uma nova migration`

Se estou em desenvolvimento, sem dados reais e quero alterar a migration original, posso usar o `migrate:fresh`;

Mas caso eu esteja em produção, e não posso perder esses dados, preciso criar uma nova migration, por exemplo, quero alterar o nome de uma tabela:

1. Criamos a migration `php artisan make:migration rename_table_user_to_clients`;
2. Dentro da migration rodamos: `Schema::rename('user', 'clients');`;
3. Subimos a migration ao DB: `php artisan migrate`;

Assim alteramos a estrutura sem apagar os dados existentes.

## Models

### O que são
- Representação das tabelas do banco de dados em forma de classe PHP;
- As models ficam guardadas na pasta `@/app/models/...`

![Model example in Laravel](./imagens-anotação/model-php.png)
<small>*Exemplo de `Model` em `Laravel`*</small>

- Exemplo MYSQL

```sql
SELECT * FROM users WHERE email = 'andre@uranus.com.br'
```

- Exemplo c/ Laravel

```php
User::where('email', 'andre@uranus.com.br')->first()
```

- `User::`: Model User;
- `where()`: Método estátido where;
- `('email')`: Coluna "email" dentro da tabela;
- `('andre@uranus.com.br')`: Condição dentro de where para a coluna "email";
- `->first()`: Retornar o primeiro resultado encontrado;

Ou seja: na model user (`User`) onde (`where`) o email 'andre@uranus.com.br' esteja dentro da coluna 'email' (`('email', 'andre@uranus.com.br')`) retorne o primeiro resultado (`->first()`);

### Fillable

As colunas marcadas como `$fillable` são campos em que o usuário ele tem controle, então, que ele possa mudar por conta própria, ou seja, todas as colunas que não estão marcadas com `$fillable` o usuário não tem controle.

### Hidden

As colunas marcadas como `$hidden` são campos em que são escondidos quando a model é convertida para array ou JSON, sendo assim, quando fizermos uma requisição em uma API para o banco de dados, os campos `$hidden` serão automaticamente removidos na resposta dessa requisição.

### Resumo

- `Migration`: Cria o banco de dados;
- `Model`: Representa o banco de dados;

## Sistema de login - Parte 1 

Optamos por fazer o sistema de login antes que o sistema de cadastro, mas para ter um sistema de login, precisamos ter um usuário cadastrado no nosso banco de dados, por isso vamos utilizar uma técnica do laravel que consiste em popular o banco de dados com dados iniciais ou dados de teste, chamado `Seeders`

### Criando seeder

```bash
php artisan make:seeder UserSeeder
```

Aqui estamos criando uma `seeder` chamada `UserSeeder`, que fica disponível em: `@/database/seeders/UserSeeder.php`.

#### Populando tabela Users

Ao criar essa `seeder` temos a função pública `run()` que será executada quando enviarmos os dados para popular o banco de dados, na maioria dos casos, uma seeder vai inserir os dados em alguma tabela, no nosso caso, só temos a tabela `User` criada, ou seja, a model `User`, por isso vamos passar a função `::create([])` para a model `User` e dentro do array, passamos os dados das colunas que temos: 

![UserSeeder Data](./imagens-anotação/userseeder-data.png)

Após definir a função `create([])` e passar os dados das colunas, podemos subir esses dados para o banco de dados utilizando:

```bash
php artisan migrate:fresh --seed
```

Passamos a flag `--seed` porque o Artisan não roda seeders automaticamente depois de migrar, então sem essa flag você teria que rodar dois comandos separados:

- `php artisan migrate`
- `php artisan db:seed`

#### Arrumando factory

Ao criar um projeto com laravel, ele tenta criar uma **Fábrica de dados falsos** para enviar ao banco de dados, só que no nosso caso, a gente deletou algumas colunas do banco de dados, então quando rodamos o `php artisan migrate:fresh --seed`, ele vai retornar um erro, dizendo que não encontrou a coluna `email_verified_at`, já que deletamos ela na última seção.

![Error on fresh](./imagens-anotação/seeder-error.png)

Desse modo, vamos entrar dentro do arquivo da `Factory` e arrumar esse problema, para que a factory envie os dados nas colunas existentes:

```php 
// UserFactory.php

class UserFactory extends Factory
{
    protected static ?string $password;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => static::$password ??= Hash::make('password'),
        ];
    }
}
```
<small>*É assim como deve ficar o `UserFactory`*</small>

Após arrumar a factory, podemos rodar o `migrate:fresh` que os dados serão enviados ao banco de dados:

![Habit tracker database on users table](./imagens-anotação/database-with-factory-data.png)
<small>*Dados enviados para a tabela users*</small>

Agora, podemos observar que os dados enviados para a tabela, foi diferente dos dados que definimos na `seeder`, isso acontece porque, criamos a `UserSeeder`, mas não estamos chamando a `seeder` em nenhum momento, vamos chamar o `seeder` dentro de `DatabaseSeeder.php` que é onde vamos chamar as `seeders` que criarmos:

```php
// DatabaseSeeder.php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

Então, dentro da função `run()`, estamos enviando `$this->call([])`, aonde, dentro desse array é onde vamos enviar a `seeder`, enviamos um array, porque caso tenhamos mais de uma `seeder`, podemos enviar ela junto ao `call([])`.

Agora podemos dar `php artisan migrate:fresh --seed`, que os dados da `UserSeeder` serão enviados para o banco de dados:

![Migration fresh --seed](./imagens-anotação/migration-fresh--seed.png)
![Table users after fresh](./imagens-anotação/database-with-user-seeder.png)

Podemos observar na tabela `password`, o dado está `hashed`, isso acontece por que na Model, é enviado uma função `casts()`, dizendo para o laravel que os dados da tabela `password` devem ser `hashed`.

```php
// User.php

protected function casts(): array
{
  return [
    'password' => 'hashed',
  ],
}
```

### Criando rota de login

Agora que criamos o nosso primeiro usuário, precisamos criar uma rota de login:

```php
// web.php

Route::get('/login', [LoginController]::class, 'index');
// Rota -> '/login'
// Controller -> 'LoginController'
// Função dentro do Controller -> 'index'
```

Aqui criamos um novo controller `LoginController` que vai estar dentro da pasta `@/App/Http/Controllers/Auth/LoginController`;

> Criamos a pasta `Auth`, para deixar todos os controllers relacionados com o login nessa pasta, podemos criar esse controller com o seguinte comando:
> ```bash
> php artisan make:controller Auth/LoginController
> ```

### Criando view login

No nosso método `index` dentro de `LoginController` vamos definir apenas a nossa view:

```php
// LoginController

public function index()
{
  return view('login');
}
```

Mas, precisamos criar a nossa view, então dentro da pasta `@resources/views` vamos duplicar o arquivo `home.blade.php` e renomear para `login.blade.php`.

### Recuperando tabela sessions

Após criar, se formos ao navegador, podemos perceber um erro `Base table or view not found: Table 'habit_tracker.sessions' doesn't exists`, por que esse erro está acontecendo, na última aula, deletamos a tabela `sessions` do banco de dado

- Para recuperar essa tabela, podemos ir no último commit do github e copiar o código em que foi criado a tabela `sessions` na função `up()` na nossa migration, e também fazer o drop dessa tabela na função `down()`:
![Recuperate sessions table](./imagens-anotação/sessions-table.png)

E para subir essa migration para o ar, e criar essa tabela no banco de dados, é só fazer: `php artisan migrate:fresh --seed`, passando a flag `--seed` para também enviar os dados da `seeder`.
