# Análise do simulador de investimentos da ArenaCash

Este documento explica, em linguagem simples, como o simulador decide qual investimento indicar para cada pessoa. Ele foi escrito a partir da leitura do arquivo `simulador.js` do repositório público `Product-Arena/arena-cash`. Nenhum teste foi rodado: os exemplos foram calculados à mão seguindo as regras do arquivo (os arredondamentos de salário nas fronteiras foram conferidos por cálculo).

---

## 1. O que o simulador faz

O simulador pede três informações: **idade**, **salário mensal** e **perfil de investidor** (conservador, moderado ou arrojado). Com elas, ele estima quanto a pessoa poderia investir por mês (sempre 15% do salário), calcula uma nota de risco entre 2 e 6 e devolve um dos cinco produtos do catálogo, junto com o valor mensal sugerido e uma lista de motivos. Duas regras especiais passam na frente da nota: uma para quem consegue guardar muito pouco e outra que limita o acesso ao produto mais arriscado.

---

## 2. Os produtos

| Produto | Rendimento | Prazo | Risco |
|---|---|---|---|
| CDB Reserva | 102% do CDI | Saque a qualquer momento | Muito baixo |
| LCI Isenta | 95% do CDI, sem Imposto de Renda | 1 ano sem poder sacar | Baixo |
| CDB Progressivo | 112% do CDI | 2 anos | Baixo |
| CDB Longo Prazo | 124% do CDI | 5 anos (saque antes do prazo tem perda) | Médio |
| Arena Quant | Variável, sem garantia | Saque em 30 dias | Alto |

*CDI é a taxa de referência da renda fixa; "102% do CDI" significa que o dinheiro rende um pouco mais que essa taxa.*

---

## 3. Como a nota é calculada, passo a passo

**Passo 1 — Quanto a pessoa investiria por mês.**
O simulador pega 15% do salário e arredonda. Exemplo: salário de R$ 4.000 vira um aporte de R$ 600. Esse número é uma estimativa; o simulador não pergunta quanto a pessoa realmente consegue guardar.

**Passo 2 — Pontos pela idade** (quanto mais nova, mais tempo para esperar o dinheiro render):

| Idade | Pontos |
|---|---|
| Menos de 30 anos | 3 |
| De 30 a 49 anos | 2 |
| 50 anos ou mais | 1 |

**Passo 3 — Pontos pelo perfil declarado:**

| Perfil | Pontos |
|---|---|
| Arrojado | 3 |
| Moderado | 2 |
| Conservador | 1 |

**Passo 4 — Somar.** A nota final é a soma dos dois, e vai de 2 (mais cauteloso possível) a 6 (mais arrojado possível).

**Passo 5 — Escolher o produto pela nota:**

| Nota | Produto indicado |
|---|---|
| 2 | CDB Reserva |
| 3 | LCI Isenta |
| 4 | CDB Progressivo |
| 5 | CDB Longo Prazo |
| 6 | Arena Quant (se o aporte permitir, veja a seção 4) |

---

## 4. As regras que passam na frente da nota

**Regra 1 — Reserva antes de tudo.**
Se o aporte estimado for **menor que R$ 100 por mês**, o resultado é sempre **CDB Reserva**, não importa a idade nem o perfil. É a primeira coisa que o simulador verifica. O arquivo explica que o objetivo é evitar que quem guarda pouco prenda o dinheiro em prazo longo e precise resgatar com prejuízo, e reconhece que essa é a regra que mais gera reclamação de clientes jovens e arrojados.

**Regra 2 — O Arena Quant exige aporte de pelo menos R$ 500 por mês.**
Quem chega à nota 6 só recebe o Arena Quant se o aporte for de R$ 500 ou mais. Se for menor, a indicação "desce um degrau" para o **CDB Longo Prazo**, e o resultado explica isso por escrito.

**Valores de salário equivalentes aos limites:**
- Aporte abaixo de R$ 100: salário até R$ 663.
- Aporte de R$ 500 ou mais: salário a partir de R$ 3.330.

---

## 5. Casos-limite, com exemplos

| # | Situação | Idade | Salário | Perfil | Resultado | O que chama atenção |
|---|---|---|---|---|---|---|
| 1 | Perfil ignorado pela Regra 1 | 25 | R$ 600 | Arrojado | CDB Reserva | Pessoa jovem e arrojada recebe o produto mais conservador; a tela ainda mostra "nota 6 de 6" |
| 2 | Fronteira dos R$ 100 | 25 | R$ 663 | Arrojado | CDB Reserva | Aporte de R$ 99 |
| 3 | Fronteira dos R$ 100 | 25 | R$ 664 | Arrojado | CDB Longo Prazo | R$ 1 a mais de salário muda de saque imediato para 5 anos de prazo |
| 4 | Aporte insuficiente para o Quant | 25 | R$ 2.000 | Arrojado | CDB Longo Prazo | A pessoa mais arrojada possível não recebe o produto arrojado |
| 5 | Salário alto, mas sem Quant | 28 | R$ 20.000 | Moderado | CDB Longo Prazo | Nota máxima para moderado é 5, então nenhum salário leva ao Quant |
| 6 | Quant liberado | 25 | R$ 5.000 | Arrojado | Arena Quant | Aporte de R$ 750 |
| 7 | Idade e perfil se compensam | 25 | R$ 5.000 | Conservador | CDB Progressivo | Mesmo resultado do caso 8 |
| 8 | Idade e perfil se compensam | 55 | R$ 5.000 | Arrojado | CDB Progressivo | Perfil oposto ao do caso 7, mesmo produto |
| 9 | Aniversário de 29 para 30 | 29 | R$ 5.000 | Conservador | CDB Progressivo | Rendimento de 112% do CDI |
| 10 | Aniversário de 29 para 30 | 30 | R$ 5.000 | Conservador | LCI Isenta | Rendimento nominal cai para 95% do CDI |
| 11 | Aniversário de 49 para 50 | 49 | R$ 5.000 | Conservador | LCI Isenta | Nota 3 |
| 12 | Aniversário de 49 para 50 | 50 | R$ 5.000 | Conservador | CDB Reserva | Nota 2 |
| 13 | Nota maior, rendimento menor | 40 | R$ 3.000 | Conservador | LCI Isenta | 95% do CDI e 1 ano de carência, contra 102% e saque imediato do CDB Reserva |

---

## 6. Perguntas para o time de produto

1. **Por que 15% do salário para todo mundo?** O percentual é fixo. Faz sentido para quem tem muitas despesas fixas ou dependentes? Podemos perguntar quanto a pessoa realmente consegue guardar?
2. **O simulador não pergunta se a pessoa já tem reserva de emergência.** Hoje o tamanho do aporte é usado como indício disso. Isso é intencional?
3. **A Regra 1 deve mesmo ignorar o perfil?** Dá para mostrar para o cliente, antes do resultado, que a decisão veio do aporte? Hoje a tela mostra a nota 6 de 6 junto de um produto conservador, o que confunde.
4. **A diferença de R$ 663 para R$ 664 é aceitável?** Faria sentido uma faixa de transição em vez de um corte seco em R$ 100?
5. **Qual é a lógica de o Arena Quant exigir R$ 500 por mês?** É uma regra de risco, de negócio ou do regulador? E por que moderados nunca chegam a ele, mesmo com salário alto?
6. **Os cortes de idade (30 e 50 anos) refletem o quê?** Um aniversário pode mudar o produto indicado e o rendimento. Há estudo por trás desses números ou foram escolhidos por simplicidade?
7. **Somar idade e perfil é a melhor forma?** Hoje uma pessoa de 55 anos arrojada e uma de 25 conservadora recebem o mesmo produto. Isso é desejado?
8. **A LCI Isenta no meio da escala é a ordem certa?** Ela tem rendimento nominal menor e carência maior que o CDB Reserva, mas fica em uma nota mais alta. O cliente entende que o ganho vem da isenção de IR?
9. **A indicação não considera o objetivo da pessoa** (casa, viagem, aposentadoria, imprevistos). É uma limitação aceita ou algo a incluir?
10. **O que o simulador faz com dados estranhos?** Por exemplo, salário negativo ou idade muito alta ou muito baixa. Há validação em outro ponto do sistema?
