# 📘 Leia-me Especializado — Ferramenta SUAP → SophiA

Este documento é destinado a bibliotecários com familiaridade em informática (intermediário/avançado) que desejam entender todos os pormenores técnicos da ferramenta, como ela processa dados, suas limitações e como estendê-la. Inclui explicações sobre o código presente em `index.html`, formato de entrada/saída, tratamento de dados, sugestões de melhoria e dicas de depuração.

---

## 🧩 Visão técnica geral

A ferramenta é uma página HTML estática que roda 100% no cliente (navegador). Ela depende de bibliotecas carregadas via CDN:

- Tailwind CSS — apenas para estilização visual.
- SheetJS (xlsx.full.min.js) — leitura de planilhas Excel (.xls, .xlsx) no navegador.

Fluxo resumido:
1. Ler planilha de referência (.xlsx) e construir um mapa (courseMap) onde a chave é o nome do curso normalizado do SUAP e o valor contém: nome convertido para SophiA, tipo de usuário e flag `importar` (boolean).
2. Ler planilha exportada do SUAP (.xls/.xlsx). Detectar a linha de cabeçalho (assumida como a primeira linha) e localizar os índices de colunas relevantes por comparação com uma lista de variações de nomes.
3. Iterar sobre linhas de dados, normalizar nomes de curso, consultar `courseMap`, filtrar por `importar === true` e montar as linhas de saída conforme o modo (inclusão/atualização).
4. Gerar conteúdo TXT (UTF-8), substituindo `;` internos por `,` e juntando colunas com `;`. Baixar o arquivo via Blob e URL.createObjectURL.

---

## 🧠 Detalhes do processamento (linha a linha)

Trechos-chave e comportamento:

- Leitura de arquivos:
  - `readFileAsArrayBuffer(file)` usa `FileReader` para obter ArrayBuffer — necessário para o `XLSX.read(..., {type:'array'})`.
  - SheetJS é chamado com `{ type: 'array' }` para referência e `{ type: 'array', cellText: false, cellDates: true }` para a planilha SUAP. `cellDates: true` tenta transformar células formatadas como data em objetos Date.

- Conversão em matriz:
  - `XLSX.utils.sheet_to_json(sheet, { header: 1, raw: false })` retorna uma matriz (array of arrays) onde a primeira linha é o cabeçalho.
  - `header:1` garante que mesmo com formatos diferentes a ordem original das colunas é mantida.

- Normalização de textos:
  - Função `normalizeStr(str)` executa: return String(str).trim().replace(/\s+/g,' ').toUpperCase();
  - Objetivo: remover espaços extras, unificar caixa e eliminar variações triviais (acentuação não é removida — cuidado).
  - Observação: acentuação e caracteres especiais permanecem; se for necessária correspondência sem acentos, adicionar `normalize('NFD').replace(/\p{Diacritic}/gu, '')` ou similar.

- Construção do courseMap:
  - Para cada linha (a partir da 2ª), pega: A=suapName, B=correctName, C=userType, D=importarStr.
  - `importarStr` é padronizado via `toUpperCase()` e comparado com "SIM" para definir Boolean `importar`.
  - A chave usada no `courseMap` é `normalizeStr(suapName)`.

- Detecção de índices (headers):
  - `getIdx(variations)` itera sobre uma lista de variações (ex.: ["Nome","Nome do aluno"]) e compara `normalizeStr(h)` com `normalizeStr(v)`.
  - Importante: a busca é exata após normalização. Nomes diferentes (ex.: "Curso - Nome") podem não ser encontrados.

- Regras de negócio principais:
  - Só são admitidos alunos cujo `originalCourse` exista no `courseMap` e `mappedData.importar === true`.
  - No modo `inclusao` a linha de saída tem 8 campos (inclui e-mail e validade). A coluna final é `"0"` (inativo = 0 significa ativo).
  - No modo `atualizacao` a linha de saída possui 7 campos (não traz e-mail) — valida e inativa conforme `situação` (só alunos com `MATRICULADO` recebem validade e `inativo=0`).
  - `sitNorm === 'MATRICULADO'` é a única condição usada para considerar ativo; variações como `MATRICULADO (REGULAR)` serão normalizadas por `normalizeStr` e devem corresponder a "MATRICULADO" exatamente — pode ser necessário melhorar a verificação.

- Geração do TXT:
  - Antes de concatenar, campos internos com `;` são convertidos para `,` para não quebrar o separador.
  - Linha terminada com CRLF (`\r\n`).
  - Blob criado com tipo `text/plain;charset=utf-8`.

---

## 🧪 Regras e casos de borda a observar

1. Cabeçalho ausente ou deslocado: a ferramenta assume que a primeira linha é cabeçalho. Planilhas com metadados (linhas acima) irão falhar. Solução: remover linhas extras antes de importar.
2. Variação de nomes de coluna: se a exportação do SUAP usar outro rótulo, `getIdx` pode não localizar. Verifique os nomes de coluna ou adicione mais variações em `getIdx`.
3. Acentuação no nome do curso: `normalizeStr` não remove acentos — por isso a correspondência com `courseMap` pode falhar se a planilha de referência e a exportação SUAP tiverem acentuação divergente (ex.: "Administração" vs "ADMINISTRACAO"). Recomenda-se normalizar removendo diacríticos em ambos os lados.
4. Datas: o campo Data de Validade é tratado como string fornecida pelo usuário (form input). Não há validação de formato. Recomenda-se implementar validação/parse com regex (ex.: `/^\d{2}\/\d{2}\/\d{4}$/`) e conversão para formato interno.
5. E-mails: no modo `inclusao` o e-mail é incluído sem validação. Pode ser útil validar sintaxe básica com regex.
6. Grandes planilhas: leitura inteira em memória. Para milhares de linhas o navegador pode consumir muita memória; testar limites no navegador alvo.
7. Encodings: UTF-8.

---

## 🧩 Sugestões de melhorias técnicas (prioritárias)

1. Normalização avançada dos nomes (remover acentos):
   - Use `str.normalize('NFD').replace(/\p{Diacritic}/gu, '')` ou `replace(/[\u0300-\u036f]/g, "")` para unificar strings sem diacríticos.

2. Validação de campos antes do processamento:
   - Verificar formato DD/MM/AAAA com regex.
   - Validar e-mail com regex simples.
   - Confirmar que `nomeBiblioteca` não está vazio.

3. Mapeamento fuzzy opcional:
   - Integrar comparação por distância de Levenshtein (ou fuse.js) para sugerir mapeamentos caso `courseMap[searchKey]` falhe.

4. Relatório de processamento:
   - Gerar um CSV/JSON de log com linhas ignoradas e motivos (curso desconhecido, importar=NÃO, campos faltantes) para auditoria.

5. Suporte a múltiplas planilhas (SheetNames):
   - Permitir que o usuário escolha a aba correta quando o arquivo tiver mais de uma.

6. Internacionalização e formatação de data:
   - Fornecer picker de data e armazenar no formato esperado pelo SophiA.

7. Testes automatizados:
   - Extrair a lógica de transformação para funções testáveis (unit tests com Jest via Node/webpack) e adicionar fixtures (ex.: small_suap.xlsx, ref_example.xlsx).

8. Versão CLI/Node:
   - Criar versão para Node.js (usando `xlsx` ou `sheetjs`) para execução em servidor ou CI e integração com pipelines institucional.

---

## 🧾 Formato esperado da planilha de referência (exemplo)

Coloque o cabeçalho na primeira linha (opcional), os dados a partir da segunda linha. Exemplo:

| A (Curso SUAP) | B (Curso SophiA) | C (Tipo Usuário) | D (Importar?) |
|---|---|---:|---|
| Administração | ADMINISTRACAO | Aluno | SIM |
| Biologia | BIOLOGIA | Aluno | NÃO |

Observações:
- Espaços em branco ao redor dos nomes são automaticamente removidos.
- A comparação é feita após `normalizeStr` (trim + uppercase + compress spaces).

---

## 🧾 Exemplo de cabeçalho esperado na exportação SUAP

A ferramenta tenta localizar colunas pelas seguintes variações (não exaustivo):

- Nome: `Nome`, `Nome do aluno`
- Matrícula: `Matrícula`, `Matricula`
- Curso: `Curso`
- E-mail: `E-mail pessoal`, `E-mail pessoa`, `Email pessoal`, `E-mail`
- Situação: `Situação`, `Situacao`, `Situação no Curso`

Se os nomes forem diferentes, ajuste a planilha ou modifique `getIdx` no código.

---

## 🛠️ Depuração e troubleshooting avançado

- Console do navegador: abra DevTools (F12) e observe `console.error()` logs; a aplicação já faz `console.error(error)` em caso de exceção.
- Inspecione `refRows` e `mainRows` (após a leitura) no console para confirmar estrutura (arrays de arrays).
- Para checar `courseMap`: após a leitura da referência execute `console.log(Object.keys(courseMap).slice(0,50))` para ver as chaves normalizadas.
- Teste com arquivos reduzidos (10–50 linhas) antes de largar em todo o dataset.
- Se houver problemas de memória, testar com outro navegador (Chrome tende a usar menos memória que Firefox em alguns casos) ou aumentar swap/recursos da máquina.

---

## 🌐 Compatibilidade e execução

- Navegadores testados: Chrome, Edge, Firefox (desktop). Mobile não recomendado para processar grandes bases.
- Requer internet apenas para carregar Tailwind e SheetJS via CDN. Para uso off-line, baixe os scripts e referencie localmente.

---

## 📁 Como estender o código (pontos de entrada)

Arquitetura atual: tudo está em `index.html` no bloco `<script>`.

Sugestão de refatoração mínima:
1. Mover funções utilitárias para `src/utils.js` (readFileAsArrayBuffer, normalizeStr, getIdx, etc.).
2. Extrair a transformação principal para `src/transformer.js` com funções exportáveis:
   - buildCourseMap(refRows)
   - parseSuap(mainRows, headers, courseMap, options)
   - generateTxt(rows, options)
3. Configurar um bundler simples (Vite, Rollup ou Webpack) para facilitar testes e modularização.
4. Escrever testes unitários para `buildCourseMap` e `parseSuap` usando Node + Jest.

---

## 🔒 Segurança, privacidade e conformidade

- Nenhum dado é enviado à rede pela aplicação por design; no entanto, as bibliotecas externas (CDN) são carregadas se o usuário não estiver em ambiente offline.
- Recomendado: armazenar uma cópia local dos assets (Tailwind + SheetJS) e servir internamente em ambiente institucional para evitar dependência externa e melhorar conformidade.
- Dados sensíveis: matrícula, nome, e-mail. Trate-os conforme as políticas da instituição (LGPD no Brasil). Remover logs sensíveis ao publicar exemplos.

---

## 🧾 Logs e relatório de auditoria (recomendado)

Implemente geração automática de um arquivo `process_log.json` (ou CSV) contendo:
- Data/hora do processamento
- Nome do arquivo SUAP de entrada (nome do ficheiro)
- Número de linhas lidas
- Número de linhas processadas
- Linhas ignoradas (com motivo: curso desconhecido / importar=NÃO / campos faltantes)
- Versão da ferramenta (inserir campo manualmente)

Isso facilita auditoria e rastreabilidade quando importando em produção.

---

## 🧩 FAQ técnica rápida

Q: Por que alguns cursos não são processados mesmo estando na referência?
A: Diferença de normalização (acentos, símbolos, múltiplos espaços) ou erro de mapeamento (coluna D = NÃO). Verifique `normalizeStr` e a presença de `SIM`.

Q: Posso usar a planilha de referência com mais colunas?
A: Sim, apenas as primeiras quatro colunas são usadas — colunas extras são ignoradas.

---

## 🧾 Boas práticas de manutenção do repositório

- Inclua `examples/` com: `ref_example.xlsx` e `suap_sample.xlsx` para testes. (Posso adicionar se desejar.)
- Crie uma branch `dev` e trabalhe mudanças significativas lá; utilize PR para revisão.
- Inclua CHANGELOG.md para rastrear mudanças que afetam o formato de saída.

---

## ⚖️ Licença

MIT — veja o arquivo LICENSE (ou adicione um se ainda não existir). 
