# Ferramenta de importação em lote SUAP-SophiA

Ferramenta simples baseada em página HTML que roda diretamente no navegador para gerar arquivos de importação compatíveis com o sistema SophiA a partir de exportações do SUAP.

A aplicação permite processar em lote listas de alunos (exportadas do SUAP) usando uma planilha de referência que mapeia os cursos SUAP para os cursos e tipos de usuário esperados pelo SophiA. O resultado é um arquivo .TXT (UTF-8) pronto para importação no SophiA.

## Funcionalidades

- Processamento em modo "Inclusão" (cria registros para novos alunos).
- Processamento em modo "Atualização" (atualiza validade e inativa alunos não matriculados).
- Utiliza uma planilha de referência para mapear nomes de curso e determinar quais cursos importar.
- Gera arquivo .TXT separado por ponto e vírgula (;) com codificação UTF-8 (com BOM) e final de linha CRLF.
- Roda 100% no cliente (navegador) — sem back-end.

## Dependências

A página carrega bibliotecas via CDN:
- Tailwind CSS (estilização)
- SheetJS (xlsx) para leitura das planilhas Excel

É necessário que o navegador tenha acesso à internet para carregar as dependências se estiver usando diretamente o arquivo `index.html` sem bundler.

## Arquivos de entrada esperados

1) Planilha de Matriculados (exportação do SUAP)
- Formato: .xls ou .xlsx
- A primeira linha é tratada como cabeçalho — a ferramenta tenta localizar colunas por nomes comuns.
- Colunas relevantes (a serem encontradas na exportação do SUAP):
  - Nome (ex.: "Nome", "Nome do aluno")
  - Matrícula (ex.: "Matrícula", "Matricula")
  - Curso (ex.: "Curso") — obrigatório
  - E-mail (ex.: "E-mail pessoal", "Email pessoal") — usado apenas no modo Inclusão
  - Situação (ex.: "Situação", "Situação no Curso") — obrigatório no modo Atualização

2) Planilha de Referência (mapeamento de cursos)
- Formato: .xlsx
- Colunas esperadas (linha de dados a partir da 2ª linha; a 1ª linha pode ser cabeçalho):
  - A = Curso SUAP (valor buscado na exportação do SUAP)
  - B = Curso SophiA (nome a ser escrito no arquivo de saída)
  - C = Tipo de Usuário (ex.: Aluno, Servidor, etc.)
  - D = Importar? (SIM/NÃO) — somente cursos com "SIM" serão processados

Observação: a chave usada para correspondência é a versão normalizada (sem espaços extras e em maiúsculas) do nome do curso na coluna A.

## Como usar (passo a passo)

1. Abra o arquivo `index.html` no navegador (ou hospede a página estática).
2. Selecione o modo de operação: "Apenas Inclusão (Novos Alunos)" ou "Atualização (Alunos Existentes)".
3. Carregue a planilha exportada do SUAP (.xls/.xlsx).
4. Carregue a planilha de referência (.xlsx) que contém o mapeamento de cursos.
5. Preencha o nome da biblioteca exatamente como deve aparecer no cadastro do SophiA.
6. Informe a data de validade que será aplicada aos usuários ativos (formato DD/MM/AAAA).
7. Clique em "Processar e Baixar Arquivo TXT" para gerar e baixar o arquivo `Importacao_SophiA.txt`.

## Diferenças entre os modos

- Inclusão (inclusao): o TXT gerado contém as colunas na ordem exigida pelo SophiA para inclusão:
  1. Nome
  2. Matrícula
  3. Curso (nome convertido pelo mapeamento)
  4. Tipo de usuário
  5. Biblioteca
  6. Contato residencial - E-mail
  7. Data de validade (DD/MM/AAAA)
  8. Inativo (0 = ativo)

- Atualização (atualizacao): o TXT gerado contém campos voltados à atualização de validade e inativação:
  1. Nome
  2. Matrícula
  3. Curso (nome convertido pelo mapeamento)
  4. Tipo de usuário
  5. Biblioteca
  6. Data de validade (preenchida apenas se o aluno estiver "MATRICULADO")
  7. Inativo (0 = ativo, 1 = inativo)

Importante: a ordem e o número de colunas do TXT devem respeitar o que o SophiA espera — ver o aviso dentro da interface.

## Formato do arquivo de saída

- Nome do arquivo: Importacao_SophiA.txt
- Codificação: UTF-8 com BOM (a página inclui o marcador U+FEFF)
- Separador de campo: ponto e vírgula (`;`)
- Final de linha: CRLF (\r\n)
- Sem cabeçalho (apenas linhas de dados)

## Tratamento de erros e mensagens

- Se nenhuma linha for processada, a ferramenta exibirá uma mensagem de erro indicando que nenhum aluno atendeu aos critérios (verifique as marcações "SIM" na planilha de referência).
- Verifique se as colunas obrigatórias (especialmente `Curso` e, no modo Atualização, `Situação`) estão presentes e corretamente nomeadas na exportação do SUAP.

## Segurança e privacidade

- A ferramenta roda localmente no navegador e não envia dados a servidores externos.
- Ainda assim, os dados carregados ficam disponíveis apenas na sessão do navegador — recomenda-se apagar arquivos e histórico quando necessário.

## Exemplos e sugestão de fluxo

- Gere a exportação de Alunos no SUAP com filtros apropriados (por exemplo: Ano de ingresso = ano vigente; Situação = Matriculado; Campus = campus da biblioteca).
- Monte a planilha de referência mapeando somente os cursos que você deseja importar e marque "SIM" na coluna D.
- Abra a página, carregue os arquivos, confirme a data de validade e execute o processamento.

## Licença

- MIT — sinta-se à vontade para adaptar a ferramenta para sua realidade.

---

Para mais detalhes, abra o `index.html` e leia os comentários no código fonte.