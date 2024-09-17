# Django ORM vs SQL Nativo

Este documento explora as diferenças entre o uso do ORM do Django e a execução de consultas SQL nativas, abordando fatores de desempenho, complexidade e manutenção.

## Considerações Gerais

### ORM do Django:
- **Abstração:** Facilita a construção de consultas sem precisar escrever SQL manualmente. O ORM também se integra ao sistema de cache do Django, otimizando algumas operações.
- **Segurança:** Protege contra injeções de SQL, uma vantagem significativa em termos de segurança.
- **Manutenção:** O código baseado em ORM tende a ser mais legível e fácil de manter, especialmente para desenvolvedores que não têm familiaridade com SQL.

### SQL Nativo:
- **Performance:** Consultas SQL nativas podem ser mais rápidas para operações complexas ou específicas que o ORM não lida bem.
- **Flexibilidade:** Permite um controle mais granular sobre as consultas e o uso de funcionalidades específicas do banco de dados.
- **Complexidade:** Geralmente, são mais difíceis de manter e exigem um conhecimento profundo de SQL e do esquema do banco de dados.

## Explicação de Código

### ORM com Django

Este exemplo usa o ORM do Django para listar clientes, com um filtro baseado no nome.

```python
from rest_framework import generics
from .models import Cliente
from .serializers import ClienteSerializer

class ClienteListView(generics.ListAPIView):
    serializer_class = ClienteSerializer

    def get_queryset(self):
        queryset = Cliente.objects.all()
        nome = self.request.query_params.get('nome', None)
        if nome is not None:
            queryset = queryset.filter(nome__icontains=nome)
        return queryset
```

- **`ClienteListView:`** Subclasse de `ListAPIView` que exibe uma lista de objetos.
- **`get_queryset():`** Sobrescreve o método padrão para aplicar um filtro com base no parâmetro de consulta `nome`. O uso de `filter(nome__icontains=nome)` realiza uma busca insensível a maiúsculas e minúsculas.
- **`serializer_class:`** Define o serializador usado para converter os objetos `Cliente` em JSON.

### SQL Nativo com Django

Este exemplo mostra como realizar consultas SQL nativas no Django para buscar clientes com base no nome.

```python
from rest_framework import generics
from rest_framework.response import Response
from .models import Cliente
from .serializers import ClienteSerializer
from django.db import connection

class ClienteSearchView(generics.GenericAPIView):
    serializer_class = ClienteSerializer

    def get(self, request, *args, **kwargs):
        nome = request.query_params.get('nome', None)
        if nome is not None:
            with connection.cursor() as cursor:
                cursor.execute("SELECT * FROM app_cliente WHERE nome LIKE %s", [f'%{nome}%'])
                columns = [col[0] for col in cursor.description]
                results = [
                    dict(zip(columns, row))
                    for row in cursor.fetchall()
                ]
            return Response(results)
        return Response({"error": "Nome não fornecido"}, status=400)
```

- **`ClienteSearchView:`** Subclasse de `GenericAPIView` que oferece maior controle e personalização.
- **`get():`** Método que lida com requisições GET, executando uma consulta SQL nativa para buscar clientes cujo nome contém a string fornecida.
- **`connection.cursor():`** Abre um cursor para executar a consulta SQL. A utilização de `%s` com parâmetros previne injeção de SQL.
- **`cursor.fetchall():`** Recupera todas as linhas da consulta e mapeia os resultados para dicionários.

## Conclusão

- Para consultas simples e operações comuns, o **ORM do Django** é geralmente a melhor escolha devido à sua facilidade de uso, segurança e integração com o framework.
- Para consultas mais complexas ou específicas, onde o desempenho é crítico, **SQL nativo** pode ser mais eficiente.
  
É sempre importante considerar o contexto e realizar testes de desempenho específicos para determinar a melhor abordagem.

---

Este README fornece uma visão geral comparativa entre o uso do ORM do Django e consultas SQL nativas, com exemplos práticos de ambas as abordagens.

**Fonte: ChatGPT4**
