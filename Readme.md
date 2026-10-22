# 📚 Ferramenta de importação em lote SUAP → SophiA

> Uma pequena ferramenta que roda no navegador para transformar planilhas exportadas do SUAP em um arquivo .TXT pronto para importação no sistema SophiA — pensada para bibliotecários, auxiliares e responsáveis por atualização de cadastros. Agora também cobre a base de servidores.

---

Vídeo tutorial deste passo-a-passo (apenas disponível aos servidores do IFB):
https://drive.google.com/file/d/1bKe3htuVONPZykqGuQbz3oQfGiwIAgRt/view?usp=drive_link

---

## 🎯 Objetivo

Gerar, localmente no navegador, um arquivo de importação (.TXT) formatado para o SophiA a partir de planilhas exportadas do SUAP.

A ferramenta agora suporta três cenários:

- Inclusão de novos alunos;
- Atualização de alunos já existentes;
- Inclusão e atualização da base de servidores.

A ferramenta não envia dados à internet — tudo roda no seu computador.

---

## 🧭 Visão geral do que você precisa

1. Arquivo exportado do SUAP (.xls, .xlsx ou .csv) — alunos ou servidores, conforme o modo escolhido.
2. Planilha de referência (.xlsx) — apenas para os modos de alunos, mapeando cursos do SUAP para nomes e tipos esperados pelo SophiA.
3. Nome da biblioteca — selecionado entre as bibliotecas cadastradas no sistema.
4. Data de validade (DD/MM/AAAA) — será aplicada aos usuários ativos.

---

## ⚠️ Antes de começar — checagem rápida (checklist)

- [ ] Você tem a exportação do SUAP correta para o tipo de usuário que deseja importar.
- [ ] No modo de alunos, você tem a planilha de referência com as colunas: A=Curso SUAP, B=Curso SophiA, C=Tipo de Usuário, D=Importar? (SIM/NÃO).
- [ ] O navegador consegue abrir o arquivo `index.html` (clique duas vezes nele ou hospede em servidor estático).
- [ ] Você conhece o nome exato da biblioteca como consta no SophiA.
- [ ] Para servidores, a planilha exportada contém as colunas de nome, matrícula, cargo e "Ativo".

Dica: mantenha cópias originais dos arquivos antes de processar. 🗂️

---

## 📝 Como funciona (resumo simples)

1. A ferramenta identifica o modo escolhido pelo usuário.
2. Para alunos, lê a planilha de referência e cria um dicionário que traduz os nomes dos cursos do SUAP para os nomes usados no SophiA.
3. Para servidores, identifica automaticamente o tipo de usuário pelo cargo e usa a coluna "Ativo" para definir quem deve permanecer ativo ou ser inativado.
4. O resultado é um arquivo `Importacao_SophiA.txt` (ou `Importacao_SophiA_Servidores.txt`) em UTF-8, separado por ponto e vírgula (`;`) e sem cabeçalho.

---

## 🖱️ Passo a passo (guia para iniciantes)

1. Abra a página da ferramenta (https://sibifb.github.io/Ferramenta_importacao_SUAP-SophiA/) no navegador (Chrome, Edge ou Firefox funcionam bem).

2. Em "Modo de Operação", escolha uma das opções:
   - "Apenas Inclusão (Novos Alunos)" — use quando for criar cadastros novos todo semestre.
   - "Atualização (Alunos Existentes)" — use quando for atualizar validade dos alunos cadastrados e inativar quem não está matriculado.
   - "Inclusão e atualização (Servidores)" — use para atualizar a base de servidores, incluindo novos e existentes, com base no cargo e no campo "Ativo".

3. Em **2. Planilha de Origem (SUAP)** clique em "Escolher ficheiro" e selecione a exportação correta.

   ➤ Importante — campos e filtros a selecionar no SUAP antes de exportar

   - Para o modo "Apenas Inclusão (Novos Alunos)":
     - Ano de Ingresso: selecione o ano vigente (ou o ano do ingresso que você quer importar)
     - Período de Ingresso: selecione o período vigente (se aplicável)
     - Situação: Matriculado
     - Campus/Unidade: selecione o campus correspondente à biblioteca
     - Observação: o objetivo é exportar somente os novos alunos que se deseja criar no SophiA.

   - Para o modo "Atualização (Alunos Existentes)":
     - Ano de Ingresso: Todos
     - Período de Ingresso: Todos
     - Situação: Todos (Matriculado, Trancado, Evasão, etc.)
     - Campus/Unidade: selecione o campus correspondente à biblioteca
     - Observação: a ferramenta usará a coluna Situação para decidir quem permanece ativo e recebe nova data de validade.

   - Para o modo "Inclusão e atualização (Servidores)":
     - Exportação de servidores do respectivo Campus do SUAP sem filtro, salvo o filtro Campus selecionado.
     - O sistema deduz automaticamente o tipo de usuário com base no cargo:
       - se o cargo contiver "PROFESSOR" → tipo de usuário: `Docente`
       - caso contrário → `Técnico-administrativo`
     - A coluna "Ativo" deve indicar se o servidor está em atividade e será usada para definir `Inativo = 0` ou `1`.

4. Em **3. Dicionário de Cursos (Excel)** clique em "Escolher ficheiro" e selecione a planilha de referência (.xlsx). Esse passo é obrigatório apenas nos modos de alunos.
   A planilha deve ter:
   - Coluna A: Curso com nome no SUAP
   - Coluna B: Curso com nome padronizado no SophiA
   - Coluna C: Tipo de Usuário
   - Coluna D: Importar? (SIM ou NÃO)

5. Em **4. Nome da Biblioteca**, selecione a biblioteca corretamente.

6. Em **5. Data de Validade**, informe a data no formato DD/MM/AAAA (ex.: `19/02/2027`).

7. Clique em **Processar e Baixar Arquivo TXT**. Aguarde a mensagem de sucesso.

- Se tudo funcionar, aparecerá uma mensagem de sucesso e o navegador fará o download do arquivo `Importacao_SophiA.txt` (ou `Importacao_SophiA_Servidores.txt`).
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

Modo: Inclusão e atualização (Servidores)
- Ordem das colunas no TXT (7 colunas):
  1. Nome;
  2. Matrícula;
  3. Tipo de usuário;
  4. Biblioteca;
  5. E-mail;
  6. Data de validade;
  7. Inativo (0 = ativo, 1 = inativo).

Observação: a ordem é importante — siga exatamente como o SophiA exige. 🔁

---

## 🧾 Formato do arquivo gerado

- Nome: `Importacao_SophiA.txt` (ou `Importacao_SophiA_Servidores.txt` no modo de servidores)
- Codificação: UTF-8 — para evitar problemas de acentuação
- Separador: ponto e vírgula (`;`)
- Fim de linha: CRLF (compatível com sistemas do SophiA)
- Sem cabeçalho — somente linhas de dados
- Quando houver arquivo grande, a ferramenta pode dividir o TXT em partes para facilitar o download.

---

## ❌ Mensagens de erro comuns e como resolver

- "Coluna 'Curso' não encontrada" → Abra a planilha do SUAP e verifique se a coluna existe e está com o nome correto.
- "Coluna 'Situação' não encontrada" (aparece no modo Atualização) → Use uma exportação do SUAP que inclua a coluna Situação (ex.: Matrículado / Trancado / Evasão).
- "Nenhum aluno atendeu aos critérios" → Verifique se na planilha de referência a coluna D está marcada como "SIM" para os cursos que você quer importar.
- "Colunas obrigatórias não encontradas" (modo de servidores) → Verifique se a exportação contém as colunas Nome, Matrícula, Cargo e Ativo.
- "Não foi possível encontrar a coluna 'E-mail Pessoal'" → No modo de inclusão de alunos, certifique-se de que a exportação contempla esse campo.

Se uma mensagem não ficar clara, copie o texto e procure um colega mais experiente ou peça auxílio ao suporte local. 💬

---

## 🔐 Privacidade e segurança

- Todo o processamento é feito no seu navegador — nada é enviado para a internet.
- Ainda assim, trate os arquivos com cuidado: contêm dados pessoais (e-mail, matrícula, nome, cargo e situação).
- Apague arquivos temporários e limpe o histórico do navegador quando necessário.

---

## 💡 Dicas práticas para evitar erros

- Sempre mantenha um backup das planilhas originais antes de processar.
- Faça um teste com poucas linhas antes de executar a base inteira.
- Use a planilha de referência como controle: marque apenas "SIM" nos cursos que realmente deseja importar.
- Em servidores, confirme se a coluna "Ativo" está correta antes de gerar o arquivo de atualização.
- Verifique a ordem das colunas no SophiA antes de importar o TXT gerado.

---

## 🧰 Exemplo rápido (mini-fluxo)

1. Exportar alunos ou servidores no SUAP conforme o cenário desejado.
2. Para alunos, abrir a planilha de referência e confirmar que os cursos estão mapeados e com "SIM" onde necessário.
3. Abrir `index.html`, selecionar arquivos, preencher Biblioteca e Data, processar, baixar TXT.
4. Abrir o TXT em um editor de texto e conferir as primeiras linhas antes de importar no SophiA.

---

## 📎 Precisa de ajuda? (quem contatar)

- Consulte o manual local da sua biblioteca ou o colega responsável por sistemas.
- Se sua instituição tiver suporte de TI/SEI, encaminhe o arquivo gerado e descreva o problema.

---

## 📝 Licença

MIT — fique à vontade para adaptar a ferramenta às suas necessidades.
