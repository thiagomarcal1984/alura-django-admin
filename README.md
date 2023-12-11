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
