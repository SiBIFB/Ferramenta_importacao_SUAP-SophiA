# 📚 Ferramenta de importação em lote SUAP → SophiA

> Uma pequena ferramenta que roda no navegador para transformar planilhas exportadas do SUAP em um arquivo .TXT pronto para importação no sistema SophiA — pensada para bibliotecários e auxiliares que precisam atualizar/adicionar cadastros em lote, sem precisar programar. (Descrição feita pela IA Copilot embutido no Github e adaptado por Daniel R.G.) ✅

---

Vídeo tutorial deste passo-a-passo (apenas disponível aos servidores do IFB):
https://drive.google.com/file/d/1bKe3htuVONPZykqGuQbz3oQfGiwIAgRt/view?usp=drive_link

---

## 🎯 Objetivo

Gerar, localmente no navegador, um arquivo de importação (.TXT) formatado para o SophiA a partir de duas planilhas:

- Exportação do SUAP com os alunos;
- Planilha de referência que mapeia nomes de curso do SUAP para os nomes e tipos de usuário esperados pelo SophiA.

A ferramenta não envia dados à internet — tudo roda no seu computador.

---

## 🧭 Visão geral do que você precisa

1. Arquivo exportado do SUAP (.xls ou .xlsx) — lista de alunos.
2. Planilha de referência (.xlsx) — mapeamento de cursos e indicação se devem ser importados.
3. Nome da biblioteca (texto) — exatamente como aparece no SophiA.
4. Data de validade (DD/MM/AAAA) — será aplicada aos usuários ativos.

---

## ⚠️ Antes de começar — checagem rápida (checklist)

- [ ] Você tem a exportação do SUAP (xls/xlsx).
- [ ] Você tem a planilha de referência com as colunas: A=Curso SUAP, B=Curso SophiA, C=Tipo de Usuário, D=Importar? (SIM/NÃO).
- [ ] O navegador pode abrir o arquivo `index.html` (clique duas vezes nele ou hospede em servidor estático).
- [ ] Tenha em mãos o nome exato da biblioteca como consta no SophiA.

Dica: mantenha cópias originais dos arquivos antes de processar. 🗂️

---

## 📝 Como funciona (resumo simples)

1. A ferramenta lê a planilha de referência e cria um “dicionário” que traduz os nomes dos cursos do SUAP para os nomes usados no SophiA.
2. Abre a planilha do SUAP, encontra as colunas necessárias (Nome, Matrícula, Curso, E-mail, Situação).
3. Para cada aluno, se o curso estiver marcado como "SIM" na planilha de referência, a linha é preparada para o SophiA.
4. O resultado é um arquivo `Importacao_SophiA.txt` em UTF-8; separado por ponto-e-vírgula (`;`) e sem cabeçalho.

---

## 🖱️ Passo a passo (guia para iniciantes)

1. Abra a página da ferramenta ( https://sibifb.github.io/Ferramenta_importacao_SUAP-SophiA/ ) no navegador (Chrome, Edge ou Firefox funcionam bem).

2. Em "Modo de Operação", escolha:
   - "Apenas Inclusão (Novos Alunos)" — use quando for criar cadastros novos todo semestre, por exemplo.
   - "Atualização (Alunos Existentes)" — use quando for atualizar validade dos alunos devidamente matriculados e inativar alunos que não estão matriculados (trancados, evadidos, etc).

3. Em **1. Planilha de Matriculados (SUAP)** clique em "Escolher ficheiro" e selecione a exportação do SUAP (.xls ou .xlsx).

   ➤ Importante — Campos/filtragem a selecionar no SUAP antes de exportar

   - Filtros recomendados para o modo "Apenas Inclusão (Novos Alunos)":
     - Ano de Ingresso: selecione o ano vigente (ou o ano do ingresso que você quer importar)
     - Período de Ingresso: selecione o período vigente (se aplicável)
     - Situação: Matriculado (para trazer apenas alunos ativos que serão incluídos)
     - Campus/Unidade: selecione o campus correspondente à biblioteca
     - Observação: o objetivo aqui é exportar somente os novos alunos que se deseja criar no SophiA — obtenha uma lista enxuta para evitar inclusões indevidas.

   - Filtros recomendados para o modo "Atualização (Alunos Existentes)":
     - Ano de Ingresso: Todos (ou deixe sem filtro) — você deseja a base geral para atualizar validade/inativar
     - Período de Ingresso: Todos (ou deixe sem filtro)
     - Situação: incluir todos, i.e., sem fltro (Matriculado, Trancado, Evasão, etc.) — a coluna Situação será usada para decidir validade/inativação
     - Campus/Unidade: selecione o campus correspondente à biblioteca
     - Observação: exporte a base completa ou a fatia desejada, pois a ferramenta usará o campo Situação para decidir quem permanece ativo e recebe renova data de validade.

4. Em **2. Lista de Cursos e Tipos de Usuários** clique em "Escolher ficheiro" e selecione a planilha de referência (.xlsx). A planilha deve ter:
   - Coluna A: Curso com nome no SUAP
   - Coluna B: Curso com nome padronizado no SophiA
   - Coluna C: Tipo de Usuário (ex.: "Ensino Médio Integrado") de acordo com o oficial SophiA
   - Coluna D: Importar? (SIM ou NÃO)

5. Em **3. Nome da Biblioteca**, digite o nome exatamente como deve aparecer no SophiA (ex.: "Biblioteca Gama").

6. Em **4. Data de Validade**, digite a data no formato DD/MM/AAAA (ex.: `19/02/2027`).

7. Clique em **Processar e Baixar Arquivo TXT**. Aguarde a mensagem de sucesso.

- Se tudo funcionar, aparecerá uma mensagem de sucesso e o navegador fará o download do arquivo `Importacao_SophiA.txt`.
- Se houver problema, aparecerá uma mensagem de erro em vermelho explicando (veja seção "Resolução de problemas").

---

## 🔍 Diferenças entre os modos (exemplo simples)

Modo: Inclusão (cria novos registros)
- Ordem das colunas no TXT (8 colunas):
  1. Nome;
  2. Matrícula;
  3. Curso;
  4. Tipo de usuário;
  5. Biblioteca;
  6. Contato residencial - E-mail;
  7. Data de validade;
  8. Inativo (0 = ativo).

Modo: Atualização (atualiza validade / inativa)
- Ordem das colunas no TXT (7 colunas):
  1. Nome;
  2. Matrícula;
  3. Curso;
  4. Tipo de usuário;
  5. Biblioteca;
  6. Data de validade (apenas se estiver matriculado);
  7. Inativo (0 = ativo, 1 = inativo).

Observação: a ordem é importante — siga exatamente como o SophiA exige. 🔁

---

## 🧾 Formato do arquivo gerado

- Nome: `Importacao_SophiA.txt`
- Codificação: UTF-8 — para evitar problemas de acentuação
- Separador: ponto e vírgula (`;`)
- Fim de linha: CRLF (compatível com sistemas do SophiA)
- Sem cabeçalho — somente linhas de dados

---

## ❌ Mensagens de erro comuns e como resolver

- "Coluna 'Curso' não encontrada" → Abra a planilha do SUAP e verifique se a coluna existe e está com o nome correto.
- "Coluna 'Situação' não encontrada" (aparece no modo Atualização) → Use uma exportação do SUAP que inclua a coluna Situação (ex.: Matrículado / Trancado / Evasão).
- "Nenhum aluno atendeu aos critérios" → Verifique se na planilha de referência a coluna D está marcada como "SIM" para os cursos que você quer importar.

Se uma mensagem não ficar clara, copie o texto e procure um colega mais experiente ou peça auxílio ao suporte local. 💬

---

## 🔐 Privacidade e segurança

- Todo o processamento é feito no seu navegador — nada é enviado para a internet.
- Ainda assim, trate os arquivos com cuidado: contêm dados pessoais (e-mail, matrícula, nome).
- Apague arquivos temporários e limpe o histórico do navegador quando necessário.

---

## 💡 Dicas práticas para evitar erros

- Sempre mantenha um backup das planilhas originais antes de processar.
- Faça um teste com poucas linhas (por exemplo: 5 alunos) para confirmar que o arquivo TXT gerado está correto antes de processar a base inteira.
- Use a planilha de referência como controle: marque apenas "SIM" nos cursos que realmente deseja importar.

---

## 🧰 Exemplo rápido (mini-fluxo)

1. Exportar alunos no SUAP com filtros: Ano de ingresso = ano atual; Situação = Matriculado; Campus = X.
2. Abrir a planilha de referência e confirmar que os cursos estão mapeados e com "SIM" onde necessário.
3. Abrir `index.html`, selecionar arquivos, preencher Biblioteca e Data, processar, baixar TXT.
4. Abrir o TXT em um editor de texto e conferir as primeiras linhas antes de importar no SophiA.

---

## 📎 Precisa de ajuda? (quem contatar)

- Consulte o manual local da sua biblioteca ou o colega responsável por sistemas.
- Se sua instituição tiver suporte de TI/SEI, encaminhe o arquivo gerado e descreva o problema.

---

## 📝 Licença

MIT — fique à vontade para adaptar a ferramenta às suas necessidades.
