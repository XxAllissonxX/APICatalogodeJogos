📘 Documentação Técnica – Consulta de Medições por Curva
📌 Visão Geral
Este endpoint permite que operadores autenticados consultem medições relacionadas a uma curva de gráfico específica. A curva está vinculada a um dispositivo e sensor, cujas medições podem ser processadas com base em uma fórmula personalizada, dependendo do período selecionado.

🔐 Endpoint
Método	URL	Autenticação
GET	/visoes/medicao/<int:id>	✅ Token JWT

📥 Requisição
🔤 Headers
http
Copiar
Editar
Content-Type: application/json  
Authorization: Bearer <token>
🧾 Parâmetros de URL
Parâmetro	Tipo	Obrigatório	Descrição
id	integer	✅	ID da curva (CurvasGrafico.id)

🔄 Query Params
Parâmetro	Tipo	Obrigatório	Descrição
periodo	string	✅	Período de agregação dos dados. Valores aceitos: 5Min-1Dia, 1Min-2Horas, 2Horas-30Dias, 30Min-7Dias, 1Dia-365Dias

📤 Resposta
✅ Sucesso – 200 OK
json
Copiar
Editar
[
  {
    "chip_id": "HA-123",
    "dispositivo_apelido": "Sensor Externo",
    "canal_sensor": "HA-GEN-27",
    "apelido_canal": "Temperatura",
    "info": "Informações técnicas do sensor",
    "descricao": "Sensor de Temperatura Externo",
    "x": "22/05/2025 10:00",
    "qtd_medicoes": 12,
    "mean": 25.3,
    "sum": 304,
    "max": 30.1,
    "min": 20.0
  },
  ...
]
Os valores mean, sum, max, e min são convertidos e normalizados, e podem ser modificados por uma fórmula definida na curva, se addForm for verdadeiro.

❌ Erros
Código	Mensagem	Causa
400	{ "error": "Periodo especificado invalido." }	Valor de periodo ausente ou não reconhecido
400	{ "error": "chip_id ou sensor_ref não definido na curva." }	Campo essencial ausente na curva
404	{ "error": "Curva não encontrada." }	ID da curva inexistente
500	{ "error": "<mensagem de erro interno>" }	Erro inesperado ao processar os dados

⚙️ Funcionamento Interno
🔍 Autenticação
Este endpoint requer autenticação JWT. A permissão e filtragem de dados está atrelada ao acesso do operador ao dispositivo vinculado à curva.

🔄 Lógica
O ID da curva é obtido via URL e usado para buscar CurvasGrafico.

A fórmula, chip_id, e sensor_ref são extraídos do JSON da curva.

Os valores de chip_id e sensor_ref são convertidos via funções auxiliares da lib cifra.py.

O sistema verifica se o dispositivo está ativo.

Dependendo do periodo informado, a medição é extraída de uma das seguintes tabelas:

LogSensor5M1D

LogSensor1M2H

LogSensor120M1M

LogSensor30M1W

LogSensor1D1Y

Se addForm = true e form contiver uma fórmula válida (com variável var), os dados de medição são recalculados conforme a fórmula.

Retorna a lista de medições com os dados processados.

🔁 Fluxo do Processo
mermaid
Copiar
Editar
graph TD
    A[Requisição GET /visoes/medicao/<id>?periodo=...] --> B[Busca Curva por ID]
    B --> C[Valida existência de chip_id e sensor_ref]
    C --> D[Converte valores usando cifra]
    D --> E[Verifica se dispositivo está ativo]
    E --> F[Consulta a tabela de acordo com o período]
    F --> G[Processa e normaliza dados]
    G --> H[Aplica fórmula se addForm = true]
    H --> I[Retorna JSON com dados de medição]
🧪 Exemplo com curl
bash
Copiar
Editar
curl -X GET "http://localhost:8000/visoes/medicao/100?periodo=5Min-1Dia" \
  -H "Authorization: Bearer <seu_token_aqui>" \
  -H "Content-Type: application/json"
Se quiser, posso gerar essa documentação em formato Markdown, PDF ou como uma página HTML. Deseja exportá-la em algum desses formatos?







Você disse:
como passar essa documenação para o markdown no github?


O ChatGPT disse:
Para transformar essa documentação em Markdown para ser usada no GitHub (por exemplo, em um README.md ou outra documentação), você pode formatá-la usando os elementos do Markdown como títulos, tabelas, blocos de código e listas.

Aqui está um exemplo da documentação que você forneceu, adaptada para Markdown:

📘 Documentação Técnica – Listagem de Medições
📌 Visão Geral
Este endpoint permite listar as medições de um dispositivo e canal específicos, vinculados a uma curva cadastrada. A curva contém os parâmetros necessários como chip_id, sensor_ref e, opcionalmente, uma fórmula (form) para transformar os dados medidos.

🔐 Endpoint
Método	URL	Autenticação
GET	/visoes/medicao/<id>	✅ Token JWT

📥 Requisição
Headers
http
Copiar
Editar
Content-Type: application/json  
Authorization: Bearer <token>
Parâmetros da URL
id: ID da curva cadastrada (CurvasGrafico).

Parâmetros de Consulta (Query)
Parâmetro	Tipo	Obrigatório	Descrição
periodo	string	✅	Define o agrupamento de tempo desejado. Pode ser: 1Min-2Horas, 5Min-1Dia, 30Min-7Dias, 2Horas-30Dias, 1Dia-365Dias.

📤 Resposta
✅ Sucesso – 200 OK
json
Copiar
Editar
[
  {
    "chip_id": "HA-656",
    "dispositivo_apelido": "Sensor 01",
    "canal_sensor": "HA-GEN-32",
    "apelido_canal": "Temp Interna",
    "info": "Temperatura ambiente",
    "descricao": "Sensor térmico",
    "x": "27/05/2024 14:00",
    "qtd_medicoes": 12,
    "mean": 25.3,
    "sum": 303.6,
    "max": 28.0,
    "min": 23.5
  },
  ...
]
❌ Erro – 404 NOT FOUND
json
Copiar
Editar
{
  "error": "Curva não encontrada."
}
❌ Erro – 400 BAD REQUEST
json
Copiar
Editar
{
  "error": "Periodo especificado invalido."
}
⚙️ Funcionamento Interno
A curva (CurvasGrafico) armazena os dados necessários: chip_id, sensor_ref e, opcionalmente, uma fórmula.

O sistema traduz esses valores em boot_chip_id e sensor_ref com base no fabricante.

Os dados são buscados de acordo com o período:

1Min-2Horas → Tabela: LogSensor1M2H

5Min-1Dia → Tabela: LogSensor5M1D

30Min-7Dias → Tabela: LogSensor30M1W

2Horas-30Dias → Tabela: LogSensor120M1M

1Dia-365Dias → Tabela: LogSensor1D1Y

Caso uma fórmula esteja presente (addForm: true), os valores de mean, sum, max, min são recalculados utilizando a fórmula aplicada sobre a variável var.

📎 Observações
O campo x será a hora ou a combinação data/hora dependendo do período.

Valores 999999.0 são considerados inválidos e substituídos por null.

A fórmula é processada usando a biblioteca SymPy.
