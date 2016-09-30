# Prelúdio

> Modelos são importantes. <br/>
> -- Oficial Alex J. Murphy / RoboCop

O objetivo deste guia é apresentar um conjunto das melhores prescrições de práticas e estilo para desenvolvimento Ruby on Rails 4. Esse é um guia complementar para o já existente liderado pela comunidade [Ruby coding style guide](https://github.com/bbatsov/ruby-style-guide).

Alguns dos conselhos aqui são aplicáveis somente no Rails 4.0+.

Você pode gerar um PDF ou uma cópia HTML deste guia usando
[Pandoc](http://pandoc.org/).

Traduções deste guia estão disponíveis nas seguintes linguagens:

* [Inglês](https://github.com/bbatsov/ruby-style-guide/blob/master/README.md)
* [Chinês simplificado](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [Chinês Traditional](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [Alemão](https://github.com/arbox/de-rails-style-guide/blob/master/README-deDE.md)
* [Japonês](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)
* [Russo](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [Turco](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [Coreano](https://github.com/pureugong/rails-style-guide/blob/master/README-koKR.md)
* [Vietnamita](https://github.com/CQBinh/rails-style-guide/blob/master/README-viVN.md)

# O Guia de Estilo Rails

Este guia de estilo Rails recomenda boas práticas para que os programadores reais de Rails possam escrever códigos que possam ser mantidos por outros programadores reais de Rails. Um guia de estilos que reflete o uso que o mundo real está acostumado e um guia de estilo que se prende a um ideal que foi rejeitado pelo povo supõe-se que o melhor é não usá-lo para nada - não importa quão bom seja.

O guia é separado em várias seções de regras relacionadas. Eu tentei adicionar a lógica por trás das regras (se estiver omitido, eu assumi que é bastante óbvio).

Eu não vim com todas as regras do nada - são na maioria baseadas na minhas extensa carreira como profissional de engenharia de software, comentários e sugestões de membros da comunidade Rails e vários recursos de programação Rails altamente recomendados.

## Tabela de conteúdos

* [Configuração](#configuração)
* [Rotas](#rotas)
* [Controladores](#controladores)
  * [Renderizando](#renderizando)
* [Modelos](#modelos)
  * [ActiveRecord](#activerecord)
  * [Consultas com ActiveRecord](#consultas-com-activerecord)
* [Migrações](#migrações)
* [Views](#views)
* [Internacionalização](#internacionalização)
* [Assets](#assets)
* [Mailers](#mailers)
* [Extensões de núcleo do Active Support](#extensões-de—núcleo-do-active-support)
* [Time](#time)
* [Bundler](#bundler)
* [Gerenciamento de processos](#gerenciamento-de-processos)

## Configuração

* <a name="config-initializers"></a>
  Coloque código de inicialização customizada em `config/initializers`. O código nessa pasta são executados quando a aplicação inicia.
<sup>[[link](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
Mantenha os códigos de inicialização de cada gem em um arquivo separado com o mesmo nome da gem, por exemplo `carrierwave.rb`, `active_admin.rb`, etc.
<sup>[[link](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Ajuste de acordo com as definições dos ambientes de desenvolvimento, teste e produção (nos arquivos correspondentes dentro de `config/environments/`)
<sup>[[link](#dev-test-prod-configs)]</sup>

  * Marcar assets adicionais para pré-compilação (se houver):

    ```Ruby
    # config/environments/production.rb
    # Pré-compila assets adicionais (application.js, application.css,
    # e todo non-JS/CSS já são adicionados)
    config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
    ```

* <a name="app-config"></a>
  Mantenha a configuração que é aplicável à todos ambientes no
  `config/application.rb` file.
<sup>[[link](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Crie um ambiente `staging` adicional que se assemelha ao `produção`.
<sup>[[link](#staging-like-prod)]</sup>

* <a name="yaml-config"></a>
  Mantenha qualquer configuração adicional em arquivos YAML dentro do diretório `config/`.
<sup>[[link](#yaml-config)]</sup>

  A partir do Rails 4.2, arquivos de configuração YAML podem ser facilmente carregados com o novo método `config_for`:

  ```Ruby
  Rails::Application.config_for(:arquivo_yaml)
  ```

## Rotas

* <a name="member-collection-routes"></a>
  Quando você precisar adicionar mais actions à um resource RESTful (você realmente precisa delas?) use rotas `member` e `collection`.
<sup>[[link](#member-collection-routes)]</sup>

  ```Ruby
  # ruim
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # bom
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # ruim
  get 'photos/search'
  resources :photos

  # bom
  resources :photos do
    get 'search', on: :collection
  end
  ```

* <a name="many-member-collection-routes"></a>
  Se você precisa definir múltiplas rotas `member/collection` use a alternativa de sintaxe em bloco.
<sup>[[link](#many-member-collection-routes)]</sup>

  ```Ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      # mais rotas
    end
  end

  resources :photos do
    collection do
      get 'search'
      # mais rotas
    end
  end
  ```

* <a name="nested-routes"></a>
  Use rotas aninhadas para expressar melhor o relacionamento entre os modelos do ActiveRecord.
<sup>[[link](#nested-routes)]</sup>

  ```Ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```

* <a name="namespaced-routes"></a>
  Se você precisa de rotas aninhadas com mais de 1 nível de profundidade, use a opção `shallow: true`. Isto vai livrar o usuário de longas urls como `posts/1/comments/5/versions/7/edit` e você de longos helpers de url como `edit_post_comment_version`.

  ```Ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

* <a name="namespaced-routes"></a>
  Use namespace em rotas para agrupar actions relacionadas.
<sup>[[link](#namespaced-routes)]</sup>

  ```Ruby
  namespace :admin do
    # Direciona /admin/products/* ao Admin::ProductsController
    # (app/controllers/admin/products_controller.rb)
    resources :products
  end
  ```

* <a name="no-wild-routes"></a>
  Nunca use o legado de controlador de rota. Esta rota fará todas actions em todos os controladores serem acessíveis via requisições GET.
<sup>[[link](#no-wild-routes)]</sup>

  ```Ruby
  # muito ruim
  match ':controller(/:action(/:id(.:format)))'
  ```

* <a name="no-match-routes"></a>
  Não use `match` para definir qualquer rota a não ser que haja necessidade de mapear múltiplos tipos de requisições dentre `[:get, :post, :patch, :put, :delete]` para uma única action usando a opção `:via`.
<sup>[[link](#no-match-routes)]</sup>

## Controladores

* <a name="skinny-controllers"></a>
  Mantenha o controlador pequeno - eles devem apenas receber informações da camada de view e não deveria conter qualquer regra de negócio (toda regra de negócio deveria naturalmente ficar dentro do modelo).
<sup>[[link](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Cada action do controlador deveria (idealmente) invocar somente um outro método além do inicial `find` ou `new`.
<sup>[[link](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Compartilhar não mais que duas variáveis de instância entre controlador e view.
<sup>[[link](#shared-instance-variables)]</sup>


### Renderizando

* <a name="inline-rendering"></a>
  Prefira usar render em um template a um inline.
<sup>[[link](#inline-rendering)]</sup>

```Ruby
# muito ruim
class ProductsController < ApplicationController
  def index
    render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>", type: :erb
  end
end

# bom
## app/views/products/index.html.erb
<%= render partial: 'product', collection: products %>

## app/views/products/_product.html.erb
<p><%= product.name %></p>
<p><%= product.price %></p>

## app/controllers/foo_controller.rb
class ProductsController < ApplicationController
  def index
    render :index
  end
end
```

* <a name="plain-text-rendering"></a>
  Prefira `render plain:` ao `render text:`.
<sup>[[link](#plain-text-rendering)]</sup>

```Ruby
# ruim - atribui tipo MIME a um `text/html`
...
render text: 'Ruby!'
...

# ruim - requer declaração explícita do tipo MIME
...
render text: 'Ruby!', content_type: 'text/plain'
...

# bom - curto e preciso
...
render plain: 'Ruby!'
...
```

* <a name="http-status-code-symbols"></a>
  Prefira [símbolos correspondentes](https://gist.github.com/mlanett/a31c340b132ddefa9cca) a um código numérico de status HTTP. Eles são significativos e não parecem números “mágicos” para códigos de status HTTP menos conhecidos.
<sup>[[link](#http-status-code-symbols)]</sup>

```Ruby
# ruim
...
render status: 500
...

# bom
...
render status: :forbidden
...
```

## Modelos

* <a name="model-classes"></a>
  Introduza livremente classes de modelos que não usem ActiveRecord.
<sup>[[link](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Nomeie modelos com significantes (porém pequenos) nomes sem abreviações.
<sup>[[link](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  Se você precisa de objetos de modelos que possuam comportamento do ActiveRecord (como validações)
  sem a funcionalidade do database do ActiveRecord use a gem
  [ActiveAttr](https://github.com/cgriego/active_attr).
<sup>[[link](#activeattr-gem)]</sup>

  ```Ruby
  class Message
    include ActiveAttr::Model

    attribute :name
    attribute :email
    attribute :content
    attribute :priority

    attr_accessible :name, :email, :content

    validates :name, presence: true
    validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
    validates :content, length: { maximum: 500 }
  end
  ```

  Para um exemplo mais completo vejo o 
  [RailsCast sobre o assunto](http://railscasts.com/episodes/326-activeattr).

* <a name="model-business-logic"></a>
  A não ser que tenham alguma importância para o regra de negócio, não ponha métodos no seu modelo que somente formata seus dados (como código gerando HTML). Estes métodos provavelmente só serão chamados na camada de view, então o lugar deles é nos helpers. Mantenhas seus modelos somente para regras de negócios e persistência de dados.
<sup>[[link](#model-business-logic)]</sup>

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Evite alterar os padrões do ActiveRecord (nomes de tabela, chave primária, etc) a não ser que você tenha uma razão muito boa pra isso (como uma base de dados que não está sob seu controle).
<sup>[[link](#keep-ar-defaults)]</sup>

  ```Ruby
  # ruim - não faça isso se você pode modificar o schema
  class Transaction < ActiveRecord::Base
    self.table_name = 'order'
    ...
  end
  ```

* <a name="macro-style-methods"></a>
  Agrupe métodos estilo macro (`has_many`, `validates`, etc) no início da definição da classe.
<sup>[[link](#macro-style-methods)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    # mantenha o escopo padrão primeiro (se houver)
    default_scope { where(active: true) }

    # contantes entram depois
    COLORS = %w(red green blue)

    # logo após, colocamos macros attr relacionados
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # Rails4+ enums depois dos macros attr, prefira sintaxe de hash
    enum gender: { female: 0, male: 1 }

    # seguido de associações macros
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # e validações macros
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true }

    # depois nós temos os callbacks
    before_save :cook
    before_save :update_username_lower

    # outras macros (como as do devise) devem ser colocadas depois dos callbacks

    ...
  end
  ```

* <a name="has-many-through"></a>
  Prefira `has_many :through` ao `has_and_belongs_to_many`. Usar `has_many
  :through` permite atributos e validações adicionais no modelo.
<sup>[[link](#has-many-through)]</sup>

  ```Ruby
  # não tão bom - usando has_and_belongs_to_many
  class User < ActiveRecord::Base
    has_and_belongs_to_many :groups
  end

  class Group < ActiveRecord::Base
    has_and_belongs_to_many :users
  end

  # forma preferida - usando has_many :through
  class User < ActiveRecord::Base
    has_many :memberships
    has_many :groups, through: :memberships
  end

  class Membership < ActiveRecord::Base
    belongs_to :user
    belongs_to :group
  end

  class Group < ActiveRecord::Base
    has_many :memberships
    has_many :users, through: :memberships
  end
  ```

* <a name="read-attribute"></a>
  Prefira `self[:attribute]` ao `read_attribute(:attribute)`.
<sup>[[link](#read-attribute)]</sup>

  ```Ruby
  # ruim
  def amount
    read_attribute(:amount) * 100
  end

  # bom
  def amount
    self[:amount] * 100
  end
  ```

* <a name="write-attribute"></a>
  Prefira `self[:attribute] = value` ao `write_attribute(:attribute, value)`.
<sup>[[link](#write-attribute)]</sup>

  ```Ruby
  # ruim
  def amount
    write_attribute(:amount, 100)
  end

  # bom
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  Sempre use as novas [validações “sexy”](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
<sup>[[link](#sexy-validations)]</sup>

  ```Ruby
  # ruim
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # bom
  validates :email, presence: true, length: { maximum: 100 }
  ```

* <a name="custom-validator-file"></a>
  Quando uma validação customizada é usada mais de uma vez ou a validação é o mapeamento de alguma expressão regular, crie um arquivo de validação customizado.
<sup>[[link](#custom-validator-file)]</sup>

  ```Ruby
  # ruim
  class Person
    validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
  end

  # bom
  class EmailValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
    end
  end

  class Person
    validates :email, email: true
  end
  ```

* <a name="app-validators"></a>
  Mantenha validações customizadas dentro de `app/validators`.
<sup>[[link](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Considere extrair validações customizadas para uma gem compartilhada se você está mantendo várias aplicações relacionadas ou se a validação customizada é genérica o suficiente.
<sup>[[link](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  Use escopos nomeados livremente.
<sup>[[link](#named-scopes)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  Quando um escopo nomeado é definido com um lambda e os parâmetros ficam muito complicados, é preferível fazer um método de classe que sirva ao mesmo propósito do escopo nomeado e retorne um objeto do tipo `ActiveRecord::Relation`. Sem dúvida você pode definir escopos ainda mais simples como este.
<sup>[[link](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

* <a name="beware-update-attribute"></a>
  Cuidado com o comportamento do método
  [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute). 
  Ele não executa as validações de modelo (diferente do `update_attributes`) e
  poderia corromper facilmente o estado do modelo.
<sup>[[link](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Use URLs amigáveis. Mostre algum atributo descritivo do modelo na URL diferente do seu `id`. Existe mais de uma maneira de conseguir isso:
<sup>[[link](#user-friendly-urls)]</sup>

  * Sobrescreva o método `to_param` do modelo. Esse método é usado pelo Rails para construir uma URL para o objeto. A implementação padrão retorna o `id` do objeto como uma String.  Ele pode ser sobrescrito para incluir algum atributo humano-legível.

      ```Ruby
      class Person
        def to_param
          "#{id} #{name}".parameterize
        end
      end
      ```

  A fim de converter isto para um valor de URL amigável, `parameterize` deve ser chamado na string. O `id` do objeto precisa estar no início para que ele possa ser achado pelo método `find` do ActiveRecord.

  * Use a gem `friendly_id`. Ela permite a criação de URLs humano-legíveis através do uso de algum atributo descritivo do modelo ao invés do seu `id`.

      ```Ruby
      class Person
        extend FriendlyId
        friendly_id :name, use: :slugged
      end
      ```

  Veja a [documentação da gem](https://github.com/norman/friendly_id) para mais informações sobre como usar.

* <a name="find-each"></a>
  Use `find_each` para iterar sobre uma coleção de objetos do AR. Fazer um looping através de uma coleção de registros da base de dados (usando o método `all`, por exemplo)
  é muito ineficiente já que ele irá tentar instanciar todos os objetos de uma só vez.
  Nesse caso, métodos de processamento batch te permitem trabalhar com registros em
  batches, reduzindo significativamente o consumo de memória.
<sup>[[link](#find-each)]</sup>


  ```Ruby
  # ruim
  Person.all.each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').each do |person|
    person.party_all_night!
  end

  # bom
  Person.find_each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').find_each do |person|
    person.party_all_night!
  end
  ```

* <a name="before_destroy"></a>
  Desde que o [Rails criou callbacks para associações dependentes](https://github.com/rails/rails/issues/3458), sempre chame callbacks
  `before_destroy` que façam validação com `prepend: true`.
<sup>[[link](#before_destroy)]</sup>

  ```Ruby
  # ruim (roles sempre serão deletados mesmo que super_admin? seja true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # bom
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

* <a name="has_many-has_one-dependent-option"></a>
  Defina a opção `dependent` para as associações `has_many` e `has_one`.
<sup>[[link](#has_many-has_one-dependent-option)]</sup>

  ```Ruby
  # ruim
  class Post < ActiveRecord::Base
    has_many :comments
  end

  # bom
  class Post < ActiveRecord::Base
    has_many :comments, dependent: :destroy
  end
  ```

* <a name="save-bang"></a>
  Quando persistir objetos AR sempre use o método com o ponto de exclamação ou manipule o valor retornado pelo método.
  Isso se aplica ao `create`, `save`, `update`, `destroy`, `first_or_create` e `find_or_create_by`.
<sup>[[link](#save-bang)]</sup>

  ```Ruby
  # ruim
  user.create(name: 'Bruce')

  # ruim
  user.save

  # bom
  user.create!(name: 'Bruce')
  # ou
  bruce = user.create(name: 'Bruce')
  if bruce.persisted?
    ...
  else
    ...
  end

  # bom
  user.save!
  # ou
  if user.save
    ...
  else
    ...
  end
  ```

### Consultas com ActiveRecord

* <a name="avoid-interpolation"></a>
  Evite interpolação de strings em consultas, isso fará seu código suscetível à ataques de SQL injection.
<sup>[[link](#avoid-interpolation)]</sup>

  ```Ruby
  # ruim - parâmetros serão interpolados sem escape
  Client.where("orders_count = #{params[:orders]}")

  # bom - parâmetros serão propriamente escapados
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Considere usar espaços reservados nomeados ao invés de espaços reservados posicionais quando tiver mais de um espaço reservado na sua consulta.
<sup>[[link](#named-placeholder)]</sup>

  ```Ruby
  # até vai..
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # bom 
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Favoreça o uso de `find` ao `where` quando precisar recuperar um único registro pelo id.
<sup>[[link](#find)]</sup>

  ```Ruby
  # ruim
  User.where(id: id).take

  # bom
  User.find(id)
  ```

* <a name="find_by"></a>
  Favoreça o uso de `find_by` ao `where` e `find_by_attribute`
quando você precisar recuperar um único registro por alguns atributos.
<sup>[[link](#find_by)]</sup>

  ```Ruby
  # ruim
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # ruim
  User.find_by_first_name_and_last_name('Bruce', 'Wayne')

  # bom
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="where-not"></a>
  Favoreça o uso de `where.not` ao SQL.
<sup>[[link](#where-not)]</sup>

  ```Ruby
  # ruim
  User.where("id != ?", id)

  # bom
  User.where.not(id: id)
  ```
* <a name="squished-heredocs"></a>
  Quando especificar uma consulta explícita em um método tal como `find_by_sql`, use
  heredocs com `squish`. Isso permite formatar legivelmente o SQL com quebra de linhas e indentações, enquanto suporta destaque de sintaxe em muitas ferramentas (incluindo o GitHub, Atom e RubyMine).
<sup>[[link](#squished-heredocs)]</sup>

  ```Ruby
  User.find_by_sql(<<-SQL.squish)
    SELECT
      users.id, accounts.plan
    FROM
      users
    INNER JOIN
      accounts
    ON
      accounts.user_id = users.id
    # further complexities...
  SQL
  ```

  [`String#squish`](http://apidock.com/rails/String/squish) remove os caracteres de indentação e de nova linha para que o log do servidor mostra uma string de SQL fluida em ve de algo como isso:

  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```

## Migrações

* <a name="schema-version"></a>
  Mantenha o `schema.rb` (ou `structure.sql`) sob controle de versão.
<sup>[[link](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Use `rake db:schema:load` em vez de `rake db:migrate` para inicializar um banco de dados vazio.
<sup>[[link](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Imponha valores padrões nas próprias migrações em vez de colocar na camada de aplicação.
<sup>[[link](#default-migration-values)]</sup>

  ```Ruby
  # ruim - aplicação impõe valor padrão
  def amount
    self[:amount] or 0
  end
  ```

  Mesmo a imposição de padrões só no Rails sendo defendida por muitos desenvolvedores Rails, essa é uma abordagem extremamente frágil que deixa seus dados vulneráveis a muitos erros de aplicação. E você terá que considerar o fato de que aplicações não triviais compartilham um banco de dados com outras aplicações, logo impor integridade de dados na aplicação Rails é impossível.

* <a name="foreign-key-constraints"></a>
  Imponha restrições de chave estrangeira. Como do Rails 4.2, ActiveRecord
  suporta restrições de chave estrangeira nativamente.
  <sup>[[link](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  Ao escrever migrações construtivas (adicionar tabelas ou colunas) use o método `change` ao invés dos métodos `up` e `down`.
  <sup>[[link](#change-vs-up-down)]</sup>

  ```Ruby
  # jeito antigo
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # o jeito novo
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Não use classes modelo em migrações. As classes modelos estão constantemente evoluindo e, em algum ponto no futuro, as migrações que costumavam funcionar podem parar por causa das mudanças no modelo usado.
<sup>[[link](#no-model-class-migrations)]</sup>

* <a name="meaningful-foreign-key-naming"></a>
  Nomeie sua chave estrangeira explicitamente em vez de confiar nos nomes de chave estrangeira automaticamente gerados pelo Rails. (http://edgeguides.rubyonrails.org/active_record_migrations.html#foreign-keys)

  ```Ruby
  # ruim
  class AddFkArticlesToAuthors < ActiveRecord::Migration
    def change
      add_foreign_key :articles, :authors
    end
  end

  # bom
  class AddFkArticlesToAuthors < ActiveRecord::Migration
    def change
      add_foreign_key :articles, :authors, name: :articles_author_id_fk
    end
  end
  ```

<sup>[[link](#meaningful-foreign-key-naming)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Nunca chame a camada de modelo diretamente da view.
<sup>[[link](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Nunca faça formatação complexa em views, exporte a formatação para um método no helper ou no modelo.
<sup>[[link](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Reduza a duplicação de código através do uso de partials e layouts.
<sup>[[link](#partials)]</sup>

## Internacionalização

* <a name="locale-texts"></a>
  Nenhuma string ou outra configuração local específica deveria ser usada nas views, modelos e controladores. 
  Esses textos deveriam ser movidos para os arquivos de localidade no diretório `config/locales`.
<sup>[[link](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  Quando as labels de um modelo do ActiveRecord precisarem ser traduzidas, use o escopo `activerecord`:
<sup>[[link](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  Logo `User.model_name.human` irá retornar "Member" e
  `User.human_attribute_name("name")` irá retornar "Full name". Essas
  traduções de atributos serão usadas como labels nas views.


* <a name="organize-locale-files"></a>
  Separe os textos usados nas views das traduções de atributos do ActiveRecord.
  Coloque os arquivos de localidade para modelos na pasta `locales/models` e os
  textos usados nas views na pasta `locales/views`.
<sup>[[link](#organize-locale-files)]</sup>

  * Quando a organização dos arquivos de localidade estiverem prontos com os diretórios adicionais,
    estes diretórios devem ser descritos no arquivo `application.rb` em ordem de carregamento.

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Coloque as opções de localização compartilhada, tal como datas e formatos de moedas, em arquivos sob a raiz do diretório `locales`.
<sup>[[link](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Use o formato curto dos métodos do I18n: `I18n.t` em vez de `I18n.translate`
  e `I18n.l` em vez de `I18n.localize`.
<sup>[[link](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Use pesquisa ”preguiçosa” para os textos nas views. Digamos que temos a seguinte estrutura:
<sup>[[link](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  O valor de `users.show.title` pode ser pesquisado no template 
  `app/views/users/show.html.haml` dessa forma:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Use as chaves separadas por ponto nos controladores e nos modelos ao invés de especificar 
  a opção `:scope`. As chamadas separadas por ponto são mais fáceis de ler e traçar sua hierarquia.
<sup>[[link](#dot-separated-keys)]</sup>

  ```Ruby
  # ruim
  I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]

  # bom
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Informações mais detalhadas sobre o Rails I18n podem ser encontradas no [Rails
  Guides](http://guides.rubyonrails.org/i18n.html)
<sup>[[link](#i18n-guides)]</sup>

## Assets

Use o [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) para aumentar a organização dentro da sua aplicação.

* <a name="reserve-app-assets"></a>
  Reserve `app/assets` para stylesheets, javascripts, ou imagens customizadas.
<sup>[[link](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Use `lib/assets` para suas próprias bibliotecas que realmente não se encaixam no escopo da aplicação.
<sup>[[link](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Código de terceiros como [jQuery](http://jquery.com/) ou
  [bootstrap](http://twitter.github.com/bootstrap/) devem ser colocados em
  `vendor/assets`.
<sup>[[link](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  Quando possível, use versões de assets em gem (ex.
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[link](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Nomeie os mailers como `AlgumaCoisaMailer`. Sem o sufixo Mailer não fica claro qual é o Mailer e a quais views ele está relacionado.
<sup>[[link](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Providencie ambos templates de view em HTML texto plano.
<sup>[[link](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Habilite elevação de erros ao tentar enviar email no ambiente de desenvolvimento.
  Os erros são desabilitados por padrão.
<sup>[[link](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Use um servidor SMTP local como o
  [Mailcatcher](https://github.com/sj26/mailcatcher) no ambiente de desenvolvimento.
<sup>[[link](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # mais configurações
  }
  ```

* <a name="default-hostname"></a>
  Providencie configurações padrões para o nome do host.
<sup>[[link](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'seu_site.com' }

  # na sua classe mailer
  default_url_options[:host] = 'seu_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  Se você precisa usar um link para o seu site no email, sempre use o `_url`, e não métodos
  `_path`. Os métodos `_url` incluem o nome do host e os métodos `_path` não.
<sup>[[link](#url-not-path-in-email)]</sup>

  ```Ruby
  # ruim
  Você pode encontrar mais informações sobre esse curso 
  <%= link_to 'aqui', curso_path(@curso) %>

  # bom
  ocê pode encontrar mais informações sobre esse curso 
  <%= link_to 'aqui', curso_url(@curso) %>
  ```

* <a name="email-addresses"></a>
  Formate os endereços de `from` e `to` devidamente. Use o seguinte formato:
<sup>[[link](#email-addresses)]</sup>

  ```Ruby
  # na sua classe mailer
  default from: 'Seu nome <info@seu_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Tenha certeza que o método de envio de email para seu ambiente de teste está configurado para `test`:
<sup>[[link](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  O método de envio para desenvolvimento e produção devem ser `smtp`:
<sup>[[link](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  Quando enviar emails html todos os estilos devem estar em linha, já que alguns clientes de email 
  têm problemas com estilos externos. Entretanto, isso os faz mais difíceis de manter 
  e leva a uma duplicação de código. Existem duas gems similares que transformam os estilos em os põe em correspondentes tags html:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) e
  [roadie](https://github.com/Mange/roadie).
<sup>[[link](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Enviar emails enquanto gera resposta para página deveria ser evitado. Isso causa atrasos 
  no carregamento da página e pode dar timeout na requisição se múltiplos emails são enviados.
  Para superar isso, emails podem ser enviados em um processo em background com a ajuda
  da gem [sidekiq](https://github.com/mperham/sidekiq).
<sup>[[link](#background-email)]</sup>


## Extensões de núcleo do Active Support

* <a name="try-bang"></a>
  Prefira o operador de navegação seguro do Ruby 2.3's `&.` em vez de `ActiveSupport#try!`.
<sup>[[link](#try-bang)]</sup>

```ruby
# ruim
obj.try! :fly

# bom
obj&.fly
```

* <a name="active_support_aliases"></a>
  Prefira métodos da Biblioteca Padrão do Ruby ao invés de alias do `ActiveSupport`.
<sup>[[link](#active_support_aliases)]</sup>

```ruby
# ruim
'the day'.starts_with? 'th'
'the day'.ends_with? 'ay'

# bom
'the day'.start_with? 'th'
'the day'.end_with? 'ay'
```

* <a name="active_support_extensions"></a>
  Prefira a Biblioteca Padrão do Ruby ao invés de extensões incomuns do ActiveSupport.
<sup>[[link](#active_support_extensions)]</sup>

```ruby
# ruim
(1..50).to_a.forty_two
1.in? [1, 2]
'day'.in? 'the day'

# bom
(1..50).to_a[41]
[1, 2].include? 1
'the day'.include? 'day'
```

* <a name="inquiry"></a>
  Prefira os operadores de comparação do Ruby ao invés dos `Array#inquiry`, `Numeric#inquiry` e `String#inquiry` do ActiveSupport.
<sup>[[link](#inquiry)]</sup>

```ruby
# ruim - String#inquiry
ruby = 'two'.inquiry
ruby.two?

# bom
ruby = 'two'
ruby == 'two'

# ruim - Array#inquiry
pets = %w(cat dog).inquiry
pets.gopher?

# bom
pets = %w(cat dog)
pets.include? 'cat'

# ruim - Numeric#inquiry
0.positive?
0.negative?

# bom
0 > 0
0 < 0
```

## Time

* <a name="tz-config"></a>
  Configure seu timezone devidamente no `application.rb`.
<sup>[[link](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # opcional - note que ele só pode ser :utc ou :local (o padrão é :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Não use `Time.parse`.
<sup>[[link](#time-parse)]</sup>

  ```Ruby
  # ruim
  Time.parse('2015-03-02 19:05:37') # => Irá assumir que a string com a hora que foi passada está no timezone do sistema.

  # bom
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Não use `Time.now`.
<sup>[[link](#time-now)]</sup>

  ```Ruby
  # ruim
  Time.now # => Retorna hora do sistema e ignora sua configuração de timezone.

  # bom
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # mesma coisa só que menor.
  ```

## Bundler

* <a name="dev-test-gems"></a>
  Ponha as gems que são usadas só em desenvolvimento e teste dentro de um grupo apropriado no Gemfile.
<sup>[[link](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Use somente gems estáveis nos seus projetos. Se você está pensando em usar uma gem pouco conhecida, você deveria dar uma revisada cuidadosa no código fonte dela primeiro.
<sup>[[link](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  Gems específicas de sistemas operacionais irão, por padrão, resultar em constantes mudanças no 
  `Gemfile.lock` para projetos com muitos desenvolvedores usando diferentes sistemas operacionais. Adicione todas a gems específicas do OS X ao grupo `darwin` no Gemfile, e
  todas as gems específicas do Linux ao grupo `linux`:
<sup>[[link](#os-specific-gemfile-locks)]</sup>

  ```Ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  Para incluir as gems apropriadas nos ambientes corretos, adicione o seguinte ao `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Não remova o `Gemfile.lock` do controle de versão. Ele não é um tipo de arquivo gerado randomicamente - ele te dá certeza de que todos os membros da sua equipe tenham as mesmas versões de gems quando executarem o `bundle install`.
<sup>[[link](#gemfile-lock)]</sup>

## Gerenciamento de processos

* <a name="foreman"></a>
  Se seu projeto depende de vários processos externos use o
  [foreman](https://github.com/ddollar/foreman) para gerenciá-los.
<sup>[[link](#foreman)]</sup>

# Leitura Adicional

Existem alguns recursos excelentes no estilo Rails que você deveria considerar, se você tiver tempo sobrando:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Rails 4 Test Prescriptions](https://pragprog.com/book/nrtest2/rails-4-test-prescriptions)
* [Better Specs for RSpec](http://betterspecs.org)

# Contribuindo

Nada escrito neste manual está escrito em pedra. O meu desejo é trabalhar em conjunto com todos interessados no estilo de programação Rails, para que por fim assim nós possamos criar um recurso que irá beneficiar a comunidade de Ruby inteira. Sinta-se livre para abrir tickets ou enviar pull requests com melhorias. Desde já agradeço pela sua ajuda!

Você pode também pode ajudar o projeto (e o RuboCop) com contribuições financeiras via
[gittip](https://www.gittip.com/bbatsov).

[![Ajuda via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## Como contribuir?

Isso é fácil! É só seguir as [orientações de contribuição](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# Licensa

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
Esse trabalho está licenciado sob [Creative Commons Attribution 3.0 Unported
License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Avise a todos

Um guia de estilo voltado para a comunidade é de pouca utilidade se a comunidade não sabe sobre sua existência. Tweet sobre o guia, compartilhe com seus amigos e colegas. Todo comentário, sugestão e opinião que tivermos fará o guia só um pouco melhor. E nós queremos ter o melhor guia possível, não é mesmo?

Até mais,<br/>
[Bozhidar](https://twitter.com/bbatsov)
