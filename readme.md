## Sistema de delivery com Laravel 5.1 + Ionic

Roteiro de atividades realizadas no curso Laravel 5.1 + Ionic da CodeEducation. 
Criado para fixar meu aprendizado e servir como referências futuras.

###Capítulo 1: Criando a base do sistema

1. Gerar APP_KEY caso não tiver sido gerada na instalação do laravel
  ```
    php artisan key:generate
  ```
2. Definir variáveis no .env
3. Mudar nome da aplicação 
  ```
    php artisan app:name [namespace]
  ```
4. Criar database
5. Em relação aos Models:
  - Criada pasta app\Models 
  - User transferi para nova paste e namespace corrigido (no config\auth.php também)
  - Category, Product, Client, Order, OrderItem models criados
6. Em relação às Migrations criadas:
  - create_categories_table
  - create_products_table
  - create_clients_table
  - create_orders_table
  - create_order_items_table
7. Em relação às Factories criadas:
  - Category
  - Product
  - Client
8. Em relação aos Seeds:
  - UserTableSeeder (Client sendo criado junto)
  - CategoryTableSeeder
  - ProductTableSeeder (não utilizado, ver CategoryTableSeeder)
9. Relacionamentos:
  - Category hasMany Product
  - Product belongsTo Category
  - User hasOne Client
  - Client hasOne User
  - Order hasMany OrderItem
  - Order belongsTo User
  - Order hasMany Product
  - OrderItem belongsTo Product
  - OrderItem belongsTo Order
  
###Capítulo 2: Repositories (padrão de projeto)

1. https://github.com/andersao/l5-repository
  - php artisan vendor:publish
  - Arrumar repository.php rootNamespace e models
  - Recriar todos os models já criados no Cap1 utilizando o make:repository
2. Criar Provider
  - Criar RepositoryServiceProvider (php artisan make:provider)
  - Fazer bind de todos os repositorios criados
  ```php
  /* dentro de register() */
  $this->app->bind(
  	'CodeDelivery\Repositories\CategoryRepository',
  	'CodeDelivery\Repositories\CategoryRepositoryEloquent'
  );
  ```
3. Adicionar repository criado a lista de providers (app.php)

###Capítulo 3: Sistema Administrativo

1. Instalar packages
  - https://github.com/bestmomo/scafold
    1. Adicionar ao composer.json
    ```
      "minimum-stability": "dev",
    ```
	2. composer require bestmomo/scafold
	3. Adicionar o Bestmomo\Scafold\ScafoldServiceProvider::class a lista de service provider
	4. php artisan vendor:publish
  - https://github.com/illuminate/html
	1. composer require illuminate/html
	2. Adicionar o Illuminate\Html\HtmlServiceProvider::class a lista de service provider
	3. Adicionar os Facades 
	```php
	  /* dentro de 'providers' em app.php */
      'Html'      => Illuminate\Html\HtmlFacade::class,
      'Form'      => Illuminate\Html\FormFacade::class,
	```
2. Controllers
  - CategoryController
  - ProductsController
3. Views
  - admin.categories
  - admin.products
4. Paginação
5. Rotas nomeadas 
6. Custom Requests
  - AdminCategoryRequest
  - AdminProductRequest
7. Refatorando Forms (_form)
8. Agrupando rotas
9. Middleware CheckRole (admin criado nas seeds)

###Capítulo 4: Clientes

1. ClientsController
2. AdminClientRequest
3. Rotas de admin.clients
4. Views de clients
5. CRUD Clients
6. ClientService
  - Recebe os repositorios de Cliente e de Usuário como parâmetros em seu construtor
  - create e update, utilizado para criar um usuário automaticamente com uma senha padrão antes de executar o create e o update do repository
  
###Capítulo 5: Pedidos

1. OrdersTableSeeder
  - Colocar na ModelFactory Order e OrderItem
2. OrdersController
3. Criar views admin.orders.index e edit
4. Relação de Order com Client
  - OrderBelongsToClient
5. Seed para deliveryman
6. Função getDeliveryman no UserRepository para criar select de deliveryman

###Capítulo 6: Checkout

1. php artisan make:repository Cupom
2. php artisan make:migration create_cupoms_table --create=cupoms
  - atenção: adicionar foreign key cupom_id na tabela orders e função down
  ```php
  Schema::table('orders',function(Blueprint $table) {
		$table->dropForeign('orders_cupom_id_foreign');
		$table->dropColumn('cupom_id');
	});
  ```
3. Criar Seeder, Factory e adicionar fillables
4. CupomsController
5. Rotas e views para admin.cupoms
6. Refatorar _form e fazer AdminCupomRequest
7. CheckoutController com depencias de OrderRepository, UserRepository e ProductRepository
8. Criação novas rotas e views a partir de 'customer' (no lugar de admin)
9. Javascript para adicionar novos produtos na tela de pedidos 
10. Criação do Service para as orders. Conceitos importantes: 
  - \DB::beginTransaction()
  - try/catch 
  - \DB::commit (dentro do try)
  - DB::rollback() (dentro to catch)	
  - OrderService com dependencias de OrderRepository, CupomRepository e ProductRepository
11. Rotas e funções para customer.index e customer.create
12. Permissões de usuários (alteração no middleware checkrole)

###Capítulo 7: OAuth 2

1. Instalando package https://github.com/lucadegasperi/oauth2-server-laravel/wiki
2. Corrigir CSRF (middleware -> kernel.php)
  - Configurar dentro do VerifyCsrfToken para ignorar as rotas da API
  ```php
   protected $except = [
        'oauth/access_token',
		'api/*'
    ];
  ```
3. Authorization Server
  - https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Choosing-a-Grant
  - Escolhido: https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Implementing-an-Authorization-Server-with-the-Password-Grant 
  ```php
  'password' => [
		'class' => '\League\OAuth2\Server\Grant\PasswordGrant',
		'callback' => '\CodeDelivery\OAuth2\PasswordVerifier@verify',
		'access_token_ttl' => 3600
	]
  ```
  - Criar client se for testar requisição do token
4. Refresh token
  - https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Implementing-an-Authorization-Server-with-the-Refresh-Token-Grant
5. Criando rotas para api
  - Route::group(['prefix' => 'api', 'middleware' => 'oauth' , 'as' => 'api.'], function(){ 	//  });
  
###Capítulo 8: Criando API de Cliente

1. Separação das rotas para os diferentes papeis de consumidores da API (client e deliveryman no caso)
  ```php
  Route::group(['prefix' => 'api', 'middleware' => 'oauth' , 'as' => 'api.'], function() 
  {
      Route::group(['prefix' => 'client', 'as' => 'client.'], function() {
      });
      Route::group(['prefix' => 'deliveryman', 'as' => 'deliveryman.'], function() {
      });
  });
  ```
2. Criando middleware oauth.checkrole
  - Inserir no kernel em $routeMiddleware e aplicar nas rotas criadas anteriormente
	```php
	// CodeDelivery\Http\Middleware\OAuthCheckRole 
	public function handle($request, Closure $next, $role)
	{
		$id = Authorizer::getResourceOwnerId();
		$user = $this->userRepository->find($id);

		if ($user->role != $role){
			abort(403,'Acess Forbidden');
		}
		return $next($request);
	}
	```
3. Criando RESTful Controller e rotas
  - ClientCheckoutController
  ```php
  //routes.php
  Route::resource('order', 
  	'Api\Client\ClientCheckoutController',
  	['except' => ['create','edit']]
  );
  ```
4. Criando funcionalidades
  - listagem de orders (index)
  - cadastro de order (store)
  - mostrar uma order (show)
5. Atenção especial:
  - nas funções da API a validação do usuário deve ser feita através do Authorizer
  ```php
  $id = Authorizer::getResourceOwnerId();
        $clientId = $this->userRepository->find($id)->client->id;
  ```
  
###Capítulo 9: Criando API de Deliveryman

1.Criando RESTful Controller e rotas
 - DeliverymanCheckoutController
```php
// routes.php
Route::resource('order',
  'Api\Deliveryman\DeliverymanCheckoutController',
  ['except' => ['create','edit','destroy','store']]
);
```
2.Criando funcionalidades do DeliverymanCheckoutController
  - listagem de orders (index)
  - mostrar uma order (show)
  - atualizar status da order (patch)
    - Atenção: dados enviados através de form-data e x-www-form-urlencoded são interpretados de maneira diferente pelo Request,no caso, utilizamos o x-www-form-urlencoded para que o parametro 'status' fosse lido e atualizado corretamente
3. Nova função adicionada no OrderService para atualizar o status
   ```php
   // OrderService.php
  public function updateStatus(Request $request, $id) 
  {
      $idDeliveryman = Authorizer::getResourceOwnerId();
      $order = $this->orderService->updateStatus($id,$idDeliveryman, $request->get('status'));
      if ($order){
          return $order;
      }
      abort(400,'Order não encontrada'); 
  }
  ```

###Capítulo 10: Validações e Serializações

1. Utilizar camada de validação (Requests) nos controllers da API 
  - Inserir Accept = application/json no HEADER da requisição para definir se a requisição é ajax ou json
2. CheckoutRequest criado
    - Validação de cupoms
    - Validação de items
      - Função para criação de rules dos items
      ```php
      public function buildRulesItems($key, array &$rules)
      {
          $rules["items.$key.product_id"] = 'required';
          $rules["items.$key.qtd"] = 'required';
      }
      ```
    - Atenção: Utilizar 'Illuminate\Http\Request as HttpRequest' no parâmetro da função rules para evitar conflito 
    com o Request que o CheckoutRequest herda
3. Aplicar o mesmo CheckoutRequest para validar o CheckoutController também (store)
4. Serializações
  - Fractal  - PHPLeague (http://fractal.thephpleague.com/)
  - https://github.com/andersao/l5-repository#presenters
  - Para utilizar:
    - Criar Transformer
    ```php
      php artisan make:transformer Order
    ```
    - Criar Presenter
    ```php
      php artisan make:presenter Order
    ```
    - Indicar no repositorio qual é o presenter usado
    ```php
    public function presenter()
    {
        return \CodeDelivery\Presenters\OrderPresenter::class;
    }
    ```
    - Não esquecer do 'use TransformableTrait;'
    - Atenção: para ignorar o presenter e continuar retornando o objeto Eloquent nas consultas deve-se utilizar
    o skipPresenter(). Ex:
    ```php
     $order = $this->orderRepository->skipPresenter()->with(['client','items.product','cupom'])->find($id);
    ```
5. Serialização de relacionamentos
  - protected $defaultIncludes = [''];  Includes que serão automaticamente adicionados
    - Criar funções include juntamente com o transformer do model relacionado. Ex:
    ```php
    public function includeCupom(Order $model)
    {
        if (!$model->cupom)
            return null;
        return $this->item($model->cupom, new CupomTransformer());
    }
    ```
  - protected $availablesIncludes = []; Includes que serão adicionados caso houver requisição
    - Enviar 'include'  no parâmetro da url da requisição com nomes dos includes que quiser adicionar (separados por virgula, sem espaços)
6. Escapando o presenter por padrão (para continuar resgatando o Eloquent normalmente)
  - Para isso é necessário ativar o skipPresenter como padrão no repositório desejado
  ```php
      protected $skipPresenter = true;
  ```
  - Explicitar o presenter nos controllers criados ClientCheckout e DeliverymanCheckout. Ex:
  ```php
      return $this->orderRepository
                  ->skipPresenter(false)
                  ->getByIdAndDeliveryman($id,$idDeliveryman);
  ```   