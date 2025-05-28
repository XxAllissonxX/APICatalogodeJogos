📘 Documentação Técnica – Listagem de Total de Medições por Contrato
📌 Visão Geral
Este endpoint permite que operadores autenticados listem os dados de medição associados a um contrato específico. Cada medição contém o total de registros coletados, a data e hora da coleta, o total convertido em GB ou MB, e uma verificação se o limite do plano contratado foi excedido.

🔐 Endpoint
Método	URL	Autenticação
GET	/total-medicoes/<id_contrato>	✅ Token JWT

📥 Requisição
Headers

http
Copiar
Editar
Content-Type: application/json  
Authorization: Bearer <token>
Parâmetros na URL

Parâmetro	Tipo	Obrigatório	Descrição
id_contrato	integer	✅	ID do contrato para filtragem

📤 Resposta
✅ Sucesso – 200 OK (GET)
json
Copiar
Editar
[
  {
    "todas_qtd_medicoes": "12.000",
    "total_medicoes_gb": null,
    "data_hora": "22/05/2025 14:00:00",
    "valor_ultima_medicao": "0.89 GB",
    "message": "Limite do Plano excedido"
  },
  ...
]
Campo	Tipo	Descrição
todas_qtd_medicoes	string	Total de registros coletados, formatado com separadores de milhar
total_medicoes_gb	string	Valor total em GB da última medição (nulo neste contexto específico)
data_hora	string	Data e hora da medição formatada (dd/mm/yyyy HH:MM:SS)
valor_ultima_medicao	string	Valor da medição em GB ou MB com duas casas decimais
message	string	Indica se o limite do plano foi excedido (null se não excedeu)

❌ Erros
❌ 403 FORBIDDEN
json
Copiar
Editar
{
  "detail": "Ação não está disponível."
}
Motivo: O operador não está vinculado ao contrato solicitado.

⚙️ Funcionamento Interno
🔍 Autenticação
O operador precisa estar autenticado com token JWT. A permissão personalizada IsOperadorDoContrato garante que o operador só acesse contratos aos quais está vinculado.

🔎 Consulta
A view acessa a tabela NovaTotalMedicoesgb para obter os registros mais recentes do contrato.

Recupera o plano associado à medição mais recente para obter o limite do plano (plano_qtd_registro).

Converte os valores de medição para MB se estiverem abaixo de 1 GB.

Compara com o limite do plano para verificar se foi excedido.

📊 Formatação
todas_qtd_medicoes é formatado com separadores de milhar (ex: 12.000).

total_medicoes_gb é convertido para MB se for menor que 1 GB.

data_hora é exibida em formato legível.

🔁 Fluxo do Processo
mermaid
Copiar
Editar
graph TD
    A[Operador Autenticado] --> B[Requisição GET /total-medicoes/<id_contrato>]
    B --> C[Valida associação operador-contrato]
    C -->|Válida| D[Consulta dados em NovaTotalMedicoesgb]
    D --> E[Obtém plano mais recente e limite]
    E --> F[Formata dados e verifica se ultrapassa o limite]
    F --> G[Retorna lista de medições]
    C -->|Inválida| H[Retorna erro 403]
🧪 Exemplo com curl
bash
Copiar
Editar
curl -X GET http://localhost:8000/total-medicoes/12 \
  -H "Authorization: Bearer <seu_token_aqui>" \
  -H "Content-Type: application/json"
📁 Arquivos Relacionados
Arquivo	Função
serializers.py	Serializa os dados do modelo TotalMedicoes com campo extra de GB
views.py	Implementa a lógica de filtragem, formatação e resposta
permissions.py	Define IsOperadorDoContrato para validar acesso ao contrato
urls.py	Mapeia a rota /total-medicoes/<id_contrato> para a view apropriada

🔧 Variáveis Importantes
User Autenticado: Recuperado via request.user

Contrato: Verificado com ContratosOperador

Plano: Determina o limite de registros permitidos (conversão MB/GB)

total_medicoes_gb: Pode ser convertido para MB se < 1 GB

📌 Considerações Finais
A segurança da API é garantida por JWT + validação de vínculo com o contrato.

Os dados são retornados em ordem cronológica crescente (data_hora).

O endpoint fornece alertas úteis sobre excedência de limite do plano contratado.
