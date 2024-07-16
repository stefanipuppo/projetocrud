python -m venv cliente1             -> criar o ambiente virtual                <br>
cd cliente1					            		-> abre o ambiente                         <br>
cd Scripts                          -> abre o scripts                          <br>
activate						              	-> ativa o ambiente                        <br>
cd .. 			               					-> opcao voltar pasta anterior             <br>

instalar django        			       -> pip install django                       <br>
criar projeto 						         -> django-admin startproject app .          <br>
rodar aplicacao                    -> py manage.pu runserver                   <br>
migrar bando de dados				       -> py manage.py migrate                     <br>


cria usuario no banco de dados      -> py manage.py createsuperuser
nome do usuario		 				-> admin
ignora o email, cria senha			-> senha qualquer 2 vezes
roda o ambiente novamente           -> py manage.pu runserver
volta pro browser                   -> colocar admin de pois da porta, exemplo: 127.0.0.1:8000/admin 

----------------------------------------------------------------------------------------------------------------------------------------------------

urls.py   ->   patch('core/', include('core.urls'))  ->   importe o include, exemplo: from django.url import patch, include
crie um aqruivo urls.py dentro da pasta que vc criou ->   patch('', home)    -> home é o nome da funcao, pode ser qualquer outro nome
registre ela no setting 

importe a views                 	-> from .views import home                                <br>
crie uma requisiçao					-> def home(request): return render(request, "index.html")

crie uma pasta templatas dentro da pasta core
crie um arquivo chamado index.html

----------------------------------------------------------------------------------------------------------------------------------------------------

models é o modo que voce cria dados no banco de dados(bd), voce descreve como seu bd pareça, voce cria a classe a partir dela vc tem acesso ao bd
exemplo de codigo:

class Pessoa(models.Model):
    nome = models.CharField(max_length=100)

depois de criar o model, voce pode ir no admin e registrar ele, primeiro importe o model como no exemplo abaixo:

from .models import Pessoa

admin.site.register(Pessoa)

----------------------------------------------------------------------------------------------------------------------------------------------------

O django vem com um sistema de makegrations que ajuda evoluir o banco de dados em codigo 

py manage.py makemigrations 

(dentro da pasta migrations em 0001_inital.py ele explica como a tabela deve ser criada)

agora voce digita o codigo abaixo para criar a migracao

py manage.py migrate

voce pode ver a tabela em 127.0.0.1:8000/admin 

use o codigo do metodo abaixo para editar o modo de exibição do nome do objeto da tabela
def __str__(self):
        return self.nome
		
----------------------------------------------------------------------------------------------------------------------------------------------------

Ja criamos, agora vá na views e import o model que voce criou, nesse caso o model pessoa                                                          <br>
crie uma variavel que recebe a classe(Pessoa) e pegue todos objetos dentro dela(nome)               ->      pessoa = Pessoa.object.all()          <br>
agora envie a variavel para o template(html) e o valor que vai ser passado pra ela                  ->     {"pessoas": pessoas}                   <br>

from .models import Pessoa

def home (request):
    pessoa = Pessoa.objects.all()
    return render(request, "index.html", {"pessoas": pessoas})
	
----------------------------------------------------------------------------------------------------------------------------------------------------
TEMPLATE(html)
 
Agora voce pode enviar seus dados para o template, o front da aplicação

{%%}  -> para comandos
{{}}  -> para variaveis 


    <ul>																			-> cria uma lista
        {% for pessoa in pessoas%}                                            		-> para cada pessoa em pessoas
            <li>{{pessoa.id}} - {{pessoa.nome}}</li>                                 
        {% endfor %}
    </ul>
	
----------------------------------------------------------------------------------------------------------------------------------------------------
CREATE

    <form action="{% url 'salvar' %}"  method="POST">                               -> ação a ser realizada, url
		{% csrf_token %}                                                                -> protege contra ataques externos
        <input type="text" name="nome"/>                                            -> tipo é tipo texto e name recebe o nome
        <button type="submit">Salvar</button>                                       -> botao submit: envia os dados dessa forma para o servidor
    </form>


quando clica no botao, ele faz a submit do form, envia os dados do input para a view que vai estar amarrada no url(nesse caso ao salvar)

importa no urls py e cria a url(nesse caso o salvar)

from .views import home, salvar

urlpatterns = [
    path('', home),
    path('salvar/', salvar, name="salvar")
]

--------------------------------------------------------------------------------------------------------------------------------------------------
cria a funcao salvar na viewa

def salvar(request):
    vnome = request.POST.get("nome")                                               -> acessa oque o usuario digitou 
    Pessoa.objects.create(nome=vnome)                                              -> cria um dado novo banco, passa a variavel para o atributo pro obejto pessoa
    pessoas = Pessoa.objects.all                                                   -> pega a lista de pessoas
    return render(request, "index.html", {"pessoas": pessoas})                     -> manda para o form
	
	
criou > pegou a lista toda -> enviou pro index -> no index faz um loop nas pessoas e imprimi cada uma

--------------------------------------------------------------------------------------------------------------------------------------------------
EDITAR

    <ul>																			
        {% for pessoa in pessoas%}    
		
            <li>{{pessoa.id}} - {{pessoa.nome}} <a href="{% url 'editar' pessoa.id %}">Editar</a></li>      
			      (um link que chama a url para editar, para editar precisa do id da pessoa)            
			
        {% endfor %}
    </ul>
	
vai pro urls.py e cria a urls e importe o metodo

path('delete/<int:id>', editar, name='editar'),

vai para view e crie a funcao editar

def editar(request, id):
    pessoa = Pessoa.objects.get(id=id)                                  -> pega a pessoa pelo id
    return render(request,"update.html", {"pessoa": pessoa})            -> envia os dados pra um novo template

--------------------------------------------------------------------------------------------------------------------------------------------------

cria um novo template ("update.html")

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Document</title>
</head>
<body>

    {{pessoa.id}} - {{pessoa.nome}} 

    <form action="{% url 'update' pessoa.id %}" method="POST">
        {% csrf_token %}
        <input type="text" name="nome" value="{{pessoa.nome}}"/>                           -> ja vem com o nome da pessoa para editat
        <button type="submit">Update</button>
    </form>

</body>
</html>


Vai pro urls, importe e crie um urls pro template update, vai pra viwa e crie a funcao update e importe o redirect

from django.shortcuts import render, redirect

def update(request, id):
    vnome = request.POST.get("nome")                     -> a pessoa mandou um novo nome 
    pessoa = Pessoa.objects.get(id=id)                   -> recupera a pessoa no banco pq agora ela existe
    pessoa.nome = vnome                                  -> pega o novo valor que veio do banco 
    pessoa.save()
	return redirect(home)                                -> redireciona pra home
	
	
------------------------------------------------------------------------------------------------------------------------------------------
DELETAR

    <ul>
        {% for pessoa in pessoas %}
            <li>{{pessoa.id}} - {{pessoa.nome}} 
                <a href="{% url 'editar' pessoa.id %}">Editar</a>
                <a href="{% url 'delete' pessoa.id %}">Deletar</a>                                           -> criar o novo link da funcao
            </li>
        {% endfor %}
    </ul>
	

CRIA A URL, IMPORTA A FUNCAO E CRIE A FUNCAO 

from .views import home, salvar, editar, update, delete                            -> importacao

path('delete/<int:id>', delete, name='delete'),									   -> urls


def delete(request, id):                                                           -> funcao 
    pessoa = Pessoa.objects.get(id=id)
    pessoa.delete()
    return render(home)

