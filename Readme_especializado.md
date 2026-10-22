# 📘 Leia-me Especializado — Ferramenta SUAP → SophiA

Este documento é destinado a bibliotecários e profissionais com familiaridade em informática que desejam entender os detalhes técnicos da ferramenta, como ela processa planilhas, quais regras de negócio são aplicadas e como o novo modo de servidores foi incorporado ao fluxo principal.

---

## 🧩 Visão técnica geral

A ferramenta é uma página HTML estática que roda 100% no navegador. Ela depende de bibliotecas carregadas via CDN:

- Tailwind CSS — apenas para apresentação visual.
- SheetJS (`xlsx.full.min.js`) — leitura de arquivos Excel (`.xls`, `.xlsx`) e CSV no cliente.

Fluxo geral:
1. Ler a planilha de origem do SUAP e identificar a linha de cabeçalho.
2. Detectar o modo de operação selecionado (`inclusao`, `atualizacao` ou `servidores`).
3. Para alunos, ler o dicionário de cursos e construir um mapa de tradução do SUAP para o SophiA.
4. Para servidores, automatizar a classificação por cargo e aplicar a regra de inatividade com base na coluna `Ativo`.
5. Montar linhas de saída no formato exigido pelo SophiA, sanitizar campos e gerar o arquivo `.txt` com separador `;`.
6. Realizar download do arquivo com `Blob` e `URL.createObjectURL()`.

---

## 🧠 Arquitetura do processamento

### 1) Leitura dos arquivos

A função principal de leitura usa `FileReader` para converter o arquivo enviado pelo usuário em `ArrayBuffer`:

- `readFileAsArrayBuffer(file)`
- em seguida: `XLSX.read(buffer, { type: 'array' })`

Isso permite que `SheetJS` leia a aba ativa do arquivo Excel e a transforme em uma matriz bidimensional, mantendo a ordem original das colunas e das linhas.

O processamento converte a planilha em um array de arrays:

- `XLSX.utils.sheet_to_json(sheet, { header: 1 })`
- cada índice representa uma coluna
- cada linha representa um registro

Essa estratégia simplifica a lógica de detecção de cabeçalhos e a leitura dos campos por posição.

### 2) Normalização de textos

A função `normalizeStr(str)` realiza:

- `String(str).trim()`
- remoção de espaços duplicados com `replace(/\s+/g, ' ')`
- conversão para maiúsculas com `toUpperCase()`

Exemplo:

- `"  biologia  "` → `"BIOLOGIA"`
- `"Engenharia de Produção"` → `"ENGENHARIA DE PRODUCAO"`

Importante: ela não remove acentos nem caracteres especiais. Isso significa que a comparação entre nomes do SUAP e da referência depende de consistência textual. Em cenários de variabilidade de grafia, a ferramenta pode falhar caso o mesmo nome apareça com acentos, espaços diferentes ou alternâncias de grafia.

---

## 🔁 Modo de operação e regras de negócio

### Modo 1 — `inclusao`

Objetivo: criar registros novos de alunos no SophiA.

Regras:
- exige o arquivo da planilha de alunos do SUAP;
- exige a planilha de referência de cursos;
- considera apenas alunos cujo curso exista no dicionário e tenha `Importar = SIM`;
- inclui e-mail na linha de saída;
- o campo `Inativo` é sempre `0` (ativo);
- data de validade é sempre preenchida com o valor informado pelo usuário.

Ordem final dos campos:
1. Nome
2. Matrícula
3. Curso
4. Tipo de usuário
5. Biblioteca
6. E-mail
7. Data de validade
8. Inativo

Exemplo de registro:

`Maria Souza;20240123;Ensino Médio Integrado;Biblioteca Gama; maria@email.com;19/02/2027;0`

### Modo 2 — `atualizacao`

Objetivo: atualizar alunos já cadastrados no SophiA e inativar os sem matrícula vigente.

Regras:
- exige dicionário de cursos e planilha geral de alunos do SUAP;
- usa a coluna `Situação` para decidir o status da linha;
- somente alunos com `Situação = MATRICULADO` recebem validade e `Inativo = 0`;
- quem não estiver matriculado recebe `Data de validade = ""` e `Inativo = 1`;
- não inclui e-mail na saída;
- a estrutura do TXT é mais curta.

Ordem final dos campos:
1. Nome
2. Matrícula
3. Curso
4. Tipo de usuário
5. Biblioteca
6. Data de validade
7. Inativo

### Modo 3 — `servidores`

Objetivo: realizar inclusão e atualização de servidores em lote.

Este é o novo fluxo adicionado à ferramenta. Ele foi desenhado para tratar o SUAP como fonte de cadastro de servidores, com a lógica seguinte:

- a planilha de referência não é usada;
- a ferramenta busca as colunas de Nome, Matrícula, Cargo e Ativo;
- o tipo de usuário é inferido automaticamente pelo cargo:
  - se o cargo contiver `PROFESSOR` → `Docente`
  - se não contiver → `Técnico-administrativo`
- a coluna `Ativo` define se o servidor permanece ativo ou deve ser inativado;
- se `Ativo` estiver com valor equivalente a `SIM`, a linha recebe:
  - `Data de validade = valor informado pelo usuário`
  - `Inativo = 0`
- se o valor for diferente de `SIM` ou a linha for considerada inativa, a linha recebe:
  - `Data de validade = ""`
  - `Inativo = 1`
- o e-mail é extraído da coluna `E-mail para contato` quando disponível.

Ordem final dos campos:
1. Nome
2. Matrícula
3. Tipo de usuário
4. Biblioteca
5. E-mail
6. Data de validade
7. Inativo

Exemplo:

`João Pereira;123456;Docente;Biblioteca Gama;joao@ifb.edu.br;19/02/2027;0`

---

## 🧩 Lógica específica do modo de servidores

### Detecção das colunas

A ferramenta percorre o cabeçalho da planilha procurando por variações textuais normalizadas. O padrão buscado inclui:

- `Nome`
- `Matrícula` / `Matricula`
- `E-mail para contato` / `Email para contato`
- `Cargo`
- `Ativo`

Se uma dessas colunas não existe, a validação falha com uma mensagem de erro clara.

### Regra de tipo de usuário

A lógica implementada é simples e direta:

```javascript
let tipoUsuario = cargoStr.includes('PROFESSOR') ? 'Docente' : 'Técnico-administrativo';
```

Isso pressupõe que a classificação dos cargos no SUAP seja suficiente para diferenciar docentes de técnicos administrativos. Caso o sistema venha a incluir outros perfis, esse ponto pode evoluir para uma regra mais robusta com mapa de cargos.

### Regras de inativação

A partir do campo `Ativo`:

- `Ativo = SIM` → servidor ativo
- qualquer outro valor → servidor inativo

Em ambiente de importação, a linha de saída é montada assim:

```javascript
let dtFmt = "";
let inativoVal = "1";

if (ativoStr === 'SIM') {
    dtFmt = dataValidade;
    inativoVal = "0";
}
```

Esse comportamento foi pensado para sincronizar corretamente com o processo institucional de renovação e inativação no SophiA.

---

## 🧮 Mapeamento de cursos para alunos

Para os modos de alunos, a ferramenta lê a planilha de referência e constrói um dicionário chamado `dicionario`, cuja chave é o nome do curso normalizado do SUAP:

```javascript
const dicionario = {};
for (let i = 1; i < matrizDic.length; i++) {
    const row = matrizDic[i];
    if (row.length > 0 && row[0]) {
        dicionario[normalizeStr(row[0])] = {
            cursoSophiA: row[1] || "",
            tipo: row[2] || "",
            importar: (normalizeStr(row[3]) === 'SIM')
        };
    }
}
```

A lógica de filtragem toma as seguintes decisões:

- o curso da linha do SUAP é normalizado;
- o nome é buscado em `dicionario`;
- se não existir, a linha é ignorada;
- se existir, mas `importar === false`, a linha é ignorada;
- se existir e `importar === true`, a linha é exportada.

Esse mecanismo garante que apenas cursos aprovados no dicionário sejam considerados para geração do arquivo.

---

## 📄 Geração do arquivo TXT

Ao final do processamento, as linhas são concatenadas em um único conteúdo textual:

- separador: `;`
- quebra de linha: `\r\n`
- codificação: UTF-8
- sem cabeçalho

Antes da concatenação, valores internos que contêm `;` são substituídos por `,` para não quebrar o formato do arquivo de importação.

Além disso, em planilhas grandes, a ferramenta divide o conteúdo em partes para não ultrapassar o limite aproximado de 950 KB por arquivo gerado:

```javascript
const MAX_BYTES = 950 * 1024;
```

Quando o tamanho excede esse limite, o sistema gera múltiplos downloads do tipo:

- `Importacao_SophiA.txt`
- `Importacao_SophiA_Parte2.txt`
- etc.

Para servidores, o sufixo do nome do arquivo é:

- `Importacao_SophiA_Servidores.txt`

---

## 🧪 Regras e casos de borda a observar

1. Cabeçalho ausente ou deslocado: a ferramenta assume que a linha do cabeçalho aparece entre as primeiras linhas. Se houver metadados acima, o processo pode falhar. Solução: remover linhas extras antes do import.
2. Variação textual em nomes de colunas: o algoritmo usa comparação normalizada. Se a exportação do SUAP usa rótulos diferentes, pode ser necessário ajustar o código ou a planilha.
3. Acentuação e espaços extras: `normalizeStr` não remove acentos. Isso pode causar falha em correspondências de curso entre duas planilhas que diferem apenas em acento ou espaçamento.
4. Data de validade: o valor é assumido como string e deve seguir o formato esperado pela interface (`yyyy-mm-dd` no input e `dd/mm/yyyy` no TXT).
5. E-mail: no modo de inclusão de alunos, o valor é inserido sem validação profunda. Em produção, vale testar conforme a política institucional.
6. Leitura em memória: arquivos muito grandes podem consumir bastante memória do navegador. A solução atual já divide em partes, mas testes com lotes grandes continuam recomendados.
7. Encodings: o arquivo final é gerado em UTF-8, o que ajuda a manter acentuação e caracteres específicos do idioma.

---

## 🧩 Sugestões de melhorias técnicas (prioritárias)

1. Normalização avançada de curso:
   - remover acentos, diacríticos e caracteres especiais antes da comparação.
   - usar `normalize('NFD').replace(/\p{Diacritic}/gu, '')` ou equivalente.

2. Validação de campos antes do processamento:
   - verificar formato `dd/mm/yyyy`
   - validar e-mail
   - garantir que `nomeBiblioteca` esteja preenchido

3. Mapeamento fuzzy para cursos e cargos:
   - comparar por similaridade textual para reduzir falhas em grafias variantes.

4. Relatório de auditoria:
   - gerar `process_log.json` com quantidade de linhas lidas, ignoradas e motivos.

5. Suporte a múltiplas abas de Excel:
   - permitir que o usuário selecione a aba correta quando a planilha tiver mais de uma.

6. Tratamento mais robusto do campo `Ativo`:
   - aceitar variações como `Sim`, `S`, `1`, `true`, `Ativo`, etc., de forma explícita.

7. Modularização do código:
   - extrair funções utilitárias para `src/utils.js`
   - mover a lógica de transformações para `src/transformer.js`

8. Testes automatizados:
   - criar fixtures de planilhas de alunos e servidores
   - validar o resultado por modos diferentes

---

## 🧾 Estrutura esperada das planilhas de entrada

### Planilha de referência de cursos

A planilha deve conter, no mínimo:

| A (Curso SUAP) | B (Curso SophiA) | C (Tipo Usuário) | D (Importar?) |
|---|---|---:|---|
| Administração | ADMINISTRACAO | Aluno | SIM |
| Biologia | BIOLOGIA | Aluno | NÃO |

Observações:
- a leitura usa apenas as colunas A, B, C e D;
- colunas extras são ignoradas;
- a comparação é feita após normalização.

### Planilha SUAP de alunos

Os campos esperados incluem:

- Nome
- Matrícula
- Curso
- E-mail pessoal
- Situação

No modo de atualização, a coluna `Situação` é fundamental para distinguir quem deve ser mantido ativo e quem deve ser inativado.

### Planilha SUAP de servidores

Os campos esperados incluem:

- Nome
- Matrícula
- Cargo
- E-mail para contato
- Ativo

A presença de `Cargo` e `Ativo` é determinante para a execução do novo fluxo.

---

## 🛠️ Depuração e troubleshooting avançado

### Console do navegador

Abra o DevTools (F12) para acompanhar os logs de exceção. A aplicação já chama `console.error(error)` em caso de falha.

### Dicas práticas

- verifique se a linha de cabeçalho está correta;
- teste com uma base pequena antes de processar o conjunto completo;
- compare os nomes das colunas e o texto exato das exportações SUAP;
- avalie se o cargo do servidor está escrito com grafia compatível (por exemplo, `Professor` ou `PROFESSOR`);
- revise a célula `Ativo` para confirmar que o valor é `SIM` para quem deve permanecer ativo.

### Diagnóstico rápido

Caso o sistema diga que não encontrou colunas obrigatórias, verifique:

- se a exportação foi feita na aba correta;
- se o nome da coluna está em português ou em outra grafia;
- se há colunas vazias, mescladas ou com cabeçalhos extras acima da tabela;
- se o arquivo foi salvo em formato compatível (`.xls`, `.xlsx` ou `.csv`).

---

## 🌐 Compatibilidade e execução

- Navegadores recomendados: Chrome, Edge e Firefox em desktop.
- Requer internet apenas para carregar as bibliotecas CDN. Em ambiente offline, baixe os arquivos localmente e altere as referências em `index.html`.
- A página é leve e não depende de backend, mas ela processa dados sensíveis no cliente, então é importante ter cuidado na máquina utilizada.

---

## 📁 Como estender o código

A arquitetura atual está concentrada em `index.html`, dentro do bloco `<script>`. Para evoluir a ferramenta com segurança, a recomendação é:

1. extrair funções utilitárias para `src/utils.js`
2. separar a transformação principal em `src/transformer.js`
3. criar testes automatizados para:
   - `normalizeStr`
   - `buildCourseMap`
   - geração de linhas por tipo de operação
   - lógica de servidores
4. separar validações por módulo, evitando que regras de negócio fiquem dispersas no código de interface

---

## 🔒 Segurança, privacidade e conformidade

- A aplicação não envia dados para a internet por design.
- Os arquivos são lidos localmente no navegador.
- Matrícula, nome, e-mail, cargo e situação são dados sensíveis. Recomendamos tratar esses dados conforme a política de privacidade da instituição.
- Em ambientes corporativos, é prudente manter uma cópia local dos scripts do projeto (Tailwind e SheetJS) para reduzir dependência de links externos.

---

## 🧾 Logs e auditoria (recomendado)

Para uso em produção, vale implementar um log com:

- data e hora do processamento
- nome do arquivo de entrada
- tipo de operação executada
- número de registros lidos
- número de registros exportados
- linhas ignoradas e motivos
- versão da ferramenta

Isso melhora a rastreabilidade e facilita auditoria em bibliotecas, unidades acadêmicas e processos de importação mensais.

---

## 🧩 FAQ técnica rápida

Q: Por que alguns cursos não são processados mesmo estando na referência?
A: Em geral, por diferença de normalização (acentos, espaços, grafias diferentes) ou por `Importar = NÃO`.

Q: Posso usar a planilha de referência com mais colunas?
A: Sim. A ferramenta utiliza apenas as colunas fundamentais; colunas extras são ignoradas.

Q: O modo de servidores exige referência de cursos?
A: Não. Ele ignora o dicionário e usa a inferência por cargo.

Q: O que acontece se o campo `Ativo` vier com outro valor além de `SIM`?
A: A linha é tratada como inativa, recebendo `Data de validade = ""` e `Inativo = 1`.

---

## 🧾 Boas práticas de manutenção do repositório

- manter exemplos em `examples/`
- documentar mudanças de regra em `CHANGELOG.md`
- utilizar branches para alterações relevantes
- validar manualmente saídas antes de agir em produção

---

## ⚖️ Licença

MIT — veja o arquivo LICENSE ou mantenha a política atual do projeto para uso e adaptação.

---

## 📌 Resumo executivo do novo modo

O terceiro modo da ferramenta foi incorporado para cobrir a base de servidores com duas finalidades principais:

- incluir novos servidores
- atualizar servidores existentes sem depender de dicionário de cursos

No processo:
- o cargo delimita o tipo de usuário;
- a coluna `Ativo` controla a data de validade e a situação inativa;
- o arquivo gerado segue a ordem exigida pelo SophiA;
- a ferramenta continua totalmente offline e operando no navegador.

Esse ajuste mantém o espírito original do projeto — automatizar e reduzir erros manuais — e amplia a utilidade da ferramenta para além do público estudantil.
