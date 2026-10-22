# Desafio Criativo N8N

## Planejando Automações com N8N Usando Apenas Bons Prompts

Quero criar uma automação no N8N para automatizar o processo de envio de Notas e Boletos a clientes.

### Público ou responsável:
Financeiro

### Resultado esperado:
Liberar o tempo que é gasto hoje com envio de boletos e notas emitidas aos clientes.

### Ferramentas envolvidas:
Gerador de relatório de boletos
Emissor de Boletos
Emissor de Notas
Envio de Email

### Fluxo desejado:
1. Ler de arquivo CSV os dados de clientes e boletos e notas/chave emitidas;
2. Obtem os boletos dentro de uma pasta. O nome do arquivo será o numero do boleto
3. Obtem o arquivo XML da nota usando o Numero da nota ou a sua chave.
4. Obtem o arquivo do DANFe em PDF usando o numero da nota ou sua chave.
5. Realiza o envio dos arquivos por email junto com um texto explicativo.
6. Move os arquivos para uma pasta de processados.

### Regras importantes:
- Se algum dos arquivos da nota não forem encontrados, a automação deve:
    1. Enviar enviar um email (ou acumular os erros e ao final enviar o email) ao responsável do financeiro informado sua falta. 
    2. Seguir para o próximo boleto 
- Se houver algum erro no envio, a automação deve registrar o erro e continuar com o próximo boleto.

## Prompt:

Atue como um especialista em N8N.

Crie uma automação para criar uma automação no N8N para automatizar o processo de envio de Notas e Boletos a clientes.

### Público:
Financeiro

### Ferramentas envolvidas:
- Leitor de CSV
    - Deve buscar o arquivo CSV dentro da pasta especificada: C:\automacao\financeiro\envio_notas_boletos
- Emissor de Boletos
    - Deve buscar os boletos na pasta especificada: C:\automacao\financeiro\boletos
- Emissor de Notas
    - Deve buscar os arquivos XML da nota e seus DANFes nas pastas especificadas: C:\automacao\financeiro\notas, C:\automacao\financeiro\danfes
- Envio de Email
    - Anexa os arquivos
    - Cria um texto explicativo com os dados do cliente e o que foi enviado, valor e dia de vencimento.

### Fluxo:
1. Ler de arquivo CSV os dados de clientes e boletos e notas/chave emitidas dentro da pasta C:\automacao\financeiro\envio_notas_boletos;
2. Obtem os boletos dentro de uma pasta. C:\automacao\financeiro\boletos. O nome do arquivo será o numero do boleto dentro do arquivo CSV.
3. Obtem o arquivo XML da nota usando o Numero da nota ou a sua chave. O numero da nota ou a chave estará no arquivo CSV. Sua pasta será C:\automacao\financeiro\notas.
4. Obtem o arquivo do DANFe em PDF usando o numero da nota ou sua chave. O numero da nota ou a chave estará no arquivo CSV. Sua pasta será C:\automacao\financeiro\danfes.
5. Realiza o envio dos arquivos por email junto com um texto explicativo, descrevendo o que esta sendo enviado e o motivo do envio, informando a data de vencimento, número do boleto e correspondente a qual nota é.
6. Ao final do processamento, os arquivos que tiveram sucesso e forma enviados ao cliente, move os arquivos para uma pasta de processados.


### Regras:
Se algum dos arquivos da nota não forem encontrados, a automação deve:
1. Ir acumulando em uma lista a nota que teve problema. 
2. Enviar um email ao responsável pelo financeiro "financeiro@empresateste.com.br" com a lista das notas que tiveram problema
3. Pular a etapa de mover os arquivos para processados.
4. Seguir para o próximo boleto.



## Explique quais nós do N8N devem ser utilizados e a lógica de funcionamento do workflow.

### 📋 Tabela de Nós do n8n para o Fluxo Completo


| Nome do Nó | Descrição | Como Utilizar neste Fluxo |
| --- | --- | --- |
| Schedule Trigger (ou Webhook) | Inicia o fluxo automaticamente por horário ou requisição. | Configure para rodar uma vez por dia (ex: todo final de tarde) ou sempre que o sistema Clipper gerar novos arquivos. |
| Execute Command | Executa comandos no terminal do servidor do n8n. | Utilize para rodar um script (como dbf2csv em Python) que converte o arquivo .DBF do Clipper em um arquivo .csv legível. |
| Read/Write Files from Disk | Lê ou grava arquivos binários diretamente no armazenamento do servidor. | 1º Uso: Lê o .csv que você acabou de converter do DBF. 2º Uso: Lê a pasta inteira onde estão armazenados todos os Boletos e todas as Notas. |
| Extract From File | Transforma o conteúdo de arquivos (como CSV, Excel) em dados de texto (JSON). | Conecte-o logo após a leitura do .csv convertido para transformar as linhas da tabela do Clipper em registros utilizáveis dentro do n8n. |
| Looping | Processa uma lista de itens (neste caso, clientes ou boletos) um por um. | Garante que o n8n trate o boleto, busque a nota e envie o e-mail de um cliente antes de passar para o próximo, evitando misturar os arquivos. |
| Code (JavaScript / Python) | Executa códigos customizados para manipular dados estruturados. | Use para tratar o nome do arquivo do boleto (ex: remover letras e manter apenas o número) para usá-lo como chave de busca. |
| Filter | Filtra os dados com base em condições lógicas. | Use para varrer a lista de Notas Fiscais e reter apenas aquela cujo nome do arquivo Contém (contains) o número que você isolou no nó anterior. |
| Gmail / Outlook / SMTP | Conecta com seu provedor de e-mail para disparar mensagens. | Configure o destinatário (obtido do DBF/CSV), o corpo do e-mail e ative a opção Attachments (Anexos) enviando os dados binários do boleto e da nota filtrada.


### 💡 Dica para a Integração

_O n8n processará múltiplos clientes, a estrutura dentro do nó Looping deve isolar o ID ou número do documento de cada iteração. Garanta também que o nó de e-mail esteja configurado para redefinir os anexos a cada ciclo do loop, evitando que o Cliente B receba a nota do Cliente A._
