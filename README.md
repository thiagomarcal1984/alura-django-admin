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
