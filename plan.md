Prova Social — Registro Consolidado de Revisão, Decisões e Plano de Aplicação

> Documento de continuidade do projeto
Período: review.md → 29/09/2026
Objetivo: consolidar tudo que foi revisado, corrigido, decidido ou planejado desde o review.md, para que a implementação possa continuar sem refazer a auditoria.




---

1. Objetivo deste documento

Este documento é o registro consolidado de continuidade do Prova Social.

Ele reúne:

revisão inicial;

auditoria técnica;

problemas encontrados;

correções realizadas;

Universal Parser v3;

arquitetura de OCR;

CI/CD;

abandono do Codemagic;

migração para GitHub Actions;

problemas encontrados na CI;

estratégia de testes;

auditoria da stack;

Supabase/PostgreSQL;

catálogo;

produto e monetização;

UI/UX;

Apple HIG;

futura biblioteca do parser;

decisões pendentes;

ordem de implementação.


Regra importante

Uma proposta registrada aqui não significa que ela já foi implementada.

O documento diferencia:

implementado;

decidido;

planejado;

pendente de validação.



---

2. Estado estratégico atual

2.1 CI/CD

Codemagic

Decisão final: abandonar o Codemagic como CI principal.

O motivo foi a limitação de uso que tornou o serviço inadequado para a rotina pretendida.

Não devemos voltar a depender dele para a pipeline principal.

Nova direção

GitHub
   ↓
GitHub Actions
   ├── Pull Request → QA rápido
   ├── main → QA completo
   └── tag v* → release Android/Web

A migração para GitHub Actions já foi iniciada e executada.


---

3. Auditoria inicial baseada no review.md

A revisão inicial mostrou que o principal problema do projeto não era falta de funcionalidades, mas confiabilidade, escalabilidade e organização interna.

Os maiores riscos estavam em:

1. importação/parser;


2. publicação segura;


3. catálogo;


4. paginação;


5. testes/CI;


6. sincronização;


7. organização de páginas complexas;


8. busca;


9. resultados;


10. acessibilidade;


11. OCR;


12. arquitetura futura.




---

4. Prioridades P0

4.1 Parser/importação

A importação foi identificada como o maior risco funcional.

O sistema precisa lidar com:

OCR;

PDFs com camada de texto;

PDFs escaneados;

uma coluna;

duas colunas;

quebra de página;

cabeçalhos;

rodapés;

numeração quebrada;

alternativas A–E;

alternativas A–G;

verdadeiro/falso;

asserções;

questões abertas;

questões sem alternativas;

tabelas;

imagens;

caracteres especiais;

textos incompletos;

baixa confiança.


A importação não pode simplesmente produzir um resultado e assumir que ele está correto.


---

4.2 Publicação segura

O cliente não deve ser a autoridade final sobre uma prova publicada.

O servidor deve validar:

enunciado;

alternativas;

quantidade de alternativas;

índice da resposta;

tópico;

tipo de fonte;

duração;

ano;

tamanho do payload;

IDs;

duplicidades;

estrutura geral da prova.



---

4.3 Catálogo

Foi encontrado um padrão N+1 no ExamRepository.

Fluxo problemático:

buscar provas
   ↓
para cada prova
   ↓
buscar questões

Isso precisa ser substituído por uma estratégia agregada.

Possibilidades:

join;

consulta agregada;

RPC;

outra consulta server-side adequada.



---

4.4 Paginação

O catálogo deve usar paginação real.

Direção preferencial:

cursor pagination

em vez de depender de grandes consultas ou offsets conforme o volume crescer.


---

4.5 CI e testes

A suíte não pode levar dezenas de minutos para uma validação comum.

O problema encontrado posteriormente confirmou essa preocupação.


---

4.6 Importação/sincronização

Devem ser tratados:

interrupção;

repetição;

falha parcial;

inconsistência;

retry;

cache;

sincronização.



---

5. Prioridades P1

5.1 PublishPage

A PublishPage possui responsabilidades demais.

Direção:

UI
 ↓
estado/orquestração
 ↓
serviços
 ↓
repositórios


---

5.2 QuizPage

A mesma preocupação existe na QuizPage.

A lógica de:

prova;

respostas;

estado;

persistência;

navegação;

resultado


deve ser separada da apresentação.


---

5.3 Busca

A busca do catálogo deve evoluir para PostgreSQL Full-Text Search.

Direção:

tsvector
   ↓
GIN
   ↓
ranking


---

5.4 Resultados

Estrutura desejada:

Resultado da prova
        ↓
Desempenho por questão
        ↓
Desempenho por tópico


---

5.5 Desempenho por tópico

Decisão de produto:

Gratuito

Desempenho por tópico dentro de uma prova.

Pago

Desempenho por tópico agregado entre várias provas.

A ideia é:

> o usuário não paga para entender a prova que acabou de fazer; paga para obter análises agregadas que facilitam sua rotina.




---

5.6 Acessibilidade

Entram na revisão:

contraste;

tamanho dos alvos;

semântica;

acessibilidade;

feedback;

navegação;

estados de erro;

estados vazios.



---

5.7 Browser QA

A execução deve ser reproduzível.

O script atualmente usa:

npm ci

Portanto, o projeto deve possuir um lockfile correspondente.


---

6. Prioridades P2

progresso da importação;

cancelamento;

OCR por página;

duas colunas;

banco local quando realmente necessário;

separação do ExamRepository;

observabilidade;

métricas;

cache;

diagnóstico de qualidade.



---

7. Prioridades P3/P4

Somente depois da base:

limpeza;

componentes;

design tokens;

empty states;

microcopy;

motion;

microinterações;

polimento visual.



---

8. Universal Parser v3

8.1 Direção

A arquitetura estabelecida é:

PDF / imagem
      ↓
extração de texto / OCR
      ↓
análise de layout
      ↓
Universal Parser
      ↓
questões estruturadas
      ↓
Quality Gate

O parser não deve saber de onde o texto veio.


---

9. Melhorias do Parser v3

Foram introduzidos/planejados:

ImportedQuestion;

QuestionFormat;

normalização;

detecção robusta do início da questão;

marcadores de alternativas;

verdadeiro/falso;

asserções;

preservação de mais de cinco alternativas;

parseColumns.



---

10. Modelo futuro do parser

Direção:

enum QuestionFormat {
  multipleChoice,
  trueFalse,
  assertion,
  openEnded,
  unknown,
}

E:

class ParsedQuestion {
  final String statement;
  final List<ParsedOption> options;
  final int? number;
  final QuestionFormat format;
  final double confidence;
  final List<ParserWarning> warnings;
}

Os nomes podem mudar durante implementação, mas a ideia é preservar:

estrutura;

confiança;

warnings;

formato;

metadados.



---

11. Confiança do parser

O parser não deve funcionar somente como:

deu certo
/
deu errado

Deve poder indicar:

confidence
warnings

Isso permite:

aceitar automaticamente;

pedir revisão;

reprocessar;

detectar páginas problemáticas.



---

12. Separação OCR ↔ Parser

Decisão arquitetural central:

> O Prova Social deve consumir o parser, e não o parser depender do Prova Social.



O parser não deve depender de:

Flutter Widgets;

Supabase;

autenticação;

feed;

perfil;

assinatura;

estatísticas;

navegação.



---

13. Futura biblioteca

Possível estrutura:

repository/
├── app/
├── packages/
│   └── exam_parser/
│       ├── lib/
│       └── test/
└── docs/

Possível evolução futura:

NAGCODE-Dev/universal_exam_parser

API conceitual:

final parser = ExamParser();
final result = parser.parse(text);

A decisão atual é:

> primeiro estabilizar o parser dentro do Prova Social; depois extrair como biblioteca.




---

14. OCR — nova arquitetura

A principal mudança em relação a um OCR tradicional é:

> não escolher um único OCR para tudo.



O sistema deve primeiro entender o documento.

Documento
   ↓
Document Analyzer
   ↓
OCR Router
   ↓
engine adequado


---

15. PDF com camada de texto

Se o PDF já possuir texto confiável:

PDF
 ↓
detectar camada de texto
 ↓
texto suficiente?
 ↓
SIM
 ↓
extração nativa

Não executar OCR sem necessidade.

Isso reduz:

tempo;

CPU;

memória;

bateria;

processamento.



---

16. Engines

16.1 ML Kit

Atual:

google_mlkit_text_recognition: 0.17.1

Uso:

câmera;

imagens simples;

leitura rápida;

casos locais.


Não existe decisão de substituir o ML Kit.


---

16.2 PaddleOCR

Uso planejado:

documentos escaneados;

PDFs complexos;

duas colunas;

layout;

bounding boxes;

documentos que exigem reconstrução espacial.


PaddleOCR entra como segundo motor, não como substituto universal.


---

16.3 Tesseract

Pode existir como fallback/offline, mas não deve ser o caminho padrão se houver alternativa melhor.


---

17. OCR Router adaptativo

O Router decide qual caminho utilizar.

Exemplo:

documento
   ↓
tem camada de texto?
 ├── sim → extração nativa
 └── não
       ↓
documento simples?
 ├── sim → OCR leve/ML Kit
 └── não → PaddleOCR


---

18. Diagnóstico do documento

Antes do OCR, analisar:

camada de texto;

páginas;

DPI;

orientação;

rotação;

número de colunas;

densidade;

tabelas;

fórmulas;

caracteres especiais;

idioma;

qualidade do texto;

cabeçalhos;

rodapés;

imagens.



---

19. Classificação

Categorias planejadas:

TEXT_ONLY
TWO_COLUMNS
TABLE
IMAGE_HEAVY
FORMULA_HEAVY
MIXED


---

20. Escada de processamento

1. extração nativa
        ↓
2. OCR leve
        ↓
3. PaddleOCR
        ↓
4. processamento especializado
        ↓
5. revisão seletiva

O objetivo não é chegar sempre ao último estágio.


---

21. Teto de tempo e qualidade

O Router deve respeitar limites:

MAX_PROCESSING_TIME
MAX_RETRIES
MAX_MEMORY_CLASS
MAX_PARALLEL_PAGES
MAX_IMAGE_RESOLUTION
MIN_ACCEPTABLE_CONFIDENCE

A meta é:

qualidade alta
+
tempo aceitável
+
consumo aceitável


---

22. Processamento por página

A decisão deve ocorrer por página quando possível.

Exemplo:

página 1 → texto nativo
página 2 → texto nativo
página 3 → ML Kit
página 4 → duas colunas/PaddleOCR
página 5 → texto nativo

Uma página difícil não deve obrigar o documento inteiro a passar pelo processamento mais pesado.


---

23. Pré-processamento

Aplicar somente quando necessário:

deskew;

rotação;

crop;

denoise;

contraste;

resolução adequada.


Evitar processamento universal desnecessário.


---

24. Duas colunas

O texto não basta.

A arquitetura deve preservar geometria:

bounding boxes
      ↓
detectar colunas
      ↓
ordenar leitura
      ↓
reconstruir texto
      ↓
parser

parseColumns faz parte dessa direção.

A ordenação correta ainda precisa ser validada/garantida.


---

25. Quality Gate

Depois do OCR/parser:

GOOD
REVIEW
RETRY

Avaliar:

OCR confidence;

layout confidence;

parser confidence;

numeração;

alternativas;

coerência;

continuidade;

duplicações;

questões incompletas.



---

26. Confiança composta

A confiança final deve considerar:

OCR
+
layout
+
parser
+
coerência
+
alternativas
+
continuidade

Não confiar exclusivamente no score do OCR.


---

27. Revisão seletiva

Se somente algumas páginas estiverem ruins:

não reprocessar o documento inteiro

Reprocessar apenas as páginas problemáticas.


---

28. Cache

Chave conceitual:

document_hash
+
page
+
configuration
+
engine_version

Permite reutilizar resultados quando o conteúdo e a configuração forem equivalentes.


---

29. Paralelismo

O processamento precisa ser limitado.

Considerar:

memória;

CPU;

páginas;

engine;

resolução.


Não processar dezenas de páginas pesadas simultaneamente em aparelhos modestos.


---

30. Modos de processamento

Possíveis modos:

Fast

Tempo primeiro.

Quality

Qualidade primeiro.

Low Resource

Memória/bateria primeiro.

O padrão deve buscar equilíbrio.


---

31. UX da importação

Mostrar:

progresso;

etapa;

página atual;

cancelamento;

páginas em revisão;

resultado;

erros.


A tela não deve parecer travada durante uma operação longa.


---

32. Métricas de OCR

Registrar:

tempo por página
engine
confidence
retry
fallback
tipo de documento
questões encontradas
questões rejeitadas
páginas revisadas

Inicialmente o Router será baseado em regras.

Depois, os dados reais podem melhorar essas regras.


---

33. Stack

Manter

Flutter;

Supabase;

PostgreSQL;

Cloudflare Pages;

ML Kit;

pdfrx;

Universal Parser.


Não trocar por trocar

A auditoria não encontrou motivo para substituir as bases principais.


---

34. Dependências

Registrado:

Flutter: 3.47.0
Node: 22
Wrangler: 4.132.0

google_mlkit_text_recognition: 0.17.1
pdfrx: 2.6.1
supabase_flutter: 2.10.0

Durante a auditoria foram identificadas versões mais novas:

pdfrx → 2.6.5
supabase_flutter → 2.17.2

Essas atualizações não devem ser consideradas implementadas até validação no repositório.


---

35. Supabase/Postgres

N+1

Corrigir antes de pensar em aumentar infraestrutura.

Direção:

consulta agregada
ou RPC
ou join adequado


---

36. Paginação

Direção:

cursor pagination


---

37. Full-Text Search

Direção:

tsvector
   +
GIN
   +
ranking


---

38. Banco local

SQLite/Drift é uma possibilidade futura.

Não deve entrar antes de existir justificativa real por:

volume;

offline;

cache;

histórico;

OCR.



---

39. CI/CD — implementação realizada

Foi criado/ajustado:

.github/workflows/flutter-ci.yml

Direções incorporadas:

Flutter 3.47.0;

Node 22;

Wrangler 4.132.0;

permissões;

concurrency;

QA;

Web build;

Android/release;

artefatos;

GitHub Release.



---

40. Primeira falha da GitHub Actions

Run:

36487267610

Falhou no Setup Node.

Causa:

cache: npm
cache-dependency-path: qa/browser/package-lock.json

O lockfile não existia.

Correção:

remover configuração de cache npm baseada no arquivo inexistente;

manter Setup Node.


Commit registrado:

ci: fix Node setup without npm cache lockfile


---

41. Segunda execução

Run:

36487886249

Commit:

d1ad332f79366465263afc6e8e36f89e55e37758

Resultados:

Setup → passou
Checkout → passou
Flutter → passou
Node → passou
QA estático → passou
Testes Flutter → gargalo


---

42. Diagnóstico da suíte Flutter

A execução chegou a aproximadamente 29 minutos na etapa de testes.

Problemas observados:

TimeoutException de 10 minutos;

erros Supabase/PostgREST;

testes de navegação/editor;

falhas acumuladas;

interação fora de viewport 320x800;

aproximadamente +66 sucessos / -16 falhas em parte da execução no diagnóstico.


A execução foi cancelada.

Decisão

Não aumentar simplesmente os timeouts.

O problema é arquitetural na organização da suíte.


---

43. Nova estratégia de testes

Separar:

unit
widget
integration
Supabase
E2E/browser

Pipeline:

PR
 └── testes rápidos

main
 ├── unit
 ├── widget
 ├── integração selecionada
 └── browser QA

release
 ├── QA completo
 ├── Android
 └── Web


---

44. Browser QA

O script:

qa/run_browser.sh

usa:

npm ci

Mas o lockfile:

qa/browser/package-lock.json

não existe.

Próxima ação

Inspecionar:

qa/browser/package.json

e gerar/versionar o lockfile se essa for a estrutura correta.

Preferência:

npm ci

para builds reproduzíveis.


---

45. Web/Cloudflare

A pipeline valida:

build/web/index.html
build/web/flutter_bootstrap.js
build/web/assets/AssetManifest.json

Antes do deploy.

Ainda precisa ser validado:

SPA fallback;

deep links;

comportamento de rotas diretamente acessadas.



---

46. Produto e monetização

Gratuito

Deve incluir:

OCR forte;

importação;

desempenho por prova;

análise básica;

uso essencial para estudar.


Pago

Pode incluir:

desempenho agregado por tópico;

análises avançadas;

recursos que economizam tempo;

facilidades adicionais.


Princípio:

> Pagar para facilitar a vida, não para bloquear a função básica.




---

47. OCR gratuito

Decisão explícita:

> O OCR não deve ser propositalmente pior no plano gratuito.



O sistema deve oferecer boa qualidade porque essa é a base do produto.


---

48. UI/design

Identidade

Verde principal:

#16A36A

Light:

#F7F8F6
#FFFFFF
#171A18
#68706B
#E3E7E4

Dark:

#101311
#181C19
#F3F5F3
#2A302C

Estados:

#E5A524 → revisão
#DC4C4C → perigo

Evitar:

azul genérico;

roxo/AI gradient;

excesso de decoração;

dashboard genérico.



---

49. Apple HIG

A referência Apple deve orientar:

hierarquia;

clareza;

espaçamento;

consistência;

acessibilidade;

feedback;

interação previsível.


Não copiar a identidade visual da Apple.

O Prova Social continua com sua identidade própria.


---

50. Componentes/estrutura visual

Direções registradas:

cards expansíveis;

catálogo com comportamento simples;

botão + verde;

accordion;

skeleton loading;

ajuda/configurações inferiores;

pouco texto;

empty states;

feedback claro.



---

51. Histórico de commits relevantes

fa174cd
→ parser v3 inicialmente integrado

6e545c3
→ feat(parser): universal exam parser v3

47b3091
→ ci: migrate and harden GitHub Actions pipeline

d1ad332f79366465263afc6e8e36f89e55e37758
→ style: format universal parser

Os hashes devem ser conferidos no repositório antes de qualquer patch novo, pois podem existir commits posteriores.


---

52. Arquivos importantes

.github/workflows/flutter-ci.yml

qa/run_browser.sh
qa/run_tests.sh
qa/browser/

lib/core/import/question_parser.dart
test/question_parser_test.dart

pubspec.yaml

ExamRepository
PublishPage
QuizPage

Também foram usados:

CODEX_QA_COMPLETO_PROVA_SOCIAL.md
docs/qa/
qa/


---

53. Arquitetura geral desejada

PROVA SOCIAL
                              │
                         Flutter App
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             UI           Importação         Backend
             │                │                │
       Design System    Document Analyzer   Supabase
             │                │                │
       Apple HIG        PDF/Text Layer     PostgreSQL
             │                │                │
       Accessibility    OCR Router          FTS/GIN
                            │
                  ┌─────────┴─────────┐
                  │                   │
             ML Kit              PaddleOCR
                  │                   │
                  └─────────┬─────────┘
                            ↓
                      Layout Engine
                            ↓
                   Universal Parser
                            ↓
                       Quality Gate
                            ↓
                  revisão seletiva
                            ↓
                  questões estruturadas
                            ↓
              ┌─────────────┴─────────────┐
              │                           │
          publicação                    quiz
              │                           │
          catálogo                    resultados
              │                           │
          comunidade                  tópicos

Infra:

GitHub
  ↓
GitHub Actions
  ├── PR QA
  ├── main QA
  └── release

Cloudflare Pages
  ↓
Web

Supabase
  ↓
PostgreSQL


---

54. Ordem de implementação

A ordem recomendada é:

1. estabilizar Universal Parser
        ↓
2. validação server-side
        ↓
3. OCR Router
        ↓
4. Document Analyzer
        ↓
5. Quality Gate
        ↓
6. cache/métricas OCR
        ↓
7. corrigir N+1
        ↓
8. paginação
        ↓
9. PostgreSQL FTS
        ↓
10. dividir testes/CI
        ↓
11. PublishPage
        ↓
12. QuizPage
        ↓
13. resultados/tópicos
        ↓
14. acessibilidade
        ↓
15. design system
        ↓
16. polimento

A ordem pode mudar caso uma implementação revele uma dependência real, mas a prioridade deve continuar sendo estabilidade antes de estética.


---

55. Próxima rotina rápida

Ao retomar o projeto:

git status
git log --oneline -15

Depois verificar:

.github/workflows/flutter-ci.yml
qa/run_tests.sh
qa/run_browser.sh
qa/browser/package.json
qa/browser/package-lock.json
pubspec.yaml
question_parser.dart

Em seguida:

1. medir novamente a suíte;


2. dividir os testes;


3. corrigir Browser QA;


4. estabilizar parser;


5. iniciar Document Analyzer;


6. implementar OCR Router;


7. implementar Quality Gate;


8. atacar N+1;


9. implementar paginação;


10. implementar FTS.




---

56. Critério de sucesso da importação

Uma importação bem-sucedida deve significar:

documento
   ↓
diagnóstico
   ↓
processamento mínimo adequado
   ↓
layout reconstruído
   ↓
parser
   ↓
confidence
   ↓
Quality Gate
   ↓
GOOD / REVIEW / RETRY
   ↓
questões confiáveis

Não simplesmente:

OCR terminou → sucesso


-
