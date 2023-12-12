# Preparando ambiente
Projeto anterior disponível no link: https://github.com/guilhermeonrails/alura_space

Criação da virtualenv com o comando `virtualenv .venv`.

> O que funcionou na minha máquina foi o comando `python -m venv .venv`.

Ativação do ambiente virtual: `.venv/Scripts/activate`.

Instalação das dependências com `pip install -r requirements.txt`.

# Nomes dinâmicos
Vamos simular um banco de dados, fornecendo para o template um dicionário contendo informações específicas de duas fotos que aparecerão no site.

Mudanças no arquivo `galerias/views.py`:
```python
from django.shortcuts import render

def index(request):
    dados = {
        1 : {
            "nome" : "Nebulosa de Carina",
            "legenda" : "webbtelescope.org / NASA / James Webb",
        },
        2 : {
            "nome" : "Galáxia NGC 1079",
            "legenda" : "nasa.org / NASA / Hubble",
        },
    }
    return render(request, 'galeria/index.html', {'cards' : dados})
# Resto do código.
```
> Repare que foi acrescentado um terceiro parâmetro na função `render`. Esse parâmetro é um dicionário com diversas variáveis que serão renderizadas no template.

Para usar o dicionário fornecido pela view, usamos uma estrutura de repetição no Django, conforme arquivo `galeria/index.html` a seguir:
```html
<!-- Resto do código -->
{% for foto_id, info in cards.items %}
<li class="card">
    <a href="{% url 'imagem' %}">
        <img class="card__imagem" src="{% static '/assets/imagens/galeria/carina-nebula.png' %}">
    </a>
    <span class="card__tag">Estrelas</span>
    <div class="card__info">
        <p class="card__titulo">{{ info.nome }}</p>
        <div class="card__texto">
            <p class="card__descricao">{{ info.legenda }}</p>
            <!-- Resto do código -->
        </div>
    </div>
</li>
{% endfor %}
<!-- Resto do código -->
```

A estrutura de repetição usada para dicionários é a seguinte:
```html
{% for chave, objeto_valor in dicionario.items %}
    Chave: {{ chave }}
    Objeto que representa o valor: {{ objeto_valor }}
{% endfor %}
```
Os loops em dicionários retornam apenas as chaves do dicionário. Para obter duas variáveis (a chave e o valor), iteramos o objeto `items` do dicionário. Cada valor no dicioinário pode conter outros atributos internos (por exemplo, o nome e a legenda para cada informação no dicionário).

Para invocarmos uma variável, nós a colocamos dentro de chaves duplas (`{{ variavel }}`).

# Banco de dados
Usamos classes que estendem de `django.db.models.Model` para cada tabela que precisarmos para o banco de dados.

Mudanças no arquivo `galeria/models.py`:
```python
from django.db import models

class Fotografia(models.Model):
    nome = models.CharField(max_length=100, null=False, blank=False)
    legenda = models.CharField(max_length=150, null=False, blank=False)
    descricao = models.TextField(null=False, blank=False)
    foto = models.CharField(max_length=100, null=False, blank=False)

    def __str__(self):
        return f"Fotografia [nome = {self.nome}]"
```

Uma vez que as classes de modelo forem criadas, precisamos criar as migrações (script de alteração no banco de dados, escritos em Python para o Django) antes de levá-las ao BD.
Isso é feito com o comando `python manage.py makemigrations`:
```
(.venv) PS D:\alura\django-admin> python manage.py makemigrations
Migrations for 'galeria':
  galeria\migrations\0001_initial.py
    - Create model Fotografia
(.venv) PS D:\alura\django-admin> 
```
> Para cada aplicativo, é criado um arquivo de migration dentro do diretóroi `migrations`. No exemplo, foi criada a migration com o nome `0001_initial.py`.

Depois que as migrações forem criadas, elas poderão ser executadas com o comando `python manage.py migrate`:

```
(.venv) PS D:\alura\django-admin> python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, galeria, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying galeria.0001_initial... OK
  Applying sessions.0001_initial... OK
(.venv) PS D:\alura\django-admin> 
```
> Repare que são mencionados diferentes arquivos de migrate, cada um prefixado com o nome do aplicativo ao qual está associado (`galeria`, `sessions`, `contenttypes`, `auth` e `admin`).

Para visualizarmos como ficou o banco SQLite, usamos a extensão SQLite Viewer (publicada pelo Florian Klampfer) do VS Code. Para visualizar o conteúdo banco, clique com o botão direito sobre o banco de dados e escolha a opção `"Abrir Com (Open With...)"` e em seguida escolher o editor SQLite Viewer.

Com essa extensão, podemos confirmar que as tabelas de cada aplicativo do projeto foram geradas no SQLite.

# Criando dados
Vamos inserir um objeto de modelo usando o shell do Django (comando `python manage.py shell`):

```
(.venv) PS D:\alura\django-admin> python manage.py shell
Python 3.11.7 (tags/v3.11.7:fa7a6f2, Dec  4 2023, 19:24:49) [MSC v.1937 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.       
(InteractiveConsole)
>>> from galeria.models import Fotografia
>>> foto = Fotografia(nome="Nebulosa de Carina", legenda="webbtelescope.org / NASA / James Webb", foto="carina-nebula.png")
>>> foto.save()
>>> Fotografia.objects.all()
<QuerySet [<Fotografia: Fotografia [nome = Nebulosa de Carina]>]>
>>>
```
> Passo a passo para salvar um objeto no banco após a abertura do shell do Django: 
> 1. Importamos a classe `Fotografia`:
> ```from galeria.models import Fotografia```
> 2. Criamos uma instância de Fotografia e a armazenamos em uma variável:
> ```foto = Fotografia(nome="Nebulosa de Carina", legenda="webbtelescope.org / NASA / James Webb", foto="carina-nebula.png")```
> 3. Salvamos a instância no banco com o método `save()`:
> ```foto.save()```
> 4. Para confirmarmos a inserção do objeto, consultamos a lista de objetos da classe com o comando `Classe.objects.all()`:
> ```Fotografia.objects.all()```

## Offtopic: como importar um aplicativo em `settings.py`
Costuma ser ensinado que basta inserir o nome do aplicativo na lista de apps instalados (`INSTALLED_APPS`) nas configurações do Django (`settings.py`):
```python
INSTALLED_APPS = [
    # Demais aplicativos...
    'galeria',
]
```

Uma boa prática é importar o arquivo que extende da classe `AppConfig`, que costuma ficar localizado no módulo `apps` do aplicativo. Por exemplo, o arquivo `galeria/apps.py` contém a classe `GaleriaConfig` (que extende de `AppConfig`):
```python
from django.apps import AppConfig

class GaleriaConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'galeria'
```

Nesse caso, o arquivo `settings.py` ficaria assim:
```python
INSTALLED_APPS = [
    # Demais aplicativos...
    'galeria.apps.GaleriaConfig',
]
```
> **Pergunta feita para a Luri:**
> Por que é melhor importarmos uma classe AppConfig ao invés de apenas referenciarmos o nome do aplicativo na lista INSTALLED_APPS do arquivo `settings.py`?
> 
> **Resposta da Luri:**
> Importar uma classe AppConfig ao invés de apenas referenciar o nome do aplicativo na lista INSTALLED_APPS do arquivo `settings.py` é uma boa prática porque a classe AppConfig permite que você personalize e configure o comportamento do aplicativo de forma mais flexível. Ela fornece métodos e atributos que podem ser usados para definir configurações adicionais, como o nome amigável do aplicativo, configurações de banco de dados, configurações de URL, entre outras. Dessa forma, você tem mais controle sobre o comportamento do aplicativo e pode organizar melhor as configurações.
> 
> **Pergunta feita para a Luri:**
> Onde consigo perceber essas configurações estendidas de AppConfig? Nome amigável, configuração de URL, de banco de dados etc.
> 
> **Resposta da Luri:**
> As configurações estendidas de AppConfig podem ser percebidas nos métodos e atributos disponíveis na classe. Alguns exemplos de configurações que podem ser definidas são:
> - Nome amigável do aplicativo: O método `verbose_name` permite definir um nome amigável para o aplicativo, que será exibido em algumas partes da interface do Django. 
> - Configurações de URL: O atributo `urlconf` permite definir o módulo de URLs específico para o aplicativo, permitindo que você organize as URLs de forma mais modular. 
> - Configurações de banco de dados: O atributo `default_auto_field` permite definir o tipo de campo automático a ser usado para as chaves primárias do modelo. Isso é útil quando você precisa personalizar o tipo de campo usado pelo Django para criar as tabelas no banco de dados. 
> 
> Essas são apenas algumas das configurações que podem ser definidas na classe AppConfig. Para saber mais sobre as configurações disponíveis, você pode consultar a documentação oficial do Django sobre a classe AppConfig.

# Acessando o banco
Mudanças no arquivo  `galeria/views.py`:
```python
from django.shortcuts import render

from galeria.models import Fotografia

def index(request):
    fotografias = Fotografia.objects.all()
    return render(request, 'galeria/index.html', {'cards' : fotografias})
# Resto do código
```
> Repare que a consulta ao banco de dados é simplificada: basta usar o comando `NomeDoModelo.objects.all()`.

Mudanças no arquiv `templates/galeria/index.html`:
```html
<!-- Resto do código -->
{% if cards %}
    {% for fotografia in cards %}
    <li class="card">
        <!-- Resto do código -->
        <div class="card__info">
            <p class="card__titulo">{{ fotografia.nome }}</p>
            <div class="card__texto">
                <p class="card__descricao">{{ fotografia.legenda }}</p>
                <!-- Resto do código -->
            </div>
        </div>
    </li>
    {% endfor %}
{% else %}
    {# Nada a exibir caso não haja os cards. #}
{% endif %}
<!-- Resto do código -->
```

> Note que o objeto `cards` que veio da view agora é uma lista de objetos, não mais um dicionário. A instância foi renomeada para `fotografia`.

# Passando uma referência
Inserindo uma nova fotografia usando o shell do Django: 
```
(.venv) PS D:\alura\django-admin> python manage.py shell    
Python 3.11.7 (tags/v3.11.7:fa7a6f2, Dec  4 2023, 19:24:49) [MSC v.1937 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.       
(InteractiveConsole)
>>> from galeria.models import Fotografia
>>> foto = Fotografia(nome="Galáxia NGC 1079", legenda="nasa.org / NASA / Hubble", foto="hubble_ngc1079.jpg")
>>> foto.save()
>>> 
```
> A fotografia `hubble_ngc1079.jpg` foi inserida no diretório `setup/static/assets/imagens/galeria`. Essa é uma etapa à parte da inserção do objeto no banco de dados, usando o shell.

Alterando a view de imagem, para que ela mostre dinamicamente a foto em função da id dessa foto, para buscá-la no banco de dados (`galeria/views.py`):
```python
from django.shortcuts import render, get_object_or_404

from galeria.models import Fotografia

# Resto do código

def imagem(request, foto_id):
    fotografia = get_object_or_404(Fotografia, pk=foto_id)
    return render(request, 'galeria/imagem.html', { 'fotografia' : fotografia })
```
> 1. Repare que a função recebeu mais um parâmetro (`foto_id`). Quando a view `imagem` for invocada, é necessário fornecer também esse parâmetro.
> 2. A função `get_object_or_404`, do pacote `django.shortcuts`, procura o objeto do modelo e retorna um erro 404 se não encontrar esse objeto. Para usar a função fornecemos dois parâmetros: o modelo (`Fotografia`) e a id/primary key procurada (`pk=foto_id` ou `id=foto_id`).

Modificando o arquivo `galeria/urls.py` para obrigar o fornecimento do parâmetro `foto_id`:
```python
from django.urls import path
from galeria.views import index, imagem

urlpatterns = [
    path('', index, name='index'),
    path('imagem/<int:foto_id>', imagem, name='imagem'),
]
```
> Note na sintaxe dentro da string do path da imagem: `<int:foto_id>`. A primeira parte é o tipo de dados; e a segunda parte é o nome do parâmetro que será enviado para a view `imagem`.

Atualizando o template `templates/galeria/index.html`:
```html
<!-- Resto do código -->
{% for fotografia in cards %}
<li class="card">
    <a href="{% url 'imagem' fotografia.id %}">
        <!-- Resto do código -->
    </a>
</li>
{% endfor %}
```
> Repare na sintaxe do comando `{% url %}`: antes ele só recebia um único parâmetro: o nome da view; agora ele também recebe a identificação da fotografia (`fotografia.id`).

Atualizando o template `templates/galeria/imagem.html`:
```html
<!-- Resto do código -->
<div class="imagem__conteudo">
    <img class="imagem__imagem" src="{% static '/assets/imagens/galeria/' %}{{ fotografia.foto }}">
    <div class="imagem__info">
        <div class="imagem__texto">
            <p class="imagem__titulo">{{ fotografia.nome }}</p>
            <p class="imagem__descricao">{{ fotografia.legenda }}</p>
            <p class="imagem__texto"></p>
        </div>
    </div>
</div>
<!-- Resto do código -->
```
> A função `{% static %}` permaneceu praticamente a mesma: o que mudou foi a inserção da interpolação do caminho da foto (`{{ fotografia.foto }}`). Os comandos `{% static %}` e a interpolação `{{ fotografia.foto }}` ficaram justapostos, um do lado do outro.
>
> De resto, temos apenas outras duas interpolações: `{{ fotografia.nome }}` e `{{ fotografia.legenda }}`.

# Django Admin
Criação de um superuser para acesso ao painel administrativo do Django com o comando `python manage.py createsuperuser`: 
```
(.venv) PS D:\alura\django-admin> python manage.py createsuperuser
Usuário (leave blank to use 'thiago'): 
Endereço de email: tma@cdtn.br
Password: 
Password (again):
A senha é muito parecida com usuário
Esta senha é muito curta. Ela precisa conter pelo menos 8 caracteres.        
Esta senha é muito comum.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
(.venv) PS D:\alura\django-admin> 
```
> Esse superuser é armazenado no banco de dados, como qualquer outro objeto de modelo do Django.

Para acessar o painel administrativo, visite a rota `admin` (geralmente no caminho http://localhost:8000/admin/). O nome de usuário e a senha cadastrados serão solicitados.

# CRUD no Admin
No momento, os objetos de modelo do aplicativo `galeria` não estão visíveis no painel administrativo do Django.

Para exibi-los no painel administrativo, é necessário modificar o arquivo `admin.py` do aplicativo (neste caso, ele está armazenado em `galeria/admin.py`):
```python
from django.contrib import admin

from galeria.models import Fotografia

admin.site.register(Fotografia)
```
Inserir os objetos no painel administrativo é simples: basta usar o comando `admin.site.register(ClasseDeModelo)`. No exemplo, importamos a classe `Fotografia` do módulo `galeria.models`

Usando essa sintaxe de registro de classe exibe para o painel `Fotografias` apenas a representação em string do objeto armazenado em banco.

Podemos personalizar o painel administrativo do Django usando a superclasse `admin.ModelAdmin`. Mudanças no arquivo `galeria/admin.py`:
```python
from django.contrib import admin

from galeria.models import Fotografia

class ListandoFotografias(admin.ModelAdmin):
    list_display = ('id', 'nome', 'legenda')
    list_display_links = ('id', 'nome')
    search_fields = ('nome',)

admin.site.register(Fotografia, ListandoFotografias)
```
> Perceba que o comando `admin.site.register` agora recebe dois parâmetros: o modelo (`Fotografia`) e a classe filha personalizada de `admin.ModelAdmin` (no caso, a classe `ListandoFotografias`).

A classe-filha de `ModelAdmin` pode ter vários parâmetros para personalizar o painel administrativo do objeto. Por exemplo:
- `list_display` define as colunas que serão exibidas no painel.
- `list_display_links` define quais colunas dentro de `list_display` vão conter links para modificação do objeto.
- `search_fields` define quais colunas podem ser pesquisadas por meio de filtros.
> Todos os campos mencionados recebem uma tupla ou uma lista com as colunas da classe de modelo.

# Incluindo categoria
Inclusão do capmo `categoria` na classe `Fotografia` (mudança no arquivo `galeria/models.py`):
```python
from django.db import models

class Fotografia(models.Model):
    OPCOES_CATEGORIA = [
        ('NEBULOSA', 'Nebulosa'),
        ('ESTRELA', 'Estrela'),
        ('GALAXIA', 'Galáxia'),
        ('PLANETA', 'Planeta'),
    ]

    # Resto do código
    categoria = models.CharField(max_length=100, choices=OPCOES_CATEGORIA, default='')
```
> Repare que criamos uma lista de tuplas com dois objetos cada (chave e valor), que foi batizada de `OPCOES_CATEGORIA`.
> 
> Essa lista vai servir de valor para o parâmetro `choices` do campo CharField recém criado (`categoria`).
> 
> Definimos um valor em branco como padrão no parâmetro `default`.

Criação da migration após a mudança da classe `Fotografia` (`python manage.py makemigrations`): 
```
(.venv) PS D:\alura\django-admin> python manage.py makemigrations
Migrations for 'galeria':
  galeria\migrations\0002_fotografia_categoria.py
    - Add field categoria to fotografia
(.venv) PS D:\alura\django-admin> 
```

Execução da migration criada (`python manage.py migrate`):
```
(.venv) PS D:\alura\django-admin> python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, galeria, sessions
Running migrations:
  Applying galeria.0002_fotografia_categoria... OK
(.venv) PS D:\alura\django-admin> 
```

Após a execução de todas essas operações, o campo `categoria` será criado com o formato de combo box no painel administrativo.

# Personalizando o admin
Mudanças no arquivo `galeria/admin.py`:
```python
from django.contrib import admin

from galeria.models import Fotografia

class ListandoFotografias(admin.ModelAdmin):
    list_display = ('id', 'nome', 'legenda')
    list_display_links = ('id', 'nome')
    search_fields = ('nome',)
    # Novos parâmetros:
    list_filter = ('categoria',)
    list_per_page = 10

admin.site.register(Fotografia, ListandoFotografias)
```

- A propriedade `list_filter` de um `ModelAdmin` serve para criar um filtro baseado numa lista de valores de alguma(s) coluna(s) do modelo.
- A propriedade `list_per_page` de um `ModelAdmin` serve para indicar o número de objetos que são exibidos por página no painel administrativo.

# Funcionalidade de publicação
O objetivo da aula é limitar a exibição das fotos apenas àquelas que estiverem configuradas como publicadas.

Inserção do campo `publicado` na model `Fotografia` (mudança no arquivo `galeria/models.py`):

```python
from django.db import models

class Fotografia(models.Model):
    # Resto do código
    publicado = models.BooleanField(default=False)
    # Resto do código
```

Criação e aplicação da migration resultante da mudança do modelo (`python manage.py makemigrations` e `python manage.py migrate`):
```
(.venv) PS D:\alura\django-admin> python manage.py makemigrations
Migrations for 'galeria':
  galeria\migrations\0003_fotografia_publicado.py
    - Add field publicado to fotografia
(.venv) PS D:\alura\django-admin> python manage.py migrate       
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, galeria, sessions
Running migrations:
  Applying galeria.0003_fotografia_publicado... OK
(.venv) PS D:\alura\django-admin> 
```

Mudança na view `index` para exibir apenas os objetos publicados (alterações no arquivo `galeria/views.py`):
```python
from django.shortcuts import render, get_object_or_404

from galeria.models import Fotografia

def index(request):
    fotografias = Fotografia.objects.filter(publicado=True)
    return render(request, 'galeria/index.html', {'cards' : fotografias})
# Resto do código.
```
> Repare que o método usado para buscar os objetos do modelo `Fotografia` mudou de `Fotografia.objects.all()` para `Fotografia.objects.filter(publicado=True)`. Outros parâmetros correspondentes aos nomes dos campos e os valores procurados neles podem ser acrescentados no método `filter`.

Mudanças no painel administrativo da classe `Fotografia` (alterações no arquivo `galeria/admin.py`):
```python
from django.contrib import admin

from galeria.models import Fotografia

class ListandoFotografias(admin.ModelAdmin):
    list_display = ('id', 'nome', 'legenda', 'publicado')
    # Resto do código
    list_editable = ('publicado',)
```
> 1. Foi acrescentado o campo `publicado` à tupla de campos que serão exibidos no painel administrativo;
> 2. O parâmetro `list_editable` contém uma lista ou tupla com todos os campos que podem ser massivamente editados no painel administrativo: no exemplo, podemos publicar ou despublicar várias fotografias de uma só vez.

# Incrementando o index
Acréscimo do campo `data_fotografia` (mudanças no arquivo `galeria/models.py`):
```python
from django.db import models
from datetime import datetime

class Fotografia(models.Model):
    # Resto do código
    data_fotografia = models.DateTimeField(default=datetime.now, blank=False)
    # Resto do código
```

Aplicação das migrations:
```
(.venv) PS D:\alura\django-admin> python manage.py makemigrations
Migrations for 'galeria':
  galeria\migrations\0004_fotografia_data_fotografia.py
    - Add field data_fotografia to fotografia
(.venv) PS D:\alura\django-admin> python manage.py migrate       
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, galeria, sessions
Running migrations:
  Applying galeria.0004_fotografia_data_fotografia...D:\alura\django-admin\.venv\Lib\site-packages\django\db\models\fields\__init__.py:1564: RuntimeWarning: DateTimeField Fotografia.data_fotografia received a naive datetime (2023-12-11 20:13:27.556632) while time zone support is active.
  warnings.warn(
 OK
(.venv) PS D:\alura\django-admin> 
```

Mudanças na lógica de exibição das fotos: elas serão exibidas pela ordem decrescente de data de publicação. Alterações no arquivo `galeria/views.py`:
```python
from django.shortcuts import render, get_object_or_404

from galeria.models import Fotografia

def index(request):
    fotografias = Fotografia.objects.order_by('-data_fotografia').filter(publicado=True)
    return render(request, 'galeria/index.html', {'cards' : fotografias})
```
> Note que antes do método `filter` inserimos o método `order_by('string_de_campos)`. Na verdade poderíamos até colocar o método `order_by` depois de `filter`.
>
> Para exibir um determinado campo em ordem decrescente (que é o nosso caso, já que queremos mostrar as fotos mais recentes primeiro), prefixamos o nome do campo ordenado com um hífen (`'-data-fotografia'`).

# Novo caminho para fotos
Definição do diretório que conterá as mídias/arquivos de upload (alterações no arquivo `settings.py`):

```python
# Resto do código
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
# Resto do código
```

Configuração das rotas de acesso à URL de mídia (alterações no arquivo `setup/urls.py`):
```python
from django.contrib import admin
from django.urls import path, include

from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('galeria.urls')),
] + static(settings.MEDIA_URL, media_root=settings.MEDIA_ROOT)
```

Mudança no modelo para definirmos um campo de imagem (alterações no arquivo `galeria/models.py`):
```python
from django.db import models
from datetime import datetime

class Fotografia(models.Model):
    # Resto do código
    foto = models.ImageField(upload_to='fotos/%Y/%m/%d/', blank=True, max_length=100)
    # Resto do código
```
> 1. Repare que o diretório de destino do upload (parâmetro `upload_to`) possui uma estrutura de vários subdiretórios: `foto/ano/mês/dia/`.
> 2. Pra constar: não confunda o diretório root `static` com o diretório root `media`. O mesmo vale para suas URLs.
> 3. O arquivo `n159.jpg` foi salvo manualmente por mim na pasta `static` apenas para fins de documentação. Ao inserir essa imagem, na verdade ela foi salva no caminho `media/fotos/%Y/%m/%d/n159.jpg`. O diretório `media` é ignorado pelo `.gitignore`.
> 4. Os outros arquivos de imagem, até então, estão salvos no diretório `static`. O novo arquivo inserido não é exibido em index porque a URL usada é a `static`, e a nova imagem foi salva no diretório mencionado em `MEDIA_ROOT`.

Execução das migrações:
```
(.venv) PS D:\alura\django-admin> python manage.py makemigrations
Migrations for 'galeria':
  galeria\migrations\0005_alter_fotografia_foto.py
    - Alter field foto on fotografia
(.venv) PS D:\alura\django-admin> python manage.py migrate       
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, galeria, sessions
Running migrations:
  Applying galeria.0005_alter_fotografia_foto... OK
(.venv) PS D:\alura\django-admin> 
```
> Eventualmente o Django pode reclamar de você não ter instalado o pacote `Pillow`. Instale-o com pip, se for o caso: `pip install Pillow`. O arquivo `requirements.txt` foi atualizado.
